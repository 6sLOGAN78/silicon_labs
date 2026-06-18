# Clinical Design Review Notes for ICUsentinel

## Recommendations from Senior Consultant, Critical Care Medicine

---

## Section 1: General ICU Monitoring Philosophy

### Clinical Statement 1

The team was advised that most ICU emergencies are not sudden events. Physiological deterioration usually begins with subtle changes that may appear 30 minutes to several hours before a patient becomes unstable.

### Clinical Statement 2

The objective of ICUsentinel should not be detection of abnormal values alone. The objective should be identification of progressive deterioration patterns before conventional alarm systems activate.

### Clinical Statement 3

The consultant emphasized that trend velocity is often more clinically important than absolute values.

Example:

Patient A:

* HR = 125 bpm
* Stable for 4 hours

Patient B:

* HR = 82 → 95 → 110 → 125 bpm in 30 minutes

Patient B should receive higher priority despite both patients having the same current heart rate.

---

# Section 2: Tachycardia

### Clinical Statement 4

Persistent tachycardia should never be interpreted as an isolated abnormality.

### Clinical Statement 5

The team was informed that tachycardia frequently represents a compensatory response to physiological stress.

Common causes include:

* Infection
* Sepsis
* Blood loss
* Pain
* Respiratory distress
* Hypovolemia
* Cardiac dysfunction

### Clinical Statement 6

The consultant stated that early septic patients may demonstrate:

* HR = 95–110 bpm
* BP within normal limits
* RR mildly elevated

This phase is frequently missed by conventional monitoring systems.

### Clinical Statement 7

A persistent heart rate above 110 bpm should trigger increased analytical attention when accompanied by:

* RR > 22
* Temperature > 38°C
* Declining HRV
* Progressive BP reduction

### Clinical Statement 8

The engineering team was advised not to build a model that predicts tachycardia alone.

The preferred approach is:

Tachycardia
+
Blood Pressure Trend
+
Respiratory Rate Trend
+
Temperature Trend
=================

Clinical Risk Estimation

---

# Section 3: Septic Shock

### Clinical Statement 9

The consultant explained that septic shock develops as a consequence of systemic infection causing widespread inflammatory activation and circulatory dysfunction.

### Clinical Statement 10

Early physiological manifestations may include:

* Temperature > 38°C
* HR > 100 bpm
* RR > 22 breaths/min

while blood pressure remains apparently normal.

### Clinical Statement 11

The consultant emphasized that waiting until MAP falls below 65 mmHg represents late recognition.

### Clinical Statement 12

The following progression was discussed:

Stage 1:

* HR increasing
* RR increasing
* Temperature increasing

Stage 2:

* HRV decreasing
* Perfusion worsening

Stage 3:

* MAP declining
* Oxygen demand increasing

Stage 4:

* Septic shock

### Clinical Statement 13

Patients demonstrating simultaneous deterioration across cardiovascular and respiratory systems should receive elevated priority scores.

...
