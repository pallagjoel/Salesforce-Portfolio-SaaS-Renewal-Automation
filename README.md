# Enterprise SaaS Subscription & Revenue Operations (RevOps) Solution

![Salesforce](https://img.shields.io/badge/Platform-Salesforce-blue)
![Architecture](https://img.shields.io/badge/Architecture-Metadata--Driven-orange)
![Status](https://img.shields.io/badge/Status-Production--Ready-brightgreen)
![Context](https://img.shields.io/badge/Context-2025--Global--Tech-purple)

## Executive Summary: Bridging the Gap Between Sales and Finance
In the high-velocity SaaS ecosystem, manual administration is the primary cause of revenue leakage and churn. This project delivers a **comprehensive, metadata-driven architectural solution** designed to automate the end-to-end subscription lifecycle. 

Rather than just a collection of custom objects, this is a **Revenue Operations (RevOps) Engine**. It synchronizes automated renewal pipelines, enforces strict data governance through a custom "State-Machine" logic, and provides real-time C-Suite analytics. By transforming raw usage data into predictive "Health Scores," the system allows stakeholders to identify upsell opportunities and mitigate churn risk pro-actively.

---

## Architectural Deep Dive: Performance & Scalability

### 1. High-Performance Process Orchestration
This solution utilizes a "Flow-First" strategy, optimized for **scalability** and **Governor Limit awareness**.

* **Proactive Subscription Overlap Prevention (Subscription_Overlap_Checker):**
    * **The Technical Challenge:** Overlapping subscriptions for a single Account lead to corrupted MRR reporting and billing disputes.
    * **The Solution:** A high-performance **Before-Save Flow**. By intercepting the save operation *before* it hits the database, we eliminate unnecessary DML statements and recursive triggers.
    * **Bulkification Logic:** The logic is designed to handle mass data imports (via Data Loader). It utilizes a sophisticated interval-overlap algorithm: `(Start_Date <= New_End_Date) AND (End_Date >= New_Start_Date)`. While the logic is optimized, it is subject to the standard 101 SOQL limit per transaction. For enterprise-scale data volumes (5,000+ records/batch), an Apex Trigger implementation needed.

* **Automated Renewal & Pipeline Engine:**
    * **The Strategy:** A Record-Triggered Flow handles the asynchronous generation of Renewal Opportunities. 
    * **Business Intelligence:** The engine doesn't simply clone records. It calculates dynamic date offsets based on contract terms, ensuring that the Sales team is alerted exactly 90 days before expiration, providing sufficient "runway" for contract negotiations.

---

### 2. Data Governance: The "State-Machine" Model
One of the most frequent failures in Salesforce implementations is "Data Rot"—the ability for users to modify historical financial data. This project enforces **Immutability**.

* **Immutable Financial Records:** Custom **Validation Rules** lock critical fields such as `Monthly_Amount__c`, `Product_Tier__c`, and `Start_Date__c` once a subscription enters a "Closed" or "Expired" status. This creates a tamper-proof audit trail for financial reporting.
* **Tier-Driven Logic Enforcement:** The architecture prevents "Tier Drift" by validating that `Data_Limit__c` remains strictly aligned with the `Product_Tier__c`. This ensures that a "Basic" customer cannot accidentally access "Professional" resources, protecting the company's value proposition and revenue streams.

---

### 3. Predictive Intelligence & Analytics Layer
The system transforms raw usage logs into a **strategic decision-support tool**.

* **Usage-Based Customer Health Scoring (Formula Engine):**
    * **CRITICAL (Utilization < 20%):** Flagged as **"At Risk"**. These accounts are identified as high-probability churn candidates, triggering immediate Customer Success intervention.
    * **OPTIMAL (20% - 80%):** The baseline for steady-state MRR retention.
    * **EXPANSION (Utilization > 80%):** Flagged as **"Upsell Potential"**. These records are automatically routed to a dedicated "Expansion Pipeline," allowing Sales to trigger tier-upgrade conversations when the customer's needs exceed their current plan.

* **The CFO Strategic Dashboard:**
    * **MRR Aggregation:** Real-time visibility into Monthly Recurring Revenue.
    * **Churn Mitigation View:** Visualizes revenue tied to low-utilization accounts.
    * **Enterprise Tier Isolation:** Strategic filters exclude "Enterprise" accounts from noise-level alerts, allowing the executive team to focus on high-volume segments where automated triggers are most effective.

---

## Technical Stack & DevSecOps
* **Salesforce DX (SFDX) Architecture:** Fully decoupled metadata structure, ensuring clean version control.
* **Metadata API Strategy:** Built using the `sf` (Salesforce CLI) for modular deployment.
* **External Data Ingestion & API Integration Architecture:** The system is designed with an API-first approach, positioning the Usage_Log__c object as an Integration Landing Zone. In a production environment, telemetry data from cloud providers (e.g., AWS S3, Heroku, or Google Cloud) is pushed into Salesforce via the standard REST API.
   * **Key Integration Features:**

      * **Decoupled Design:** By using a dedicated log object, the business logic (Health Scoring, Renewal Triggers) is decoupled from the data ingestion layer, ensuring system stability.

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

## Projected Business Impact
* **95% Reduction** in manual administrative labor for Opportunity creation.
* **100% Data Accuracy** regarding subscription timeline overlaps.
* **Proactive Revenue Protection:** Sales teams identify churn risks 3-6 months earlier than manual review cycles.
* **Accelerated Upsell Cycles:** Data-driven identification of power users ready for higher-value tiers.

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

## Future Roadmap (The Next Level)
* **Apex Trigger Implementation:** Transitioning logic to an Apex Trigger Framework for extreme-scale bulk handling (10,000+ records).
* **External Service Integration:** Connecting to Slack/Teams for real-time "Red Alert" notifications when a Tier-1 customer's usage drops below 10%.
* **Einstein Discovery Integration:** Utilizing Machine Learning to refine Health Scores based on historical churn patterns.

---
**Lead Architect:** [Pallag Joel](https://github.com/pallagjoel) | Salesforce Solutions Specialist & RevOps Strategist
