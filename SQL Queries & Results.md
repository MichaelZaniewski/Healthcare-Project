# SQL Queries & Results

## Section 1: Late Payments and Revenue Cycle (Recommendation: Negotiate with underperforming insurers)
### 1) Which insurance providers have the highest share of late payments (paid and unpaid)?

<img width="393" height="713" alt="Section 1 Q1" src="https://github.com/user-attachments/assets/2e873a8b-220f-4de7-8fbf-2ce8298f65bb" />

- Insights Gained: 
  
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
### 2) Total financial loss from late/unpaid bills by insurer?
<img width="1127" height="713" alt="Section 1 Q2" src="https://github.com/user-attachments/assets/69d53451-3893-44b1-9492-d406f1cbaead" />

- Insights Gained:
  
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
LIMIR 20;
```
### 3) Do uninsured/self-pay patients have significantly higher default rates?
<img width="592" height="121" alt="Section 1 Q3" src="https://github.com/user-attachments/assets/93d5ab4b-9724-4f08-8c6d-512f8647577c" />

- Insights Gained:
  
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

### 4) What is the rate of late payments by condition? - Improves financial forecasting and collection strategy
<img width="856" height="614" alt="Section 1 Q4" src="https://github.com/user-attachments/assets/4874b873-0f50-413e-ac59-bcc657cb061a" />

- Insights Gained:
  
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

## Section 2: Length of Stay Costs (Recommendation: Optimize dischage planning for conditions with high los)
### 1) What is the avg cost per day as LOS increases?
<img width="374" height="549" alt="Section 2 Q1" src="https://github.com/user-attachments/assets/1dcc4a72-2bb1-454e-baf9-d88962182065" />

- Insights Gained:
  
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

- Insights Gained:
  
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

- Insights Gained:
  
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

### 4) Are there hospitals with higher-than-average LOS for the same condition? (inefficient)
<img width="454" height="713" alt="Section 2 Q4" src="https://github.com/user-attachments/assets/157f4a21-3c74-4185-857a-293a484721e8" />

- Insights Gained:
  
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
-- AND hc.stays >= 10   -- optional volume guardrail
ORDER BY delta_los DESC, hc.stays DESC
)
GROUP BY hospital
ORDER BY count DESC
LIMIT 20;

```

## Section 3: Follow-up Visits (Recommendation: Introduce stronger collection strategies for uninsured patients)
### 1) Which conditions generate the most follow-ups? Helps hospitals anticipate repeat care and allocate resources
<img width="335" height="581" alt="Section 3 Q1" src="https://github.com/user-attachments/assets/367739da-b168-4639-989e-52cd77b5bd5b" />

- Insights Gained:
  
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

- Insights Gained:
  
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

### 3) Are certain doctors driving unnecessary repeat visits? (inefficient)
<img width="765" height="220" alt="Section 3 Q3" src="https://github.com/user-attachments/assets/d096e367-97e7-4fed-8017-db3fb0651de7" />

- Insights Gained:
  
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

### 2) What are the most prevalent conditions for age ranges 0-18 (children), 19-64 (adults), 65+ (elderly) and what is the sum total of their visits in the last comlpete year of the dataset?
<img width="457" height="154" alt="Section 4 Q2" src="https://github.com/user-attachments/assets/3546b7f4-41a4-4379-936e-bd9b0524e320" />

- Insights Gained:
  
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
### 1A) Which hospitals have the highest patient volume. - workload balancing and staffing optimization
<img width="454" height="713" alt="Section 5 Q1" src="https://github.com/user-attachments/assets/79f30792-41c5-4780-8df6-25f6e1c833c3" />

- Insights Gained:
  
```
SELECT
hospital, COUNT(*) as count
FROM visit
GROUP BY hospital
ORDER BY count DESC
LIMIT 20;
```

### 1B) For the hospital with the highest patient volume, what percentage of patients are discharged the same day vs. multi-day stays?  and compare that to the rest of the dataset. Informs bed availability forcasting 
<img width="676" height="121" alt="Section 5 Q2" src="https://github.com/user-attachments/assets/0586ead8-bc49-4f32-b0bd-e1b2fa2f5468" />

- Insights Gained:
  
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
