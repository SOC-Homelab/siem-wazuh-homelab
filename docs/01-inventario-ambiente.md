# 01 — Inventario dell'ambiente

> Host fisici, macchine virtuali, ruoli e indirizzamento del laboratorio.

[← Torna al README](../README.md)

---

## Host fisici

| # | Host | SO host | Ruolo | VM ospitate |
|---|---|---|---|---|
| 1 | PC fisso (Carlo) | Windows | Host dei componenti SIEM | VM1 (Manager), VM2 (Agent) |
| 2 | Lenovo (Carlo) | CachyOS | Host attaccante | VM3 (Kali) |
| 3 | MacBook Pro 2015 (Giulia) | CachyOS | Host attaccante | VM4 (Kali) |

**Totale:** 3 host fisici · 4 VM · 1 rete condivisa.

---

## Macchine virtuali

Hypervisor: **VirtualBox** su tutti e tre gli host.

| VM | Host | SO guest | Ruolo | Hostname | IP |
|---|---|---|---|---|---|
| VM1 | PC fisso | Ubuntu 24.04 | Wazuh **Manager + Indexer + Dashboard** | `ubuntu-manager-VirtualBox` | `192.168.1.28` |
| VM2 | PC fisso | Ubuntu 24.04 | Wazuh **Agent** — vittima | `ubuntu-agent-VirtualBox` | `192.168.1.27` |
| VM3 | Lenovo | Kali Linux | Attaccante 1 | — | DHCP (LAN) |
| VM4 | MacBook | Kali Linux | Attaccante 2 | — | DHCP (LAN) |

**Agente registrato sul Manager:**

| Campo | Valore |
|---|---|
| `agent.name` | `UbuntuEndPoint` |
| `agent.id` | `001` |
| `agent.ip` | `192.168.1.27` |
| SO rilevato | Ubuntu 24.04.4 LTS |
| Stato finale | `Active` |

---

## Rete

- **Segmento:** `192.168.1.0/24` — LAN WiFi domestica
- **Modalità VirtualBox:** `Scheda con bridge` su **tutte** le VM
- **Porte Agent → Manager:** `1514/tcp` (invio eventi) · `1515/tcp` (enrollment)
- **Porta esposta sulla vittima:** `22/tcp` (OpenSSH 9.6p1 Ubuntu, protocol 2.0)

> Durante la prima configurazione le VM erano in **NAT**, dove VirtualBox assegna a ciascuna un IP isolato (tipicamente `10.0.2.15`) nella propria rete privata. Questo ha impedito ogni comunicazione tra Agent e Manager. Il dettaglio della diagnosi è in [06 — Problemi e soluzioni](06-problemi-soluzioni.md).

---

## Perché Windows come host del SIEM

Non è una scelta ideologica, è una conseguenza del vincolo hardware.

Wazuh in configurazione all-in-one è **esigente in termini di RAM**: il Manager, l'Indexer (basato su OpenSearch) e la Dashboard girano insieme sulla stessa macchina, e l'Indexer da solo occupa la fetta più grossa. Sommando due VM Ubuntu più una VM Kali, i laptop saturavano.

Il PC fisso è l'unica macchina con risorse sufficienti a ospitare contemporaneamente le due VM Ubuntu. I laptop restano dedicati alle VM Kali, molto più leggere — un Kali che lancia `nmap` e `Hydra` non ha bisogno di quasi nulla.

**Lezione:** il dimensionamento delle risorse va verificato **prima** del deployment, non scoperto a metà.

---

## Perché due attaccanti e non uno

Non era strettamente necessario, ma ha aggiunto valore didattico: la vittima riceveva traffico da **due IP sorgente distinti**, permettendo di osservare lato Manager **lo stesso pattern d'attacco visto da due host diversi**. È un piccolo esercizio di correlazione multi-sorgente — che è esattamente quello che fa un analista quando deve capire se sta guardando un attaccante singolo o una campagna distribuita.

In più, ha permesso a entrambi di eseguire ogni vettore in prima persona invece di guardare l'altro farlo. Vedi [Modalità di lavoro](../README.md#modalità-di-lavoro).

---

**Precedente:** [00 — Cronologia](00-timeline.md) · **Prossimo:** [02 — Deployment e enrollment](02-deployment.md)
