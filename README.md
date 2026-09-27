# SOC Investigation Lab — SSH Brute-Force Detection & Analysis (Splunk)

A self-directed, hands-on SOC analyst training project. Built a Splunk Enterprise home lab, ingested realistic security log data, and independently investigated a simulated SSH brute-force attack — from initial anomaly detection through to a formal incident report with MITRE ATT&CK mapping and remediation recommendations.

This project was built to develop and demonstrate practical SIEM / log-analysis skills relevant to a **Tier 1 SOC Analyst** role, going beyond theoretical (TryHackMe/CTF) learning into applied, self-directed investigation.

---

## 🎯 Objective

Simulate the real workflow of a Tier 1 SOC analyst:
1. Ingest and index security log data into a SIEM (Splunk)
2. Triage an anomaly / potential alert
3. Investigate using SPL (Search Processing Language) queries
4. Distinguish a genuine threat from a false positive
5. Document findings in a formal, industry-style incident report

---

## 🛠️ Environment & Tools

| Component | Detail |
| SIEM | Splunk Enterprise 10.4.2 (free trial) |
| OS | Parrot OS (Debian-based Linux) |
| Dataset | Official Splunk Search Tutorial dataset (`tutorialdata.zip`) — simulated "Buttercup Games" e-commerce environment |
| Log sources analyzed | `access_combined_wcookie` (web access logs), `secure-2` (SSH authentication logs), `vendor_sales` (transaction logs) |
| Query language | SPL (Splunk Search Processing Language) |

---

## 🔍 Investigation Summary

### Step 1 — Initial anomaly and false-positive triage
Reviewing HTTP access logs, one IP (`87.194.216.51`) stood out for generating a wide spread of HTTP error codes (400, 404, 406, 408, 500, 503, 505). On deeper inspection of the full request sequence for that IP, the traffic matched normal user browsing behavior (consistent session IDs, natural navigation: category → product → cart → checkout). **Ruled out as a false positive.**

This step is intentionally included in the write-up — correctly disproving your own initial hypothesis is a core, everyday Tier 1 skill, not a failure.

### Step 2 — Real finding: SSH brute-force attack
Pivoting to SSH authentication logs (`sourcetype=secure-2`) surfaced a genuine pattern:

- **264 failed login attempts** from a single external IP (`194.8.74.23`)
- Targeting host `home` (`mailsv1`)
- Recurring at the **same timestamp across multiple separate days** — a strong signal of scripted/automated attack behavior rather than manual login attempts
- Multiple distinct usernames attempted (`root`, `appserver`, `testuser`) — consistent with credential-guessing tools (e.g., Hydra, Medusa)

### Step 3 — Impact verification
Searched explicitly for successful authentication events (`"Accepted password"` / `"Accepted publickey"`) against the same source IP. **Zero results** — confirming the attack did not result in a compromise.

---

## 📊 Key SPL Queries Used

# Discover what log sources exist in the index
index=main | stats count by sourcetype

# Preview raw SSH authentication events
index=main sourcetype=secure-2 | head 20

# Pull every failed login attempt from the suspect IP
index=main sourcetype=secure-2 "Failed password" "194.8.74.23" | table _time, host, user

# Critical check: did any attempt succeed?
index=main sourcetype=secure-2 "194.8.74.23" ("Accepted password" OR "Accepted publickey")

# Ruling out the false-positive IP: reconstructing its full session
index=main sourcetype=access_combined_wcookie clientip="87.194.216.51" | table _time, status, uri, method
```

---

## 🧩 MITRE ATT&CK Mapping

| Field | Value |
| Tactic | Credential Access |
| Technique | T1110 – Brute Force |
| Sub-technique | T1110.001 – Password Guessing |

---

## ✅ Findings & Recommended Actions

**Impact assessment:** Contained. No successful authentication occurred; no evidence of lateral movement or privilege escalation was found.

**Recommendations:**
1. Block/rate-limit source IP `194.8.74.23` at the perimeter firewall.
2. Deploy `fail2ban` or equivalent account-lockout policy on the affected host.
3. Create a correlation rule alerting when a single source IP generates 10+ failed SSH logins within a 5-minute window across multiple usernames.
4. Harden SSH config — confirm `PermitRootLogin no` is set.
5. Continue monitoring the source IP for renewed or altered activity.

---

## 📁 Repository Contents

- [`SSH_Bruteforce_Investigation_Report.md`](./SSH_Bruteforce_Investigation_Report.md) — full formal incident report
- [`SPL_Cheat_Sheet_SOC_Analyst.md`](./SPL_Cheat_Sheet_SOC_Analyst.md) — SPL reference notes built while completing this project
- `/screenshots` — Splunk search results and evidence supporting the investigation

---

## 📚 What This Project Demonstrates

- Practical SIEM/log analysis using industry-standard tooling (Splunk)
- Ability to write and iterate SPL queries independently to answer investigative questions
- Critical triage skill: distinguishing real threats from false positives using evidence, not assumption
- Formal security documentation aligned with MITRE ATT&CK
- Self-directed learning — this lab was built independently, outside of a guided course, as applied practice alongside TryHackMe SOC Level 1 coursework

---

## 🔗 About Me

Cybersecurity B.Tech student preparing for SOC Analyst / Tier 1 roles. See my [resume](https://mayankpriyadarshi.xyz/About.html)for more.
# SOC Investigation Lab — SSH Brute-Force Detection & Analysis (Splunk)

A self-directed, hands-on SOC analyst training project. Built a Splunk Enterprise home lab, ingested realistic security log data, and independently investigated a simulated SSH brute-force attack — from initial anomaly detection through to a formal incident report with MITRE ATT&CK mapping and remediation recommendations.

This project was built to develop and demonstrate practical SIEM / log-analysis skills relevant to a **Tier 1 SOC Analyst** role, going beyond theoretical (TryHackMe/CTF) learning into applied, self-directed investigation.

---

## 🎯 Objective

Simulate the real workflow of a Tier 1 SOC analyst:
1. Ingest and index security log data into a SIEM (Splunk)
2. Triage an anomaly / potential alert
3. Investigate using SPL (Search Processing Language) queries
4. Distinguish a genuine threat from a false positive
5. Document findings in a formal, industry-style incident report

---

## 🛠️ Environment & Tools

Component | Detail :
SIEM / Splunk Enterprise 10.4.2 
OS / Parrot OS (Debian-based Linux) 
Dataset, Official Splunk Search Tutorial dataset (`tutorialdata.zip`) — simulated "Buttercup Games" e-commerce environment, Log sources analyzed, `access_combined_wcookie` (web access logs), `secure-2` (SSH authentication logs), `vendor_sales` (transaction logs), Query language, SPL (Splunk Search Processing Language)
---

## 🔍 Investigation Summary

### Step 1 — Initial anomaly and false-positive triage
Reviewing HTTP access logs, one IP (`87.194.26.55`) stood out for generating a wide spread of HTTP error codes (400, 404, 406, 408, 500, 503, 505). On deeper inspection of the full request sequence for that IP, the traffic matched normal user browsing behavior (consistent session IDs, natural navigation: category → product → cart → checkout). **Ruled out as a false positive.**

This step is intentionally included in the write-up — correctly disproving your own initial hypothesis is a core, everyday Tier 1 skill, not a failure.

### Step 2 — Real finding: SSH brute-force attack
Pivoting to SSH authentication logs (`sourcetype=secure-2`) surfaced a genuine pattern:

- **264 failed login attempts** from a single external IP (`194.168.74.23`)
- Targeting host `home` (`mailsv1`)
- Recurring at the **same timestamp across multiple separate days** — a strong signal of scripted/automated attack behavior rather than manual login attempts
- Multiple distinct usernames attempted (`root`, `appserver`, `testuser`) — consistent with credential-guessing tools (e.g., Hydra, Medusa)

### Step 3 — Impact verification
Searched explicitly for successful authentication events (`"Accepted password"` / `"Accepted publickey"`) against the same source IP. **Zero results** — confirming the attack did not result in a compromise.

---

## 📊 Key SPL Queries Used:

# Discover what log sources exist in the index
index=main | stats count by sourcetype

# Preview raw SSH authentication events
index=main sourcetype=secure-2 | head 20

# Pull every failed login attempt from the suspect IP
index=main sourcetype=secure-2 "Failed password" "194.168.74.23" | table _time, host, user

# Critical check: did any attempt succeed?
index=main sourcetype=secure-2 "194.168.74.23" ("Accepted password" OR "Accepted publickey")

# Ruling out the false-positive IP: reconstructing its full session
index=main sourcetype=access_combined_wcookie clientip="87.194.26.55" | table _time, status, uri, method

---

## 🧩 MITRE ATT&CK Mapping

| Field                     | Value                         |
| Tactic                    | Credential Access             |
| Technique n               | T1110 – Brute Force           |
| Sub-technique             | T1110.001 – Password Guessing |

---

## ✅ Findings & Recommended Actions

**Impact assessment:** Contained. No successful authentication occurred; no evidence of lateral movement or privilege escalation was found.

**Recommendations:**
1. Block/rate-limit source IP `194.168.74.23` at the perimeter firewall.
2. Deploy `fail2ban` or equivalent account-lockout policy on the affected host.
3. Create a correlation rule alerting when a single source IP generates 10+ failed SSH logins within a 5-minute window across multiple usernames.
4. Harden SSH config — confirm `PermitRootLogin no` is set.
5. Continue monitoring the source IP for renewed or altered activity.

---

## 📁 Repository Contents

- [`SSH_Bruteforce_Investigation_Report.md`](./SSH_Bruteforce_Investigation_Report.md) — full formal incident report
- [`SPL_Cheat_Sheet_SOC_Analyst.md`](./SPL_Cheat_Sheet_SOC_Analyst.md) — SPL reference notes built while completing this project
- `/screenshots` — Splunk search results and evidence supporting the investigation

---

## 📚 What This Project Demonstrates

- Practical SIEM/log analysis using industry-standard tooling (Splunk)
- Ability to write and iterate SPL queries independently to answer investigative questions
- Critical triage skill: distinguishing real threats from false positives using evidence, not assumption
- Formal security documentation aligned with MITRE ATT&CK
- Self-directed learning — this lab was built independently, outside of a guided course, as applied practice alongside TryHackMe SOC Level 1 coursework

---

## 🔗 About Me

Cybersecurity B.Tech student preparing for SOC Analyst / Tier 1 roles. See my [resume](https://mayankpriyadarshi.xyz/About.html) for more.
