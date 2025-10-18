![healthcare](https://github.com/user-attachments/assets/917c0969-65dc-4edd-9919-a9db6e985e40)

# Healthcare Analytics Project Part 1: Company-Focused
A data analytics project focused on helping healthcare organizations identify inefficiencies, optimize revenue cycles, and improve hospital performance through data-driven insights derived from a custom realistic synthetic healthcare dataset generator.

## Background and Overview

The goal of this project is to leverage SQL to analyze utilization, length of stay, and payment behavior so leaders can improve capacity planning, streamline care pathways, and strengthen financial performance.

Insights and recommendations are provided on the following key areas:

- The relationships between condition, LOS, and total charge
- Hospital and doctor efficiency in treating patients in terms of followup visits and 7-day bouncebacks
- Insurance provider members performance: late payments, defaults 
- High and low risk financial exposure per condition and strategies for combating late payments
- Reducing average LOS to improve operational efficiency, turnover, and revenue performance

EDA strategies in Excel and SQL used to inspect, clean, and quality-check the dataset can be found [here](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/Excel%20Cleaning%20&%20SQL%20EDA.md).

Targeted SQL queries used to answer business questions and extract insights can be found [here](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20%26%20Results.md).

## ⚙️ Dataset Generator and Structure Overview

This project includes a fully custom Healthcare Dataset Generator that creates realistic, U.S.-based healthcare data for analysis — entirely synthetic, with built-in randomness so no two datasets are ever the same. It’s an analytics-ready foundation for exploring trends in patient outcomes, billing efficiency, and hospital performance — without exposing real patient data.

The dataset simulates how hospitals operate day-to-day: patients are admitted for various conditions, receive treatments of differing severities, and are billed through diverse insurance and payment plans. Each record is generated using probabilistic logic to mimic real-world complexity — from follow-up visits and recurring conditions to late payments and same-day discharges.

Each dataset models real-world hospital operations through three linked tables: `Patients`, `Visits`, and `Billing`.

### Generator Logic
**Realism:** Data behaves like a real hospital: visit lengths range from same-day to multi-night, follow-ups occur where they make sense, and charges scale with condition severity and length of stay.

**Customizability:** Choose how many patients to generate, set the date window, and target real U.S. ZIP codes. You can also use a fixed seed so results are repeatable.

**Validation:** Built-in checks confirm clean links between patients, visits, and billing, enforce age and condition rules, and flag issues before you start analyzing.

To download or learn more about the dataset generator, click [here](https://github.com/MichaelZaniewski/Healthcare-Dataset-Generator/blob/main/README.md).

## Executive Summary
### Glossary

- **LOS (Length of Stay) -** The number of nights a patient spends in the hospital for a visit.
- **Late-Paid -** A patient-responsibility balance that was eventually paid, but after the due date.
- **Late-Unpaid -** A patient-responsibility balance that is past due and still unpaid.
- **Follow-up -** A return visit for the same condition after an initial visit, typically for monitoring, treatment progression, or recovery checks.
- **Same-day -** A visit where the patient is admitted and discharged on the same day (LOS = 0).
- **One-night -** A visit with one overnight stay (LOS = 1).
- **Multi-night -** A visit with two or more overnight stays (LOS ≥ 2).

### Overview of Findings

Most cash-flow strain traces back to a few sources. Two insurers account for a large share of late and unpaid balances (by members), and patients without insurance default nearly twice as often as those with coverage. Complex conditions like cancer, kidney failure, and stroke show the highest late-payment rates because care is longer and involves repeat visits. 

Costs also climb quickly as stays get longer, peaking around the eight to ten-night range, which means even a small reduction in length of stay can save money. At the same time, follow-up visits are cheaper but far more frequent, so they make up a large share of the work that fills clinics and drives predictable revenue. Put together, the data points to a focused playbook: work directly with the two insurers linked to the most late balances, give uninsured and high-risk patients clear costs and support up front, tighten pre-authorization and benefits checks for complex care, and expand same-day pathways and strong discharge planning to reduce unnecessary extra days.

### Findings

1) Late-payment risk is concentrated by two insurers.
   - Brown LLC and Williams LLC lead both in the count of late payments by insurer members and in late revenue exposure, totaling about $2.6M and $1.7M respectively, with roughly 35–45% of that exposure still unpaid. This concentration indicates that a small number of payers disproportionately drive cash-flow drag. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20&%20Results.md#2-what-is-the-total-financial-loss-from-lateunpaid-bills-by-insurer-members)
2) Uninsured patients default almost twice as often.
   - Default rate is 20.36% for uninsured patients vs 11.89% for insured, meaning insurance status is a major driver of nonpayment. Bad-debt risk increases whenever the payer mix shifts toward self-pay. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20&%20Results.md#3-do-uninsuredself-pay-patients-have-significantly-higher-default-rates)
3) Complex conditions drive the highest late-payment rates.
   - Cancer (49%), Kidney failure (47%), and Stroke (46%) combine high visit volume with high lateness, creating the largest delayed-cash exposure. These conditions typically involve multi-day care and repeated encounters, which extend billing and reimbursement timelines. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20&%20Results.md#4-what-is-the-rate-of-late-payments-by-condition)
4) Costs climb quickly with longer stays
   - Average charges rise from same-day visits to $63K+ around 8–10 nights; per-day costs are highest for Kidney failure, Stroke, and Cancer (~$17–18K/day). Small changes in length of stay for these groups translate into large swings in total cost. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20&%20Results.md#1-what-is-the-average-cost-per-day-as-los-increases)
5) Follow-ups cost less but make up much of the workload
   - Follow-ups are typically 35–65% of the cost of initial visits (often ~$30K less for complex cases) and occur more often, so they represent a large share of total care. As a result, capacity planning should account for steady follow-up demand even when initial visits fluctuate. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20%26%20Results.md#2-what-is-the-incremental-cost-of-follow-ups-compared-to-initial-visits)
6) Majority of the patients are discharged after a night spent in the hospital
   - Statistics: 7.29% same-day, 63.51% one-night, 29.20% multi-night from 262,664 visits. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20%26%20Results.md#1b-for-the-hospital-with-the-highest-patient-volume-what-percentage-of-patients-are-discharged-the-same-day-vs-multi-day-stays-compare-to-the-rest-of-the-dataset)
7) The longest LOS is attributed to cancer, but the most expensive condition is cancer (in the last full year of the dataset, 2024)
   - COVID has a maximum LOS of 14 while cancer has 10, but the total charge for COVID at 14 LOS is only 3.5 million whereas cancer at 10 LOS is 78 million. [Query](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20%26%20Results.md#2-what-conditions-drive-the-longest-and-most-expensive-hospital-stays)

## Recommendations
1) Partner with insurers associated with the most late payments to cut bad debt 
   - Introduce real-time eligibility and cost estimate shared up front to educate patients, allowing them to see their out-of-pocket costs before care and decide the best course of action.
   - Exchange data with those insurers to flag high-risk conditions and patients who are most likely to fall behind, and allocate special attention and education to that cohort.
   - Align on faster explanations of benefits and clearer member letters so patients know exactly what they owe and when.

2) Preemptive financial clearance with support & stronger collection strategies to reduce defaulting and late payments
   - Create condition-focused pre-authorization & benefits checks for high-risk cohorts
     - A short, mandatory, front-end checklist before treatment to confirm coverage, authorizations, and patient financial setup. The hospital will provide a transparent estimate of charges, explain copay and coinsurance, available payment plans, and frequent payment reminders for high-risk late-payment conditions to reduce the frequency of late payments.
     - Roll out where risk and cost are highest:
        -  Oncology (cancer): infusions, imaging, biopsies, radiation therapy.
        -  Renal (kidney failure): dialysis, inpatient renal procedures, high-cost medications.
        -  Neuro/Cardiac (stroke): advanced imaging, interventions, rehab placement.
   - Offer a short follow-up call within two to three days to answer billing questions and confirm the plan.
   - Make it easy to pay: one-click links, saved payment methods, and clear instructions on how to get help.
          
3) Create a "same-day criteria" bundle to reduce LOS from one night to same day
   - For a small list of low-risk, high-volume conditions (asthma, migraine, allergies, flu), set clear go-home-today rules. If the patient has stable vitals, controlled pain with medication, no warning signs, a safe ride home, and basic support at home, they meet the criteria for same-day release.
   - Moving appropriate patients home the same day frees beds and reduces cost.
     
4) Optimize discharge planning for conditions with high LOS
   - Since cost rises drastically as LOS increases, plan the discharge path from day one.
   - Line up rehab or home-care early (have pre-generated lists to pull from) and remove common blockers: earlier imaging reads, faster transportation scheduling, and clear weekend and holiday handoffs.

### Cost Saving Projection After Implementation

Cost saving recommendations are anticipated to decrease outstanding debt by 8%. 5% and 10% figures were included in the calculation to show sensitivity. 

<img width="1213" height="169" alt="Implemented Recommendations" src="https://github.com/user-attachments/assets/c0e90c5a-6c8e-43ef-bf47-07fbd33ab0d4" />

#### Projection Explanation
- **Scope:** The hospital with the highest outstanding debt was used for comparison
- **Timeframe:** 2024 - the last full year of the dataset
- **Billing type selected:** Patient responsibility amount - The portion of revenue most influenced by implementation 

More information about the projection can be found [here](https://github.com/MichaelZaniewski/Healthcare-Project/blob/main/SQL%20Queries%20%26%20Results.md#implementation-of-recommendations--revenue-recovery-focus).

## Assumptions, Limitations, and Caveats
### Assumptions
- Follow‑ups are correctly linked to the original visit. “Follow‑up” ties back to the initial episode for the same patient and condition.
- Business rules remained stable. Payer rules, common care pathways, and charging practices did not change during the analysis window to invalidate comparisons.
  
### Limitations
- Synthetic data, despite realistic rules, may not capture true clinical variation like seasonal spikes. Real‑world behavior (patients, providers, payers) can differ.
- Simplified payer behavior. Denials, appeals, carve‑outs, and clawbacks are not modeled, so “revenue at risk” may be conservative or optimistic.
  
### Caveats
- "Unnecessary repeat visits" should be risk-adjusted to avoid penalizing complex-case physicians.
- Pilot before scaling. Treat recommendations as a starting playbook; run small tests, measure results, and expand what works.
