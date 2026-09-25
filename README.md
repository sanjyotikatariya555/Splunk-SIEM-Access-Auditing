# SIEM Access Auditing & Brute-Force Detection Lab

## 📌 Project Overview
This project demonstrates the deployment and configuration of Splunk Enterprise as a centralized **Security Information and Event Management (SIEM)** sandbox. The primary objective is to build a simulated Security Operations Center (SOC) environment to monitor internal application telemetry, analyze authentication control logs, and engineer detection mechanisms for access anomalies and automated brute-force attack vectors.

## 🛠️ Technical Skills & Architecture
* **SIEM Platform:** Splunk Enterprise (Local Sandbox Deployment)
* **Log Framework:** Internal Audit Telemetry (`index=_audit`)
* **Query Language:** Search Processing Language (SPL)
* **Threat Focus:** Credential Access, Brute-Force Tactics, and Insider Threat Mitigation

---

## 🚀 Step-by-Step Implementation

### Phase 1: Environment Setup
* Deployed a local sandbox instance of Splunk Enterprise on a host machine (`http://localhost:8000`).
* Configured administrative access control credentials to establish an initial security and operational baseline.

### Phase 2: Attack Simulation & Telemetry Generation
To simulate an insider brute-force vector and generate actionable log data, targeted malicious authentication events were intentionally executed against the local Splunk console login interface:
1. Generated **5 failed authentication attempts** targeting the unauthorized username `hacker_bob`.
2. Executed a secondary cluster of **5 failed authentication attempts** utilizing the username `malicious_user`.

### Phase 3: Threat Hunting & Detection Engineering
To isolate the malicious brute-force attempts from standard user typos, a targeted SPL pipeline was authored to aggregate and analyze failed authentication events:

```spl
index=_audit action=login info=failed 
| stats count by user, clientip 
| sort - count
```

#### Query Analysis:
* `index=_audit`: Targets Splunk's permanent internal audit system logs to track console activity.
* `action=login info=failed`: Isolates strict authentication failures, filtering out successful logins.
* `| stats count by user, clientip`: Aggregates raw events into an analytical table, grouping failure volume by targeted user accounts and source network addresses.
* `| sort - count`: Dynamically sorts data from highest to lowest volume to instantly highlight high-frequency anomalies.

---

## 📊 Lab Results & Artifacts

### Splunk Search Results
The SPL threat hunting query successfully aggregated the simulated traffic, isolating both target threat vectors into an actionable security intelligence view:

| User | Client IP | Count |
| :--- | :--- | :--- |
| `hacker_bob` | `127.0.0.1` | 5 |
| `malicious_user` | `127.0.0.1` | 5 |

### Console Evidence
Below is the live execution output captured directly from the Splunk Enterprise analytical sandbox dashboard:

![Splunk Search Results Dashboard](splunk_results.png)

---

## 🎯 Incident Response & Key Takeaways
The implemented pipeline successfully identified and isolated the anomalous authentication spikes. 

**Production Recommendations:** In a live enterprise infrastructure, this logic would be operationalized by establishing a **volume threshold alert** (e.g., `where count > 5` within a 5-minute window). This threshold would trigger a high-priority alert to downstream SOAR playbooks or network firewalls to instantly isolate and block the offending source network address (`clientip`).
