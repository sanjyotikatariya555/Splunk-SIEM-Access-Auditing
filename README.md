# SIEM Access Auditing & Brute-Force Detection Lab

## Project Overview
This project demonstrates how to utilize **Splunk Enterprise** as a centralized Security Operations Center (SOC) auditing sandbox. The objective of this lab is to actively monitor internal application telemetry and authenticate control logs to detect access anomalies, automated brute-force attacks, and insider threat vectors.

## Technical Skills & Architecture Components
* **SIEM Platform:** Splunk Enterprise (Local Sandbox Deployment)
* **Log Framework:** Internal Audit Telemetry (`index=_audit`)
* **Query Language:** Search Processing Language (SPL)
* **Threat Focus:** Authentication Brute-Force & Credential Anomaly Tracking

---

## Step-by-Step Implementation

### Phase 1: Environment Setup
1. Deployed a local sandbox instance of **Splunk Enterprise** running on a host machine (`http://localhost:8000`).
2. Configured standard administrative access control credentials to establish the initial security baseline.

### Phase 2: Generating Simulated Attack Telemetry
To simulate an insider brute-force vector, I intentionally generated multiple malicious authentication events:
1. Navigated to the local Splunk console login interface.
2. Intentionally submitted 5 failed authentication attempts using the unauthorized username `hacker_bob`.
3. Executed a secondary cluster of 5 failed attempts using the username `malicious_user`.

### Phase 3: Threat Hunting & Data Aggregation via SPL
To isolate the malicious brute-force attempts from standard user typos, I authored a targeted SPL pipeline tracking failed authentication events:

```splunk
index=_audit action=login info=failed 
| stats count by user, clientip 
| sort - count
```

**Query Analysis:**
* `index=_audit`: Targets Splunk's permanent internal audit system architecture.
* `action=login info=failed`: Filters out successful actions to isolate only authentication failures.
* `| stats count by user, clientip`: Aggregates the raw events, counting exact failure occurrences grouped by the targeted account username and source network address.
* `| sort - count`: Dynamically sorts the output from the highest volume of failures to the lowest.

---

## Lab Results & Artifacts

### Splunk Search Results Table
Below is the execution output of the SPL threat hunting query, successfully aggregating the malicious traffic and isolating both target users:

![Splunk Query Output](YOUR_SCREENSHOT_FILENAME_HERE.png)

### Incident Response Conclusion
The query successfully isolated the malicious activity. In a live enterprise infrastructure environment, a volume threshold rule (e.g., `where count > 5`) would be established to trigger a high-priority alert to instantly isolate the source network address (`clientip`) at the perimeter firewall.
