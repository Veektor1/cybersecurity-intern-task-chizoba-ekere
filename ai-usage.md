# AI Usage Documentation

This document outlines the collaborative role Artificial Intelligence played in the development of this security assessment, in accordance with the transparency requirements of the Nexcell Solutions internship task.

## 1. Where AI was Utilized
I leveraged AI as a research and structural collaborator for:
* **Attack Surface Brainstorming:** Generating a broad list of cloud-based CRM vulnerabilities to ensure comprehensive coverage.
* **Incident Response Framework:** Structuring the 60-minute triage steps based on NIST standards.
* **Editorial Review:** Ensuring the technical explanations were clear, concise, and professional.

## 2. Key Prompts Used
To maintain high-quality outputs, I used specific, context-heavy prompts:
1. *"Identify high-impact attack surfaces for a multi-tenant CRM specifically using AWS S3 and RDS PostgreSQL."*
2. *"Explain a BOLA/IDOR attack sequence and provide a real-world parallel from 2023 or 2024 involving customer PII."*
3. *"Review my incident response checklist for a URL parameter manipulation scenario and suggest one detection-focused control."*

## 3. Critical Evaluation: What I Changed or Rejected
Independent reasoning was applied to all AI-generated suggestions:
* **Prioritization Pivot:** AI initially suggested SQL Injection as the #1 risk. I **rejected** this in favor of **BOLA (Broken Object Level Authorization)**. Based on the 18-month age of ClientHub and the scenario in Part 3, BOLA is a more realistic and catastrophic "existential threat" for this specific company.
* **Control Selection:** AI suggested complex biometric MFA for all field staff. I **modified** this to focus on **Session Token Encryption** and **UUID implementation**, as these are more cost-effective and practical for a 200-client startup.
* **Contextual Nuance:** I manually integrated the trade-offs section in Part 2, focusing on the UX impact on "Sales Reps", a perspective the AI did not initially provide.

## 4. Original Analysis & University Contributions
The following sections represent my original work and reasoning, independent of AI:
* **The "Blast Radius" Framework:** My logic for prioritizing risks was developed through the lens of **multi-tenancy integrity**, a key focus of my Cybersecurity studies at the **University of Aberdeen**.
* **ML Security Integration:** Drawing from my Machine Learning coursework, I added analysis regarding the protection of customer datasets which serve as the foundation for future predictive analytics features.
* **Practical Fix Validation:** I designed the specific "Negative Testing" scenarios (e.g., testing with `company_id=NULL` and deleted IDs) based on defensive coding principles learned in my lab sessions.

---
**Candidate Statement:** *I view AI as a tool to augment efficiency, but the security reasoning, risk prioritization, and ethical judgment within this project are entirely my own.*
