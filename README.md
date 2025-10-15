![HEALTHCARE](https://github.com/user-attachments/assets/be123116-11b6-4910-92df-735d27852e1d)

# Healthcare Analytics Project Part 1: Company-Focused
A data analytics project focused on helping healthcare organizations identify inefficiencies, optimize revenue cycles, and improve hospital performance through data-driven insights derived from a tailor-made realistic synthetic healthcare dataset generator.

## Background and Overview


The ultimate goal of this project is to analyze patterns in care quality, hospital efficiency, and payment behavior using SQL — demonstrating how data-driven insights can improve both operational performance and patient outcomes.

- EDA strategies in Excel and SQL used to inspect, clean, and quality-check the dataset can be found [here](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/Excel%20Cleaning%20%26%20EDA.md).
- Targeted SQL queries used to answer business questions and extract insights can be found [here](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20%26%20Results.md).


## ⚙️ Dataset Generator and Structure Overview

This project includes a fully custom Healthcare Dataset Generator that creates realistic, U.S.-based healthcare data for analysis — entirely synthetic, with built-in randomness so no two datasets are ever the same. It’s an analytics-ready foundation for exploring trends in patient outcomes, billing efficiency, and hospital performance — without exposing real patient data.

The dataset simulates how hospitals operate day-to-day: patients are admitted for various conditions, receive treatments of differing severities, and are billed through diverse insurance and payment plans. Each record is generated using probabilistic logic to mimic real-world complexity — from follow-up visits and recurring conditions to late payments and same-day discharges.

Each dataset models real-world hospital operations through three linked tables: Patients, Visits, and Billing.

Generator Logic
- Condition, treatment, and billing logic follow medically realistic patterns.
- Payment behavior, insurance coverage, and follow-up care evolve dynamically.
- All data passes a validation system ensuring consistent ages, conditions, billing rules, and table relationships.

To download or learn more about the dataset generator, click [here](https://github.com/MichaelZaniewski/Healthcare-Dataset-Generator/blob/main/README.md)

## Executive Summary
### Overview of Findings
Paragraph form of main takeaway/how companies should leverage the findings
  
  - Highlights where hospitals face the greatest revenue risk, displaying the importance of prioritizing insurer negotiations.

### Findings

1) Late-payment risk is concentrated by two insurers.
   - Brown LLC and Williams LLC lead both in the count of late payments and in late revenue exposure, totaling about $2.6M and $1.7M respectively, with roughly 35–45% of that exposure still unpaid. This concentration indicates that a small number of payers disproportionately drive cash-flow drag.
2) Uninsured patients default almost twice as often.
   - Default rate is 20.36% for uninsured patients vs 11.89% for insured, meaning insurance status is a major driver of nonpayment. Bad-debt risk increases whenever the payer mix shifts toward self-pay.
3) Complex conditions drive the highest late-payment rates.
   - Cancer (49%), Kidney failure (47%), and Stroke (46%) combine high visit volume with high lateness, creating the largest delayed-cash exposure. These conditions typically involve multi-day care and repeated encounters, which extend billing and reimbursement timelines.
4) Costs climb quickly with longer stays
   - Average charges rise from same-day visits to $63K+ around 8–10 nights; per-day costs are highest for Kidney failure, Stroke, and Cancer (~$17–18K/day). Small changes in length of stay for these groups translate into large swings in total cost.
5) Follow-ups cost less but make up much of the workload
   - Follow-ups are typically 35–65% of the cost of initial visits (often ~$30K less for complex cases) and occur more often, so they represent a large share of total care.
As a result, capacity planning should account for steady follow-up demand even when initial visits fluctuate.

### Recommendations
1) Partner with insurers associated with the most late payments to cut bad debt 
   - Introduce real-time eligibility and cost estimate share up-front to educate patients, allowing them to see their out-of-pocket costs before care and decide the best course of action.
   - Share data to identify high-risk cohorts requiring special attention.
  
2) Introduce stronger collection strategies for uninsured patients
   - test
   - test
     
3) Create condition-focused pre-authorization & benefits checks for high-risk cohorts
   - A short, mandatory, front-end checklist before treatment to confirm coverage, authorizations, and patient financial setup. The hospital will provide a transparent estimate of charges, explain copay/coinsurance, available payment plans, and frequent payment reminders for high-risk late-payment conditions to reduce the frequency of late payments.
   - Rollout first to:
        -  Oncology (cancer): infusions, imaging, biopsies, radiation therapy.
        -  Renal (kidney failure): dialysis, inpatient renal procedures, high-cost meds.
        -  Neuro/Cardiac (stroke): advanced imaging, interventions, rehab placement.
          
4) Create a "same-day criteria" bundle to reduce one-night stays
   - For a small list of low-risk, high-volume conditions (asthma, migraine, allergies, flu)
   - A list of minor conditions with low risk-high volume cases (athsma, migraine, dehydration) with eligiblity crieria (stable vitals, pain controlled on oral meds, no red flags, has safe transporation home, support at home)
   - test
     
5) Optimize dischage planning for conditions with high los
   - test
   - test


## Assumptions, Limitations, and Caveats
### Assumptions
- Follow‑ups are correctly linked to the original visit. “Follow‑up” ties back to the initial episode for the same patient and condition.
- Business rules remained stable. Payer rules, common care pathways, and charging practices did not change during the analysis window to invalidate comparisons.
  
### Limitations
- Synthetica data—despite realistic rules— may not capture true clinical variation like seasonal spikes. Real‑world behavior (patients, providers, payers) can differ.
- Simplified payer behavior. Denials, appeals, carve‑outs, and clawbacks are not modeled, so “revenue at risk” may be conservative or optimistic.
  
### Caveats
- "Unnecessary repeat visits" should be risk-adjusted to avoid penalizing complex-case physicians.
- Pilot before scaling. Treat recommendations as a starting playbook; run small tests, measure results, and expand what works.
