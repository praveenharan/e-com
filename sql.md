```sql
-- Busiest day of Week
-- Staffing optimization for busy mornings or Mondays.
SELECT a A.day_of_week
FROM gold_lh_for_ambulatory_analyst.dbo.dim_date AS A
JOIN gold_lh_for_ambulatory_analyst.dbo.fact_encounters  AS B
ON A.FullDate = B.EncounterDate
GROUP BY A.day_of_week;

-- Revenue by Specialty
-- Identify which departments are the highest revenue drivers.
SELECT A.Specialty, B.GrossCharge
FROM gold_lh_for_ambulatory_analyst.dbo.dim_providers AS A
JOIN gold_lh_for_ambulatory_analyst.dbo.fact_encounters AS B
ON A.ProviderID = B.ProviderID
GROUP by A.Specialty, B.GrossCharge

-- Geographic Leakage
-- See if patients are traveling far, suggesting a need for a new clinic location.
SELECT 
    p.PatientID,
    p.PatientName,
    p.ZipCode AS Patient_Zip,
    l.ClinicName,
    l.ZipCode AS Clinic_Zip,
    CASE 
        WHEN p.ZipCode = l.ZipCode THEN 'Local'
        ELSE 'Leakage' 
    END AS Patient_Proximity
FROM gold_lh_for_ambulatory_analyst.dbo.dim_patients p
JOIN gold_lh_for_ambulatory_analyst.dbo.fact_encounters f 
    ON p.PatientID = f.PatientID
JOIN gold_lh_for_ambulatory_analyst.dbo.dim_locations l 
    ON f.LocationID = l.LocationKey
WHERE p.ZipCode <> l.ZipCode;

-- Wait Time Outliers
-- Identify specific clinics or days where patient experience failed.
WITH GlobalMetrics AS (
    SELECT AVG(WaitTimeMin) AS AvgWaitTime
    FROM gold_lh_for_ambulatory_analyst.dbo.fact_encounters
)
SELECT 
    f.EncounterDate,
    l.ClinicName,
    f.PatientID,
    f.WaitTimeMin,
    g.AvgWaitTime,
    (f.WaitTimeMin / g.AvgWaitTime) AS MultipleOfAvg
FROM gold_lh_for_ambulatory_analyst.dbo.fact_encounters f
CROSS JOIN GlobalMetrics g
JOIN gold_lh_for_ambulatory_analyst.dbo.dim_locations l 
    ON f.LocationID = l.LocationKey
WHERE f.WaitTimeMin > (g.AvgWaitTime * 2)
ORDER BY f.WaitTimeMin DESC;


```
