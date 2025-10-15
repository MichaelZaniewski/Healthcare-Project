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







EXTRA STUFF
Questions to answer:



Patient Care and Outcomes:

1) Avg LOS per condition and severity?
```
SELECT
DISTINCT condition, severity, PERCENTILE_CONT(.05) WITHIN GROUP (ORDER BY los) AS median_los
FROM visit
GROUP BY condition, severity
ORDER BY median_los DESC
```






3) Which hospitals are most frequently used to treat given conditions (group by condition)?
```
SELECT
hospital, condition, COUNT(condition) AS treated
FROM visit
GROUP BY condition
```




4) Is there any correlation between patient age and total_charge?

```
SELECT
p.age, b.total_charge
FROM patient AS p JOIN billing AS b ON patient.id = billing.patient_id
GROUP BY age, total_charge
ORDER BY b.total_charge DESC
```









X) Which hospital generates the most revenue?
```
SELECT hospital, SUM(total_charge) AS total_revenue
FROM visit v JOIN billing b ON v.visit_id = b.visit_id
GROUP BY hospital
ORDER BY total_revenue DESC
LIMIT 5
```





#Triple Joins:

1) what conditions lead to the highest out of pocket cost by insurance providers?
```
SELECT
condition, insurance_provider, patient_responsibility_amount
FROM patient p
JOIN visit v ON p.id=v.patient_id JOIN billing b ON v.visit_id=b.visit_id
GROUP BY condition, insurance_provider, patient_responsibility_amount
```

2) which patient demographics drive the highest revenue for the top 5 hospitals?
```
SELECT v.hospital, p.gender, p.age, v.condition, v.severity, b.
FROM patient p
JOIN visit v ON p.id=v.patient_id JOIN billing b ON v.visit_id=b.visit_id.
```







