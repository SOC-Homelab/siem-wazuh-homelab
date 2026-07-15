# 00 — Cronologia del progetto

> Le fasi del lavoro in ordine, dall'ipotesi iniziale (sbagliata) al laboratorio funzionante.

[← Torna al README](../README.md)

---

## Perché questa pagina esiste

Il laboratorio è stato costruito prima di essere documentato: questo repository raccoglie il lavoro **a posteriori**, non è il log in tempo reale del deployment. Questa cronologia serve a ricostruire l'ordine reale degli eventi, comprese le strade sbagliate — che sono la parte più interessante.

---

## Le fasi

| Fase | Cosa abbiamo fatto | Esito |
|---|---|---|
| **0** | Architettura iniziale: tutto il lab (Wazuh + vittima + attaccanti) sui due laptop | ❌ **Rollback** — RAM insufficiente |
| **0-bis** | Riprogettazione: Manager + Agent sul PC fisso Windows, Kali sui laptop | ✅ |
| **1** | Provisioning delle VM su VirtualBox (2× Ubuntu Server, 2× Kali) | ✅ |
| **2** | Deploy dello stack Wazuh all-in-one (Manager + Indexer + Dashboard) su VM1 | ✅ |
| **3** | Installazione Wazuh Agent su VM2 (vittima) ed enrollment | ❌ Agente non visibile sulla dashboard |
| **3-bis** | Diagnosi rete: VM in NAT → passaggio a bridged su entrambe | ⚠️ Ancora disconnesso |
| **3-ter** | Diagnosi residua: `ossec.conf` dell'agente puntava al vecchio IP NAT del Manager | ✅ Agente `Active` |
| **4** | Abilitazione SSH (`:22`) sulla vittima per creare superficie d'attacco | ✅ |
| **5** | Ricognizione `nmap` da entrambi i Kali | ✅ Detection |
| **5-bis** | Brute force SSH con `Hydra` da entrambi i Kali | ✅ Detection (rule 2502 / 5710) |
| **5-ter** | Tentativo di SQL injection via `curl` contro Apache | ❌ Nessun alert — ambiente web non configurato → rimandato |
| **6** | Detection engineering: audit dei comandi privilegiati e delle autenticazioni | ✅ Detection (rule 5402 / 5404 / 5503) |
| **7** | Abilitazione del modulo Vulnerability Detection | ✅ 2.225 CVE inventariate |
| **8** | Redazione del report tecnico e pubblicazione su questo repository | ✅ |

---

## Il percorso in una riga

**Ipotesi sbagliata → vincolo hardware scoperto → riprogettazione → deployment → tre errori di rete in sequenza → laboratorio funzionante → attacchi → detection → vulnerability management → documentazione.**

Nessuna delle tre fasi marcate ❌ era prevista. Sono quelle che hanno insegnato di più: il dettaglio di ciascuna è in [06 — Problemi e soluzioni](06-problemi-soluzioni.md).

---

## Cosa è rimasto fuori

Due cose sono state **volutamente** rimandate invece di essere forzate:

- **Detection di SQL injection** — provata, non funzionante, causa individuata (manca la configurazione `<localfile>` per l'`access.log` di Apache). Documentata come tale, non nascosta.
- **File Integrity Monitoring (syscheck)** — la documentazione ufficiale è stata studiata come riferimento, ma l'implementazione non è stata completata.

Entrambe in [07 — Sviluppi futuri](07-sviluppi-futuri.md).

---

**Prossimo:** [01 — Inventario dell'ambiente](01-inventario-ambiente.md)
