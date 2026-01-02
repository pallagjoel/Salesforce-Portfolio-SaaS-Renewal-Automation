# Enterprise SaaS Subscription & Revenue Operations Solution

![Salesforce](https://img.shields.io/badge/Platform-Salesforce-blue)
![Architecture](https://img.shields.io/badge/Architecture-Metadata--Driven-orange)
![Status](https://img.shields.io/badge/Status-Production--Ready-brightgreen)
![Context](https://img.shields.io/badge/Context-2025--Global--Tech-purple)

## Summary: Bridging the Gap Between Sales and Finance
In the SaaS ecosystem, manual administration is the primary cause of revenue leakage and churn. This project delivers a **comprehensive, metadata-driven architectural solution** designed to automate the end-to-end subscription lifecycle. 

Rather than just a collection of custom objects, this is a **Revenue Operations Engine**. It synchronizes automated renewal pipelines, enforces strict data governance through a custom logic, and provides real-time C-Suite analytics. By transforming raw usage data into predictive "Health Scores," the system allows stakeholders to identify upsell opportunities and mitigate churn risk pro-actively.

---

## Architectural Deep Dive: Performance & Scalability

### 1. High-Performance Process Orchestration
This solution utilizes a "Flow-First" strategy, optimized for **scalability** and **Governor Limit awareness**.

* **Proactive Subscription Overlap Prevention (Subscription_Overlap_Checker):**
    * **The Technical Challenge:** Overlapping subscriptions for a single Account lead to corrupted MRR reporting and billing disputes.
    * **The Solution:** A high-performance **Before-Save Flow**. By intercepting the save operation *before* it hits the database, we eliminate unnecessary DML statements and recursive triggers.
    * **Bulkification Logic:** The logic is designed to handle mass data imports (via Data Loader). It utilizes a sophisticated interval-overlap algorithm: `(Start_Date <= New_End_Date) AND (End_Date >= New_Start_Date)`. While the logic is optimized, it is subject to the standard 101 SOQL limit per transaction. For enterprise-scale data volumes (5,000+ records/batch), an Apex Trigger implementation is recommended.

* **Multi-Stage Intelligent Renewal & Pipeline Engine:**
    * **The Strategy:** Utilizes **Scheduled Paths** within a Record-Triggered Flow to manage the subscription lifecycle.
  * **The 90/30 Rule:**
      * **T-90 Days (Early Warning):** Identifies Subscriptions with low Usage utilization (Under 20%), and high  Usage utilization (Above 80%) and generates a Churn Prevention, or an Upsell Opportunity 90 days prior the End_date.

      * **T-30 Days (Standard Renewal):** Generates a standard Renewal Opportunity for healthy Subscriptions 30 days prior the End_date.

   * **Duplicate Prevention:** Every Scheduled path includes a Pre-Execution Check (Get Records) to detect existing open opportunities, preventing "CRM Noise" and redundant pipelines.

* **Event-Driven Expansion (Immediate Path):**

   * **100% Limit Trigger:** Monitors usage logs in real-time. When utilization hits 100%, it triggers an immediate Upsell Opportunity.

   * **Prior Value Logic:** Utilizes $Record__Prior to ensure the automation only triggers once at the threshold, ensuring high data hygiene.

---

### 2. Data Governance:
One of the most frequent failures in Salesforce implementations is "Data Rot" - the ability for users to modify historical financial data. This project enforces Full Lifecycle Immutability.

* **Total Subscription Lock:** Beyond locking just financial fields, custom **Validation Rules** enforce a complete record lock once a subscription enters an **"Expired"** status. This prevents any further modifications to the record, ensuring that historical data remains a "Single Source of Truth" for auditing.
*  **Usage Log Integrity (Zero-Post Lockout):** To prevent financial and usage-based discrepancies, the system implements a strict lockout policy: **no new** `Usage_Log__c` **records can be created** for subscriptions that are Expired. This ensures that telemetry data is strictly aligned with the valid contract period.
* **Tier-Driven Logic Enforcement:** The architecture prevents "Tier Drift" by validating that `Data_Limit__c` remains strictly aligned with the `Product_Tier__c`. This ensures that a "Basic" customer cannot accidentally access "Professional" resources, protecting the company's value proposition and revenue streams.
*  **Enterprise Tier Isolation (Soft-Limit Strategy):** To ensure VIP customers enjoy an "unlimited" experience, the Enterprise tier is programmatically assigned a high-buffer limit (999999999999999999999999999999999999 GB) and is excluded from automated churn/upsell triggers.

---

### 3. Predictive Intelligence & Analytics Layer
The system transforms raw usage logs into a **strategic decision-support tool**.

* **Usage-Based Customer Health Scoring (Formula Engine):**
    * **CRITICAL (Utilization < 20%):** Flagged as **"At Risk"**. These accounts are identified as high-probability churn candidates, triggering immediate Customer Success intervention.
    * **OPTIMAL (20% - 80%):** Flagged as **"Healthy"**.The baseline for steady-state MRR retention.
    * **EXPANSION (Utilization > 80%):** Flagged as **"Upsell Potential"**. These records are automatically routed to a dedicated "Expansion Pipeline," allowing Sales to trigger tier-upgrade conversations when the customer's needs exceed their current plan.

* **The CFO Strategic Dashboard:**
    * **MRR Aggregation:** Real-time visibility into Monthly Recurring Revenue.
    * **Upcoming Renewals - Next 30 Days:** Lihtning Table, to show all upcoming renewal subscriptions.
    * **Usage Trend:** Line chart where Directors can actively see, all the usage of the Software daily.
    * **Churn Mitigation View:** Visualizes revenue tied to low-utilization accounts.
    * **Enterprise Tier Isolation:** Strategic filters exclude "Enterprise" accounts from noise-level alerts, allowing the executive team to focus on high-volume segments where automated triggers are most effective.

---

## Technical Stack & DevSecOps
* **Salesforce DX (SFDX) Architecture:** Fully decoupled metadata structure, ensuring clean version control.
* **Metadata API Strategy:** Built using the `sf` (Salesforce CLI) for modular deployment.
* **External Data Ingestion & API Integration Architecture:** The system is designed with an API-first approach, positioning the Usage_Log__c object as an Integration Landing Zone. In a production environment, telemetry data from cloud providers (e.g., AWS S3, Heroku, or Google Cloud) is pushed into Salesforce via the standard REST API.
   * **Key Integration Features:**

      * **Decoupled Design:** By using a dedicated log object, the business logic is decoupled from the data ingestion layer, ensuring system stability.

      * **Auditability & Transparency:** Every external data point is stored as a record, providing a 100% transparent audit trail for usage-based billing and dispute resolution.

      * **Scalability:** The architecture is built to handle high-frequency telemetry updates, utilizing Salesforce's bulk-compatible data model to process incoming logs in real-time.
    
   * **Sample API Payload (JSON Specification):**
    ```json {
  "Subscription__c": "SUB-001",
  "Usage_Date__c": "2026-01-01",
  "Active_Users__c": "7",
  "Storage_Used_GB__c": 15.75,
  "Source_System__c": "AWS-CloudWatch-Monitor"


* **Environment-Agnostic Design:** Zero Hardcoding. The system contains **no hardcoded IDs** (e.g., `001...`). All logic is driven by API names and dynamic references, ensuring the package can be deployed from Sandbox to Production with 100% success rates.
* **Optimized Data Model:** Strategic use of **Master-Detail Relationships** between `Usage_Log__c` and `Subscription__c`, enabling high-speed **Roll-up Summaries** for real-time reporting without the overhead of Batch Apex.

---

## Deployment Instructions

> [!CAUTION]
> Deployment requires Salesforce CLI and an authorized Developer Edition or Scratch Org.

1.  **Clone the Repository:**
    ```bash
    git clone [https://github.com/pallagjoel/Salesforce-Portfolio-SaaS-Renewal-Automation.git](https://github.com/pallagjoel/Salesforce-Portfolio-SaaS-Renewal-Automation.git)
    ```
2.  **Authorize the Target Org:**
    ```bash
    sf org login web -a SaaS_Production
    ```
3.  **Deploy Metadata:**
    ```bash
    sf project deploy start
    ```


---
**Lead Architect:** [Pallag Joel](https://github.com/pallagjoel) | Salesforce Solutions Specialist & RevOps Strategist
