# Nashville Housing: Data Cleaning Project

*A messy real-estate spreadsheet goes in. A clean, documented dataset comes out. Every change is written down.*

Hi, I'm Mahdi. This is a client-style data cleaning project. I imagined a real-estate company in Nashville handing me a rough Excel file and asking: *"Can you make this ready for analysis, and show us exactly what you changed?"*

The client is made up, but I worked as if they were real. I wrote a plan first, logged every change, double-checked every number, and wrote a report that someone with no Python knowledge can read.

If you only have two minutes, read **The short version** below and then open the [Data Quality Report](reports/data_quality_report.md).

---

## The short version

The file had 56,477 home sales (2013 to 2019) in 19 columns. It was usable, but it needed care:

- Labels were written in more than one way (`Yes`, `No`, `Y`, `N`).
- Almost every address had extra spaces inside.
- A few labels had typos.
- 29 sales had no property address.
- 104 sales had been recorded twice.
- More than half of the rows had no property assessment details at all.

I fixed what was clearly broken, and I **left alone** what only the client can explain. Those cases are flagged or reported, never guessed.

| | Before | After |
|---|---|---|
| Rows | 56,477 | 56,373 |
| Columns | 19 | 29 |
| Empty property addresses | 29 | 0 |
| Labels in `SoldAsVacant` | 4 (`Yes`, `No`, `Y`, `N`) | 2 (`Yes`, `No`) |
| Distinct `LandUse` labels | 39 | 37 |
| Duplicate sales | 104 | 0 |
| Final quality checks passed | | 12 of 12 |

---

## What was wrong, and what I did

| Problem | Rows affected | What I did |
|---|---|---|
| Extra spaces in `PropertyAddress` | 55,863 | Trimmed and collapsed spaces |
| Extra spaces in `OwnerAddress` | 25,315 | Trimmed and collapsed spaces |
| `SoldAsVacant` written as `Y` / `N` | 451 | Changed to `Yes` / `No` |
| Typos and split labels in `LandUse` | 1,557 | Mapped to one clean label each |
| Empty `PropertyAddress` | 29 | Filled from other rows with the same `ParcelID` |
| Same sale recorded twice | 104 | Removed the extra copy (kept the lowest `UniqueID`) |
| Column header `UniqueID ` had a trailing space | 1 column | Renamed to `UniqueID` |
| Addresses stored as one text | all | Added separate street, city and state columns (originals kept) |

## What I chose NOT to change (and why)

Cleaning is not only about fixing things. It is also about knowing when to stop.

- **Missing assessment details (30,462 rows, 53.9%).** Owner address, acreage, land value, building value, total value and tax district are empty together in the same rows. Every condo row is in this group. Inventing values would create fake data, so I added a `HasAssessorData` column instead.
- **243 parcels with more than one address (526 rows).** I can't tell which address is right, so I saved them in `reports/parcel_address_conflicts.csv` for the client.
- **`CONDO` vs `RESIDENTIAL CONDO` (247 rows).** They might be the same thing, or two different codes. The evidence points both ways (54 parcels appear under both labels, but the date ranges and prices differ), so I kept them apart and asked the client.
- **Unusual values.** Very low prices, zero bedrooms in a house, very large lots and homes "built" after they were sold are **flagged**, not removed. A flag means "take a second look", not "this is wrong".

---

## How I worked

1. **Plan first.** `PROJECT_CONTEXT.md` holds the client brief, what I found while profiling, the decisions I made, and a nine-step plan with the numbers I expected for each step.
2. **One step at a time.** Every step prints before/after counts, adds a line to the change log, and ends with `assert` checks.
3. **Stop when the numbers don't match.** Two real moments from this project:
   - The duplicate check expected 206 rows but found 208. Filling the 29 empty addresses had revealed one more hidden duplicate pair (one copy had no address, so it looked different before). I understood why before changing the expected number to 104.
   - Before filling addresses, my safety check found 243 parcels with several addresses. I checked that none of them belonged to the 29 rows I needed to fill, and I left them untouched and reported them.
4. **AI help, with rules.** I used a coding assistant in VS Code (the Continue extension with a Gemini model) and an AI chat assistant as a mentor. The coding assistant had to follow one rule: *propose first, wait for my "APPROVED", write code, then stop*. The prompts I wrote for it are in `.continue/prompts/`. Nothing ran without my approval.

---

## New columns

| Column | Meaning |
|---|---|
| `PropertySplitAddress`, `PropertySplitCity` | `PropertyAddress` split into street and city |
| `OwnerSplitAddress`, `OwnerSplitCity`, `OwnerSplitState` | `OwnerAddress` split into street, city and state |
| `Flag_LowSalePrice` | `True` if `SalePrice` is below 10,000 (94 rows) |
| `Flag_ZeroBedrooms` | `True` if `Bedrooms` is 0 on a residential land use (4 rows) |
| `Flag_LargeAcreage` | `True` if `Acreage` is above 10 (48 rows) |
| `Flag_UnusualYearBuilt` | `True` if `YearBuilt` is more than one year after the sale year (175 rows) |
| `HasAssessorData` | `True` if all six assessment columns have values (25,969 rows); `False` otherwise (30,404 rows) |

**Careful:** `False` in a flag column means "not flagged". It does **not** mean "checked and correct", because rows with no data can't be flagged either.

---

## Repository structure

```
nashville-housing-cleaning/
├── data/
│   ├── raw/                  # put the original Excel file here (not included)
│   └── cleaned/              # nashville_housing_cleaned.csv and .xlsx
├── notebooks/
│   └── 01_cleaning.ipynb     # the whole cleaning pipeline, step by step
├── reports/
│   ├── data_quality_report.md
│   ├── change_log.csv        # every change, with rows affected
│   ├── issue_register.csv    # every problem found
│   ├── validation_report.csv # 12 final checks (all PASS)
│   └── parcel_address_conflicts.csv
├── .continue/prompts/        # prompts I wrote for the coding assistant
├── PROJECT_CONTEXT.md        # brief, findings, decisions and the plan
├── requirements.txt
└── README.md
```

## Run it yourself

1. Download the original file (see **About the data**) and save it as `data/raw/Nashville Housing Data for Data Cleaning.xlsx`.
2. Create an environment and install the packages:

   ```bash
   python -m venv .venv
   .venv\Scripts\activate        # on macOS/Linux: source .venv/bin/activate
   pip install -r requirements.txt
   ```

3. Open `notebooks/01_cleaning.ipynb` and choose **Run All**. (Reading the Excel file takes a little while.)

Running the notebook rebuilds the cleaned data, the change log, the issue register and the parcel conflicts file. The notebook uses relative paths, so open it from the `notebooks` folder (VS Code does this by default).

**Tools:** Python 3.12.0, pandas 3.0.0, numpy 2.4.1, openpyxl 3.1.5, Jupyter in VS Code.

## About the data

The original file, *Nashville Housing Data for Data Cleaning*, is a popular practice dataset of Nashville home sales. I don't redistribute it here. Download it from the source you prefer: https://raw.githubusercontent.com/AlexTheAnalyst/PortfolioProjects/main/Nashville%20Housing%20Data%20for%20Data%20Cleaning.xlsx

## Things to keep in mind

- Rows are sorted by `UniqueID` after the duplicate step.
- The thresholds behind the flags (10,000 dollars, 10 acres, "more than one year after the sale") are my working choices. The client can change them.
- In `change_log.csv`, `rows_affected` does not always mean "values changed": for the split columns it counts rows that received new values, for the flags it counts flagged rows, and for the column rename it counts one column.
- I did not check addresses against any outside source such as a map or county record.

## What's next

- Send the seven open questions in the report to the client and update the data with their answers.
- Use the flags and `HasAssessorData` to look at prices separately for vacant land and built homes, because a vacant lot and a finished home are not like-for-like.

---

## About me

**Mahdi Yaqobi**, learning data science and AI, and building projects like this one to practice working the way a real client job works.
