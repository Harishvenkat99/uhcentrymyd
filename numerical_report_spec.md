# Numerical Report Feature — Aggregation Spec (RESOLVED)

Goal: admin panel → pick a date → download an Excel file in the exact
"HUD Wise (Cumulative Number) - Daily Reporting Format" layout, block-wise,
computed automatically from that date's patient records.

Status: **Implemented** in `admin.html` (Numerical Report card + `buildNumericalReportRow()`).
Field names below are the real Firestore keys used by `index.html`'s form
(`FIELD_IDS`/`RADIO_GROUPS`), confirmed against the live form code, not placeholders.

## Value conventions confirmed in the actual data

- Binary fields (Inj.Td given, referred-to-higher-facility, portal-entered, etc.)
  are stored as the **string** `"Yes"` or `"No"` (radio pill values), never
  booleans or `"Y"/"N"`.
- Positive/negative lab-style fields store the **string** `"Positive"` or `"Negative"`.
- Blank/unanswered fields are always an **empty string** `""` (dropdowns default to
  `value=""`), never `undefined`/`null` — though the aggregation code checks for
  all three defensively.

## Output shape

One row per block (5 rows, fixed order): **Sirkali**, Kollidam, Kuttalam,
Mayiladuthurai, Sembanarkovil. Note: the block is spelled `"Sirkali"` in the
app's actual Block→PHC→HSC hierarchy, not "Sirkazhi" as in the original state
template wording — the report uses the real data spelling.

RULE types:
- **COUNT_NONBLANK(field)** — number of records where `field` is filled in
- **COUNT_EQ(field, value)** — number of records where `field` equals `value`
- **COUNT_STARTS_WITH(field, prefix)** — number of records where `field` starts with `prefix`
- **SUM(field)** — numeric sum of `field` across records
- **DISTINCT_COUNT(field)** — number of distinct values of `field` across records
- **ROSTER** — comes from the Block→PHC→HSC hierarchy already embedded in the app
- **DERIVED** — computed from other columns in this same report
- **BLANK** — left empty; comes from a separate register, not the patient record

| # | Column | Rule |
|---|--------|------|
| 1 | No.of HWC-HSCs | ROSTER — count of HSCs under this block in `HIERARCHY` |
| 2 | No.of HWC-HSCs Reported | DISTINCT_COUNT(`hsc`) among this block's records for the date |
| 3 | No.of HWC-HSCs Not Reported | DERIVED = (1) − (2) |
| 4 | No.of Patients | COUNT_NONBLANK(`pName`) |
| 5 | Type of Consultation → MLHP | COUNT_EQ(`consultType`, "MLHP") |
| 6 | Type of Consultation → eSanjeevani | COUNT_EQ(`consultType`, "eSanjeevani") |
| 7 | No.of Fever Cases | COUNT_NONBLANK(`feverComplaint`) |
| 8 | No.of Cough & Cold Cases | COUNT_NONBLANK(`coughColdComplaint`) |
| 9 | No.of Wheezing Cases | COUNT_NONBLANK(`wheezingComplaint`) |
| 10 | No.of Headache Cases | COUNT_NONBLANK(`headacheComplaint`) |
| 11 | No.of Skin lesions Cases | COUNT_NONBLANK(`skinLesionsComplaint`) |
| 12 | No.of Diarrhea/Dysentry Cases | COUNT_NONBLANK(`diarrheaComplaint`) |
| 13 | No.of Vomiting Cases | COUNT_NONBLANK(`vomitingComplaint`) |
| 14 | No.of Abdominal Pain cases | COUNT_NONBLANK(`abdominalComplaint`) |
| 15 | No.of Long standing Pain Cases | COUNT_NONBLANK(`painComplaint`) |
| 16 | No.of Chest Pain cases | COUNT_NONBLANK(`chestPainComplaint`) |
| 17 | No.of Cardiac Cases managed with loading dose and referred | COUNT_EQ(`cardiacManaged`, "Yes") |
| 18 | No.of Eye Cases | COUNT_NONBLANK(`eyeComplaint`) |
| 19 | No.of ENT Cases | COUNT_NONBLANK(`entComplaint`) |
| 20 | No.of Mental Illness Cases | COUNT_NONBLANK(`mentalIllness`) — see resolution below (not a Yes/No field) |
| 21 | No.of MNS cases Referred to Psychiatrist | COUNT_EQ(`mentalReferred`, "Yes") |
| 22 | No.of Geriatric Cases | COUNT_NONBLANK(`geriatricComplaint`) |
| 23 | No.of Palliative Cases | COUNT_NONBLANK(`palliativeComplaint`) |
| 24 | No.of Dog bite Cases | COUNT_NONBLANK(`dogCategory`) |
| 25 | No.of Dog bite Cases referred for Immunoglobulin | COUNT_EQ(`arvCategory3Referred`, "Yes") |
| 26 | No.of Injury/Emergency Cases | COUNT_NONBLANK(`injuryType`) |
| 27 | No.of Emergency cases given Inj.Td | COUNT_EQ(`injTd`, "Yes") |
| 28 | No.of Emergency cases referred to Higher Facility | COUNT_EQ(`emergencyReferred`, "Yes") |
| 29 | No of new NCD Screening | COUNT_STARTS_WITH(`ncdScreeningType`, "New") — matches "New HT"/"New DM"/"New Both (HT & DM)" |
| 30 | No of known NCD Screening cases | COUNT_STARTS_WITH(`ncdScreeningType`, "Known") — matches "Known HT"/"Known DM"/"Known Both (HT & DM)" |
| 31 | No of Follow up cases | COUNT_NONBLANK(`ncdFollowupDetail`) |
| 32 | No.of Oral Screening Cases | COUNT_NONBLANK(`oralScreening`) |
| 33 | No.of Positive Cases (Oral) | COUNT_EQ(`oralResult`, "Positive") |
| 34 | No.of SBE Screening Cases | COUNT_EQ(`breastScreening`, "SBE") — see resolution below (one field, two values, not two fields) |
| 35 | No.of CBE Screening Cases | COUNT_EQ(`breastScreening`, "CBE") |
| 36 | No.of Positive Cases (Breast) | COUNT_EQ(`breastResult`, "Positive") |
| 37 | No.of VIA Screening Cases | COUNT_EQ(`cervicalScreening`, "Yes") — Yes/No field, not free text |
| 38 | No.of Positive Cases (Cervical) | COUNT_EQ(`cervicalResult`, "Positive") |
| 39 | No.of Cases referred to Higher Facility (Cervical) | COUNT_EQ(`cervicalReferred`, "Yes") |
| 40 | No.of TB Symptom cases | COUNT_NONBLANK(`tbSymptoms`) |
| 41 | No.of TB screening cases Positive | COUNT_EQ(`tbScreening`, "Positive") |
| 42 | No.of Positive cases Treatment initiated (TB) | COUNT_NONBLANK(`tbTreatmentDate`) |
| 43 | No.of positive cases, Contact Screening done (TB) | COUNT_EQ(`tbContactDone`, "Yes") |
| 44 | No.of Contact Screening cases positive (TB) | COUNT_EQ(`tbContactScreening`, "Positive") |
| 45 | No.of Positive Contact Screening Treatment initiated (TB) | COUNT_NONBLANK(`tbContactTreatmentDate`) |
| 46 | No.of Cases Followup (TB) | COUNT_NONBLANK(`tbFollowup`) |
| 47 | No.of Symptom cases (Leprosy) | COUNT_NONBLANK(`leprosySymptoms`) |
| 48 | No.of Leprosy screening Positive | COUNT_EQ(`leprosyScreening`, "Positive") |
| 49 | No.of Positive cases Treatment initiated (Leprosy) | COUNT_NONBLANK(`leprosyTreatmentDate`) |
| 50 | No.of positive cases, Contact Screening done (Leprosy) | COUNT_EQ(`leprosyContactDone`, "Yes") |
| 51 | No.of Contact Screening cases positive (Leprosy) | COUNT_EQ(`leprosyContactScreening`, "Positive") |
| 52 | No.of Positive Contact Screening Treatment initiated (Leprosy) | COUNT_NONBLANK(`leprosyContactTreatmentDate`) |
| 53 | No.of Cases Followup (Leprosy) | COUNT_NONBLANK(`leprosyFollowup`) |
| 54 | No.of Urine Pregnancy Test done | COUNT_NONBLANK(`upt`) |
| 55 | No of UPT positive | COUNT_EQ(`upt`, "Positive") |
| 56 | No.of Hemoglobin test done | COUNT_NONBLANK(`hbValue`) |
| 57 | No.of Anemia Positive Cases | COUNT_NONBLANK(`anemia`) — see resolution below (renamed from "test done") |
| 58 | No.of Blood Sugar test done | COUNT_NONBLANK(`bloodSugar`) |
| 59 | No.of Urine Albumin test done | COUNT_NONBLANK(`urineAlbumin`) |
| 60 | No.of Urine Sugar test done | COUNT_NONBLANK(`urineSugar`) |
| 61 | No.of Visual Inspection Acetic Acid test done | COUNT_NONBLANK(`viaAceticAcid`) — separate Diagnostics-section field, not the same as (37) |
| 62 | No.of Dengue test done | COUNT_NONBLANK(`dengue`) |
| 63 | No.of HIV test done | COUNT_NONBLANK(`hiv`) |
| 64 | No.of HbsAg test done | COUNT_NONBLANK(`hbsag`) |
| 65 | No.of VDRL test done | COUNT_NONBLANK(`vdrl`) |
| 66 | No. of Iodine in salt test done | SUM(`iodineTests`) — see resolution below |
| 67 | No.of Water testing for Coliform test done | SUM(`waterTests`) |
| 68 | No.of Sputum for AFB test done | COUNT_NONBLANK(`sputumAfb`) — free-text field |
| 69 | Peripheral smear for Malaria | COUNT_NONBLANK(`peripheralSmear`) — free-text field |
| 70 | No.of Smear for Filaria test done | COUNT_NONBLANK(`smearFilaria`) — free-text field |
| 71 | Others if any (Diagnostics) | BLANK — free text, not aggregated |
| 72 | WHV - Suspected HT | SUM(`whvHT`) |
| 73 | WHV - Suspected DM | SUM(`whvDM`) |
| 74 | WHV - Suspected Both (HT & DM) | SUM(`whvBoth`) |
| 75 | WHV - Invited Oral Screening | SUM(`whvOralScreening`) |
| 76 | WHV - Invited SBE/CBE Screening | SUM(`whvSbeCbeScreening`) |
| 77 | WHV - Invited VIA Screening | SUM(`whvViaScreening`) |
| 78–83 | No.of Kit A & Kit B drugs provided | BLANK — separate register |
| 84–89 | No.of Essential Drugs provided | BLANK — separate register |
| 90 | AAM Portal | COUNT_EQ(`aamPortal`, "Yes") |
| 91 | PICME | COUNT_EQ(`picme`, "Yes") |
| 92 | MTM | COUNT_EQ(`mtm`, "Yes") |
| 93 | HMIS | COUNT_EQ(`hmis`, "Yes") |
| 94 | IHIP | COUNT_EQ(`ihip`, "Yes") |

## Resolutions to the original "Open items to confirm"

- **Field names**: all replaced above with real Firestore keys, cross-checked
  against `index.html`'s `FIELD_IDS`/`RADIO_GROUPS` — every field referenced
  exists on the actual patient record schema.
- **Yes/No and Positive/Negative string values**: confirmed as the literal
  strings `"Yes"`/`"No"` and `"Positive"`/`"Negative"` (see "Value conventions"
  above) — not booleans, not "Y"/"N".
- **Anemia (57)**: the `anemia` field is a severity dropdown (Mild/Moderate/
  Severe) that health workers only fill in when anemia is present — there is
  no separate "test performed" flag distinct from `hbValue` (already column 56).
  Resolved: column 57 counts anemia-**positive** cases (`COUNT_NONBLANK(anemia)`),
  confirmed with the district admin.
- **Mental Illness (20)**: same issue as anemia — `mentalIllness` is a
  categorical select (illness type), not Yes/No, so it's `COUNT_NONBLANK`
  rather than `COUNT_EQ(..., "Yes")`.
- **Breast screening (34/35)**: `breastScreening` is a **single** field with
  values `"SBE"`/`"CBE"`, not two separate boolean fields as the original
  placeholder names (`sbe_screening`/`cbe_screening`) implied. Resolved as two
  `COUNT_EQ` rules against the one field.
- **Two VIA fields (37 vs 61)**: confirmed these are genuinely two separate
  fields — `cervicalScreening` (Yes/No, under Cervical Cancer screening) and
  `viaAceticAcid` (Positive/Negative, under general Diagnostics). No
  double-counting risk; both counted independently as instructed.
- **Iodine/water testing (66/67)**: these fields exist per-patient-record as
  number inputs, but represent a facility-level daily tally a health worker
  enters on (typically) one record for the day. Resolved: `SUM()` the field
  across the block's records for that date — confirmed with the district admin,
  who also asked that the same "sum as a per-HSC/block total" treatment apply
  to the WHV fields (72–77), which are likewise numeric per-record tallies.
- **Portal columns (90–94)**: these are **five separate** Yes/No radio fields
  (`aamPortal`, `picme`, `mtm`, `hmis`, `ihip`), not one multi-select field as
  the placeholder guessed.

## Implementation notes

- Implemented in `admin.html`: new "Numerical Report" card with a date picker,
  `buildNumericalReportRow()` aggregation function, and `NUMERICAL_REPORT_HEADERS`.
- Reuses the existing `xlsx-js-style` library already loaded for the 64-column
  district Excel export (`XLSX.utils.aoa_to_sheet`, bold header row via cell
  styling, `XLSX.writeFile`) — no new dependency added.
- Roster-based columns (1, 3) read from the same `HIERARCHY` Block→PHC→HSC
  object already embedded in `admin.html`.
- One row per block, fixed order: Sirkali, Kollidam, Kuttalam, Mayiladuthurai,
  Sembanarkovil.
- Query: all `entries` where `pDate == <selected date>`, grouped client-side
  by `block`.
