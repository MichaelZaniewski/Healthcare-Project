
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
