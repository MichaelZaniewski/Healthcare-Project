# Healthcare-Project

### Code for Table Generation

CREATE TABLE patient(
  id GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(255) NOT NULL,
  last_name VARCHAR(255) NOT NULL,
  date_of_birth DATE NOT NULL,
  age INTEGER NOT NULL,
  gender VARCHAR(6) NOT NULL,
  phone_number VARCHAR(50) NOT NULL,
  email VARCHAR(255) NOT NULL,
  address VARCHAR(255) NOT NULL,
  city VARCHAR(255) NOT NULL,
  state VARCHAR(255) NOT NULL,
  zipcode VARCHAR(255) NOT NULL,
  insurance_provider VARCHAR(255),
  insurance_policy_number VARCHAR(30) NOT NULL,
 );
 
   ALTER TABLE patient
   ADD CONSTRAINT chk_insurance_policy_number
   CHECK (
   (insurance_provider IS NULL AND insurance_policy_number IS NULL)
   OR
   (insurance_provider IS NOT NULL AND policy_number IS NOT NULL)
    );
   
CONDITIONAL CONSTRAINT: IF INSURANCE PROVIDER NULL, POLICY NUMBER NULL ALSO
IF INSURANCE PROVIDER NOT NULL, THEN POLICY NUMBER NOT NULL ALSO


 CREATE TABLE medical(
  medical_record_id GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  patient_id FOREIGN KEY NOT NULL,
  flight_number VARCHAR(20) NOT NULL,
  tail_number VARCHAR(45),
  origin VARCHAR(4) NOT NULL,
  destination VARCHAR(4) NOT NULL,
  sched_departure TIME NOT NULL,
  actual_departure TIME NOT NULL,
  sched_flt_time INTEGER NOT NULL,
  actual_flt_time INTEGER NOT NULL,
  departure_delay INTEGER NOT NULL,
  wheels_up_time TIME NOT NULL,
  taxi_out_time INTEGER NOT NULL,
  carrier_delay INTEGER NOT NULL,
  weather_delay INTEGER NOT NULL,
  national_aviation_sys_delay INT NOT NULL,
  security_delay INTEGER NOT NULL,
  late_ac_arrival_delay INTEGER NOT NULL
 );


 CREATE TABLE billing(
  id SERIAL PRIMARY KEY,
  date DATE NOT NULL,
  flight_number VARCHAR(20) NOT NULL,
  tail_number VARCHAR(45),
  origin VARCHAR(4) NOT NULL,
  destination VARCHAR(4) NOT NULL,
  sched_departure TIME NOT NULL,
  actual_departure TIME NOT NULL,
  sched_flt_time INTEGER NOT NULL,
  actual_flt_time INTEGER NOT NULL,
  departure_delay INTEGER NOT NULL,
  wheels_up_time TIME NOT NULL,
  taxi_out_time INTEGER NOT NULL,
  carrier_delay INTEGER NOT NULL,
  weather_delay INTEGER NOT NULL,
  national_aviation_sys_delay INT NOT NULL,
  security_delay INTEGER NOT NULL,
  late_ac_arrival_delay INTEGER NOT NULL
 );
