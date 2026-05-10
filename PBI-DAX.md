# Key Performance Indicators (KPIs)

### These are the high-level "tiles" for the top of dashboard.
```
Actual Encounters = 
    IF(
        ISBLANK(
            DISTINCTCOUNT(gold_fact_encounters[EncounterID])
            ),
            0, 
            DISTINCTCOUNT(gold_fact_encounters[EncounterID])
    )
```
### Average Wait Time: AVG(WaitTimeMin). Used to identify patient experience bottlenecks.

```
Average Wait Time = IF(
    ISBLANK(
        AVERAGE(gold_fact_encounters[WaitTimeMin])
        ),
        0,
        AVERAGE(gold_fact_encounters[WaitTimeMin])
)
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
### No-Show Rate: (Total No Shows / Total Encounters) * 100. A critical metric for clinic efficiency.
```

No-Show Rate = 
DIVIDE(
    [Total No-Shows],
    [Actual Encounters], 
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




Gross Charges: Total revenue generated before adjustments.

Provider Productivity: Encounters / Unique Providers. Measures how busy your clinicians are.

Patient Growth: New PatientID counts month-over-month.
```
