
# Drone Flight Log Prompt Engineering: Unstructured Text to CSV

## Project Overview
A prompt engineering experiment using Google Gemini to extract and normalize unstructured drone pilot debrief notes into clean, database-ready CSV telemetry. This project demonstrates how replacing basic prompts with strict schema constraints, negative rules, and one-shot examples eliminates formatting drift.

## Tools & Techniques Used
* **AI Model:** Google Gemini
* **Prompting Techniques:** System Role Assignment, Negative Constraints, Schema Enforcement, One-Shot Prompting
* **Output Format:** Raw CSV (Comma-Separated Values)

---

## Test Dataset (Unstructured Input)
Five synthetic pilot debrief notes containing inconsistent aircraft IDs, mixed time/battery units, and free-form status descriptions:

```text
1. "UAV-03 flew roof inspection mission in Class D airspace with LAANC clearance up to 200 ft. Flight lasted 18 mins, landed with 34 percent battery. No issues."
2. "Drone unit 7 lost GPS lock briefly near cell tower during bridge survey, switched to ATTI mode and returned to home safely. Duration 12 minutes, battery 22%. Flagged for compass recalibration."
3. "Quadcopter #01 coastal mapping run aborted due to 24 kt wind gusts. Only flew 4 mins, battery still at 81 percent. Rescheduled for tomorrow."
4. "Hexacopter 11 thermal powerline check completed—spotted hot spot on transformer #4. Total air time 27 mins, battery level not logged. Ready for next sortie."
5. "Mini-UAV 04 night training flight in open field, minor gimbal vibration noticed at 150 ft AGL. Flew 15 minutes, landed at 45 percent battery, grounded for gimbal check."
```

---

## Iteration 1: Baseline Prompt & Failure Analysis

### Prompt v1
```text
Turn these drone flight notes into CSV format with Aircraft ID, Mission, Flight Time, Battery, and Status:
[Unstructured Input]
```

### Observed Flaws (Why v1 Needed Improvement)
* **Raw String Copying:** Extracted unstandardized aircraft IDs (`UAV-03`, `Drone unit 7`, `Quadcopter #01`) instead of a single uniform ID format.
* **Mixed Data Types:** Left text units inside numeric fields (`18 mins` vs. `12 minutes`; `34 percent` vs. `22%`), breaking spreadsheet sorting and formulas.
* **Inconsistent Statuses:** Generated free-form phrases rather than standardized, filterable categories.

---

## Iteration 2: Engineered System Prompt (Final)

### Prompt v2
```text
You are a strict unmanned aircraft (UAS) flight log and telemetry extraction engine. Convert the unstructured pilot debrief notes below into raw CSV format.

Output Rules:
1. Output ONLY raw CSV text. Do NOT include greetings, markdown code fences, explanations, or closing remarks.
2. Use these exact column headers: Aircraft_ID,Mission_Type,Duration_Min,Battery_Pct,Flight_Status
3. Standardize Aircraft_ID to UAS-XX format (e.g., UAS-03, UAS-07, UAS-01, UAS-11, UAS-04).
4. Categorize Mission_Type strictly as one of: Inspection, Survey, Mapping, or Training.
5. Format Duration_Min as a plain integer representing minutes (e.g., 18).
6. Format Battery_Pct as a plain integer with NO percent sign (e.g., 34). If no battery reading is logged, write N/A.
7. Standardize Flight_Status strictly to one of four labels: Operational, Aborted, Maintenance, or Grounded.
8. Wrap any text field containing a comma in double quotes.

Example Input:
"Drone 09 perimeter patrol flight, 21 minutes air time, landed at 39 percent battery, all systems normal."
Example Output:
Aircraft_ID,Mission_Type,Duration_Min,Battery_Pct,Flight_Status
UAS-09,Inspection,21,39,Operational

Input Data to Process:
[Unstructured Input]
```

### Standardized Output (v2 Result)
```csv
Aircraft_ID,Mission_Type,Duration_Min,Battery_Pct,Flight_Status
UAS-03,Inspection,18,34,Operational
UAS-07,Survey,12,22,Maintenance
UAS-01,Mapping,4,81,Aborted
UAS-11,Inspection,27,N/A,Operational
UAS-04,Training,15,45,Grounded
```
