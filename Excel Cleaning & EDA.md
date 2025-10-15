# Excel Data Cleaning

## Patients
### Date formatting  

### Remove salutation from first name and concatenate into full_name

### Remove duplicates

### Trim all leading and trailing spaces

### Filter for nulls in primary key and essential columns


# Exploratory Data Analysis
Below are the queries used to explore and familiarize with the dataset

Aggregated a LOS column
```
ALTER TABLE visits
ADD COLUMN los INT GENERATED ALWAYS AS (
    GREATEST((date_of_discharge::date - date_of_admission::date), 0)
) STORED;
```

1) Frequency of LOS 

```
SELECT los, COUNT(*)
FROM visit
GROUP BY los
ORDER BY los ASC
```

3) Frequency of followups
```
SELECT 
condition, count(follow_up_required) as Followups
FROM visit
WHERE follow_up_required = 'Y' 
GROUP BY condition
ORDER BY followups DESC 
```

4) Frequency by condition and unique conditions
```
SELECT condition, COUNT(visit_id) AS visits_for
FROM visit
GROUP BY condition
ORDER BY visits_for DESC
```

5) Total late payments where insurance_provider IS NOT NULL (92836)
```
SELECT SUM(late_paid + late_unpaid)
FROM (SELECT COUNT(*) FILTER(WHERE payment_status = 'Late-Paid') AS late_paid,
COUNT(*) FILTER(WHERE payment_status = 'Late-Unpaid') as late_unpaid
FROM billing b JOIN patient p ON b.patient_id = p.id
WHERE insurance_provider IS NOT NULL)
```

6) most common insurance providers
```
SELECT insurance_provider, COUNT(*) AS count
FROM patient
WHERE insurance_provider IS NOT NULL
GROUP BY insurance_provider
ORDER BY count DESC
```

X) Which hospital generates the most revenue?
```
SELECT hospital, SUM(total_charge) AS total_revenue
FROM visit v JOIN billing b ON v.visit_id = b.visit_id
GROUP BY hospital
ORDER BY total_revenue DESC
LIMIT 5

