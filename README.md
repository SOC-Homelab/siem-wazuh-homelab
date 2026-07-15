# Homelab SIEM con Wazuh

> Laboratorio di sicurezza difensiva (Blue Team / SOC): deployment di un SIEM, simulazione di attacchi in ambiente controllato e analisi della detection.

**Autori:** Carlo Alessio e Giulia Mangraviti
**Ambiente:** rete domestica isolata — 3 host fisici, 5 macchine virtuali (VirtualBox)
**Stack:** Wazuh (Manager + Indexer + Dashboard) · Ubuntu 24.04 · Kali Linux · VirtualBox

---

## Di cosa si tratta

Abbiamo progettato e realizzato un laboratorio SIEM completo per acquisire competenze pratiche di **detection**, **log management** e **incident analysis** — il cuore del lavoro di un SOC Analyst di Tier 1.

Un server Wazuh raccoglie e correla i log di una macchina vittima (Wazuh Agent), che viene attaccata da due host indipendenti con Kali Linux. Ogni evento generato dagli attacchi — scansioni di rete, brute force SSH, escalation di privilegi, comandi eseguiti da root — viene rilevato, normalizzato e visualizzato sulla dashboard sotto forma di alert. In parallelo, il modulo di Vulnerability Detection mappa le vulnerabilità note dell'endpoint.

**Il progetto è partito da un'architettura sbagliata**, che è stata diagnosticata, corretta e ricostruita. Proprio la gestione degli errori — RAM insufficiente e connettività di rete tra le VM — è la parte più formativa del lavoro, ed è documentata per intero: vedi [Problemi e soluzioni](docs/06-problemi-soluzioni.md).
