# Project: Nashville Housing - Data Cleaning (client-style)

## Brief
Fictional real-estate client in Nashville sent a raw Excel file of home sales.
Client wants: (1) clean, standardized data ready for analysis, (2) a report of the problems
found, (3) an exact list of every change made, so they can trust the result.

## Files and names
- Raw file (never edited): data/raw/Nashville Housing Data for Data Cleaning.xlsx
- Notebook: notebooks/01_cleaning.ipynb
- Variables: df_raw (untouched), df_clean (working copy), change_log (DataFrame with
  columns: step, description, rows_affected)
- Outputs: data/cleaned/nashville_housing_cleaned.xlsx and .csv,
  reports/change_log.csv, reports/data_quality_report.md, reports/change_log.md, README.md

## Verified dataset facts (from profiling on df_raw)
- Shape: 56,477 rows x 19 columns: UniqueID, ParcelID, LandUse, PropertyAddress, SaleDate,
  SalePrice, LegalReference, SoldAsVacant, OwnerName, OwnerAddress, Acreage, TaxDistrict,
  LandValue, BuildingValue, TotalValue, YearBuilt, Bedrooms, FullBath, HalfBath.
- SaleDate is already datetime (2013-01-02 to 2019-12-13). No conversion needed.
- Address format: PropertyAddress = "street, city"; OwnerAddress = "street, city, state".
- No fully identical rows. No repeated UniqueID. UniqueID ranges 0-56,635 (about 158 IDs absent).
- SoldAsVacant: No 51,403 | Yes 4,623 | N 399 | Y 52.
- PropertyAddress: 55,863 rows have repeated spaces; 7 have leading/trailing spaces;
  29 are null (all recoverable via ParcelID).
- OwnerAddress: 25,315 rows have repeated spaces; 30,462 are null.
- Structural missingness: OwnerAddress, Acreage, LandValue, BuildingValue, TotalValue,
  TaxDistrict are null in EXACTLY the same 30,462 rows (no partial cases).
  Bedrooms, YearBuilt, FullBath, HalfBath are about 57% null (more than the six above).
  OwnerName is 55.27% null.
- Business-key duplicates on [ParcelID, PropertyAddress, SaleDate, SalePrice, LegalReference]:
  103 extra rows (103 groups of 2). 102 groups are identical except UniqueID.
  1 odd group: ParcelID '110 03 0A 005.00', 2616 LAKESIDE MEADOWS DR, MOUNT JULIET:
  UniqueID 11912 LandUse 'VACANT RES LAND' vs UniqueID 12629 LandUse 'SINGLE FAMILY'
  (both SoldAsVacant = Yes, both without assessor data).
- LandUse has 39 distinct values. Problems: 'VACANT RES LAND' (1,549) and
  'VACANT RESIENTIAL LAND' (3) vs 'VACANT RESIDENTIAL LAND' (3,547); 'RESTURANT/CAFETERIA' (2);
  'GREENBELT/RES_x000D_\nGRRENBELT/RES' (3; Excel carriage-return artifact + typo);
  'CONDOMINIUM OFC  OR OTHER COM CONDO' (35; double space).
  'CONDO' (247) and 'RESIDENTIAL CONDO' (14,080) may or may not be the same category.
- About 82% of SoldAsVacant = Yes rows have vacant-land LandUse; about 13% are SINGLE FAMILY.
  So Yes + SINGLE FAMILY is NOT a contradiction by itself (hypothesis, unverified:
  LandUse may describe current use, SoldAsVacant the status at sale).
- Suspicious values (flag only): SalePrice min 50, max 54,278,060 (mean 327k, median 205k);
  Bedrooms min 0; Acreage max 160.06 (median 0.27); YearBuilt min 1799.

## Decisions
- Do NOT impute structural missing values. Add HasAssessorData flag instead.
- Do NOT delete outliers or suspicious values. Add boolean flag columns.
- Keep original address columns when splitting.
- Keep CONDO and RESIDENTIAL CONDO separate.
- Duplicates: keep the lowest UniqueID in each group (this also resolves the odd group
  by keeping 11912).

## Assumptions that need client confirmation (put in the report)
1. Odd duplicate pair resolved by lowest UniqueID rule (kept 11912).
2. CONDO vs RESIDENTIAL CONDO may overlap; left unmerged.
3. SoldAsVacant vs LandUse mismatch pattern (see above) is unexplained.
4. Meaning of very low SalePrice values (e.g. 50) is unknown (non-market transfer?).
5. Assessor data absent in about 54% of rows; reason unknown.

## PLAN (each step: Do / Expected / Validate)

### Step 0 - Setup
Do: df_clean = df_raw.copy(); create empty change_log; confirm SaleDate with
pd.api.types.is_datetime64_any_dtype.
Validate: df_clean is not df_raw; shape == (56477, 19); change_log empty.

### Step 1 - Text standardization
1a Do: PropertyAddress and OwnerAddress: strip + collapse repeated whitespace (keep NaN).
   Expected: PropertyAddress rows changed >= 55,863; OwnerAddress about 25,315.
   Null counts unchanged: PropertyAddress 29, OwnerAddress 30,462.
1b Do: LandUse: strip + collapse whitespace, then ONLY this mapping (log each as its own row):
   'VACANT RES LAND' -> 'VACANT RESIDENTIAL LAND' (1,549)
   'VACANT RESIENTIAL LAND' -> 'VACANT RESIDENTIAL LAND' (3)
   'RESTURANT/CAFETERIA' -> 'RESTAURANT/CAFETERIA' (2)
   any value containing 'GRRENBELT' -> 'GREENBELT/RES' (3)
   Do NOT merge CONDO with RESIDENTIAL CONDO.
   Expected: distinct LandUse 39 -> 37; 'VACANT RESIDENTIAL LAND' count == 5,099.
1c Do: UniqueID column name have some spaces in the raw file. Rename to 'UniqueID' (no spaces).
Validate: no repeated/leading/trailing spaces and no '_x000D_' left in the three columns.

### Step 2 - SoldAsVacant
Do: 'Y' -> 'Yes', 'N' -> 'No'. Expected: 451 rows changed; Yes 4,675; No 51,802.
Validate: set of values == {'Yes', 'No'}; no nulls introduced.

### Step 3 - Fill PropertyAddress via ParcelID
Pre-check: each ParcelID has exactly ONE distinct non-null PropertyAddress (after step 1).
If not, STOP. Do: fill the 29 null rows only. Expected: 29 rows changed.
Validate: PropertyAddress nulls == 0.

### Step 4 - Split addresses (keep originals)
Do: PropertySplitAddress, PropertySplitCity (split on comma, expect 2 parts);
OwnerSplitAddress, OwnerSplitCity, OwnerSplitState (expect 3 parts); strip each part.
Rows with an unexpected number of parts: STOP and show me.
Validate: split columns non-null wherever the source is non-null; Owner split nulls == 30,462.

### Step 5 - Duplicates
Key: [ParcelID, PropertyAddress, SaleDate, SalePrice, LegalReference], recomputed on df_clean
(after steps 1-3). Expected on raw: 103 extra rows in 103 groups of 2. If the recomputed
count is not 103, or any group size is not 2: STOP and show me before dropping.
Do: keep the lowest UniqueID per group; log the odd pair as an assumption.
Validate: 103 rows removed; final rows 56,374; 11912 present and 12629 absent;
zero duplicates on the key.

### Step 6 - Flag suspicious values (no deletions)
Do: PROPOSE thresholds from the distributions first (wait for my approval), then add boolean
columns: Flag_LowSalePrice, Flag_ZeroBedrooms, Flag_LargeAcreage, Flag_UnusualYearBuilt.
Validate: row count unchanged; flag columns are boolean; counts logged.

### Step 7 - HasAssessorData
Do: boolean column, True where all six structural columns are non-null. No imputation.
Expected: False count == df_clean['TotalValue'].isna().sum() (30,462 minus removed duplicates
that had no assessor data; measure and report the exact number).

### Step 8 - Final validation
Use the /validate skill. Every check must PASS or be explained.

### Step 9 - Export
Do: save data/cleaned/nashville_housing_cleaned.xlsx and .csv (UTF-8, index=False) and
reports/change_log.csv. Validate: reload both files; shape and columns match df_clean.