# Company-Focused Section

## Section 1: Late Payments and Revenue Cycle (Recommendation: Negotiate with underperforming insurers)
1) Which insurance providers have the highest share of late payments (paid and unpaid)?
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
2) A) Total financial loss from late/unpaid bills by insurer?
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
B) Put into context of top 3 hospitals. For the top 3 hospitals, what is the most prevelant insurance provider patients have and what is the sum of late payments. 
```
CODE HERE
```
3) Do uninsured/self-pay patients have significantly higher default rates?
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

##Section 2: Length of Stay Costs (Recommendation: Optimize dischage planning for conditions with high los)
1) What is the avg cost per day of length of stay?
2) What conditions drive the longest and most expensive hospital stays?
3) What is the average cost per day for each condition (total_charge / los)
4) Are there doctors/hospitals with higher-than-average LOS for the same condition? (inefficient)

##Section 3: Follow-up Visits (Recommendation: Introduce stronger collection strategies for uninsured patients)
1) Which conditions generate the most follow-ups?
2) What is the incremental cost of follow-ups compared to initial visits?
3) Are certain doctors/hospitals driving unnecessary repeat visits? (inefficient)

##Section 4: Payment Plans (Recommendation: Redesign payment plan terms to reduce defaults)
1) How does the incremental vs full plan affect late payments?
2) What is the risk-adjusted cost of offering incremental payment options?

____________________________________________________________________________________________________________________________

#Patient-Focused Section
##Section 1: Condition-Specific Cost Comparison
1) What is the avg out-of-pocket expense for each condition (by insurance provider)?
2) Which hospitals offer the lowest average total charge for common conditions?
3) Do severe conditions have dramatically difference costs across hospitals/insurers?

##Section 2: Insurance Provider Analysis
1) Which insurers cover the highest share of cost for high expense conditions (bucket high expense conditions with CASE)
2) Which insurers have patients with the largest out of pocket costs?
3) How do patient responsibility percentages compare across insurers?

##Section 3: Hospital Performance
1) Which hospitals are consistently cheap for same-day, 1-night, and multi-night stays?
2) Are some hospitals better for specific specialties? ex. Hospital A better for childbirth, B for surgery
3) Which hospitals have the lowest rates of follow-up visits? (indicating efficiency of treatment)

##Section 4: Patient-Focused Benchmarks
1) For a patient with insurance, what is the bets provider/hospital combo for minimizing costs by condition?
2) For uninsured patients, which hospitals are cheapest overall?
