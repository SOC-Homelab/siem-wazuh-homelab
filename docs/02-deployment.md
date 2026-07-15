# 02 — Deployment e enrollment

> Dal primo tentativo fallito al Wazuh Agent in stato `Active`: provisioning delle VM, deploy dello stack, registrazione dell'agente e i tre errori di rete incontrati lungo la strada.

[← Torna al README](../README.md)

---

## Fase 0 — Il primo errore: tutto sui laptop

**Cosa avevamo pianificato:** ospitare l'intero laboratorio — Wazuh, vittima e attaccanti — sui due soli laptop.

**Cosa è successo:** i laptop non reggevano il carico. L'Indexer di Wazuh, basato su OpenSearch, satura rapidamente la RAM; con l'aggiunta delle VM Kali il sistema diventava inutilizzabile.

**Cosa abbiamo fatto:** rollback e riprogettazione dell'architettura. I componenti pesanti (Manager + Agent Ubuntu) sono stati spostati sul PC fisso Windows, molto più capiente; ai laptop è rimasto il solo compito di ospitare le VM Kali.

> **Prima lezione del progetto.** Il dimensionamento delle risorse va verificato prima del deployment, non scoperto a metà. Un SIEM non è un servizio leggero: prima di installarlo si leggono i requisiti.

---

## Fase 1 — Provisioning delle VM

Su VirtualBox, PC fisso:

- **VM1 → Wazuh Manager** — Ubuntu, destinata allo stack all-in-one (Manager + Indexer + Dashboard)
- **VM2 → Wazuh Agent** — Ubuntu, la futura vittima

Su ciascun laptop, una **VM Kali Linux** (Attaccante 1 e Attaccante 2).

Dettagli completi in [01 — Inventario dell'ambiente](01-inventario-ambiente.md).

---

## Fase 2 — Deploy dello stack e registrazione dell'agente

1. **Installazione dello stack all-in-one** su VM1 (Manager + Indexer + Dashboard) e avvio della Dashboard di Wazuh.
2. **Installazione del pacchetto Wazuh Agent** sulla vittima (VM2).
3. **Enrollment dell'agente**: recupero della chiave di registrazione dal Manager e installazione della stessa sull'agente da terminale, per autenticare la VM2 presso il Manager.

Procedura seguita dalla [documentazione ufficiale Wazuh](https://documentation.wazuh.com/current/) — vedi [09 — Riferimenti](09-riferimenti.md).

---

## Fase 3 — Il secondo problema: l'agente non compariva

**Sintomo.** Nonostante l'agente risultasse avviato e la chiave installata correttamente, sulla dashboard del Manager l'agente non compariva, oppure restava in stato `Disconnected` / `Never connected`.

### Diagnosi

Entrambe le VM erano configurate in VirtualBox con scheda di rete in **NAT**. In modalità NAT ogni VM riceve un IP privato isolato — tipicamente `10.0.2.15` — all'interno della *propria* rete NAT. Le due VM **non si vedono tra loro**, perché non condividono lo stesso segmento di rete. L'agente non poteva quindi raggiungere il Manager sulle porte `1514` / `1515`.

### Correzione progressiva

La soluzione è arrivata in tre passaggi, ognuno dei quali ha rivelato il problema successivo:

**1. Cambiata la sola VM1 (Manager) in `Scheda con bridge`.**
L'IP del Manager è cambiato: da `10.0.2.15` (NAT) a `192.168.1.28` (LAN di casa).
→ *Ancora nessun agente visibile.* La vittima era ancora in NAT, su una rete diversa.

**2. Cambiata anche la VM2 (Agent) in `Scheda con bridge`.**
Ora entrambe erano sulla stessa LAN `192.168.1.0/24`, con l'agente su `192.168.1.27`.
→ *L'agente continuava a non connettersi.*

**3. Causa residua individuata.**
Nel file di configurazione dell'agente — `/var/ossec/etc/ossec.conf` — l'indirizzo del Manager era **ancora quello vecchio in NAT** (`10.0.2.x`). Cambiando la rete era cambiato l'IP del Manager, ma l'agente puntava ancora al vecchio indirizzo.

### Soluzione definitiva

Da terminale sulla vittima: aggiornato manualmente l'indirizzo del Manager nell'`ossec.conf` dell'agente con il nuovo IP bridged (`192.168.1.28`) e riavviato il servizio dell'agente.

A quel punto, sulla dashboard del Manager, l'agente è comparso come **`Active`** — `agent.name` `UbuntuEndPoint`, `id` `001`, `192.168.1.27`.

---

## Concetto chiave: NAT vs Bridged

| | **NAT** | **Bridged** |
|---|---|---|
| Cosa fa | La VM è isolata dietro la propria rete NAT | La VM appare come un host reale sulla LAN fisica |
| IP tipico | `10.0.2.15` | `192.168.1.x` |
| Visibilità | Non raggiungibile dalle altre VM | Visibile a tutti gli host della LAN |
| Uso qui | ❌ Impedisce la comunicazione Agent ↔ Manager | ✅ Necessario |

Perché serviva il bridge: Agent, Manager e i due Kali dovevano **condividere lo stesso segmento di rete** — l'agente per parlare col Manager, i Kali per attaccare la vittima.

E la regola che ne discende:

> **Quando cambia l'IP del Manager, va sempre aggiornato l'indirizzo del Manager nell'agente** — altrimenti l'agente continua a cercare il vecchio, in silenzio.

---

## Fase 4 — Esposizione della vittima

Una VM Ubuntu Server appena installata non ha **nessun servizio da attaccare**. Senza superficie d'attacco non c'è telemetria, e senza telemetria non c'è niente da rilevare.

Abbiamo quindi abilitato manualmente il servizio **SSH sulla porta 22** sulla VM2.

> **Lezione:** in un lab difensivo la vittima va *costruita* apposta. La detection si testa solo se prima c'è qualcosa da attaccare.

Da qui in poi il laboratorio era operativo → [03 — Fase di attacco](03-attacchi.md).

---

## Riepilogo della fase

| Problema | Causa | Soluzione |
|---|---|---|
| Laptop saturati | Indexer/OpenSearch affamato di RAM | Manager + Agent sul PC fisso |
| Agente non visibile | Entrambe le VM in NAT → reti isolate | Bridged su entrambe |
| Agente ancora disconnesso | `ossec.conf` puntava al vecchio IP NAT | Aggiornato indirizzo Manager + restart |
| Nessuna porta da attaccare | Vittima con servizi minimi | Abilitato SSH `:22` |

Sintesi completa in [06 — Problemi e soluzioni](06-problemi-soluzioni.md).

---

**Precedente:** [01 — Inventario dell'ambiente](01-inventario-ambiente.md) · **Prossimo:** [03 — Fase di attacco](03-attacchi.md)
