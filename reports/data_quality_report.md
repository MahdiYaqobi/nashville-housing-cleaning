# Data Quality Report: Nashville Housing Sales

**Prepared by:** Mahdi Yaqobi
**Date:** October 2026
**Data:** 56,477 home sales, 2013 to 2019, 19 columns
**Note:** This is a practice project. The client is fictional, but the work was done as if they were real.

---

## 1. The short version

Thank you for sharing your sales file. Here is what you need to know:

- **The data is in good shape for analysis now.** I fixed the clear problems: inconsistent labels, extra spaces, typos, empty addresses and duplicate sales.
- **I removed 104 duplicate sales** and **filled 29 missing property addresses**. Nothing else was deleted.
- **I did not guess.** When I could not tell what was right, I left the data as it was and listed the question for you in section 5.
- **Everything is traceable.** Every change is in the change log (Appendix A), and 12 final quality checks all passed.
- **You now have 29 columns instead of 19.** The new ones split addresses into parts and mark rows that deserve a second look (section 6).

---

## 2. Before and after

| | Before | After |
|---|---|---|
| Rows | 56,477 | 56,373 |
| Columns | 19 | 29 |
| Empty property addresses | 29 | 0 |
| Labels in `SoldAsVacant` | 4 (`Yes`, `No`, `Y`, `N`) | 2 (`Yes`, `No`) |
| Distinct `LandUse` labels | 39 | 37 |
| Duplicate sales | 104 | 0 |

The 10 extra columns are: 5 address parts, 4 review flags, and 1 column called `HasAssessorData`.

---

## 3. What I found and what I did

Each problem has a short ID (I01, I02, ...) that matches the file `reports/issue_register.csv`.

"Share" is the percentage of the original 56,477 rows. For the review flags (I11 to I14) and the two "possible" issues (I09, I10) it is the percentage of the cleaned 56,373 rows.

### Fixed

| ID | Problem | Rows | Share | What I did |
|---|---|---|---|---|
| I01 | `SoldAsVacant` used `Y` and `N` next to `Yes` and `No` | 451 | 0.80% | Changed to `Yes` / `No` |
| I02 | Extra spaces in `PropertyAddress` | 55,863 | 98.91% | Trimmed and collapsed spaces |
| I03 | Extra spaces in `OwnerAddress` | 25,315 | 44.82% | Trimmed and collapsed spaces |
| I04 | Typos and split labels in `LandUse` | 1,557 | 2.76% | Mapped to one standard label each |
| I05 | Empty `PropertyAddress` | 29 | 0.05% | Filled from the same `ParcelID` |
| I17 | The `UniqueID` column header had a trailing space | 1 column | n/a | Renamed to `UniqueID` |

### Removed

| ID | Problem | Rows | Share | What I did |
|---|---|---|---|---|
| I07 | The same sale recorded twice | 104 | 0.18% | Removed the extra copy, keeping the lowest `UniqueID` |

### Left unchanged, and reported

| ID | Problem | Rows | Share | Why I left it |
|---|---|---|---|---|
| I06 | Assessment details missing (six columns together) | 30,462 | 53.94% | Inventing values would create fake data |
| I08 | Parcels that appear with more than one address | 526 (243 parcels) | 0.93% | I can't tell which address is right |
| I09 | `CONDO` and `RESIDENTIAL CONDO` may be the same | 247 | 0.44% | The evidence is mixed (section 4), and only you know if they are the same |
| I10 | Land-use says vacant land, but `SoldAsVacant` says `No` | 1,296 | 2.30% | I can't tell which column to trust (852 more rows disagree the other way, see section 4) |
| I15 | Bedrooms, year built and bathrooms often empty | 32,320 (bedrooms) | 57.23% | Not recorded in the source |
| I16 | Gaps in the `UniqueID` sequence | 159 IDs | 0.28% | Harmless, but good to know |

### Marked for a second look (nothing removed)

| ID | Flag column | Rule | Rows | Share |
|---|---|---|---|---|
| I11 | `Flag_LowSalePrice` | `SalePrice` below 10,000 | 94 | 0.17% |
| I12 | `Flag_ZeroBedrooms` | 0 bedrooms on a residential land use | 4 | 0.01% |
| I13 | `Flag_LargeAcreage` | `Acreage` above 10 | 48 | 0.09% |
| I14 | `Flag_UnusualYearBuilt` | Built more than 1 year after the sale year | 175 | 0.31% |

---

## 4. The details, in plain words

### Duplicate sales (I07)

A sale counted as a duplicate when the parcel, address, sale date, sale price and legal reference were all the same. I found 104 such pairs and kept the one with the lowest `UniqueID` in each pair.

Two things are worth knowing:

- **One pair was not an exact copy.** The two rows for 2616 Lakeside Meadows Dr (Mount Juliet) matched in everything except `LandUse`: one said `VACANT RES LAND` (`UniqueID` 11912) and the other said `SINGLE FAMILY` (`UniqueID` 12629). I kept 11912 because of my "lowest `UniqueID`" rule, **not** because I know it is the correct label. Please confirm (section 5, question 2).
- **One duplicate was hidden at first.** One copy of the sale at 815 31st Ave N (Nashville) had no address, so it did not look like a duplicate until I filled the address. That is why the final count is 104 and not 103. This is also why I fill addresses *before* looking for duplicates.

### Missing assessment details (I06)

For 30,462 rows (53.9% of the file), six columns are empty **together**: `OwnerAddress`, `Acreage`, `LandValue`, `BuildingValue`, `TotalValue` and `TaxDistrict`. There is not a single row where only some of them are missing. That tells me the data was simply never recorded for those sales, so there is nothing to repair.

I did not fill these in. Instead I added `HasAssessorData`, which is `True` when all six columns have values. After removing duplicates, 25,969 rows are `True` and 30,404 are `False`.

One pattern stands out: **none** of the 14,311 condo rows (247 `CONDO` and 14,064 `RESIDENTIAL CONDO`) has assessment details or a build year. They make up about 47% of the rows where `HasAssessorData` is `False`.

Heads up: `Bedrooms`, `YearBuilt`, the bathroom columns and `OwnerName` are empty in **even more** rows (roughly 55% to 57%). `HasAssessorData` does not cover them.

### Parcels with more than one address (I08)

Normally one `ParcelID` means one property. But 243 parcels show up with two or three different addresses (526 rows in total). Some differ by a tiny typo (for example `118 A LARKIN SPRINGS RD` and `118 ALARKIN SPRINGS RD`). Others are completely different streets. I did not change any of them. The full list is in `reports/parcel_address_conflicts.csv`.

This matters for one more reason. To fill the 29 empty addresses I looked up other rows with the same `ParcelID`. I first checked that each of those 29 parcels had exactly **one** known address, and none of them was among the 243 conflicts. Still, those 29 addresses rely on the assumption that the other rows are right.

### Typos and split labels in `LandUse` (I04)

| Before | After | Rows |
|---|---|---|
| `VACANT RES LAND` | `VACANT RESIDENTIAL LAND` | 1,549 |
| `VACANT RESIENTIAL LAND` | `VACANT RESIDENTIAL LAND` | 3 |
| `RESTURANT/CAFETERIA` | `RESTAURANT/CAFETERIA` | 2 |
| `GREENBELT/RES` with a line break and the typo `GRRENBELT` | `GREENBELT/RES` | 3 |

In addition, 38 labels had stray spaces or a line break, which I cleaned up.

### `CONDO` vs. `RESIDENTIAL CONDO` (I09)

247 rows are labeled `CONDO` and 14,064 are labeled `RESIDENTIAL CONDO`. The evidence points both ways, so I kept them apart:

- **Similar:** neither group has assessment details or a build year, and 54 parcels appear under both labels.
- **Different:** `CONDO` sales only appear between January 2013 and May 2014, while `RESIDENTIAL CONDO` sales run until October 2016. The median price is 362,000 dollars for `CONDO` and 220,000 dollars for `RESIDENTIAL CONDO`. None of the `CONDO` rows was sold as vacant, compared with 1.6% of the other group.

One possible reading is that `CONDO` is an older label (I have **not** confirmed this), but the price gap alone doesn't prove it. This is question 4 in section 5.

### Vacant land vs. `SoldAsVacant` (I10)

The two columns agree for most rows, but not for all. Of the 5,113 rows with a vacant land use, 3,817 say `SoldAsVacant = Yes` and 1,296 say `No`. Of the 4,669 rows with `SoldAsVacant = Yes`, 852 have a land use that is not vacant (for example 593 `SINGLE FAMILY` and 223 `RESIDENTIAL CONDO`). In total, 2,148 rows (3.8%) disagree in one direction or the other.

One possible explanation (I have **not** confirmed it) is that `LandUse` describes the property as recorded in the assessment, while `SoldAsVacant` describes its state on the day of the sale. A lot sold empty and built on later would produce exactly this pattern.

### Homes "built" after they were sold (I14)

175 rows have a `YearBuilt` that is more than one year after the sale year.

- For 174 of them the gap is 2 to 3 years, and for one it is 4. That looks like a normal construction timeline, not random typing mistakes.
- 50 of them (29%) were sold as vacant land. Among all rows that have a build year, only about 1.8% were sold as vacant, so this is much higher than usual. A lot sold empty and built on later fits well.
- 29 more are classified as vacant residential land even though `SoldAsVacant` says `No`.
- That leaves 87 single-family homes that this story does not explain. They could be sales during construction, or recorded differently.

---

## 5. Questions for you

Until I hear from you, I left everything as described above.

1. **Missing assessment details (I06).** Do you know why 54% of sales have no assessment details (every condo sale is in this group)? Can you provide them?
2. **The duplicate pair with different land uses (I07).** For 2616 Lakeside Meadows Dr, which is right: `VACANT RESIDENTIAL LAND` (kept) or `SINGLE FAMILY` (removed)?
3. **Parcels with several addresses (I08).** Can you or your team check the 243 parcels in `parcel_address_conflicts.csv`? Which address is correct for each?
4. **Condos (I09).** Are `CONDO` and `RESIDENTIAL CONDO` the same category? They look alike in some ways (54 parcels appear under both labels) and different in others (date ranges and prices, see section 4). If they are the same, I'll merge the 247 rows.
5. **Vacant land vs. `SoldAsVacant` (I10).** Which column should I trust when they disagree?
6. **Missing property details (I15).** Can you share bedroom, bathroom and year-built data for the rows where it is empty?
7. **Very low prices.** The lowest recorded price is 50 dollars, and 94 sales are under 10,000 dollars. What do these represent (family transfers, data errors, something else)? Should they be excluded from price analysis?

---

## 6. How to read the new columns

| Column | What it tells you |
|---|---|
| `PropertySplitAddress`, `PropertySplitCity` | Street and city from `PropertyAddress` |
| `OwnerSplitAddress`, `OwnerSplitCity`, `OwnerSplitState` | Street, city and state from `OwnerAddress` (empty where the owner address is empty) |
| `Flag_LowSalePrice` | `True` for 94 sales under 10,000 dollars |
| `Flag_ZeroBedrooms` | `True` for 4 single-family homes with 0 bedrooms. Zero bedrooms is normal for churches and vacant lots, so those are **not** flagged. |
| `Flag_LargeAcreage` | `True` for 48 parcels above 10 acres. A typical parcel in this file is about a quarter of an acre. |
| `Flag_UnusualYearBuilt` | `True` for 175 rows built more than a year after the sale |
| `HasAssessorData` | `True` for 25,969 rows with full assessment details, `False` for 30,404 |

Two things to remember:

- **`True` means "take a second look", not "this is wrong".** Many flagged rows are perfectly real.
- **`False` means "not flagged", not "checked and correct".** Rows with no data can't be flagged.

The thresholds (10,000 dollars, 10 acres, one year after the sale) are my working choices. If you prefer different ones, I can recalculate the flags.

---

## 7. How I checked my work

The cleaning notebook stops with an error if a number is different from what I expected, so a problem can't slip through quietly. After cleaning, I ran 12 final checks, and all of them passed:

- The original data was not modified, and the row math adds up (56,477 minus 104 equals 56,373).
- All original columns are still there, and only the expected new ones were added.
- No empty property addresses remain.
- `SoldAsVacant` has only `Yes` and `No`, and `LandUse` has 37 distinct labels.
- The text columns have no extra spaces left.
- No duplicates remain, and the right `UniqueID` was kept in the pair I mentioned above.
- The flag columns are true/false columns, the row count did not change, and the flag counts match what I expected (94, 48 and 175 for the three I tested).

I also reloaded the exported CSV and Excel files to make sure they have the same size and columns as the cleaned data. The full results are in `reports/validation_report.csv`.

---

## 8. Limits of this cleaning

- I filled in only one kind of missing value: the 29 empty property addresses, copied from other rows with the same `ParcelID` (see section 4). I did not fill in anything else: not the assessment details, bedrooms, year built, bathrooms or owner names.
- I did not check addresses against outside sources such as maps or county records.
- Prices, sizes and years were checked for strange values, but not verified against real records.
- After the duplicate step, rows are sorted by `UniqueID` instead of the original order.

---

## 9. Tips for your analysis

- **Look at vacant land and built homes separately.** A vacant lot and a finished home are not like-for-like when you compare prices.
- **Check the flags before averaging prices.** A few extreme values can move an average a lot (for example, the average sale price is far above the median).
- **Be careful with the property details.** About 57% of rows have no bedroom or year-built value, so any result based on those columns only covers part of your data.

---

## Appendix A: Full change log

Same content as `reports/change_log.csv`. A note on the last column: it counts rows whose value changed, except for the address split (rows that received new split values), the review flags (rows flagged), the column rename (one column) and `HasAssessorData` (rows where it is `False`).

| Step | What changed | Rows affected |
|---|---|---|
| 1a | Strip and collapse whitespace in `PropertyAddress` | 55,863 |
| 1a | Strip and collapse whitespace in `OwnerAddress` | 25,315 |
| 1b | `LandUse`: strip and collapse whitespace | 38 |
| 1b | `LandUse`: `VACANT RES LAND` to `VACANT RESIDENTIAL LAND` | 1,549 |
| 1b | `LandUse`: `VACANT RESIENTIAL LAND` to `VACANT RESIDENTIAL LAND` | 3 |
| 1b | `LandUse`: `RESTURANT/CAFETERIA` to `RESTAURANT/CAFETERIA` | 2 |
| 1b | `LandUse`: `GRRENBELT...` to `GREENBELT/RES` | 3 |
| 1c | Rename column header `UniqueID ` (trailing space) to `UniqueID` | 1 |
| 2 | `SoldAsVacant`: `Y` to `Yes`, `N` to `No` | 451 |
| 3 | Fill missing `PropertyAddress` via `ParcelID` | 29 |
| 4 | Split `PropertyAddress` into `PropertySplitAddress` and `PropertySplitCity` | 56,477 |
| 4 | Split `OwnerAddress` into `OwnerSplitAddress`, `OwnerSplitCity`, `OwnerSplitState` | 26,015 |
| 5 | Remove business-key duplicates (kept lowest `UniqueID`) | 104 |
| 6 | Flag low sale price (below 10,000) | 94 |
| 6 | Flag zero bedrooms in residential properties | 4 |
| 6 | Flag large acreage (above 10) | 48 |
| 6 | Flag unusual year built (after sale year + 1) | 175 |
| 7 | Add `HasAssessorData` flag | 30,404 |

## Appendix B: Files in this delivery

| File | What it is |
|---|---|
| `data/cleaned/nashville_housing_cleaned.csv` and `.xlsx` | The cleaned dataset (56,373 rows, 29 columns) |
| `reports/change_log.csv` | Every change, with rows affected |
| `reports/issue_register.csv` | Every problem found, with severity and action taken |
| `reports/validation_report.csv` | The 12 final quality checks |
| `reports/parcel_address_conflicts.csv` | The 243 parcels with several addresses |
| `notebooks/01_cleaning.ipynb` | The full cleaning code, step by step |
