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
- Finding 1
- 2
- 3
- 4
- 5

### Recommendations
Section 1 :(Recommendation: Negotiate with underperforming insurers)


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
