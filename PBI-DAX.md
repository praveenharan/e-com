# Key Performance Indicators (KPIs)

### These are the high-level "tiles" for the top of dashboard.
```
Actual Encounters =
  DISTINCTCOUNT(gold_fact_encounters[EncounterID])
```

```
Average Wait Time =
  AVERAGE(gold_fact_encounters[WaitTimeMin])
```

```
Encounter Performance % = 
DIVIDE(
    [Actual Encounters], 
    [Target Encounters], 
    0
)
```

```
Encounter Variance = [Actual Encounters] - [Target Encounters]

```

```
NewPatientsCount = 
CALCULATE(
    DISTINCTCOUNT(gold_fact_encounters[PatientID]),
    KEEPFILTERS(gold_fact_encounters[DateKey] = MIN(gold_fact_encounters[DateKey]))
)
```

```
No-Show Rate = 
DIVIDE(
    [Total No-Shows], [Actual Encounters], 
    0
)
```

```
PatientGrowthMoM = 
VAR current_month = [NewPatientsCount]
VAR previous_month = 
    CALCULATE(
        [NewPatientsCount], 
        DATEADD('gold_dim_date'[Date], -1, MONTH)
    )
RETURN
    current_month - previous_month

```
```
Total No-Shows = 
CALCULATE(
    COUNT(gold_fact_encounters[EncounterID]), 
    gold_fact_encounters[is_noshow] = 1
)
```


Total Encounters: Total volume of patient visits (Actuals vs. Target).

No-Show Rate: (Total No Shows / Total Encounters) * 100. A critical metric for clinic efficiency.

Average Wait Time: AVG(WaitTimeMin). Used to identify patient experience bottlenecks.

Gross Charges: Total revenue generated before adjustments.

Provider Productivity: Encounters / Unique Providers. Measures how busy your clinicians are.

Patient Growth: New PatientID counts month-over-month.
```
