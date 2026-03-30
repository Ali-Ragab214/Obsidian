
---

Name : Ali Ragab Ghonim Track : OS
Hospital System
User Stories :

1- As a Receptionist ,I want to register a new patient in the system So that the hospital can store and access patient info easily . 
-Acceptance criteria : age , address ,name generate id 
-priority : High / 90
-weight : 3
-Details : Receptionist enters patient data then system validates inputs and saves the patient record and generates a unique patient ID.

### Tasks:

|Task|Description|
|---|---|
|Task 1.1|Design patient registration form with fields for name, age, and address.|
|Task 1.2|Implement input validation for all patient fields.|
|Task 1.3|Develop logic to generate a unique Patient ID.|
|Task 1.4|Save patient data to the database.|
|Task 1.5|Display success message upon successful registration.|

---

2-As a Patient , I want to book an appointment with a docs so that I can receive medical Consultation without waiting
-Acceptance criteria : choose specialization ,show available doctors ,choose date and time
-priority : High / 100
-weight : 5
-Details : Patient selects specialization, system displays available doctors, patient chooses date and time, system checks availability and confirms booking.

### Tasks:

|Task|Description|
|---|---|
|Task 2.1|Display list of medical specializations for the patient.|
|Task 2.2|Filter and display doctors based on selected specialization.|
|Task 2.3|Show available dates for chosen doctor.|
|Task 2.4|Show available time slots for selected date.|
|Task 2.5|Implement availability check to prevent double booking.|
|Task 2.6|Confirm appointment and update patient and doctor schedules.|
|Task 2.7|Display confirmation message to the patient.|

---

3-As a Doctor , I want to see my daily appointments So that I can organize my work 
-Acceptance criteria : show patient lists , show visit time
-priority : Medium / 40
-weight : 1 
-Details : Doctor logs into the system and views a list of daily appointments including patient names and visit times.

### Tasks:

|Task|Description|
|---|---|
|Task 3.1|Implement doctor login functionality.|
|Task 3.2|Fetch today's appointments for the logged-in doctor from the database.|
|Task 3.3|Display patient names and visit times in a list format.|
|Task 3.4|Sort appointments by time.|

---

4-As a Cashier , I want to generate and record patient payment so the hospital can track financial transactions 
-Acceptance criteria : Calculate total fees ,choose payment method
-priority : High / 90
-weight : 3

5-As an Admin I want to see system Statistics in a dashboard so that I can monitor hospital performance Acceptance criteria : Number of all patients ,number of all Appointments ,Number of available doctors priority : Low / 30 weight : 2

---

I sliced the third Ustory =>

Original User Story:

2-As a Patient , I want to book an appointment with a docs so that I can receive medical Consultation without waiting
-Acceptance criteria : choose specialization ,show available doctors ,choose date and time
-priority : High / 100
-weight : 5 Details :Patient selects specialization, system displays available doctors, patient chooses date and time, system checks availability and confirms booking.

## Slice 1 — Select Specialization & View Doctors

User Story Slice: As a Patient, I want to choose a medical specialization so that I can see the relevant doctors available.

Acceptance Criteria:

Patient can view a list of available specializations Selecting a specialization shows a list of doctors in that field Doctors list shows basic info (name, rating) Priority: High / 100 Weight: 2 Details: Patient selects specialization and the system filters and displays available doctors for that specialization.

## Slice 2 — Choose Date & Time

User Story Slice: As a Patient, I want to choose a date and time for my appointment so that I can book a suitable slot.

Acceptance Criteria:

Patient can select a date from available dates Patient can select a time slot from available times System prevents double booking Priority: High / 100 Weight: 2 Details: Patient chooses the date and time, and the system checks availability before confirming the selection.

## Slice 3 — Confirm Appointment

User Story Slice: As a Patient, I want to confirm my appointment so that the system registers it and prevents conflicts.

Acceptance Criteria:

System confirms appointment successfully Appointment appears in patient and doctor schedule Patient receives confirmation message Priority: High / 100 Weight: 1 Details: System registers the appointment, updates schedules, and confirms booking to the patient.