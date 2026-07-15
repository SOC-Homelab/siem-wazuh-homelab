# 03 — Fase di attacco

> Ricognizione con `nmap` e brute force SSH con `Hydra`, eseguiti da entrambi gli attaccanti Kali. Obiettivo: generare telemetria realistica e verificare che Wazuh la rilevi.

[← Torna al README](../README.md)

---

## Principio di fondo

In un laboratorio difensivo l'attacco non è il fine, è il **mezzo per generare eventi da rilevare**. Ci interessa poco se un exploit riesce; ci interessa che l'attacco lasci una traccia e che il SIEM la catturi. Il metro di successo è la **detection**, non il compromesso.

> **Exploit non riuscito, detection riuscita** — è esattamente l'obiettivo di un lab SOC.

Entrambi i vettori sono stati eseguiti da **entrambi gli attaccanti**, ognuno dalla propria VM Kali (Carlo dal Lenovo, Giulia dal MacBook). La vittima riceveva quindi traffico da due IP sorgente distinti — un piccolo esercizio di correlazione multi-sorgente lato Manager.

---

## Fase 4 — Ricognizione (`nmap`)

Prima si guarda, poi si colpisce. La scansione serve a capire quali servizi espone la vittima.

Da ciascun Kali, verso la vittima `192.168.1.27`:

```bash
sudo nmap -sC -sV -A 192.168.1.27
```

Cosa fanno le opzioni:

| Opzione | Significato |
|---|---|
| `-sC` | Esegue gli script NSE di default |
| `-sV` | Rileva versione dei servizi |
| `-A` | Aggressive: OS detection, traceroute, script avanzati |

**Risultato:** 65534 porte filtrate e **un'unica porta aperta**, la `22/tcp` SSH (`OpenSSH 9.6p1 Ubuntu`, protocol 2.0). Coerente con la configurazione minimale della vittima, che espone solo SSH ([Fase 4 del deployment](02-deployment.md#fase-4--esposizione-della-vittima)).

📷 **Evidenza:** [Fig. 1 — Ricognizione nmap](../img/fig1-nmap.png)

Ogni scansione veniva registrata come evento sul Manager.

---

## Fase 5 — Brute force SSH (`Hydra`)

Individuata l'unica porta aperta, l'attacco naturale è tentare l'accesso SSH con una lista di password.

Da ciascun Kali:

```bash
sudo hydra -l soleXploit -P /usr/share/wordlists/fuzz/others/common_pass.txt ssh://192.168.1.27
```

Cosa fanno le opzioni:

| Opzione | Significato |
|---|---|
| `-l soleXploit` | Username singolo da provare |
| `-P <file>` | Wordlist di password da provare in sequenza |
| `ssh://192.168.1.27` | Protocollo e bersaglio |

**Esito dell'attacco:** `0` password valide trovate. L'account non era indovinabile con quella lista.

**Esito difensivo — quello che conta:** Wazuh ha rilevato l'attacco. Sulla dashboard sono comparsi in raffica:

| Rule ID | Descrizione | Livello |
|---|---|---|
| **2502** | *User missed the password more than one time* | 10 |
| **5710** | *Attempt to login using a non-existent user* | 5 |

📷 **Evidenza:** [Fig. 2 — Hydra brute force + alert](../img/fig2-hydra.png)

La rule **2502** è quella pesante (livello 10): tanti fallimenti ravvicinati sono la firma classica di un brute force. La **5710** compare perché Hydra prova anche username che sul sistema non esistono.

Il meccanismo di detection dietro questi alert è lo stesso su cui poggia l'audit delle autenticazioni → [04 — Detection engineering](04-detection.md).

---

## Un tentativo che NON ha funzionato (e perché lo diciamo)

Abbiamo provato anche una **SQL injection** via `curl` contro un Apache sulla vittima:

```bash
curl "http://192.168.1.27/page?id=1' OR '1'='1"
```

**Nessun alert sul Manager.** Non perché l'attacco fosse innocuo, ma perché **l'ambiente non era configurato per la detection web**: mancava il monitoraggio dell'`access.log` di Apache lato agente (`<localfile>` con `log_format apache`). Senza quella sorgente, Wazuh non aveva nulla da leggere.

Lo documentiamo apposta invece di nasconderlo: è la differenza tra "l'attacco non è avvenuto" e "l'attacco è avvenuto ma non l'abbiamo visto".

---

## Riepilogo

| Vettore | Comando | Esito attacco | Detection |
|---|---|---|---|
| Ricognizione | `nmap -sC -sV -A` | Solo `22/SSH` aperta | ✅ Evento su Manager |
| Brute force SSH | `hydra -l ... -P ...` | 0 password valide | ✅ Rule 2502 / 5710 |
| SQL injection | `curl` su Apache | — | ❌ Ambiente web non configurato → rimandato |

---

**Precedente:** [02 — Deployment e enrollment](02-deployment.md) · **Prossimo:** [04 — Detection engineering](04-detection.md)
