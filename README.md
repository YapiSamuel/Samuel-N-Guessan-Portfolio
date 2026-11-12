# Samuel-N-Guessan-Portfolio

Hands‑on cybersecurity portfolio featuring SOC detection work, digital forensics, IAM/Entra ID practice, cloud incident response, Python automation, and network security projects.

## 👋 About Me

Hi, I'm Samuel N'Guessan, a cybersecurity graduate student with a background in Computer Science and a strong focus on blue‑team security. I build and break my own environments — detection labs, forensic investigations, identity configurations, and the tooling that supports them — working toward SOC operations, incident response, and detection engineering.

## 🎓 What I'm Studying

- Master of Science in Cybersecurity
- Bachelor of Science in Computer Science
- TryHackMe SOC Level 2 path (in progress — Advanced Elastic completed; SOC Level 1 completed)
- CompTIA Security+ (in progress)

## 🔐 Cybersecurity Focus Areas

- SOC analysis, alert triage & detection engineering
- Incident response (on‑premises and cloud)
- Digital forensics & artifact analysis
- Identity & Access Management (Entra ID)
- Python automation for security
- Network security & segmentation
- SIEM (Splunk, Elastic, SOF‑ELK)

---

## 🛡️ Security Labs & Projects

### 🔴🔵 Red vs. Blue Range — CSIS Course × DCSA
*November 2026*

Built and operated a live red/blue team environment at the request of a CSIS course, run as a joint exercise between the class (Blue Team) and the Dragons Cybersecurity Student Association.

- Designed and built the network: switched infrastructure with static IP addressing across three parallel ranges.
- Deployed on bare‑metal Windows workstations, a file server, a web server, and Kali attack hosts — no virtualization layer.
- Configured the live services the Blue Team defends, giving the range realistic attack surface instead of empty hosts.
- Handled range reset and repeatability through disk imaging (Clonezilla/FOG), since bare metal allows no snapshots.
- Served as Red Team attacker alongside DCSA board members, and built a Python validation harness to verify range state before the event.
- Scoped around a rules‑of‑engagement model emphasizing detection, investigation, and response over prevention.

---

### 🏢 SOC Lab Environment — Splunk SIEM
*May 2025*

A full enterprise SOC home lab built from scratch to practice detection engineering against real Windows attack techniques.

- Deployed Active Directory, Windows Server 2022, and Sysmon endpoint monitoring.
- Configured Group Policy Objects and deployed Splunk Universal Forwarder across all endpoints for centralized log ingestion.
- Wrote Splunk SPL detection queries for kerberoasting, pass‑the‑hash, brute force, and lateral movement.

---

### 🔎 SIEM Log Correlation — Splunk
*May 2025*

- Analyzed system and network logs to identify abnormal behavior and potential security incidents.
- Built and tuned correlation searches to reduce false positives and improve alert accuracy.
- Investigated security alerts and user activity logs to support access control and incident response.

---

### 🌐 DNS & HTTP Log Analysis — C2 Investigation
*March 2025*

- Investigated DNS and HTTP logs in Splunk Search & Reporting, extracting key fields with SPL queries.
- Identified a Command & Control (C2) attack by correlating log patterns with threat intelligence, tracing an infected host communicating with a malicious domain.
- Analyzed traffic data (src_ip, src_port, dst_ip, dst_port) to flag a system sending excessive requests.
- Documented findings and traced suspicious communication patterns to support risk mitigation and incident reporting.

---

### 📧 Phishing Email Investigation — SOC Simulator
*January 2025*

- Analyzed email headers and attachments to identify phishing and social engineering attempts.
- Correlated events within a SIEM platform to uncover malicious activity.
- Produced incident reports with clear remediation and prevention recommendations.

---

### 🕵️ Digital Forensics Labs

- Windows registry analysis to recover evidence of user and system activity.
- Timeline reconstruction to establish the sequence of events in an incident.
- Autopsy investigations across disk images and host artifacts.
- Artifact extraction (NTUSER.DAT, SAM, SYSTEM).

---

### 🔑 Identity & Access Management — Entra ID

- Entra ID user and group management.
- RBAC role assignments.
- MFA and Conditional Access configuration.
- Identity lifecycle management (Joiner–Mover–Leaver).

---

### 🖧 Network Security Lab

- Switched network build with static IP addressing and segmented ranges.
- Service deployment and configuration across Windows and Linux hosts.
- Traffic monitoring and analysis across TCP/IP, DNS, and HTTP.

---

### 🐍 Python Security Automation

- Log parsing and enrichment scripts.
- Data processing for investigation workflows.
- VirusTotal API integration for hash verification and malware triage.

---

## 🎓 Workshops & Training

### Azure Incident Response — Antisyphon Training (BHIS)
*November 2026 · 4 CEU hours · Instructor: Gerard Johansen*

- Completed five hands‑on labs building a SOF‑ELK analysis platform and investigating Azure and M365 incidents end to end.
- Analyzed the M365 Unified Audit Log to investigate Business Email Compromise (BEC) and adversary‑in‑the‑middle (AiTM) attacks.
- Investigated the Azure Activity Log and Entra ID sign‑in and audit logs to trace identity‑based attacks — OAuth abuse, device code phishing, and token/consent misuse.
- Examined AI agent misuse in Azure, including identity exposure through Agent 365 and the EchoLeak vulnerability (CVE‑2025‑32711).
- Worked through containment, eradication, recovery, and post‑incident activity for cloud environments where identity, not the endpoint, is the blast radius.

### Information Security Core Skills in the Age of AI — Antisyphon Training (BHIS)
*November 2026 · 16 hours · Instructor: John Strand*

- Worked through 11 core defensive controls mapped to MITRE ATT&CK and the CIS Critical Security Controls — application allow listing, egress traffic analysis, UEBA, logging, host firewalls, Active Directory hardening, vulnerability management, and more.
- Built automated security workflows using n8n, integrating AI into existing detection and response processes to reduce analyst workload.
- Focused on controls that measurably stop real attacks, based on BHIS penetration testing data, rather than compliance‑driven checklists.

---

## 🔥 Featured Security Tool

### 🛡️ VIGIL — Pre‑Execution Malware Triage CLI
Statically analyzes suspicious scripts and returns an explainable verdict in under 30 seconds — without ever executing the file.
🔗 https://github.com/YapiSamuel/VIGIL_2
📄 [Full write‑up](Projects/VIGIL.md)

### 🤖 Anomaly‑Based Intrusion Detection System (Random Forest)
Machine learning IDS that classifies network traffic as normal or malicious using a Random Forest classifier trained on the UNSW‑NB15 dataset — roughly 90% accuracy on unseen test data, with feature‑importance ranking and confusion‑matrix analysis to show where the model fails.
🔗 https://github.com/YapiSamuel/Anomaly-IDS-RandomForest

---

## 🔧 Other Builds

### 📝 ExamReady — AI Exam Composer
Turns a lecturer's PowerPoint chapters into a print‑ready exam paper, with AI‑drafted questions they can edit. Bilingual English / French.
🔗 https://github.com/YapiSamuel/QuizGenerator
📄 [Full write‑up](Projects/ExamReady.md)

### 💰 BudgetBuddy — Excel Budget Tracker Generator
Guided wizard that generates a working six‑sheet Excel tracker built around your real accounts and categories. Free and open source.
🔗 https://github.com/YapiSamuel/BudgetBuddy---Finance-Tracker-App
📄 [Full write‑up](Projects/BudgetBuddy.md)

<details>
<summary><b>Earlier coursework</b></summary>

### 🤝 Peer Programming Project
Collaborative development project focused on teamwork, Git workflows, and shared problem‑solving.
🔗 https://github.com/epack2/Peer_Programming
📄 [Full write‑up](Projects/Peer_Programming.md)

### 🎮 OOP Final Project — Dynamic Tic‑Tac‑Toe
Object‑oriented game with dynamic board sizes and AI opponent.
🔗 https://github.com/asmith2133/OOP-Final-Project
📄 [Full write‑up](Projects/OOP-TicTacToe.md)

### 📅 Java Calendar Application
Java console‑based calendar tool with month navigation.
🔗 https://github.com/YapiSamuel/Java-Calendar-
📄 [Full write‑up](Projects/Java-Calendar.md)

</details>

---

## 📈 My Goal

To build a strong foundation in SOC operations, grow through incident response, and move into detection engineering — designing and tuning the detections, automating the workflows, and securing both on‑premises and cloud environments.
