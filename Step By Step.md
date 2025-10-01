Everything Ive done:
Aggregated a LOS column
```
ALTER TABLE visits
ADD COLUMN los INT GENERATED ALWAYS AS (
    GREATEST((date_of_discharge::date - date_of_admission::date), 0)
) STORED;
```
easy join
FROM patient p JOIN billing b ON p.id = b.patient_id JOIN visit v ON b.visit_id = v.visit_id

Exploratory Analysis:
1) Frequency of LOS 

```
SELECT los, COUNT(*)
FROM visit
GROUP BY los
ORDER BY los ASC
```

2) Frequency of conditions (and distinct conditions)
```
SELECT condition
FROM visit
GROUP BY condition
ORDER BY COUNT(*) DESC
```
3) Frequency of followups
SELECT 
condition, count(follow_up_required) as Followups
FROM visit
WHERE follow_up_required = 'Y' 
GROUP BY condition
ORDER BY followups DESC 

4) Frequency by condition
SELECT condition, COUNT(visit_id) AS visits_for
FROM visit
GROUP BY condition
ORDER BY visits_for DESC

5) Total late payments where insurance_provider IS NOT NULL (92836)
SELECT SUM(late_paid + late_unpaid)
FROM (SELECT COUNT(*) FILTER(WHERE payment_status = 'Late-Paid') AS late_paid,
COUNT(*) FILTER(WHERE payment_status = 'Late-Unpaid') as late_unpaid
FROM billing b JOIN patient p ON b.patient_id = p.id
WHERE insurance_provider IS NOT NULL)

6) most common insurance providers

SELECT insurance_provider, COUNT(*) AS count
FROM patient
WHERE insurance_provider IS NOT NULL
GROUP BY insurance_provider
ORDER BY count DESC



-------
For the hospital with the highest patient volume, find the percentage of patients discharged on the same day, after one night, and after multi nights. How do these percentages compare to the rest of the dataset? (CTE's)

WITH Lewis_Inc_Hospital AS (SELECT 'Lewis Inc Hospital' AS scope,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 0) / COUNT(*)::NUMERIC),2),'999D99%') AS sameday,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 1) / COUNT(*)::NUMERIC),2),'999D99%') AS one_night,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los > 1) / COUNT(*)::NUMERIC),2),'999D99%') AS multi_night
FROM visit
WHERE hospital = 'Lewis Inc Hospital'),

National AS (SELECT 'Nationally' AS scope,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 0) / COUNT(*)::NUMERIC),2),'999D99%') AS sameday,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los = 1) / COUNT(*)::NUMERIC),2),'999D99%') AS one_night,
TO_CHAR(ROUND(100*(COUNT(*) FILTER(WHERE los > 1) / COUNT(*)::NUMERIC),2),'999D99%') AS multi_night
FROM visit)

SELECT scope, sameday, one_night, multi_night
FROM Lewis_Inc_Hospital
UNION
SELECT scope, sameday, one_night, multi_night
FROM National
------------




-- What is the top insurance provider in the top 5 hospitals IN PROGRESS
SELECT * 
FROM
(SELECT *, 
ROW_NUMBER () OVER (PARTITION BY hospital ORDER BY insurance_patient_count DESC) as provider_rank
FROM (SELECT hospital, insurance_provider, COUNT(DISTINCT p.id) as insurance_patient_count
FROM visit v JOIN patient p ON v.patient_id=p.id 
WHERE insurance_provider IS NOT NULL 
and hospital IN (SELECT hospital
				FROM visit
				GROUP BY hospital
				ORDER BY COUNT(*) DESC
				LIMIT 5)
GROUP BY hospital, insurance_provider
ORDER BY insurance_patient_count DESC)
)
WHERE provider_rank = 1



CONTUINED 
-- What is the top insurance provider in the top 5 hospitals
SELECT hospital, top_hospitals_ranked, cnt_patients
FROM(SELECT hospital, cnt_patients,
ROW_NUMBER() OVER (ORDER BY cnt_patients DESC) AS top_hospitals_ranked
FROM(SELECT hospital, COUNT(*) as cnt_patients
				FROM visit
				GROUP BY hospital
				ORDER BY cnt_patients DESC
				LIMIT 5)
)



EXTRA STUFF
Questions to answer:



Patient Care and Outcomes:

1) Avg LOS per condition and severity?



SELECT

DISTINCT condition, severity, PERCENTILE_CONT(.05) WITHIN GROUP (ORDER BY los) AS median_los

FROM visit

GROUP BY condition, severity

ORDER BY median_los DESC







2) Which conditions most often require follow up visits? - Helps hospitals anticipate repeat care and allocate resources. 



SELECT 

condition, count(follow_up_required) as Followups

FROM visit

WHERE follow_up_required = 'Y' 

GROUP BY condition





3) Can you deduce which hospitals are most frequently used to treat given conditions (group by condition)?



SELECT

hospital, condition, COUNT(condition) AS treated

FROM visit

GROUP BY condition





4) Is there any correlation between patient age and total_charge?



SELECT

p.age, b.total_charge

FROM patient AS p JOIN billing AS b ON patient.id = billing.patient_id

GROUP BY age, total_charge

ORDER BY b.total_charge DESC









Operational Efficiency:

1) Which hospitals have the highest patient volume. - workload balancing and staffing optimization
```
SELECT
hospital, COUNT(hospital)
FROM visit
GROUP BY hospital
ORDER BY COUNT(*) DESC
```


2) For the hospital(s) with the highest patient volume, what percentage of patients are discharged the same day vs. multi-day stays?  and compare that to the rest of the dataset. Informs bed availability forcasting 


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



Billing questions

1) Average patient responsibility by insurance provider 
```
SELECT p.insurance_provider,
ROUND(AVG(patient_responsibility_amount),0) AS AVG_patient_responsibility
FROM billing b JOIN patient p ON b.patient_id=p.id
WHERE p.insurance_provider IS NOT NULL
GROUP BY p.insurance_provider
ORDER BY AVG_patient_responsibility DESC
```

3) What is the rate of late payments/payment types by condition? AKA Which condition has the most late payments. (Improves financial forecasting and collection strategy)

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





3) Are chronic conditions leading to higher billing totals



X) Which hospital generates the most revenue?

SELECT hospital, SUM(total_charge) AS total_revenue

FROM visit v JOIN billing b ON v.visit_id = b.visit_id

GROUP BY hospital

ORDER BY total_revenue DESC

LIMIT 5



X) What is the average total cost per day of a stay in a hospital as LOS increases?

REALLY LIKE THIS QUESTION

Have to join visits and billing



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











Triple Joins:

1) what conditions lead to the highest out of pocket cost by insurance providers?



SELECT

condition, insurance_provider, patient_responsibility_amount

FROM patient p

JOIN visit v ON p.id=v.patient_id JOIN billing b ON v.visit_id=b.visit_id

GROUP BY condition, insurance_provider, patient_responsibility_amount





2) which patient demographics drive the highest revenue for the top 5 hospitals?



SELECT v.hospital, p.gender, p.age, v.condition, v.severity, b.

FROM patient p

JOIN visit v ON p.id=v.patient_id JOIN billing b ON v.visit_id=b.visit_id.





3) What are the most prevalent conditions for age ranges 0-18 (children), 19-64 (adults), 65+ (elderly) and what is the median total of their visits in 2024?




Forestdale hospital has made a clerical error and needs to get in contact with every patient who was discharged on (DATE) (must join patient and visit to get phone number and hospital)



Find redundant (costly) readmissions?

What qualifies as redundant?













WITH top_hospitals AS (

  SELECT hospital, COUNT(DISTINCT patient_id) AS cnt_patients

  FROM visit

  GROUP BY hospital

  ORDER BY cnt_patients DESC

  LIMIT 5

),

hospital_billing AS (

  SELECT v.hospital, SUM(b.total_charge) AS total_billed

  FROM visit v

  JOIN billing b ON b.visit_id = v.visit_id

  WHERE v.hospital IN (SELECT hospital FROM top_hospitals)
  GROUP BY v.hospital




