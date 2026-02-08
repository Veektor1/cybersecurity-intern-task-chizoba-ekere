# Part 3: Incident Response Plan
**Scenario:** Unauthorized data exposure via URL parameter manipulation (IDOR/BOLA).

## 1. Immediate Actions (The First 60 Minutes)
When a vulnerability like this is reported, the goal is **Containment** and **Preserved Evidence**.

1.  **Triage & Confirmation (T+0:05):** Security lead reproduces the bug in a staging environment to confirm the report's validity using the provided example (`company_id=1235`).
2.  **Short-Term Containment (T+0:15):** * Deploy an emergency WAF (Web Application Firewall) rule to block or flag requests where the `company_id` in the URL does not match the `company_id` claim in the user's JWT/Session token.
    * If a WAF rule is not possible, temporarily disable the specific `/customers/view` endpoint to stop active exfiltration.
3.  **Preserve Evidence (T+0:30):** Snapshot the API access logs and RDS query logs. Ensure log retention is extended so evidence isn't overwritten.
4.  **Internal Notification (T+0:45):** Notify the CTO, Legal/Compliance, and the Customer Success lead. Prepare a "Holding Statement" for affected clients if a breach is confirmed.

---

## 2. Investigation Checklist
To understand the "Blast Radius," we must answer three questions:

### A. How widespread is this?
- [ ] **Scan Codebase:** Use `grep` or automated static analysis (SAST) to find all instances where `company_id` is pulled from `request.params` or `request.query` without a secondary ownership check.
- [ ] **API Discovery:** Check if similar parameters exist in the Mobile App endpoints or other dashboard views (e.g., `/appointments/view?company_id=...`).

### B. What data was potentially exposed?
- [ ] **Log Analysis:** Filter logs for the `/customers/view` endpoint. Identify requests where the `user_session_id` accessed a `company_id` different from their registered employer.
- [ ] **Data Sensitivity Audit:** Confirm if the exposed records contained plain-text payment history or PII protected under GDPR/UK-DPA.

### C. Was it exploited before?
- [ ] **Historical Log Review:** Analyze the last 90 days of logs for "Sequential ID Enumeration" (e.g., one IP requesting IDs 1001, 1002, 1003 in rapid succession).
- [ ] **Anomaly Detection:** Look for spikes in outbound data transfer (egress) from the API.

---

## 3. Root Cause Analysis
I have identified three possible technical causes, ranked by likelihood:

1.  **Missing Ownership Validation (Most Likely):** The code verifies that a user is *authenticated* (logged in) but assumes that any logged-in user can access any record if they have the ID. This is the hallmark of early-stage SaaS "growth-over-security" coding.
2.  **Implicit Trust in Client-Side Input:** The backend developer likely trusted the `company_id` sent by the frontend rather than looking up the user’s authorized `company_id` from the secure server-side session.
3.  **Inconsistent Middleware Application:** The application may have a "CheckOwnership" middleware, but it was not applied to this specific `/customers/view` route during a recent update.

---

## 4. Fix Validation
Before the "Fixed" tag is accepted, the following tests must pass:

| Test Case | Method | Expected Result |
| :--- | :--- | :--- |
| **Cross-Tenant Test** | Log in as Company A; request `company_id` for Company B. | `403 Forbidden` or `404 Not Found`. |
| **Boundary Test** | Request `company_id=0`, `company_id=-1`, and `company_id=NULL`. | Graceful error handling; no stack trace leaked. |
| **No-Session Test** | Request `company_id=1234` without an Auth header. | `401 Unauthorized`. |
| **Negative Testing** | Attempt to access a `company_id` that has been deleted or suspended. | `403 Forbidden`. |
| **Logging Verification** | Trigger an unauthorized access attempt. | Verify an "Authorization Failure" alert is generated in the SIEM/Logs. |

---
*Note: This incident response framework emphasizes the "Secure by Design" principles taught at the University of Aberdeen, focusing on protecting multi-tenant integrity.*
