# Excel Data Cleaning

## Patients
### Date formatting  
<img width="637" height="486" alt="dateformat" src="https://github.com/user-attachments/assets/e3f1ed89-95de-49b3-ade1-28a6e4167dfa" />

### Remove salutation from first name and concatenate into full_name
<img width="614" height="414" alt="name cleaning" src="https://github.com/user-attachments/assets/1815060c-43f4-4a7e-ae5b-981febbd3108" />
<img width="412" height="432" alt="concat fullname" src="https://github.com/user-attachments/assets/8c2aa902-0102-4801-89b9-f92043231822" />

### Remove duplicates
<img width="229" height="369" alt="remove duplicates" src="https://github.com/user-attachments/assets/9333315f-506f-4f26-9542-0617b9620e0e" />

### Other important cleaning checks
- Trim all leading and trailing spaces
- Filtering for nulls in primary key and essential columns
- 

# Exploratory Data Analysis
Below are the queries used to explore and familiarize with the dataset

1) Frequency of LOS 
```
SELECT los, COUNT(*)
FROM visit
GROUP BY los
ORDER BY los ASC
```

2) Frequency of followups
```
SELECT 
condition, count(follow_up_required) as Followups
FROM visit
WHERE follow_up_required = 'Y' 
GROUP BY condition
ORDER BY followups DESC 
```

3) Frequency by condition and unique conditions
```
SELECT condition, COUNT(visit_id) AS visits_for
FROM visit
GROUP BY condition
ORDER BY visits_for DESC
```

4) Total late payments where insurance_provider IS NOT NULL (92836)
```
SELECT SUM(late_paid + late_unpaid)
FROM (SELECT COUNT(*) FILTER(WHERE payment_status = 'Late-Paid') AS late_paid,
COUNT(*) FILTER(WHERE payment_status = 'Late-Unpaid') as late_unpaid
FROM billing b JOIN patient p ON b.patient_id = p.id
WHERE insurance_provider IS NOT NULL)
```

5) most common insurance providers
```
SELECT insurance_provider, COUNT(*) AS count
FROM patient
WHERE insurance_provider IS NOT NULL
GROUP BY insurance_provider
ORDER BY count DESC
```

6) Which hospital generates the most revenue?
```
SELECT hospital, SUM(total_charge) AS total_revenue
FROM visit v JOIN billing b ON v.visit_id = b.visit_id
GROUP BY hospital
ORDER BY total_revenue DESC
LIMIT 5
```

7) Aggregating a LOS column
```
ALTER TABLE visits
ADD COLUMN los INT GENERATED ALWAYS AS (
    GREATEST((date_of_discharge::date - date_of_admission::date), 0)
) STORED;
```
- Serves as a founadtion for queries in the analysis section, prevents the need for calculating each time a code is run. 
