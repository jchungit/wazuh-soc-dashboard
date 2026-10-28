# 🛡️ Home Lab SOC — Wazuh SIEM + Firewall Logs + Custom Dashboard

A small **SOC (Security Operations Center)** for my home network: **Wazuh SIEM (Security Information and Event Management)** collecting endpoint and firewall logs, custom detection rules for my UniFi gateway, real-time **Telegram** alerting, and a custom-built **Next.js** dashboard that summarizes it all on one screen.

This is the v2 of my earlier [home-lab-siem-wazuh](https://github.com/jchungit/home-lab-siem-wazuh) project — moved from a laptop VM to an always-on server, and extended from "it collects logs" to "it tells me when something matters."

> 🔒 IP addresses, hostnames, ports, credentials, and alert contents have been intentionally left out.

---

## 🧱 Architecture

```
 ┌──────────────┐   syslog (UDP)    ┌──────────────────────────────┐
 │ UniFi Gateway├──────────────────►│                              │
 └──────────────┘                   │   Wazuh (Docker single-node) │
 ┌──────────────┐   Wazuh agent     │   • Manager  (decode + rules)│
 │ Mac mini     ├──────────────────►│   • Indexer  (storage/search)│
 └──────────────┘                   │   • Dashboard (HTTPS)        │
 ┌──────────────┐   Wazuh agent     │                              │
 │ Windows PC   ├──────────────────►│                              │
 └──────────────┘                   └───────┬──────────────┬───────┘
                                            │ level ≥ 10   │ read-only API user
                                            ▼              ▼
                                   ┌────────────────┐ ┌───────────────────────┐
                                   │ Telegram bot   │ │ Home Lab SOC Dashboard│
                                   │ (phone alerts) │ │ Next.js · Docker      │
                                   └────────────────┘ └───────────────────────┘
```

---

## 🛠️ What I Built

### 1. Wazuh on Docker
- Deployed **Wazuh 4.x single-node** via the official Docker Compose stack on my 24/7 Mac mini
- Moved the dashboard off port 443 (already used by my reverse proxy) to avoid a port conflict
- **Changed the default admin password** immediately after first login
- Enrolled agents on the server itself and on my Windows desktop

### 2. Firewall Logs → Wazuh
- Forwarded the **UniFi gateway's syslog** into Wazuh's syslog listener
- Wrote a **custom decoder** to parse UniFi log lines into structured fields Wazuh can search on
- Wrote **custom rules** so UniFi firewall events raise alerts at the right severity levels

### 3. Alerting
- Integrated Wazuh with a personal **Telegram bot** — any alert at **level 10 or higher** pushes to my phone
- Set the threshold at level 10 so routine events stay in the dashboard and only high-severity ones reach my phone (avoiding alert fatigue)

### 4. Custom SOC Dashboard (Next.js)
- Built a **Next.js** web app, containerized in Docker, that summarizes Wazuh alerts and agent activity on one screen
- Connects through a dedicated **read-only indexer user** (`readall` backend role) — the dashboard can *see* the data but can't change or delete anything (**principle of least privilege**)
- Includes a **demo mode** toggle so I can show the dashboard to others without exposing real alerts

---

## 🧯 Problems Solved

| Problem | Cause | Fix |
|---|---|---|
| Wazuh dashboard wouldn't bind to HTTPS | Port 443 already held by the reverse proxy | Remapped the dashboard to a different port |
| UniFi syslog wasn't parsed into usable fields | Wazuh has no built-in decoder for UniFi's format | Wrote a custom decoder + rule set |
| Needed phone alerts without notification spam | Forwarding every event would be noise | Only level 10+ forwarded to Telegram |
| Dashboard needed data access | Using the admin account would be over-privileged | Created a read-only service account |

---

## 🎓 Security+ / SOC Concepts Practiced
- **Log collection & normalization** — decoders turn raw text into structured fields
- **Detection engineering** — writing and tuning rules, severity levels
- **Alert triage & tuning** — reducing false positives / alert fatigue
- **Least privilege** — read-only service account for the dashboard
- **Default credential hygiene** — changing vendor defaults
- **Defense in depth** — firewall + SIEM + alerting layered together

---

## 📌 Next Steps
- [ ] Add Pi-hole DNS query logs as another data source
- [ ] Map custom rules to **MITRE ATT&CK** techniques
- [ ] Add uptime monitoring alerts to the same Telegram channel

---

## 🧠 Skills Demonstrated
`Wazuh` · `SIEM` · `Syslog` · `Custom decoders & rules` · `Detection engineering` · `Docker` · `Next.js` · `REST API` · `Telegram Bot API` · `Least privilege` · `UniFi`
