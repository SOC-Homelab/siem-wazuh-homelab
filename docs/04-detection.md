# 04 — Detection engineering

> "Vedere ogni mossa del root": audit dei comandi privilegiati e delle autenticazioni sulla vittima. Non solo *sapere che* è successo qualcosa, ma *chi* l'ha fatto, *da dove* e con *quale comando esatto*.

[← Torna al README](../README.md)

---

## Dagli attacchi all'audit

La [fase di attacco](03-attacchi.md) ha mostrato che Wazuh rileva le minacce che arrivano da *fuori* (brute force da un IP esterno). Qui verifichiamo l'altra metà del lavoro di un SOC: tracciare cosa succede *dentro* la macchina, in particolare le **azioni privilegiate** — quelle che un attaccante compie dopo essere entrato.

La fonte è la stessa che alimenta la detection del brute force: il **monitoraggio continuo delle autenticazioni** (login/logout PAM, sessioni, tentativi falliti) e l'**audit dei comandi `sudo`**.

---

## Escalation di privilegi FALLITA

Simulazione: qualcuno prova a diventare root sbagliando la password.

Tre password errate consecutive su `sudo su` hanno generato:

| Rule ID | Descrizione | Livello |
|---|---|---|
| **5404** | *Three failed attempts to run sudo* | 10 |
| **5503** | *PAM: User login failed* | — |

📷 **Evidenza:** [Fig. 3 — Escalation fallita](../img/fig3-sudo-fail.png)

La **5404** a livello 10 è significativa: tre fallimenti `sudo` di fila non sono un errore di battitura, sono un pattern sospetto. È il tipo di alert su cui un analista di Tier 1 apre un ticket.

---

## Escalation RIUSCITA e audit del comando

Questa è la parte più interessante del lab difensivo.

Un `sudo` andato a buon fine ha prodotto:

| Rule ID | Descrizione |
|---|---|
| **5402** | *Successful sudo to ROOT executed* |
| **5501** | *PAM: Login session opened* |
| **5502** | *PAM: Login session closed* |

Ma il valore vero non è sapere *che* c'è stata un'escalation — è sapere **cosa è stato fatto** dopo. Wazuh ha catturato il **comando esatto** eseguito come root. Nel dettaglio dell'evento (`Document Details`):

| Campo | Valore |
|---|---|
| `data.command` | `/usr/bin/sudo mv hosts hosts2` |
| `data.srcuser` | `ubuntu_agent` (chi ha lanciato il comando) |
| `data.dstuser` | `root` (con quali privilegi) |
| TTY | il terminale di origine |
| Decoder note | *First time user executed the sudo command* |

📷 **Evidenze:** [Fig. 4 — Escalation riuscita + PAM](../img/fig4-sudo-ok.png) · [Fig. 5 — Audit del comando](../img/fig5-command-audit.png)

---

## Perché questo è "vedere ogni mossa del root"

Mettendo insieme i campi, per ogni azione privilegiata sappiamo:

- **CHI** — l'utente di origine (`ubuntu_agent`)
- **CON QUALI PRIVILEGI** — l'utente di destinazione (`root`)
- **COSA** — il comando preciso (`mv hosts hosts2`)
- **DA DOVE** — la TTY

È esattamente la ricostruzione che un analista SOC fa durante un'indagine. La differenza tra un log che dice *"root ha fatto qualcosa"* e uno che dice *"l'utente ubuntu_agent, alle 14:32, dalla TTY pts/0, ha rinominato il file hosts con privilegi di root"* è la differenza tra sapere che è successo un incidente e poterlo **investigare**.

> L'esempio `mv hosts hosts2` è innocuo di proposito. Ma lo stesso meccanismo cattura qualsiasi comando: se un attaccante avesse eseguito `cat /etc/shadow` o `wget <payload>`, sarebbe finito nell'evento con la stessa precisione.

---

## Il legame con il brute force

Non è un caso che l'audit delle autenticazioni e la detection del brute force ([03](03-attacchi.md)) condividano le stesse rule PAM (55xx). Sono lo stesso flusso: Wazuh legge i log di autenticazione della vittima e li normalizza. Un brute force è "tanti login falliti da fuori"; un'escalation fallita è "login falliti verso root da dentro". Stessa sorgente, contesti diversi.

---

## Nota: File Integrity Monitoring

Il **syscheck** (File Integrity Monitoring) — che genererebbe alert su modifica/cancellazione/creazione di file in directory sensibili come `/root` o `/etc` (rule ID 550 / 553 / 554) — è stato **studiato come riferimento** sulla documentazione ufficiale, ma **non implementato** in questa fase.

Le prove raccolte qui riguardano l'audit dei comandi e delle autenticazioni, **non** il syscheck. Lo diciamo esplicitamente per non attribuire al lab una capacità che non è ancora stata verificata con uno screenshot. Implementazione pianificata in → [07 — Sviluppi futuri](07-sviluppi-futuri.md).

---

## Riepilogo

| Scenario | Rule ID | Cosa dimostra |
|---|---|---|
| `sudo` fallito ×3 | 5404 (liv.10), 5503 | Detection di tentata escalation |
| `sudo` riuscito | 5402, 5501, 5502 | Tracciamento dell'escalation |
| Audit del comando | `data.command` + `srcuser`/`dstuser`/TTY | **Chi ha fatto cosa, da dove** |

---

**Precedente:** [03 — Fase di attacco](03-attacchi.md) · **Prossimo:** [05 — Vulnerability Detection](05-vulnerability.md)
