# Part 1: Threat Model & Attack Surface Analysis

## 1. Attack Surface Identification

I have identified the 5 most critical attack surfaces for ClientHub. My analysis focuses on the specific architecture of an 18-month-old SaaS CRM migrating toward enterprise readiness.

| Attack Surface | Description |
| :--- | :--- |
| **1. Multi-Tenant API Logic** | The core logic handling data requests. The primary risk is "Vertical or Horizontal Privilege Escalation" between different business clients. |
| **2. Public Cloud Storage (S3)** | Used for file uploads (contracts, PII). Misconfigured bucket policies or "signed URL" leaks could expose raw data. |
| **3. Mobile App Auth Layer** | Field staff access points. Vulnerable to insecure token storage and Man-in-the-Middle (MitM) attacks on public Wi-Fi. |
| **4. Third-Party Integrations** | OAuth-based syncs for Email/Calendar. A compromise here leaks data outside of the ClientHub ecosystem. |
| **5. Database Layer (PostgreSQL)** | The RDS instance. Vulnerable to SQL injection in reporting modules or credential theft via leaked environment variables. |

---

## 2. Risk Assessment

### Surface 1: Multi-Tenant API Logic (BOLA/IDOR)
* **The Threat:** An attacker modifies a URL or body parameter (e.g., `company_id=123`) to access records belonging to a competitor.
* **Impact:** **CRITICAL.** Total exposure of the 50,000 monthly records and catastrophic loss of client trust.
* **Likelihood:** **High.** This is the most common flaw in rapidly scaled multi-tenant platforms.

### Surface 2: Publicly Accessible S3 Buckets
* **The Threat:** S3 buckets intended for private customer files are accidentally set to "Public" during a deployment update.
* **Impact:** **High.** Massive PII leak (addresses, phone numbers, payment histories) leading to GDPR/UK-DPA violations.
* **Likelihood:** **Medium.** Configuration drift is common as startups grow their AWS footprint.

### Surface 3: Stolen Mobile Session Tokens
* **The Threat:** JWTs or session cookies stored in unencrypted local storage on mobile devices are harvested by mobile malware.
* **Impact:** **Medium.** Targeted account takeover of specific field staff, allowing record manipulation.
* **Likelihood:** **Medium.** Field staff often use personal devices with varying security postures.

### Surface 4: OAuth Token Theft
* **The Threat:** Weak redirect URI validation allows an attacker to steal OAuth tokens for Google/Outlook integrations.
* **Impact:** **High.** Unauthorized access to business communications and sensitive appointment schedules.
* **Likelihood:** **Low.** Requires specialized knowledge of the integration's handshake process.

### Surface 5: RDS Data Exfiltration via SQLi
* **The Threat:** Improperly sanitized inputs in the "Global Search" feature allow for Blind SQL Injection.
* **Impact:** **Critical.** Full database dump, including administrative credentials and system metadata.
* **Likelihood:** **Low.** Modern frameworks provide significant protection, though custom reporting queries are often a weak point.

---

## 3. Prioritized Top 3 Risks

1. **Cross-Tenant Data Leakage (BOLA/IDOR)**
2. **S3 Bucket Misconfiguration (PII Exposure)**
3. **Compromised Mobile Auth (Field Staff Exploitation)**

---

## 4. Prioritization Logic

My prioritization logic is driven by the **"Business Existentialism"** framework, meaning I prioritized risks that could end the company if exploited:

* **Why #1 (BOLA)?** For a CRM, the "Wall of Privacy" between clients is the product. As evidenced by the reported issue in Part 3, this is an active vulnerability. If not solved, enterprise clients will not onboard.
* **Why #2 (S3)?** Modern privacy laws (GDPR) carry heavy fines for PII leaks. File storage is often less audited than database code, making it a "high-impact" sleeper risk.
* **Why #3 (Mobile Auth)?** Field staff are the "boots on the ground." Their devices are the most exposed part of the infrastructure. Securing the mobile entry point is vital for a system that processes 50k records monthly.
