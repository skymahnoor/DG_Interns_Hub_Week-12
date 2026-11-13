WEEK 12 - SOC FINAL CAPSTONE: THREAT DETECTION & INCIDENT RESPONSE
Screenshots folder | Incident DG-SOC-2026-001 | Prepared by: Mah Noor | 13 Nov 2026

Organized from the Week 10 + Week 11 screenshot packs, renamed and grouped
so every Week 12 task has its evidence ready to paste into the report/PPT.

LAB FACTS (use these in the ticket, timeline, IOC table and metrics)
- Windows host (attacker/test machine, PowerShell): 192.168.56.1
- soc-snort (Ubuntu 24.04, Snort + Wazuh agent "snort-agent"): 192.168.56.102
- wazuh-server (Wazuh 4.14.9 - manager, indexer, dashboard): 192.168.56.103
- Apache on soc-snort = the service behind open port 80
- Linux logs are UTC; Pakistan time = UTC+5
- 8 failed SSH passwords (invalid user "wronguser") then
  Accepted password for "snortadmin" at 09:10:59 UTC
- Wazuh rules: 5710 (failed SSH), 5760, 40112 (level 12, multiple failed
  auth attempts), FIM 553/554 from Week 10
- Snort SIDs: 1:1000001 (ICMP ping), 1:1418 / 1:1420 / 1:1421 (SNMP,
  triggered by the nmap scan)
- MITRE ATT&CK: T1110 Brute Force / Valid Accounts

FOLDER -> WEEK 12 TASK MAP
Task1_Lab_Architecture          Task 1 (lab verification) + report "SOC
                                Architecture" and "Lab Environment":
                                both VMs, IPs, network config, Apache
Task1_Wazuh_Setup               Report "Wazuh Setup": OVA import, manager/
                                indexer/dashboard services, health check
Task1_Snort_Setup               Report "Snort Setup": install, HOME_NET,
                                local.rules, test mode, live console
Task1_Wazuh_Agent               Task 1: agent installed, service active,
                                "Active" in dashboard
Task2_Incident_Simulation       Task 2, Scenario D (recommended):
                                Stage 1 recon (ping + nmap) ->
                                Stage 2 auth events (8 failed SSH,
                                then success) -> Stage 3 system/file
                                activity (sudo fail, file creation, FIM)
Task3_Detection_Snort           Task 3: Snort detections, SIDs, priority
Task3_Detection_Wazuh           Task 3: Wazuh rules 5710/5760/40112,
                                MITRE, severity level 12, searches
Task4_5_Investigation_...       Task 4 (investigation) + Task 5
                                (correlation): Snort logs read by Wazuh,
                                ossec.conf localfile, 62 snort hits,
                                no-result queries (honest negatives)
Task6_IOC_FIM_Evidence          Task 6: FIM file added/deleted events,
                                rules 553/554 (IOC file evidence)
Task9_SOC_Metrics_Dashboard     Task 9: alert totals, top rules, PCI DSS
                                groups (use with your Excel sheet)
Task10_Appendix_Troubleshooting Task 10 "Challenges faced": typos, proxy
                                error, XML errors, service restarts

IOC QUICK TABLE (fill into Task 6 / Task 8 ticket)
- Source IP            192.168.56.1          (Snort + Wazuh)
- Destination IP       192.168.56.102        (Snort + Wazuh)
- Accounts             wronguser (invalid), snortadmin (valid)  (Wazuh)
- Ports                22 (SSH), 80 (HTTP), 161/162 (SNMP)      (Snort)
- Protocols            TCP/SSH, HTTP, ICMP, SNMP                (Both)
- File/Path            /tmp/fim-test.txt (added/deleted)        (Wazuh FIM)
- Rule IDs / SIDs      Wazuh 5710, 5760, 40112, 553, 554;
                       Snort 1:1000001, 1:1418, 1:1420, 1:1421  (Both)
- Timestamp            09:10:59 UTC successful login            (Both)

STILL TO CREATE BY YOU (screenshots cannot exist yet)
- Task 5 timeline table  -> build from Task2/Task3 screenshots (timestamps
  in auth.log + Snort alerts; Wazuh clock ~22s behind logs)
- Task 8 ticket (DG-SOC-2026-001) -> then screenshot it for evidence
- Task 9 metrics sheet (Excel/Google Sheets) -> screenshot it
- LinkedIn post -> text only, no screenshot needed

NOTES
- "No Results" screenshots are kept on purpose: they prove which rules
  did NOT fire in this run (rule 5715, FIM 550/553/554 this run) - good
  analyst practice to document negative searches.
- Older 28 Oct 2026 entries in auth.log are earlier tests, not part of
  this incident.
- Ubuntu/GRUB installation-wizard shots from the old pack were left out;
  only evidence relevant to Week 12 is included.

ADDED 13 Nov 2026 - Task1_Final_Verification_13Nov
Final lab verification after restarting both VMs (Task 1: "verify all components are working").
01 both VMs up: soc-snort 192.168.56.102 (enp0s8), wazuh-server 192.168.56.103 (eth1)
02 Windows host ping to 192.168.56.102, 4/4 replies
03 Wazuh Dashboard Overview, agent Active (1)
04 Wazuh Endpoints, agent Active (1), ubuntu
05 Snort live console, ICMP Ping Detected, SID 1:1000001 (13 Nov 12:30 PKT)
These are NOT part of the 5 Nov incident; use them only as final verification evidence.

NOTES
- Task6_IOC_FIM_Evidence/03 and /04 are the same image (identical file). Use only one in the report.
- Rule 40112 (level 12, 14:10:36 PKT = 09:10:36 UTC) is the Wazuh alert for the failures-then-success login.
- 8 failed SSH passwords = 6 wronguser + 2 snortadmin; successful login snortadmin 09:10:59 UTC.
- sudo lines at 09:09:52 and 09:13:13 in auth.log are the analyst's own grep commands, not attacker activity.
