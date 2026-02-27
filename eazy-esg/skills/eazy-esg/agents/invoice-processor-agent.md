# Invoice Processor Agent

**Purpose:** Extract structured ESG-relevant data from uploaded utility invoices (electricity,
natural gas, LPG, water, diesel, etc.) — including batch mixed-type uploads — and output a
clean, formatted Excel workbook ready for ESG data intake or framework mapping.

---

## Trigger Conditions
- User uploads one or more PDF files that are utility invoices or bills
- User says things like "process my invoices", "extract data from my electricity bills",
  "I have a batch of utility bills", "pull the consumption data from these PDFs"
- User wants to convert invoice data into ESG-usable consumption figures

---

## Workflow

### Step 1: Acknowledge Upload & Detect Invoice Types

Use Python with pdfplumber to extract text from each uploaded file. Classify each invoice by
scanning for key identifiers:

| Invoice Type | Detection Signals |
|---|---|
| **Electricity** | "NMI", "kWh", "kilowatt", "electricity", "AEMO", "network tariff", "meter read" |
| **Natural Gas (mains)** | "MJ", "GJ", "natural gas", "MIRN", "gas supply", "therms" |
| **LPG / Bottled Gas** | "LPG", "cylinder", "bottle", "45kg", "swap", "propane", "litres delivered" |
| **Water** | "kL", "kilolitres", "water usage", "sewerage", "water meter" |
| **Diesel / Fuel** | "litres", "diesel", "petrol", "fuel", "fleet card", "cents/litre" |
| **Other** | Flag and ask user to classify manually |

Announce what was detected:
```
📄 INVOICE SCAN COMPLETE
──────────────────────────────────────────
Files detected: [n] invoice(s)

  • invoice_jan.pdf  → ⚡ Electricity
  • gas_q1.pdf       → 🔥 Natural Gas (mains)
  • lpg_delivery.pdf → 🫙 LPG / Bottled Gas
  • unknown_bill.pdf → ❓ Unrecognised — needs manual classification

Proceed to configure extraction fields?
```

If any file is unrecognised, ask the user to confirm the type before continuing.

---

### Step 2: Configure Extraction Fields

Present suggested data points for each detected invoice type. Ask the user to **confirm,
remove, or add** fields — including any custom computed fields — before extraction begins.

Use this format:

```
📋 EXTRACTION CONFIGURATION
──────────────────────────────────────────
I've pre-selected standard fields for each invoice type. Please confirm, adjust, or add fields.

⚡ ELECTRICITY invoices
  ✅ NMI (National Metering Identifier)
  ✅ Billing period (start date / end date)
  ✅ Total consumption (kWh)
  ✅ Peak consumption (kWh)          [remove if not needed]
  ✅ Off-peak consumption (kWh)      [remove if not needed]
  ✅ Total charges ($ excl. GST)
  ✅ Site / account name
  ✅ Tariff code / rate type
  ✅ Converted: MWh                  [=kWh/1000, for GRI 302-1]
  ➕ Add a field or computed logic?

🔥 NATURAL GAS invoices
  ✅ MIRN (Meter Installation Reference Number)
  ✅ Billing period (start date / end date)
  ✅ Total consumption (MJ)
  ✅ Total consumption (GJ)          [=MJ/1000]
  ✅ Total charges ($ excl. GST)
  ✅ Site / account name
  ➕ Add a field or computed logic?

🫙 LPG / BOTTLED GAS invoices
  ✅ Site / account name
  ✅ Delivery date
  ✅ Number of bottles / cylinders delivered
  ✅ Bottle size detected (kg)       [e.g. 45kg — detected from invoice]
  ✅ Total kg delivered              [=bottles × bottle_size]
  ✅ Total volume (litres)           [=kg ÷ 0.51 approximate density]
  ✅ Total energy (MJ)               [=kg × 25.3 calorific value]
  ✅ Total charges ($ excl. GST)
  ➕ Add a field or computed logic?

Reply with any changes, or type "confirmed" to begin extraction.
```

If the user defines a **custom computed field**, ask them to specify:
- The input column(s) to use
- The formula logic (e.g. "multiply bottles by 0.051 to get CO₂e tonnes")
Store this as an Excel formula, not a hardcoded value.

---

### Step 3: Extract Data from Each Invoice

For each file, use Python + pdfplumber to extract text and tables.

**PDF extraction:**
```python
import pdfplumber, re

def extract_invoice_text(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        full_text = ""
        for page in pdf.pages:
            full_text += page.extract_text() or ""
            for table in page.extract_tables():
                for row in table:
                    full_text += " | ".join([str(c) for c in row if c]) + "\n"
    return full_text
```

Install if needed: `pip install pdfplumber --break-system-packages`

**Parsing patterns by invoice type:**

```python
# NMI
re.search(r'NMI[:\s]+(\d{10,11})', text, re.IGNORECASE)

# Electricity consumption
re.search(r'Total\s+[Uu]sage[:\s]+([\d,]+\.?\d*)\s*kWh', text)

# Billing period
re.search(
    r'(?:Billing Period|Period)[:\s]+(\d{1,2}[\/\-]\d{1,2}[\/\-]\d{2,4})'
    r'\s*(?:to|–|-)\s*(\d{1,2}[\/\-]\d{1,2}[\/\-]\d{2,4})', text, re.IGNORECASE
)

# Gas MJ
re.search(r'Total\s+[Uu]sage[:\s]+([\d,]+\.?\d*)\s*MJ', text)

# LPG: bottles × bottle size
re.search(r'(\d+)\s*[Xx×]\s*(\d+)\s*kg', text)

# Total charges excl. GST
re.search(r'(?:Total\s+charges?|Amount\s+due)[^\$]*\$([\d,]+\.?\d*)', text, re.IGNORECASE)
```

**Confidence scoring:** Tag each extracted value as:
- ✅ **Found** — clear regex match
- ⚠️ **Uncertain** — ambiguous match (flag for review)
- ❌ **Not found** — no match (leave blank, highlight red in Excel)

---

### Step 4: Apply Computed Fields

Calculate derived columns using **Excel formulas** (not Python-hardcoded values):

| Derived Field | Formula | Notes |
|---|---|---|
| kWh → MWh | `=kWh_col/1000` | GRI 302-1 unit |
| MJ → GJ | `=MJ_col/1000` | GRI 302-1 unit |
| LPG kg → litres | `=kg_col/0.51` | Approximate density |
| LPG kg → MJ | `=kg_col*25.3` | Calorific value |
| LPG litres → MJ | `=litres_col*12.9` | Calorific value |
| User-defined | Per user spec | Store as formula |

---

### Step 5: Flag Extraction Issues

Before writing the Excel file, show an **Extraction Report**:

```
📊 EXTRACTION REPORT
──────────────────────────────────────────
Files processed: 4  |  Records extracted: 4

ISSUES:
🔴 lpg_delivery.pdf — Bottle size not found. Assumed 45kg. Verify highlighted cell.
⚠️  invoice_feb.pdf — kWh figure found in two locations (body: 1,234; summary: 1,230).
    Used summary table value.
⚠️  gas_q1.pdf — MIRN not found. Site name used instead.
✅ invoice_jan.pdf — All fields extracted successfully.
──────────────────────────────────────────
```

---

### Step 6: Generate Formatted Excel Output

Create a workbook using openpyxl with three sheets.

**Sheet 1: "Invoice Data"** — one row per invoice, all extracted + computed fields.

**Sheet 2: "Summary"** — aggregated by site and invoice type:
```
Site Name | Invoice Type | Period Covered | Total Consumption | Unit | Total Charges ($)
```

**Sheet 3: "ESG Mapping"** — pre-mapped to GRI 302-1 (Energy):
```
GRI Indicator | Description             | Total Value      | Unit | Source Files
GRI 302-1     | Electricity consumption | =SUM(range)      | MWh  | invoice_jan.pdf, ...
GRI 302-1     | Natural gas consumption | =SUM(range)      | GJ   | gas_q1.pdf
GRI 302-1     | LPG consumption         | =SUM(range)      | GJ   | lpg_delivery.pdf
```

**Formatting:**
```python
from openpyxl.styles import Font, PatternFill, Alignment
from openpyxl.utils import get_column_letter

# Header: dark teal background, white bold text
header_fill = PatternFill("solid", start_color="1F5C5C")
header_font = Font(bold=True, color="FFFFFF", name="Arial", size=10)

# Alternating row fills: white / light grey (F2F2F2)
# Extraction status cells:
#   ✅ Found     → green  (C6EFCE)
#   ⚠️ Uncertain → yellow (FFEB9C)
#   ❌ Not found → red    (FFC7CE)

# Freeze header row
sheet.freeze_panes = "A2"

# Auto-fit column widths (cap at 40)
for col in sheet.columns:
    max_len = max(len(str(cell.value or "")) for cell in col)
    sheet.column_dimensions[get_column_letter(col[0].column)].width = min(max_len + 4, 40)
```

**After creating the file**, recalculate formulas:
```bash
python /sessions/ecstatic-sleepy-ramanujan/mnt/.skills/skills/xlsx/scripts/recalc.py output.xlsx
```

Verify zero formula errors before delivering.

Save to: `/sessions/ecstatic-sleepy-ramanujan/mnt/outputs/invoice_extraction_[YYYYMMDD].xlsx`

---

### Step 7: Hand Off

Present the result and offer next steps:

```
✅ INVOICE EXTRACTION COMPLETE
──────────────────────────────────────────
[n] invoices processed → [n] records extracted

📎 [View your extracted invoice data](computer:///sessions/ecstatic-sleepy-ramanujan/mnt/outputs/invoice_extraction_YYYYMMDD.xlsx)

Sheets:
  • Invoice Data  — raw + computed values, one row per invoice
  • Summary       — aggregated by site and period
  • ESG Mapping   — values pre-mapped to GRI 302-1 (energy consumption)

⚠️ [n] extraction issues flagged — highlighted cells need review.
──────────────────────────────────────────
Would you like me to:
1. Feed this data into the ESG intake pipeline (map to GRI / TCFD / ASRS)?
2. Run a gap analysis against your full ESG dataset?
3. Process another batch of invoices?
```

Route to `data-intake-agent` or `gap-analyser-agent` based on response.

---

## Invoice Type Defaults

### ⚡ Electricity
NMI, billing period, total kWh, peak/off-peak kWh (if time-of-use), demand kVA (if applicable),
total charges excl. GST, retailer name, GHG factor if printed. Computed: MWh.

### 🔥 Natural Gas (mains)
MIRN, billing period, total MJ, total charges excl. GST, site name. Computed: GJ.

### 🫙 LPG / Bottled Gas
Site name, delivery date, bottle count, bottle size (kg, detected), total charges excl. GST.
Computed: total kg (bottles × size), litres (kg ÷ 0.51), MJ (kg × 25.3).
**If bottle size not found:** default to 45kg and highlight assumption in yellow.

### 💧 Water
Account/property number, billing period, total kL, sewerage kL, total charges excl. GST.
Computed: ML (kL ÷ 1000).

### ⛽ Diesel / Fuel
Account/vehicle/site, date(s), total litres, fuel type, total charges excl. GST.
Computed: GJ (litres × 0.03854 for diesel).

---

## Error Handling

- **Scanned image PDF:** Alert user. Attempt OCR via `pdf2image` + `pytesseract` as fallback.
  Flag all OCR values as ⚠️ Uncertain.
- **Multiple invoices in one PDF:** Detect page breaks and create multiple rows. Alert user
  if row count exceeds file count.
- **Duplicate detected:** Alert if same NMI/MIRN + billing period appears twice.
- **Non-AUD currency:** Flag and confirm with user before proceeding.
- **Missing bottle size (LPG):** Default 45kg, highlight yellow, note in Extraction Report.
- **pdfplumber not installed:** Run `pip install pdfplumber --break-system-packages` before
  extraction. If OCR needed: `pip install pdf2image pytesseract --break-system-packages`.
