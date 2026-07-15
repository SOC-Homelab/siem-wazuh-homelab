# 09 — Riferimenti

> La documentazione ufficiale seguita durante il progetto. Saper leggere e applicare la documentazione è parte integrante del metodo: garantisce configurazioni corrette e riflette la capacità di auto-aggiornarsi su una tecnologia in evoluzione.

[← Torna al README](../README.md)

---

## Perché citiamo le fonti

Un homelab costruito "a sensazione" e uno costruito seguendo la documentazione ufficiale si distinguono al primo problema serio. Aver lavorato sulle **Proof of Concept guide** di Wazuh significa che le configurazioni non sono improvvisate, e che sappiamo dove tornare quando qualcosa non torna. In un ruolo SOC, saper leggere la documentazione di uno strumento è una competenza quotidiana quanto saper leggere un alert.

---

## Documentazione Wazuh

### Generale

- **Wazuh — Documentazione ufficiale**
  https://documentation.wazuh.com/current/
  Riferimento per installazione dello stack all-in-one, enrollment dell'agente e configurazione di base.

### Capability usate

- **Vulnerability Detection**
  https://documentation.wazuh.com/current/user-manual/capabilities/vulnerability-detection/index.html
  Incrocio dei pacchetti installati con i feed CVE. Usata in [05 — Vulnerability Detection](05-vulnerability.md).

### Proof of Concept guide

- **File Integrity Monitoring**
  https://documentation.wazuh.com/current/proof-of-concept-guide/poc-file-integrity-monitoring.html
  Consultata come riferimento per il syscheck su `/root` (rule ID 550 / 553 / 554). Implementazione **pianificata** → [07 — Sviluppi futuri](07-sviluppi-futuri.md#2-file-integrity-monitoring-syscheck).

- **Detecting an SQL injection attack** *(sviluppo futuro)*
  https://documentation.wazuh.com/current/proof-of-concept-guide/detect-web-attack-sql-injection.html
  Riferimento per la detection web via `access.log` di Apache. **Pianificata** → [07 — Sviluppi futuri](07-sviluppi-futuri.md#1-detection-di-sql-injection-sul-livello-web).

---

## Strumenti offensivi (lato attaccante)

- **nmap** — https://nmap.org/book/man.html
  Ricognizione delle porte. Usato in [03 — Fase di attacco](03-attacchi.md#fase-4--ricognizione-nmap).

- **Hydra** — https://github.com/vanhauser-thc/thc-hydra
  Brute force SSH. Usato in [03 — Fase di attacco](03-attacchi.md#fase-5--brute-force-ssh-hydra).

---

## Strumenti pianificati (sviluppi futuri)

- **DVWA** (Damn Vulnerable Web Application) — https://github.com/digininja/DVWA
  Applicazione volutamente vulnerabile per testare la detection di SQL injection.

- **Suricata** — https://suricata.io/
  IDS di rete per la correlazione host + rete.

---

## Nota metodologica

Le configurazioni del laboratorio (installazione dello stack, enrollment dell'agente, brute force SSH) seguono le rispettive guide ufficiali della documentazione Wazuh. Dove abbiamo deviato — o dove qualcosa non ha funzionato al primo colpo — lo abbiamo documentato in [06 — Problemi e soluzioni](06-problemi-soluzioni.md), perché è lì che sta il vero apprendimento.

---

**Precedente:** [08 — Q&A tecnico](08-colloquio-qa.md) · [↑ Torna al README](../README.md)
