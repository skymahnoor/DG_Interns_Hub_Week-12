# Week 12 — SOC Final Capstone: Snort + Wazuh Incident Investigation

> **Program:** SOC Analyst Program — DG Interns Hub
> **Week:** 12 (Final Capstone) · **Date:** 13 November 2026
> **Incident:** DG-SOC-2026-001 · **Author:** Mah Noor

---

## 🎯 Week Goal

Bring everything from the past 4 weeks together into one end-to-end incident investigation:
detect a simulated multi-stage attack with **Snort (network IDS)** and **Wazuh (SIEM)**,
correlate both, respond, and document it like a real SOC analyst.

---

## 🖥️ Lab Setup Recap

| Asset | IP | Role |
|---|---|---|
| Windows host | 192.168.56.1 | Attacker / test machine (Nmap, SSH client) |
| soc-snort (Ubuntu 24.04) | 192.168.56.102 | Apache server + Snort IDS + Wazuh agent |
| wazuh-server (Ubuntu) | 192.168.56.103 | Wazuh 4.14.9 manager, indexer, dashboard |

- Both VMs on a Host-Only network (`192.168.56.0/24`) + NAT for updates
- Snort `HOME_NET` set to the lab segment
- Wazuh agent `snort-agent` active and reporting on soc-snort

![Lab Architecture](screenshots/lab_architecture.png)

---

## 📋 What I Did This Week

1. **Re-verified the whole lab** (13 Nov) — both VMs restarted, agent Active, Snort monitoring live, ping test passed.
2. **Ran the capstone incident (Scenario D)** — an authorized multi-stage attack on 5 Nov, 09:03–09:13 UTC:
   - **Stage 1 — Recon:** ICMP ping sweep + Nmap SYN scan (ports 22, 80 open)
   - **Stage 2 — Brute force:** 8 failed SSH logins (6 × `wronguser`, 2 × `snortadmin`)
   - **Stage 3 — Compromise:** successful login as `snortadmin` at **09:10:59 UTC**, then post-login activity
3. **Detected it from both sides:**
   - **Snort:** custom rule `1:1000001` fired on the ping; SIDs `1:1418 / 1:1420 / 1:1421` on the scan
   - **Wazuh:** rules `5710`, `5760` on the failures, then **rule 40112 — severity level 12** on failures-followed-by-success
4. **Correlated the tools:** Snort alerts shipped into Wazuh via `ossec.conf` localfile → 62 hits on a "snort" search
5. **Built the incident timeline** and analyzed the 71-alert investigation window
6. **Documented IOCs** (source IP, usernames, ports, rule IDs, timestamps)
7. **Wrote the 6-phase response plan** and a formal SOC ticket
8. **Compiled everything** into a full report, presentation and evidence archive

---

## ⏱️ Incident Timeline

| Time (UTC) | Event | Source | Finding |
|---|---|---|---|
| 09:03:42 | ICMP echo requests from 192.168.56.1 | Snort 1:1000001 | Host discovery |
| 09:04:56 | TCP probes to ports 161/162/705 | Snort 1:1418/1420/1421 | Nmap scan |
| 09:06:46 | Invalid user `wronguser` over SSH | Wazuh 5710 | Brute force begins |
| 09:07:00–09:07:57 | 8 failed password attempts | Wazuh 5760 | 6 wronguser + 2 snortadmin |
| 09:10:36 | Multiple failures → success | Wazuh 40112 (**level 12**) | High-severity alert |
| 09:10:59 | Accepted password for `snortadmin` | auth.log | Possible compromise |
| after login | whoami, ls, sudo attempt, file created | Terminal + Wazuh | Post-exploitation |

---

## 🚨 Key Alerts & IOCs

- **Most important alert:** Wazuh rule 40112, severity **12**, 09:10:36 UTC
- **MITRE ATT&CK:** T1110 (Brute Force), T1078 (Valid Accounts)
- **Investigation window:** 71 alerts — 17 failures / 8 successes
- **Top IOCs:** source IP `192.168.56.1`, accounts `wronguser` / `snortadmin`, port 22, login at 09:10:59 UTC
- **Classification:** True Positive (1 incident, 0 false positives)

![Rule 40112](screenshots/wazuh_rule_40112.png)

---

## 📚 What I Learned

- **One tool is not enough.** Snort sees the network but can't tell if a login succeeded.
  Wazuh reads server logs but never saw the scanning that came first. Together, the full attack
  chain appeared: recon → scan → brute force → compromise.
- **Detection ≠ intent.** The SNMP rules fired because Nmap *probed* those ports, not because real
  SNMP traffic existed. Signatures match literally, so tuning matters.
- **Document negative results too.** Rule 5715 and the FIM rules (550/553/554) returned nothing
  this run — writing that down honestly is part of a defensible investigation.
- **A level-1 analyst's real job:** monitor, triage, verify against raw logs, classify,
  write a clear ticket, escalate with timeline + IOCs attached.

---

## 🧗 Challenges & How I Solved Them

| Challenge | Fix |
|---|---|
| Wrong interface name in Snort (`enp0s8`) | Verified with `ip a` first, validated config with `snort -T` |
| Proxy error on Wazuh dashboard | Read the error properly and fixed proxy settings |
| XML typos in `ossec.conf` | Checked agent logs, found the exact broken line |
| Non-existent `wazuh.service` | Real units are `wazuh-manager`, `wazuh-indexer`, `wazuh-dashboard` — checked each with `systemctl status` |
| ~22s clock drift (Wazuh vs auth.log) | Documented it; NTP sync added to improvement list |

**The meta-lesson: read the error message first, then verify, then document.**

---

## 📈 SOC Metrics (Snapshot)

| Metric | Value |
|---|---|
| Total lab alerts (baseline) | 358 |
| Incident-window alerts | 71 |
| True Positives | 1 incident |
| False Positives | 0 |
| High alerts (level 12+) | 1 (rule 40112) |
| Snort SIDs fired | 4 |

![Metrics](screenshots/metrics_358.png)

---

## 🚀 Improvements for the Future

- [ ] Enable Wazuh **active response** to auto-block brute-force sources
- [ ] Write a dedicated Snort rule for SSH brute-force thresholds
- [ ] Tune the noisy SNMP signatures
- [ ] Disable SSH password login (key-only) + add fail2ban
- [ ] NTP time sync on all hosts
- [ ] Email/chat alerting + per-incident dashboards

---

## 📦 Deliverables

- 📄 Full report: `Week12_SOC_Report.pdf` (20 pages, 16 chapters)
- 📊 Presentation: `Week12_SOC_Capstone.pptx`
- 🗂️ Evidence archive: 92 screenshots (`Week12_Screenshots.zip`)
- 💼 LinkedIn post: `Week12_LinkedIn_Post.txt`

---

*End of Week 12 — from a blank VM to a closed SOC ticket in four weeks.* 🎓
