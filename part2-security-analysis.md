# Part 2: Deep-Dive Security Analysis
**Focus Area:** Broken Object Level Authorization (BOLA / IDOR)

## 1. Technical Explanation: How the Attack Works
Broken Object Level Authorization occurs when an application provides access to objects based on user-supplied input but fails to verify if the requester has the right to access that specific object.

### The Attack Flow
In the context of ClientHub, the attack follows these steps:

1.  **Discovery:** An attacker logs in as a "Sales Rep" for *Company A*. They navigate to their own customer list and notice the URL: `https://clienthub.com/api/v1/customers?company_id=1234`.
2.  **Manipulation:** The attacker uses a proxy tool (like Burp Suite) or simply edits the browser URL to change the ID: `company_id=1235`.
3.  **Exploitation:** The backend receives the request. It checks if the user is logged in (Authentication), which they are. However, it fails to check if the user belongs to `company_id=1235` (Authorization).
4.  **Data Exfiltration:** The system queries the PostgreSQL database for all records where `company_id = 1235` and returns them to the attacker. The attacker then writes a simple script to iterate through IDs from 1000 to 9999, effectively scraping the entire database.



---

## 2. Real-World Example: Honda eCommerce Platform (2023)
In 2023, a security researcher discovered a massive BOLA/IDOR vulnerability in Honda's dealer management systems. By simply changing a non-encrypted ID in the URL, the researcher gained access to over **21,000 customer orders**, including names, addresses, and phone numbers, as well as 1,500 dealer sites.

**Source:** [SecurityWeek: Vulnerabilities in Honda eCommerce Platform Exposed Customer, Dealer Data](https://www.securityweek.com/vulnerabilities-in-honda-ecommerce-platform-exposed-customer-dealer-data/)

---

## 3. Defense Strategy
To secure ClientHub, I propose a "Defense-in-Depth" approach focusing on the root cause: the lack of relationship validation.

| Control Type | Implementation | Why it Works | How to Verify |
| :--- | :--- | :--- | :--- |
| **Prevention** | **Policy-Based Access Control (PBAC)** | It moves authorization from the endpoint code to a centralized middleware that validates the `user_id` -> `company_id` relationship before every query. | Attempt to access `company_id=999` from a session tied to `company_id=111` and ensure a `403 Forbidden` response. |
| **Detection** | **API Rate Limiting & Anomaly Detection** | Attackers exploiting BOLA usually iterate IDs quickly. Detection triggers an alert when a single user accesses >5 unique `company_id` values in 1 minute. | Use a script to rapidly request 10 different IDs and verify the account is automatically throttled or flagged in logs. |
| **Response** | **UUIDs (Non-Enumerable IDs)** | Replacing sequential integers (1234) with version 4 UUIDs makes "ID guessing" computationally impossible. | Verify that all public-facing API endpoints return strings like `550e8400-e29b-41d4...` instead of integers. |

---

## 4. Trade-offs & Practical Challenges

1.  **PBAC Middleware (Complexity):** Implementing centralized authorization adds latency to every API call. Developers must be disciplined in ensuring *every* new endpoint is wrapped in this middleware, or "shadow endpoints" will remain vulnerable.
2.  **Rate Limiting (UX Impact):** If set too strictly, power users (like Admins running bulk reports) might get accidentally blocked, leading to "false positives" and increased support tickets.
3.  **UUID Migration (Cost/Legacy):** Changing from integers to UUIDs requires a significant database migration. Existing third-party integrations (Calendar/Email sync) may break if they expect integer-based IDs, requiring a phased "Legacy ID" support period.

---
*Note: As a student of both Cybersecurity and Machine Learning at the University of Aberdeen, I have analyzed these risks with a focus on data privacy. My recommendations aim to protect the underlying customer datasets from unauthorized exposure, which is critical for maintaining the integrity of future data-driven features.*
