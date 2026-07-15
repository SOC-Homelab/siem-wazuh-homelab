# 06 — Problemi e soluzioni

> La parte più formativa del progetto. Ogni problema è raccontato con lo schema **Sintomo → Diagnosi → Soluzione → Lezione**, perché è così che ragiona un analista: non "ha funzionato per caso", ma "ho capito perché non funzionava".

[← Torna al README](../README.md)

---

## Perché questa pagina è la più importante

Un homelab che funziona al primo colpo non insegna niente e non lo racconti in nessun colloquio. Il valore sta negli errori: dimostrano **metodo di troubleshooting**, cioè la capacità di isolare una causa invece di procedere per tentativi casuali. Questa pagina è quella che un intervistatore tecnico leggerà per prima.

Le narrazioni estese vivono nelle pagine di fase; qui sta la sintesi consultabile.

---

## Tabella riepilogativa

| # | Problema | Causa | Soluzione | Lezione |
|---|---|---|---|---|
| 1 | I laptop non reggevano il carico | Wazuh/Indexer (OpenSearch) affamato di RAM | Manager + Agent spostati sul PC fisso; Kali sui laptop | Dimensionare le risorse **prima** del deployment |
| 2 | Agente non visibile sulla dashboard | Entrambe le VM in NAT → reti isolate | Passaggio a scheda **bridged** su entrambe | NAT isola le VM; bridged le mette sulla stessa LAN |
| 3 | Agente ancora disconnesso dopo il bridge | `ossec.conf` dell'agente puntava al vecchio IP NAT del Manager | Aggiornato l'indirizzo del Manager + restart del servizio | Se cambia l'IP del Manager, aggiorna sempre l'agente |
| 4 | Nessuna porta da attaccare | Vittima con servizi minimi | Abilitato SSH (`:22`) manualmente | Serve una superficie d'attacco per generare telemetria |
| 5 | SQL injection senza detection | `access.log` di Apache non monitorato dall'agente | Rimandato: config `<localfile>` mancante | Un evento che non arriva al SIEM è un punto cieco |

---

## Dettaglio

### Problema 1 — RAM insufficiente sui laptop

**Sintomo.** Con l'intero lab (Wazuh + vittima + attaccanti) sui due laptop, il sistema diventava inutilizzabile.

**Diagnosi.** L'Indexer di Wazuh è basato su OpenSearch, che è affamato di RAM per costruzione. Sommato alle VM Kali, saturava la memoria disponibile.

**Soluzione.** Riprogettazione dell'architettura: componenti pesanti (Manager + Agent Ubuntu) sul PC fisso Windows, VM Kali leggere sui laptop.

**Lezione.** Il dimensionamento hardware è un vincolo di progetto reale. I requisiti di un SIEM si leggono *prima* di installarlo, non si scoprono a metà. → [02, Fase 0](02-deployment.md#fase-0--il-primo-errore-tutto-sui-laptop)

---

### Problema 2 — Agente non visibile (rete in NAT)

**Sintomo.** Agente avviato, chiave installata, ma sulla dashboard non compariva (`Disconnected` / `Never connected`).

**Diagnosi.** Entrambe le VM in **NAT**. In NAT ogni VM riceve un IP isolato (`10.0.2.15`) nella propria rete privata: le due VM non si vedono, perché non condividono lo stesso segmento. L'agente non poteva raggiungere il Manager sulle porte `1514`/`1515`.

**Soluzione.** Passaggio a **scheda bridged** su entrambe le VM → stessa LAN `192.168.1.0/24`.

**Lezione.** La rete delle VM è la fonte di problemi numero uno in un homelab. NAT isola, bridged espone sulla LAN fisica. → [02, Fase 3](02-deployment.md#fase-3--il-secondo-problema-lagente-non-compariva)

---

### Problema 3 — Agente ancora disconnesso dopo il bridge

**Sintomo.** Anche dopo aver messo entrambe le VM in bridged, l'agente non si connetteva.

**Diagnosi.** Cambiando la rete era cambiato l'IP del Manager (da `10.0.2.x` a `192.168.1.28`), ma nell'`ossec.conf` dell'agente (`/var/ossec/etc/ossec.conf`) l'indirizzo del Manager era **ancora quello vecchio in NAT**. L'agente puntava a un IP che non esisteva più.

**Soluzione.** Aggiornato manualmente l'indirizzo del Manager nell'`ossec.conf` con il nuovo IP bridged (`192.168.1.28`) e riavviato il servizio dell'agente. → agente `Active`.

**Lezione.** Questo è il problema più didattico dei tre, perché è "invisibile": nessun errore urlato, l'agente cercava silenziosamente un indirizzo morto. Quando cambia l'IP del Manager, va **sempre** aggiornato l'agente. È la trappola classica del passaggio NAT → bridged.

---

### Problema 4 — Nessuna superficie d'attacco

**Sintomo.** Vittima installata ma niente da attaccare: `nmap` non trovava porte interessanti.

**Diagnosi.** Una Ubuntu Server appena installata non espone servizi di rete.

**Soluzione.** Abilitato manualmente SSH sulla porta `22`.

**Lezione.** In un lab difensivo la vittima va costruita apposta. Senza superficie d'attacco non c'è telemetria, e senza telemetria non c'è detection da testare. → [02, Fase 4](02-deployment.md#fase-4--esposizione-della-vittima)

---

### Problema 5 — SQL injection senza alert

**Sintomo.** Tentativo di SQL injection via `curl` contro Apache sulla vittima → nessun alert sul Manager.

**Diagnosi.** Non un fallimento dell'attacco, ma un **punto cieco della detection**: l'agente non monitorava l'`access.log` di Apache. Mancava la configurazione `<localfile>` con `log_format apache`. Senza quella sorgente, Wazuh non aveva log web da leggere.

**Soluzione.** Rimandato consapevolmente: la configurazione web è pianificata come sviluppo futuro, con soluzione già individuata. → [07 — Sviluppi futuri](07-sviluppi-futuri.md)

**Lezione.** La più importante di tutte. C'è differenza tra *"l'attacco non è avvenuto"* e *"l'attacco è avvenuto ma non l'abbiamo visto"*. La seconda è il vero incubo di un SOC: un evento che non raggiunge il SIEM è cieco per definizione. Riconoscere un punto cieco vale più che coprire un altro vettore già coperto.

---

## Il filo conduttore

Tre dei cinque problemi (2, 3, 4) sono di **rete e configurazione**, non di sicurezza in senso stretto. È esattamente ciò che rende un homelab formativo: il 90% del lavoro reale in un SOC non è "hackerare", è capire perché un log non arriva, perché un agente non si connette, perché una sorgente è muta. La sicurezza applicata è, prima di tutto, saper far parlare i sistemi tra loro.

---

**Precedente:** [05 — Vulnerability Detection](05-vulnerability.md) · **Prossimo:** [07 — Sviluppi futuri](07-sviluppi-futuri.md)
