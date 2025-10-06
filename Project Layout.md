# Company-Focused Section

## Section 1: Late Payments and Revenue Cycle (Recommendation: Negotiate with underperforming insurers)
### 1) Which insurance providers have the highest share of late payments (paid and unpaid)?
```
WITH late AS (SELECT insurance_provider,
COUNT(*) FILTER (WHERE payment_status = 'Late-Paid') AS late_paid,
COUNT(*) FILTER (WHERE payment_status = 'Late-Unpaid') AS late_unpaid
FROM billing b JOIN patient p ON b.patient_id=p.id
GROUP BY insurance_provider)

SELECT insurance_provider, SUM(late_paid + late_unpaid) AS total_late
FROM late
WHERE insurance_provider IS NOT NULL
GROUP BY insurance_provider
ORDER BY total_late DESC
LIMIT 50
```
### 2A) Total financial loss from late/unpaid bills by insurer?
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
JOIN patient p ON p.id = b.patient_id
WHERE p.insurance_provider IS NOT NULL
GROUP BY p.insurance_provider
ORDER BY total_late_exposure DESC;
```
### 2B) Put into context of top 3 hospitals. For the top 3 hospitals, what is the most prevelant insurance provider patients have and what is the sum of late payments. 

```
CODE HERE
```

### 3) Do uninsured/self-pay patients have significantly higher default rates?
CHATGPT CODE:
```
WITH base AS (
  SELECT 
    CASE WHEN p.insurance_provider IS NULL THEN 'Uninsured' ELSE 'Insured' END AS ins_status,
    b.payment_status
  FROM patient p
  JOIN billing b ON p.id = b.patient_id
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

MY CODE:

```
WITH Top_Hospital AS (SELECT (SELECT hospital
	FROM visit
	GROUP BY hospital
	ORDER BY count(*) DESC
	LIMIT 1) AS scope,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 0) / COUNT(*)::NUMERIC),2),'999D99%') AS sameday,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 1) / COUNT(*)::NUMERIC),2),'999D99%') AS one_night,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los > 1) / COUNT(*)::NUMERIC),2),'999D99%') AS multi_night
FROM visit),

National AS (SELECT 'Nationally' AS scope,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 0) / COUNT(*)::NUMERIC),2),'999D99%') AS sameday,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 1) / COUNT(*)::NUMERIC),2),'999D99%') AS one_night,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los > 1) / COUNT(*)::NUMERIC),2),'999D99%') AS multi_night
FROM visit)

SELECT scope, sameday, one_night, multi_night
FROM Top_Hospital
UNION
SELECT scope, sameday, one_night, multi_night
FROM National
```
### 4) What is the rate of late payments by condition? - Improves financial forecasting and collection strategy

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

```
CODE HERE
```

### 3) What is the average cost per day for each condition (total_charge / los)

```
CODE HERE
```

### 4) Are there doctors/hospitals with higher-than-average LOS for the same condition? (inefficient)

```
CODE HERE
```

## Section 3: Follow-up Visits (Recommendation: Introduce stronger collection strategies for uninsured patients)
### 1) Which conditions generate the most follow-ups?

```
SELECT 
condition, count(follow_up_required) as Followups
FROM visit
WHERE follow_up_required = 'Y' 
GROUP BY condition
```

### 2) What is the incremental cost of follow-ups compared to initial visits?

```
CODE HERE
```

### 3) Are certain doctors/hospitals driving unnecessary repeat visits? (inefficient)

```
CODE HERE
```

## Section 4: Demographics 
### 1) which patient demographics drive the highest revenue for the top 5 hospitals?

```
-- 1) which patient demographics drive the highest revenue for the top 5 hospitals?
-- Demographics: age, gender, insurance status (CASE WHEN insurance_provider IS NULL THEN 'Uninsured' ELSE 'insured'

WITH top5 AS (
  SELECT hospital, COUNT(DISTINCT patient_id) AS cnt_patients
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
	gender, condition, SUM(total_charge) as sum_revenue, COUNT(*) AS patient_count
FROM(	SELECT v.hospital, p.gender, v.condition, b.total_charge, v.age
		FROM patient p
		JOIN visit v ON p.id=v.patient_id JOIN billing b ON v.visit_id=b.visit_id
		WHERE hospital IN (SELECT hospital FROM top5))
GROUP BY hospital, age_bracket, gender, condition
ORDER BY sum_revenue DESC
LIMIT 5


  
```

### 2) What are the most prevalent conditions for age ranges 0-18 (children), 19-64 (adults), 65+ (elderly) and what is the sum total of their visits in the last comlpete year of the dataset?

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
ORDER BY age_bracket
```

## Section 5: Operational Efficiency
### 1A) Which hospitals have the highest patient volume. - workload balancing and staffing optimization

```
SELECT
hospital
FROM visit
GROUP BY hospital
ORDER BY COUNT(*) DESC
```

### 1B) For the hospital(s) with the highest patient volume, what percentage of patients are discharged the same day vs. multi-day stays?  and compare that to the rest of the dataset. Informs bed availability forcasting 

```
WITH Top_Hospital AS (SELECT (SELECT hospital
	FROM visit
	GROUP BY hospital
	ORDER BY count(*) DESC
	LIMIT 1) AS scope,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 0) / COUNT(*)::NUMERIC),2),'999D99%') AS sameday,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 1) / COUNT(*)::NUMERIC),2),'999D99%') AS one_night,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los > 1) / COUNT(*)::NUMERIC),2),'999D99%') AS multi_night
FROM visit),

National AS (SELECT 'Nationally' AS scope,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 0) / COUNT(*)::NUMERIC),2),'999D99%') AS sameday,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 1) / COUNT(*)::NUMERIC),2),'999D99%') AS one_night,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los > 1) / COUNT(*)::NUMERIC),2),'999D99%') AS multi_night
FROM visit)

SELECT scope, sameday, one_night, multi_night
FROM Top_Hospital
UNION
SELECT scope, sameday, one_night, multi_night
FROM National
```
____________________________________________________________________________________________________________________________
____________________________________________________________________________________________________________________________
# Patient-Focused Section
## Section 1: Condition-Specific Cost Comparison
### 1) What is the avg out-of-pocket expense by condition?

```
CODE HERE
```
   
### 2) Which hospitals offer the lowest average total charge for common conditions?

```
CODE HERE
```

### 3) Do severe conditions have dramatically difference costs across hospitals/insurers?

```
CODE HERE
```

## Section 2: Insurance Provider Analysis
### 1) Which insurers cover the highest share of cost for high expense conditions (bucket high expense conditions with CASE)

```
CODE HERE
```

### 2) Which insurers have the largest out of pocket costs?

```
CODE HERE
```

AVERAGE PATIENT RESPONSIBILITY BY PROVIDER:
```
SELECT p.insurance_provider,
ROUND(AVG(patient_responsibility_amount),0) AS AVG_patient_responsibility
FROM billing b JOIN patient p ON b.patient_id=p.id
WHERE p.insurance_provider IS NOT NULL
GROUP BY p.insurance_provider
ORDER BY AVG_patient_responsibility DESC
```
### 3) How do patient responsibility percentages compare across insurers?

```
CODE HERE
```

## Section 3: Hospital Performance
### 1) Which hospitals are consistently cheap for same-day, 1-night, and multi-night stays?

```
CODE HERE
```

### 2) Which hospitals are most frequently used to treat given conditions?

```
CODE HERE
```

### 3) Are some hospitals better for specific specialties? ex. Hospital A better for childbirth, B for surgery (less followups)

```
CODE HERE
```

### 4) Which hospitals have the lowest rates of follow-up visits? (indicating efficiency of treatment)

```
CODE HERE
```

## Section 4: Patient-Focused Benchmarks
### 1) For a patient with insurance, what is the bets provider/hospital combo for minimizing costs by condition?

```
CODE HERE
```

### 2) For uninsured patients, which hospitals are cheapest overall?

```
CODE HERE
```

### 3) Is there any corelation between patient age and total_charge?

```
SELECT
p.age, SUM(b.total_charge)
FROM patient AS p JOIN billing AS b ON patient.id = billing.patient_id
GROUP BY age, total_charge
ORDER BY b.total_charge DESC
```

### Customer Recommendations (Examples)
Patients with diabetes should prefer Hospital X + Insurance Provider Y for lowest recurring costs
.
Cancer patients are better off with Insurance A due to higher coverage, even if charges are higher.

For uninsured patients, Hospital Z consistently delivers lowest-cost same-day care.
