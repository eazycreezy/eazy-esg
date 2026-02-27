# ESG Data Schema — Standard Field Reference

Used by the Data Intake Agent to normalise uploaded data fields into a consistent schema.

## Table of Contents
1. [Environmental — Emissions](#emissions)
2. [Environmental — Energy](#energy)
3. [Environmental — Water](#water)
4. [Environmental — Waste](#waste)
5. [Social — Workforce](#workforce)
6. [Governance](#governance)
7. [Financial Context](#financial)

---

## 1. Environmental — Emissions {#emissions}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `scope_1_emissions_tco2e` | tCO₂e | Direct GHG emissions |
| `scope_2_lb_emissions_tco2e` | tCO₂e | Indirect — location-based method |
| `scope_2_mb_emissions_tco2e` | tCO₂e | Indirect — market-based method |
| `scope_3_total_tco2e` | tCO₂e | All categories combined |
| `scope_3_cat1_tco2e` | tCO₂e | Purchased goods & services |
| `scope_3_cat11_tco2e` | tCO₂e | Use of sold products |
| `scope_3_cat15_tco2e` | tCO₂e | Investments (financial sector) |
| `biogenic_co2_tco2e` | tCO₂e | Reported separately per GHG Protocol |
| `emissions_intensity_revenue` | tCO₂e / $M | Normalised by revenue |
| `emissions_intensity_employee` | tCO₂e / FTE | Normalised by headcount |
| `ghg_base_year` | YYYY | Base year for intensity calculations |
| `ghg_protocol_standard` | text | e.g. "GHG Protocol Corporate Standard" |

**Common column aliases to map from:**
- "CO2e", "GHG", "Carbon emissions" → scope fields
- "Scope 1 (kg)" → convert: divide by 1000 to get tCO₂e
- "Scope 1 (Mt)" → convert: multiply by 1,000,000 to get tCO₂e

---

## 2. Environmental — Energy {#energy}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `energy_consumption_mwh` | MWh | Total energy consumed |
| `energy_renewable_mwh` | MWh | Renewable energy consumed |
| `energy_renewable_pct` | % | Renewable as % of total |
| `energy_intensity_revenue` | MWh / $M | |
| `energy_intensity_employee` | MWh / FTE | |
| `electricity_consumption_mwh` | MWh | Electricity only (subset of total) |

**Common aliases:** "kWh" → divide by 1000 for MWh; "GJ" → multiply by 0.2778 for MWh

---

## 3. Environmental — Water {#water}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `water_withdrawal_ml` | ML | Total water withdrawn |
| `water_discharge_ml` | ML | Total water discharged |
| `water_consumption_ml` | ML | Withdrawal minus discharge |
| `water_recycled_ml` | ML | |
| `water_stress_area_pct` | % | Operations in water-stressed areas |

**Common aliases:** "kL" → divide by 1000 for ML; "m³" → divide by 1000 for ML

---

## 4. Environmental — Waste {#waste}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `waste_total_tonnes` | tonnes | |
| `waste_to_landfill_tonnes` | tonnes | |
| `waste_recycled_tonnes` | tonnes | |
| `waste_hazardous_tonnes` | tonnes | |
| `waste_diverted_pct` | % | Diverted from landfill |

---

## 5. Social — Workforce {#workforce}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `total_headcount` | integer | All employees at period end |
| `fte_count` | integer | Full-time equivalent |
| `gender_female_pct` | % | |
| `gender_male_pct` | % | |
| `gender_female_senior_pct` | % | Senior leadership female % |
| `ltifr` | per million hours | Lost time injury frequency rate |
| `trifr` | per million hours | Total recordable injury frequency rate |
| `fatalities` | integer | Work-related fatalities |
| `employee_turnover_pct` | % | Voluntary turnover rate |
| `training_hours_per_fte` | hours | Average training per employee |
| `indigenous_employee_pct` | % | Relevant for AU operations |

---

## 6. Governance {#governance}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `board_size` | integer | Total directors |
| `board_female_pct` | % | |
| `board_independent_pct` | % | |
| `esg_committee_exists` | boolean | Dedicated ESG/sustainability committee |
| `climate_risk_governance_statement` | text | Narrative — board oversight of climate |
| `anti_corruption_policy` | boolean | |
| `whistleblower_policy` | boolean | |
| `tax_transparency_report` | boolean | |

---

## 7. Financial Context {#financial}

| Standard Field Name | Unit | Notes |
|---|---|---|
| `total_revenue` | $M | Used for intensity calculations |
| `total_assets` | $M | Used for ASRS scoping |
| `reporting_currency` | ISO code | e.g. "AUD", "USD" |
| `reporting_period_start` | YYYY-MM-DD | |
| `reporting_period_end` | YYYY-MM-DD | |
| `reporting_boundary` | text | e.g. "Operational control, consolidated group" |
| `entity_name` | text | Legal entity name for disclosure |
