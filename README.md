# BI Data Validator

A small Python 3.11+ command-line tool for Domo-to-Power-BI migrations.
It compares a Domo card CSV baseline with an equivalent Databricks query CSV.
It identifies discrepancies and summarizes missing populations; it does not infer upstream causes.

## Setup

From this folder, using Python 3.11 or newer:

```powershell
python -m venv .venv
.venv\Scripts\python -m pip install -r requirements.txt
```

## Run

Edit `config.yaml` to specify a validation name, a single key column, and a nonempty
list of fields to compare. Both CSVs must contain all configured columns.

```powershell
.venv\Scripts\python validate.py --domo data/domo.csv --databricks data/databricks.csv --config config.yaml --output output/
```

With dependencies installed in your active Python environment, the equivalent command is:

```text
python validate.py --domo data/domo.csv --databricks data/databricks.csv --config config.yaml --output output/
```

Supply your own CSV exports. Files use UTF-8 (with or without a BOM), commas, and a
header row with unique, nonempty column names. Header-only files are supported.

## Comparison rules

- Domo is the baseline. Row difference is Databricks minus Domo.
- All values are read as text, preserving leading zeros. Case and whitespace matter;
  `1` differs from `1.0`. Dates are not normalized for field comparisons.
- Empty cells are missing values and match each other. Literal `NA` and `NULL` remain text.
- Blank keys are errors. Duplicate keys are allowed and reported both as repeated
  distinct keys and as excess rows beyond each first occurrence.
- Missing/extra counts are distinct keys. Their CSVs contain every original row
  for those keys, with original columns and order.
- Common keys are compared per configured column. For duplicate keys, the collection
  of values, including repetitions, must match. There is no row pairing within a key.
  This can miss differences in how field values are combined across duplicate rows;
  use a truly unique key when row-level correspondence matters.
- Field mismatch count means differing key/column pairs. Exports contain the configured
  key, `column`, `domo_values`, and `databricks_values`. Values are sorted JSON lists;
  empty cells appear as `null`. Those three metadata names cannot be used as the key name.
- Missing-record breakdowns count Domo rows. Fields containing `date` in their name or
  ending in `_at` use years when all populated values are ISO dates/timestamps;
  otherwise values are grouped literally. No timezone conversion occurs. Blank values
  display as `(blank)`. All categories are included.
- PASS requires equal row counts and unique-key counts, no missing/extra keys, and no
  configured field differences. Identical duplicate populations may PASS.

## Outputs

The output directory is created automatically. Each successful validation run writes:

- `validation_report.html`: status, metrics, duplicate counts, and missing breakdowns.
- `missing_from_databricks.csv`
- `extra_in_databricks.csv`
- `field_mismatches.csv`

Empty exports still include headers. Subsequent runs replace these four files.
Exit codes: **0** PASS, **1** FAIL, **2** invalid input or an execution error.
Input errors do not generate a new report; any previous output remains from its prior run.

## Tests

```powershell
.venv\Scripts\python -m pytest -q
```

The comparison functions in `comparison.py` use generic source/target terminology.
`validate.py` supplies the Domo/Databricks CLI and report labels. No service integrations
or credentials are needed.
