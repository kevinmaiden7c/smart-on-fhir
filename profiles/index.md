---
layout: main
title: SMART on FHIR Profiles
---

# SMART on FHIR resource profiles

To support apps that run unmodified across different health IT systems, we
construct a set of "ground rules" that define which data fields are required
vs.  optional, and which coding systems are used in a given context. The FHIR
specification leaves many of these decisions open to downstream implementers,
to ensure that FHIR can work with a variety of use cases. But for a viable app
platform, we need more.

Below, we'll define the data elements and coding systems that are considered
essential for a SMART on FHIR implementation. Eventually we'd like to document
these decisions using FHIR's built-in "profiling" mechanism, but for now we
host the following human-readable explanations, along with sample data to
demonstrate what we mean.

## Why do we need profiles?

Imagine an app that queries a patient's problem list and wants to create a
timeline view of problems in 2014. According to the FHIR specification, an app
can query for `/Condition?subject=123&onset=2014` to find problems that began
in 2014, or it can query for `/Condition?subject=123&date-asserted=2014` to
find problems that were first detected in 2014. 

In practice, any given FHIR server might have data with both `onsetDate` and
`dateAsserted` or might populate just one of them, or neither (or any mixture
of these). This suggests that if you're building an app for the "general case,"
you can't rely on either field, which makes it hard to build rich, expressive
queries. In the worst case, you might query by `dateAsserted` against a server
that only provides `onset`, and then upon finding no results, might mistakenly
conclude that the patient has no problems. 

In short: if you get no guarantees about which fields will be populated, you're
limited to just querying for all Conditions and doing all of your filtering
work client-side.

Profiles improve the situation by giving you a reliable set of agreed-upon
guarantees that go above and beyond FHIR's base specification.

# Patient Demographcs

Each
[Patient](http://www.hl7.org/implement/standards/fhir/patient.html#Patient)
must have:

 * `1 or more` medical record numbers in `Patient.identifier`
 * `1 or more` names in `Patient.name`
 * `1` administrative gender in `Patient.gender`

##### Example: [https://fhir-open-api.smartplatforms.org/Patient/1032702](https://fhir-open-api.smartplatforms.org/Patient/1032702?_format=json)

# Allergy or intolerance

Each [AllergyIntolerance](http://www.hl7.org/implement/standards/fhir/allergyintolerance.html) must have:

 * `1` sensitivity type (allergy, intolerance, or unknown) in `AllergyIntolerance.sensitivityType`
 * `1` patient in `AllergyIntolerance.subject`
 * `1` substance in `AllergyIntolerance.substance`

##### Example: [https://fhir-open-api.smartplatforms.org/AllergyIntolerance/59](https://fhir-open-api.smartplatforms.org/AllergyIntolerance/59?_format=json)

The coding system used to record the substance depends on the substance type.
There are three	cases: drug allergies, drug class allergies, and other.

### Allergy to a specific drug
#### Coded with RxNorm Ingredient codes

Allergies to a specific drug are coded using RxNorm at the ingredient level
(using codes with `TTY=IN`). It is almost certainly an error to claim that a
patient is allergic to a specific dose form such as a 100mg tablet of
Sulfamethoxazole
([rxnorm:402625](http://schemes.caregraf.info/rxnorm#!402625)). Instead, the
allergy is captured using the ingredient code for Sulfamethoxazole
([rxnorm:10180](http://schemes.caregraf.info/rxnorm#!10180)).

`Substance.type.coding.system`: `http://rxnav.nlm.nih.gov/REST/rxcui`
##### Example: cipro [https://fhir-open-api.smartplatforms.org/Substance/rxnorm-203563](https://fhir-open-api.smartplatforms.org/Substance/rxnorm-203563?_format=json)

### Allergy to a drug class
#### Coded with NDF-RT

Allergies to a drug class are captured using NDF-RT. For example, an allergy to
Sulfonamide drugs is captured using the drug class code
[NDFRT:N0000175503](http://purl.bioontology.org/ontology/NDFRT/N0000175503)

`Substance.type.coding.system`: `http://rxnav.nlm.nih.gov/REST/Ndfrt`

##### Example: sulfonamides [https://fhir-open-api.smartplatforms.org/Substance/ndfrt-N0000175503](https://fhir-open-api.smartplatforms.org/Substance/ndfrt-N0000175503?_format=json)

### Food and environmental allergies
#### Coded with FDA UNII

Allergies to other substances (foods ane environmental allergies) are captured
using FDA UNII codes.

`Substance.type.coding.system`: `http://fda.gov/UNII/`

##### Example: shrimp [https://fhir-open-api.smartplatforms.org/Substance/unii-1891LE191T](https://fhir-open-api.smartplatforms.org/Substance/unii-1891LE191T?_format=json)

### "No known allergies"

Encoded as 
[List](http://www.hl7.org/implement/standards/fhir/list.html#List)
must have:

 * `1` patient in `List.subject`
 * `1` code value of `52473-6` and system of `http://loinc.org` in `List.code`
 * `1` date in `List.date`
 * `1` value of `snapshot` in `List.mode`
 * `1` code value of `nilknown` and system of `http://hl7.org/fhir/list-empty-reason` in `List.emptyReason`

##### Example: [https://fhir-open-api.smartplatforms.org/List/24](https://fhir-open-api.smartplatforms.org/List/24?_format=json)

### "No known history of drug allergy"

Encoded as 
[List](http://www.hl7.org/implement/standards/fhir/list.html#List)
must have:

 * `1` patient in `List.subject`
 * `1` code value of `11382-9` and system of `http://loinc.org` in `List.code`
 * `1` date in `List.date`
 * `1` value of `snapshot` in `List.mode`
 * `1` code value of `nilknown` and system of `http://hl7.org/fhir/list-empty-reason` in `List.emptyReason`

##### Example: [https://fhir-open-api.smartplatforms.org/List/62](https://fhir-open-api.smartplatforms.org/List/62?_format=json)

# Immunizations

Each
[Immunization](http://www.hl7.org/implement/standards/fhir/immunization.html#Immunization)
must have:

 * `1` patient in `Immunization.subject`
 * `1` refusal indicator in `Immunization.refusalIndicator`
 * `1` self-reported indicator in `Immunization.reported`
 * `1` vaccine code in `Immunization.vaccineType`

#### Vaccines coded with CVX

Each immunizations is coded with a `vaccineType` drawn from CDC's [CVX
vocabulary](http://www2a.cdc.gov/vaccines/iis/iisstandards/vaccines.asp?rpt=cvx).
This is a coding with:
System: `http://www2a.cdc.gov/vaccines/iis/iisstandards/vaccines.asp?rpt=cvx`

##### Example: [https://fhir-open-api.smartplatforms.org/Immunization/18](https://fhir-open-api.smartplatforms.org/Immunization/18?_format=json)

# Conditions

Each
[Condition](http://www.hl7.org/implement/standards/fhir/condition.html#Condition)
must have:

 * `1` patient in `Condition.subject`
 * `1` condition in `Condition.code` with System of `http://snomed.info/sct`
 * `1` code (confirmed) in `Condition.status`
 * `1` date in `Condition.onsetDate`
 
##### Example: fever [https://fhir-open-api.smartplatforms.org/Condition/454](https://fhir-open-api.smartplatforms.org/Condition/454?_format=json)
 
# Procedures

Each
[Procedure](http://www.hl7.org/implement/standards/fhir/procedure.html#Procedure)
must have:

 * `1` patient in `Procedure.subject`
 * `1` procedure in `Procedure.type` with System of `http://snomed.info/sct`
 * `1` date in `Procedure.date.start`
 * `1` date in `Procedure.date.end`

##### Example: mastectomy [https://fhir-open-api.smartplatforms.org/Procedure/5](https://fhir-open-api.smartplatforms.org/Procedure/5?_format=json)

# Family History

Each
[FamilyHistory](http://www.hl7.org/implement/standards/fhir/familyhistory.html#FamilyHistory)
must have:

 * `1` patient in `FamilyHitory.subject`
 * `up to 1` height measurement as an `valueQuantity` extension of url `http://fhir-registry.smartplatforms.org/Profile/family-history#height` with units of `centimeters` and system of `http://unitsofmeasure.org`
 * `1` relationship to patient in `FamilyHitory.reation.relationship` with system `http://hl7.org/fhir/v3/RoleCode`

##### Example: [https://fhir-open-api.smartplatforms.org/FamilyHistory/8](https://fhir-open-api.smartplatforms.org/FamilyHistory/8?_format=json)

# Smoking Status

Each Smoking Status
[Observation](http://www.hl7.org/implement/standards/fhir/observation.html#Observation)
must have:

 * `1` patient in `Observation.subject`
 * `1` code value of `72166-2` and system of `http://loinc.org` in `Observation.name`
 * `1` code with system `http://snomed.info/sct` in `Observation.valueCodeableConcept`
 * `1` status of `final` in `Observation.status`
 * `1` value of `ok` in `Observation.reliability`

##### Example: former smoker [https://fhir-open-api.smartplatforms.org/Observation/6-smokingstatus](https://fhir-open-api.smartplatforms.org/Observation/6-smokingstatus?_format=json)

# Medication Prescription

Each 
[MedicationPrescription](http://www.hl7.org/implement/standards/fhir/medicationprescription.html#MedicationPrescription)
must have:

 * `1` patient in `MedicationPrescription.subject`
 * `1` [Medication] (http://www.hl7.org/implement/standards/fhir/medication.html#Medication) object in `MedicationPrescription.medication` with system `http://www.nlm.nih.gov/research/umls/rxnorm` in `Medication.coding.system`
 * `1` status of `active` in `MedicationPrescription.status`
 * `1` patient in `MedicationPrescription.patient`
 * `1` object in `MedicationPrescription.dosageInstruction.timingSchedule` with `1` date in `event.start` and `0 or 1` date in `event.end` and `0 or 1` objects in `repeat` (with `1` value in `repeat.frequency`, `1` value in `repeat.units`, and `1` value in `repeat.duration`)
 * `0 or 1` code in `MedicationPrescription.dosageInstruction.doseQuantity` with system of `http://unitsofmeasure.org`
 * `0 or 1` objects in `MedicationPrescription.dispense` with `1` value in `numberOfRepeatsAllowed`, `1` code with system of `http://unitsofmeasure.org` in `quantity`, and `0 or 1` codes with system of `http://unitsofmeasure.org` in `expectedSupplyDuration` 

##### Example: [https://fhir-open-api.smartplatforms.org/MedicationPrescription/102](https://fhir-open-api.smartplatforms.org/MedicationPrescription/102?_format=json)

# Medication Dispense

Each 
[MedicationDispense](http://www.hl7.org/implement/standards/fhir/medicationdispense.html#MedicationDispense)
must have:

 * `1` patient in `MedicationDispense.subject`
 * `1` patient in `MedicationDispense.patient`
 * `1` reference to `MedicationPrescription` in `MedicationDispense.authorizingPrescription`
 * `1` object in `MedicationDispense.dispense` with `1` extension of `http://fhir-registry.smartplatforms.org/Profile/dispense#days-supply` of type `valueQuantity` with system of `http://unitsofmeasure.org` with units of `days` and code of `d`
 * `1` [Medication] (http://www.hl7.org/implement/standards/fhir/medication.html#Medication) object in `MedicationDispense.dispense.medication` with system `http://www.nlm.nih.gov/research/umls/rxnorm` in `Medication.coding.system`
 * `1` status of `completed` in `MedicationDispense.dispense.status`
 * `1` quantity with system `http://unitsofmeasure.org` and code of `{tablets}` and units of `tablets` in `MedicationDispense.dispense.quantity`
 * `1` date in `MedicationDispense.dispense.whenHandedOver`

##### Example: [https://fhir-open-api.smartplatforms.org/MedicationDispense/1229](https://fhir-open-api.smartplatforms.org/MedicationDispense/1229?_format=json)

# TODO: work in progress.

```
# Vital Signs
### Blood Pressure
# Lab result
```
