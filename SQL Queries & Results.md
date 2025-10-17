# SQL Queries & Results

## Section 1: Late Payments and Revenue Cycle
### 1) Which insurance providers have the highest share of late payments (paid and unpaid)?

<img width="393" height="713" alt="Section 1 Q1" src="https://github.com/user-attachments/assets/2e873a8b-220f-4de7-8fbf-2ce8298f65bb" />

- Functions:
     - Leveraged CTEs to pre-aggregate Late-Paid and Late-Unpaid transactions, applying FILTER() for conditional counts within the same query. Joined to the patient table for accurate insurer association and used GROUP BY and ORDER BY DESC to rank the top 20 insurance providers by total late payments.
       
- Insights Gained:
     - Brown LLC and Williams LLC stand out with 355 and 268 late payments respectively — far exceeding other insurers. The distribution shows a gradual 10-20 count increase between providers, suggesting a consistent increase until a sharp jump to the top two, indicating potential systemic delays or claim-processing inefficiencies within those insurers’ payment systems.
  
```
WITH late AS (SELECT insurance_provider,
COUNT(*) FILTER (WHERE payment_status = 'Late-Paid') AS late_paid,
COUNT(*) FILTER (WHERE payment_status = 'Late-Unpaid') AS late_unpaid
FROM billing b JOIN patient p ON b.patient_id=p.patient_id
GROUP BY insurance_provider)

SELECT insurance_provider, SUM(late_paid + late_unpaid) AS total_late
FROM late
WHERE insurance_provider IS NOT NULL
GROUP BY insurance_provider
ORDER BY total_late DESC
LIMIT 20
```
### 2) What is the total financial loss from late/unpaid bills by insurer members?
<img width="1127" height="713" alt="Section 1 Q2" src="https://github.com/user-attachments/assets/69d53451-3893-44b1-9492-d406f1cbaead" />

- Calculation Logic:
     - Calculated how much money hospitals risk losing due to late or unpaid bills from each insurance provider. The totals include both amounts that were eventually paid late and amounts still outstanding, giving a full picture of revenue delays and exposure.
       
- Insights Gained:
     - Brown LLC again leads with the highest total late exposure ($2.58M) and outstanding debt ($0.93M), followed by Williams LLC ($1.7M exposure, $0.73M debt) and Thompson and Sons ($1.25M exposure, $0.47M debt) — mirroring their high late-payment counts from the previous query.
     - The repeating presence of the same insurers in both metrics suggests persistent claim-processing or reimbursement delays, possibly structural issues in payer operations rather than random billing inefficiencies.
       
 - Why It Matters:
     - These findings identify where hospitals face the most financial risk and helps direct collection efforts or policy negotiations toward insurers with the highest late payment exposure.
       
```
SELECT
  p.insurance_provider,
  SUM(CASE WHEN b.payment_status IN ('Late-Paid','Late-Unpaid')
           THEN b.patient_responsibility_amount ELSE 0 END)::numeric(14,0) AS total_late_exposure,
  SUM(CASE WHEN b.payment_status = 'Late-Unpaid'
           THEN b.patient_responsibility_amount ELSE 0 END)::numeric(14,0) AS total_outstanding_debt,
  COUNT(*) FILTER (WHERE b.payment_status IN ('Late-Paid','Late-Unpaid')) AS cnt_late_bills,
  COUNT(*) FILTER (WHERE b.payment_status = 'Late-Paid')                  AS cnt_late_paid,
  COUNT(*) FILTER (WHERE b.payment_status = 'Late-Unpaid')                AS cnt_late_unpaid
FROM billing b
JOIN patient p ON p.patient_id = b.patient_id
WHERE p.insurance_provider IS NOT NULL
GROUP BY p.insurance_provider
ORDER BY total_late_exposure DESC
LIMIT 20;
```
### 3) Do uninsured/self-pay patients have significantly higher default rates?
<img width="592" height="121" alt="Section 1 Q3" src="https://github.com/user-attachments/assets/93d5ab4b-9724-4f08-8c6d-512f8647577c" />

- Functions:
     - Used a CTE (`base`) to centralize joins and keep the final query readable. Applied a JOIN between `patient` and `billing` to associate each bill with insurance status, then a CASE expression to bucket records into Insured vs Uninsured. Counted late-unpaid bills with COUNT(*) FILTER (WHERE …), calculated the default rate with a safe divide using NULLIF, and formatted the percentage via TO_CHAR. Finally, ordered by the default rate to compare groups at a glance.
       
- Insights Gained:
     - Uninsured patients default at 20.36% vs 11.89% for Insured, about 1.7× higher. Although uninsured patients account for 22% of all bills, they represent 33% of all late-unpaid bills, indicating a disproportionate share of defaults and a clear area for targeted interventions, for instance a detailed payment-plan enrollment. 
  
```
WITH base AS (
  SELECT 
    CASE WHEN p.insurance_provider IS NULL THEN 'Uninsured' ELSE 'Insured' END AS ins_status,
    b.payment_status
  FROM patient p
  JOIN billing b ON p.patient_id = b.patient_id
)
SELECT
  ins_status,
  COUNT(*) AS total_bills,
  COUNT(*) FILTER (WHERE payment_status = 'Late-Unpaid') AS late_unpaid_bills,
  TO_CHAR(
    ROUND(
      100.0 * COUNT(*) FILTER (WHERE payment_status = 'Late-Unpaid')
      / NULLIF(COUNT(*), 0)
    , 2),
    'FM999990D00%'
  ) AS default_rate_pct
FROM base
GROUP BY ins_status
ORDER BY default_rate_pct DESC;
```

### 4) What is the rate of late payments by condition?
<img width="856" height="614" alt="Section 1 Q4" src="https://github.com/user-attachments/assets/4874b873-0f50-413e-ac59-bcc657cb061a" />

- Insights Gained:
     - Conditions tied to intensive/ongoing care show the highest late payment rates: Cancer (49%, - 12,117 late), Kidney failure (47%, - 8,259 late), Stroke (46% - 8,094 late). Cancer is the condition that sees the most visits and the most late payments. On the other end of the spectrum, heart disease is notable at 44% late but significantly fewer visits, flagging a small, high-risk cohort.
       
- How to Use This Information: Forecasting and Collections
     - Forecast delayed cash flow by condition.
     - Tighten pre-authorization and financial counseling to confirm guaranteed payment for conditions like cancer or kidney failure where treatment is expensive and late payments are common.
     - Tailor reminders and installment options early.
   
```
WITH by_condition AS (
  SELECT
    v.condition,
    COUNT(*) FILTER (WHERE b.payment_status = 'Late-Paid')    AS count_late_paid,
    COUNT(*) FILTER (WHERE b.payment_status = 'Late-Unpaid')  AS count_late_unpaid,
    COUNT(*) FILTER (WHERE b.payment_status = 'Paid')         AS count_paid,
    COUNT(*) FILTER (WHERE b.payment_status = 'In-progress')  AS count_in_progress,
    COUNT(*)                                                  AS visits
  FROM billing b
  JOIN visit v ON v.visit_id = b.visit_id
  GROUP BY v.condition
)
SELECT
  condition,
  (count_late_paid + count_late_unpaid) AS total_late,
  count_late_paid,
  count_late_unpaid,
  visits,
  TO_CHAR(ROUND(100*(count_late_paid + count_late_unpaid)/visits,2),'999D99%') AS percent_late
FROM by_condition
ORDER BY total_late DESC;
```

## Section 2: Length of Stay Costs
### 1) What is the average cost per day as LOS increases?
<img width="374" height="549" alt="Section 2 Q1" src="https://github.com/user-attachments/assets/1dcc4a72-2bb1-454e-baf9-d88962182065" />

- Insights Gained:
     - Average cost per visit rises sharply from $5,678 (same-day) to over $63,000 for 8–10-night stays, reflecting the heavy resource demands of extended hospitalizations. However, charges decline beyond 10 nights, which may indicate smaller sample sizes.
       
- Use Case:
     - This analysis can strengthen financial forecasting and budgeting models, allowing hospitals to predict expected revenue and patient cost by LOS. Administrators can use it to identify where length-of-stay efficiency has the greatest financial impact and to inform staffing, bed management, and insurance negotiations tied to high-cost, multi-day hospitalizations.
  
```
SELECT 
    v.los,
    CASE v.los
        WHEN 0 THEN 'Same Day'
        WHEN 1 THEN 'One Night'
        WHEN 2 THEN 'Two Nights'
        ELSE v.los || ' Nights'
    END AS los_label,
    ROUND(AVG(b.total_charge), 0) AS avg_charge
FROM visit v
JOIN billing b ON v.visit_id = b.visit_id
GROUP BY v.los
ORDER BY v.los;
```

### 2) What conditions drive the longest and most expensive hospital stays?
<img width="479" height="713" alt="Section 2 Q2" src="https://github.com/user-attachments/assets/bbbb4b0f-222d-4021-827f-fa7798249f3b" />

- Methodology:
     - Joined `visit` and `billing` tables to link each medical condition with its total charges and Length of Stay (LOS). Used GROUP BY on both fields to aggregate charges by condition and duration, then ordered results by LOS (descending) and total charges to highlight the costliest and longest hospitalizations.
       
- Insights Gained:
     - COVID appears as the maximum LOS between 9-14 days, which is in accordance with the years this dataset targets (peak COVID). This shows substantial resource utilization for extended respiratory cases. The sum of total charges for these stays hovers consistently around 3.1M to 3.5M.  
     - Cancer, Kidney Failure, and Stroke show the largest total charges across 8–10 day stays, with totals often exceeding $50–75 million.
     - Heart disease maintains consistent costs but at a smaller scale, suggesting shorter or less variable stays.
     - These results confirm that critical and chronic illnesses are the primary cost drivers for prolonged admissions.
  
```
SELECT condition, LOS, SUM(total_charge) AS sum_total_charge
FROM visit v 
JOIN billing b ON v.visit_id=b.visit_id 
GROUP BY condition, LOS
ORDER BY LOS DESC, sum_total_charge DESC
LIMIT 20;
```

### 3) What is the average LOS for each condition and what is the average cost per visit and day?
<img width="659" height="614" alt="Section 2 Q3" src="https://github.com/user-attachments/assets/b32b9c58-efc0-4565-8a7e-a0245c3eaeea" />

- Functions:
     - Joined `visit` and `billing` tables to calculate average stay length, average, cost per visit, and average cost per day per condition. Used AVG() with ROUND() for clarity and precision, and derived per-day cost by dividing total charges by average LOS. Results were ordered by LOS and total charge to highlight the costliest and longest conditions first.
       
- Insights Gained:
     - Similar to the previous query, conditions requiring intensive inpatient care (Kidney failure, Stroke, Cancer, Heart disease, and COVID) average the longest LOS at 3 nights. The top 4 conditions have the highest average cost per visit and cost per day with a steep drop off with COVID.
Meanwhile, shorter-duration conditions like asthma, arthritis, and high blood pressure remain costly on a per-day basis ($6K–9K/day) but have fewer total days of care. Common low-acuity conditions such as flu, migraine, and allergies show minimal daily costs under $2K, reflecting low complexity and outpatient management. 

```
SELECT condition, 
	ROUND(AVG(LOS),0) AS avg_stay, 
	ROUND(AVG(total_charge),0) AS AVG_cost_per_visit, 
	ROUND(AVG(total_charge) / AVG(LOS),0) AS avg_cost_per_day
FROM visit v 
JOIN billing b ON v.visit_id=b.visit_id 
GROUP BY condition
ORDER BY avg_stay DESC, avg_cost_per_visit DESC, avg_cost_per_day DESC;
```

### 4) Are there hospitals with higher-than-average LOS for the same condition?
<img width="454" height="713" alt="Section 2 Q4" src="https://github.com/user-attachments/assets/157f4a21-3c74-4185-857a-293a484721e8" />

- Methodology:
     - To identify hospitals with unusually long patient stays, the average Length of Stay (LOS) for each condition was first calculated across all hospitals. Then, each hospital’s own average LOS for those same conditions was compared against the broader average. Hospitals with a consistently higher LOS than the condition norm were flagged and ranked by how often this occurred.
     - This approach highlights where patients tend to remain hospitalized longer than expected. However, longer LOS doesn’t always indicate inefficiency — it can also reflect valid factors such as case severity, hospital specialization, or availability of post-acute care options. For example, a facility focused on complex cancer treatment or rehabilitation may naturally show longer stays than general hospitals.
       
- Insights Gained:
     - Several hospitals, including Clark, Jackson and Garcia Hospital, Malone Group Hospital, and Davis LLC Hospital, appeared most frequently with higher-than-average LOS across conditions. These institutions may serve as regional or specialty centers where complex or chronic patients are treated, explaining the extended durations. At the same time, consistently elevated LOS across a wide range of conditions may signal an inefficient turnover rate. More information is needed to determine causality.
  
```
SELECT hospital, COUNT (*) as COUNT
FROM(SELECT
  hc.hospital,
  hc.condition,
  hc.stays,
  ca.avg_los_condition,
  hc.avg_los_hospital,
  (hc.avg_los_hospital - ca.avg_los_condition) AS delta_los,
  (hc.avg_los_hospital / NULLIF(ca.avg_los_condition, 0)) AS ratio_to_avg
FROM (
  SELECT
    v.hospital,
    v.condition,
    ROUND(AVG(v.los),2)::numeric AS avg_los_hospital,
    COUNT(*)            AS stays
  FROM visit v
  GROUP BY v.hospital, v.condition
) AS hc
JOIN (
  SELECT
    v.condition,
    ROUND(AVG(v.los),2)::numeric AS avg_los_condition
  FROM visit v
  GROUP BY v.condition
) AS ca
USING (condition)
WHERE hc.avg_los_hospital > ca.avg_los_condition
ORDER BY delta_los DESC, hc.stays DESC
)
GROUP BY hospital
ORDER BY count DESC
LIMIT 20;

```

## Section 3: Follow-up Visits 
### 1) Which conditions generate the most follow-ups?
<img width="335" height="581" alt="Section 3 Q1" src="https://github.com/user-attachments/assets/367739da-b168-4639-989e-52cd77b5bd5b" />

- Methodology:
     - Counted visits where follow_up_required = 'Y' and grouped by condition to see which diagnoses most often trigger additional appointments. This measures total visit follow-up volume, not unique patients.
       
- Insights Gained:
     - Follow-ups are concentrated in conditions tied to ongoing care or staged recovery: Cancer (24,360), Broken bones (17,548), Diabetes (17,431), Stroke (17,424), and Kidney failure (17,386) lead the list, with Asthma (16,670) close behind.
  
```
SELECT 
condition, count(follow_up_required) as Followups
FROM visit
WHERE follow_up_required = 'Y' 
GROUP BY condition
ORDER BY followups DESC;
```

### 2) What is the incremental cost of follow-ups compared to initial visits?
<img width="1228" height="612" alt="Section 3 Q2" src="https://github.com/user-attachments/assets/a54ea14c-2edb-4073-b202-274b97a18f15" />

- Walkthrough: Each patient’s care pattern was examined to see how much a follow-up visit costs compared to their first visit for the same condition.
     - The first recorded visit for each condition was treated as the initial visit, and any later ones were counted as follow-ups.
     - By averaging the total charges for both types of visits, we could measure whether follow-ups tend to cost more or less than the first encounter.
     - The difference between the two averages represents the incremental cost — showing how expenses change once initial diagnostics, procedures, and setup work are already done.
     - The analysis also compared how often follow-ups occur versus first visits, revealing which conditions generate the most ongoing care needs.
     - This approach provides a clear view of how costs evolve throughout a patient’s treatment journey and helps hospitals understand where most long-term care expenses are concentrated.
       
- Insights Gained:
     - In this dataset, follow-ups are cheaper than initial visits across all conditions.
     - The largest drops are for complex, costly conditions like heart disease, stroke, kidney failure, and cancer. While the smallest cost drops are from mild conditions such as migraines, allergies, and anxiety.
     - The assumption behind this data is that most of the costly procedures (such as CAT scans, X-rays, etc) were done in the initial visit and the follow-up visits are mostly monitoring progress.
  
```
WITH labeled AS (
  SELECT
    v.condition,
    v.patient_id,
    v.visit_id,
    v.date_of_admission,
    ROW_NUMBER() OVER (
      PARTITION BY v.patient_id, v.condition
      ORDER BY v.date_of_admission, v.visit_id
    ) AS rn,
    b.total_charge
  FROM visit v
  JOIN billing b USING (visit_id)
)
SELECT
  condition,
  ROUND(AVG(total_charge) FILTER (WHERE rn = 1)::numeric, 2) AS avg_initial_visit_cost,
  ROUND(AVG(total_charge) FILTER (WHERE rn > 1)::numeric, 2) AS avg_follow_up_cost,
  ROUND(
    (AVG(total_charge) FILTER (WHERE rn > 1)
     - AVG(total_charge) FILTER (WHERE rn = 1))::numeric, 2
  ) AS incremental_cost,
  COUNT(*) FILTER (WHERE rn = 1) AS initial_visits,
  COUNT(*) FILTER (WHERE rn > 1) AS follow_up_visits,
  ROUND(
    (AVG(total_charge) FILTER (WHERE rn > 1))
    / NULLIF(AVG(total_charge) FILTER (WHERE rn = 1), 0)::numeric, 3
  ) AS followup_to_initial_ratio
FROM labeled
GROUP BY condition
ORDER BY incremental_cost DESC NULLS LAST;
```

### 3) Are certain doctors driving unnecessary repeat visits?
<img width="765" height="220" alt="Section 3 Q3" src="https://github.com/user-attachments/assets/d096e367-97e7-4fed-8017-db3fb0651de7" />

- Methodology: This analysis looks for situations where patients return to the hospital shortly after discharge for the same condition — sometimes called a “bounce-back.”
     - For every patient, their visits were placed in order by date to track whether they came back to the hospital within 7 days of their last discharge.
     - Each doctor was then evaluated based on how many of their patients returned within that short window.
     - Doctors with a high number of quick return visits might appear to have higher repeat rates, possibly signaling inefficient care.
     - There are important caveats similar to Section 2 Question 4 - Hospitals with longer than average LOS. Some doctors may specialize in complex or chronic conditions like cancer or kidney failure, where multiple visits are medically necessary and not an inefficient process by the doctor. Higher repeat visit counts could simply reflect that these physicians see sicker patients or manage intensive follow-up care, not poor outcomes.
  
- Insights Gained:
     - A handful of doctors show higher-than-average 7-day return rates, meaning their patients often return sooner after discharge. These could indicate unnecessary or inefficient practices by the practitioner.
       
```
WITH base AS (
  SELECT v.patient_id, v.condition, v.doctor, v.hospital, v.visit_id,
         v.date_of_admission,
         COALESCE(v.date_of_discharge, v.date_of_admission + (v.los - 1))::date AS date_of_discharge
  FROM visit v

),
ordered AS (
  SELECT b.*,
         LEAD(b.date_of_admission) OVER (PARTITION BY b.patient_id, b.condition
                                         ORDER BY b.date_of_admission, b.visit_id) AS next_admit_date,
         LEAD(b.doctor)  OVER (PARTITION BY b.patient_id, b.condition
                                ORDER BY b.date_of_admission, b.visit_id) AS next_doctor,
         LEAD(b.hospital) OVER (PARTITION BY b.patient_id, b.condition
                                ORDER BY b.date_of_admission, b.visit_id) AS next_hospital
  FROM base b
),
bouncebacks AS (
  SELECT
    o.patient_id, o.condition,
    o.doctor  AS discharging_doctor,
    o.hospital AS discharging_hospital,
    o.date_of_discharge, o.next_admit_date,
    (o.next_admit_date - o.date_of_discharge) AS days_to_return,
    CASE WHEN o.next_admit_date IS NOT NULL
           AND o.next_admit_date >  o.date_of_discharge
           AND o.next_admit_date <= o.date_of_discharge + INTERVAL '7 days'
         THEN 1 ELSE 0 END AS is_bounceback_7d
  FROM ordered o
  WHERE o.next_admit_date IS NOT NULL
)
SELECT
  discharging_doctor AS doctor,
  COUNT(*)                                   AS index_visits_with_followup,
  SUM(is_bounceback_7d)                      AS bouncebacks_7d,
  ROUND( (SUM(is_bounceback_7d)::numeric / NULLIF(COUNT(*),0)), 3) AS bounceback_rate_7d
FROM bouncebacks
GROUP BY discharging_doctor
HAVING COUNT(*) >= 20          
ORDER BY bounceback_rate_7d DESC, bouncebacks_7d DESC, index_visits_with_followup DESC, doctor
LIMIT 5;

```

## Section 4: Demographics 
### 1) Which patient demographics drive the highest revenue for the top 3 hospitals?
<img width="1088" height="154" alt="Section 4 Q1" src="https://github.com/user-attachments/assets/a9caa156-ce06-49d8-8aea-3192a0bef36b" />

- Insights Gained:
     - The top 3 hospitals are, from top to bottom, Edwards-Lamb, Lopez, Warren, and Marsh, and Peterson LLC.
     - Revenue is concentrated in adult age bracket (18-64), mostly male, insured, with complex conditions of stroke and kidney failure.
     - The top hospital shows adult, female, stroke, with a frequency count of 106 as the most revenue generating condition. In second place, adult, male, kidney failure, with a count of 71. In third place, similar to first, adult, male, stroke, count of 66.
       
- Use Case:
     - Resource planning: Hospitals should ensure they have proper equipment and staff to treat their most profitable demographic
     - Simpler budgeting: Track simple metrics like revenue per patient and average days in the hospital for these groups to predict busy periods and set realistic budgets.
     - Insurer strategy: Since most revenue is from insured patients, focus conversations with the biggest insurers covering stroke and kidney care to reduce billing hassles and speed up payment.
       
```
WITH top5 AS (
  SELECT hospital, COUNT(distinct patient_id) AS cnt_patients
  FROM visit
  GROUP BY hospital
  ORDER BY cnt_patients DESC
  LIMIT 5)

SELECT  hospital,
	CASE
      WHEN age <= 17 THEN 'Child'
      WHEN age BETWEEN 18 AND 64 THEN 'Adult'
      ELSE 'Senior'
    END AS age_bracket, 
	gender, condition, insurance_status, SUM(total_charge) as sum_revenue, COUNT(*) AS patient_count
FROM (SELECT v.hospital, p.gender, v.condition, b.total_charge, v.age, insurance_provider,
		CASE WHEN insurance_provider IS NULL THEN 'Uninsured' 
			ELSE 'Insured'
			END as insurance_status
		FROM patient p
		JOIN visit v ON p.patient_id=v.patient_id JOIN billing b ON v.visit_id=b.visit_id
		WHERE hospital IN (SELECT hospital FROM top5))
GROUP BY hospital, age_bracket, gender, condition, insurance_status
ORDER BY sum_revenue DESC
LIMIT 3;
```

### 2) What are the most prevalent conditions for age ranges 0-18 (children), 19-64 (adults), 65+ (elderly) and what is the sum total of their visits in the last complete year of the dataset?
<img width="457" height="154" alt="Section 4 Q2" src="https://github.com/user-attachments/assets/3546b7f4-41a4-4379-936e-bd9b0524e320" />

- Functions:
     - Built a CTE(`counts`) that joins `visit` and `billing` tables, assigns each record to an age bracket (Child 0–17, Adult 18–64, Senior 65+), and aggregates visit counts and total charges per age bracket and condition for the last full year of the dataset.
     - Used a second CTE(`ranked`) with ROW_NUMBER() to rank conditions within each age group by visit count and then picked the #1 per group.
     - Returned the top condition for each age bracket along with its sum of charges to show financial weight, not just frequency.
       
- Insights Gained:
     - Cancer is the most prevalent and costly condition across all age brackets in 2024 with adult cancer cases totaling 130 million. Child and Senior cancer totals approximately 69.5 million each.
     - Adult oncology represents nearly double the financial volume of pediatric or senior oncology, suggesting the strongest demand for adult oncology capacity (beds, infusion chairs, specialists).
     - Since prevalence is measured by visit count (not unique patients), high totals likely reflect multiple encounters per patient—consistent with cancer treatment cycles. This supports prioritizing oncology scheduling, care coordination, and payer management across all ages, with the largest operational impact in the adult cohort.
  
```
WITH counts AS (
  SELECT
    CASE
      WHEN v.age <= 17 THEN 'Child'
      WHEN v.age BETWEEN 18 AND 64 THEN 'Adult'
      ELSE 'Senior'
    END AS age_bracket,
    v.condition,
    COUNT(*) AS cnt,
    SUM(b.total_charge) AS sum_total_charge
  FROM visit v
  JOIN billing b USING (visit_id)
  WHERE v.date_of_admission >= DATE '2024-01-01'
    AND v.date_of_admission <  DATE '2025-01-01'
  GROUP BY 1, 2
),
ranked AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY age_bracket ORDER BY cnt DESC, condition) AS rn
	  FROM counts)
SELECT age_bracket, condition, sum_total_charge
FROM ranked
WHERE rn = 1
ORDER BY age_bracket;
```

## Section 5: Operational Efficiency
### 1A) Which hospitals have the highest patient volume?
<img width="454" height="713" alt="Section 5 Q1" src="https://github.com/user-attachments/assets/79f30792-41c5-4780-8df6-25f6e1c833c3" />

- Use Case:
     - Informs workload balancing and staffing optimization
- Insights Gained:
     - The top 3 hospitals with the highest patient volume, are Ferrell Group (2,592), Peterson LLC (2,554), and Woods and Sons (2,545).
  
```
SELECT
hospital, COUNT(*) as count
FROM visit
GROUP BY hospital
ORDER BY count DESC
LIMIT 20;
```

### 1B) For the hospital with the highest patient volume, what percentage of patients are discharged the same day vs. multi-day stays? Compare to the rest of the dataset. 
<img width="676" height="121" alt="Section 5 Q2" src="https://github.com/user-attachments/assets/0586ead8-bc49-4f32-b0bd-e1b2fa2f5468" />

- Methodology:
     1) Find the hospital with the most visits.
     2) Split all visits into two groups: that hospital vs everyone else (nationally).
     3) For each group, classify stays by LOS: same-day (0 days), one night (1 day), multi-night (>1 day), and compute each category as a percent of total visits.
        
- Insights Gained:
     - The top hospital is a great representation of the national percentage for LOS across same-day, one-night, and multi-night metrics.
     - Top hospital: 6.13% same-day, 65.70% one-night, 28.16% multi-night from 2,592 visits.
     - Nationally: 7.29% same-day, 63.51% one-night, 29.20% multi-night from 262,664 visits.
     - The top hospital performs fewer same-day and fewer multi-night stays than average, with a heavier tilt toward one-night admissions.
  
```
WITH hosp_counts AS (
  SELECT hospital, COUNT(*) AS visit_cnt
  FROM visit
  GROUP BY hospital
),
top_hospitals AS (
  SELECT hospital
  FROM hosp_counts
  WHERE visit_cnt = (SELECT MAX(visit_cnt) FROM hosp_counts)
),
scoped AS (
  SELECT
    v.*,
    CASE WHEN v.hospital IN (SELECT hospital FROM top_hospitals)
         THEN 'Top hospital'
         ELSE 'Nationally'
    END AS scope
  FROM visit v
),
agg AS (
  SELECT
    scope,
    COUNT(*)::numeric AS n_visits,
    COUNT(*) FILTER (WHERE los = 0)::numeric AS n_sameday,
    COUNT(*) FILTER (WHERE los = 1)::numeric AS n_one_night,
    COUNT(*) FILTER (WHERE los > 1)::numeric AS n_multi_night
  FROM scoped
  GROUP BY scope
)
SELECT
  scope,
  TO_CHAR(100 * n_sameday / NULLIF(n_visits,0), 'FM999D00%')   AS sameday,
  TO_CHAR(100 * n_one_night / NULLIF(n_visits,0), 'FM999D00%') AS one_night,
  TO_CHAR(100 * n_multi_night / NULLIF(n_visits,0), 'FM999D00%') AS multi_night,
  n_visits::int AS visit_volume
FROM agg
ORDER BY CASE WHEN scope = 'Top hospital(s)' THEN 0 ELSE 1 END;
```

## Simulating Recommendations (Cost Saving Figures)

Cost saving recommendations are anticipated to decrease outstanding debt by between 5-10% with 8% being most likely.



How much revenue would have been collected by the hospital with the most outstanding debt had the debt-prevention strategies been in place? 
```
WITH params AS (
  SELECT
    DATE '2024-01-01' AS start_yr,
    DATE '2025-01-01' AS next_yr,
    DATE '2024-12-31' AS cutoff
),
cohort AS (
  SELECT
    b.billing_id,
    b.visit_id,
    b.patient_id,
    b.billing_date,
    b.expected_payment_date,
    b.actual_payment_date,
    b.payment_status,
    b.patient_responsibility_amount::numeric AS patient_resp,
    v.hospital
  FROM billing b
  JOIN visit v ON v.visit_id = b.visit_id
  CROSS JOIN params p
  WHERE b.billing_date >= p.start_yr
    AND b.billing_date <  p.next_yr
),
outstanding AS (
  SELECT
    hospital,
    billing_id,
    patient_resp,
    CASE
      WHEN payment_status IN ('Late-Unpaid','In-progress')
           AND (actual_payment_date IS NULL OR actual_payment_date > (SELECT cutoff FROM params))
      THEN patient_resp
      ELSE 0::numeric
    END AS outstanding_amt
  FROM cohort
),
by_hospital AS (
  SELECT
    hospital,
    COUNT(*) FILTER (WHERE outstanding_amt > 0) AS n_outstanding_bills,
    SUM(outstanding_amt)                         AS outstanding_total
  FROM outstanding
  GROUP BY hospital
),
top_hospital AS (
  SELECT *
  FROM by_hospital
  ORDER BY outstanding_total DESC NULLS LAST
  LIMIT 1
),
rates AS (
  SELECT UNNEST(ARRAY[0.05::numeric, 0.08::numeric, 0.10::numeric]) AS recovery_rate
)
SELECT
  th.hospital,
  th.n_outstanding_bills,
  th.outstanding_total                                 AS baseline_outstanding,
  r.recovery_rate,
  ROUND(th.outstanding_total * r.recovery_rate, 2)     AS estimated_recovery,
  ROUND(th.outstanding_total
        - th.outstanding_total * r.recovery_rate, 2)   AS remaining_after_recovery
FROM top_hospital th
CROSS JOIN rates r
ORDER BY r.recovery_rate;
```









