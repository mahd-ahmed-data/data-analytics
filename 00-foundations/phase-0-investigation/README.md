# Phase 0: data investigation of inventory.csv

## Source and grain
inventory.csv: 106,367 rows, 15 columns, no blank cells. MRP forecast quantities for notebook and desktop components. Source: Source: "inventory" by veeralakrishna on Kaggle
(https://www.kaggle.com/datasets/veeralakrishna/inventory). License: CC0: Public Domain.

Grain: one component (COMP_ITEM_ID) for one product (PRODUCT_ID), in one channel (CHANNEL_ID), for one demand week (DMND_WEEK_STRT_DATE), as forecast in one forecast version (VRSN_WEEK_STRT_DATE), with the quantity in MRP_FCST_QTY. The file has 3,098 distinct items, so one row is not one item.

## Quality problems
| Problem | Count | Why it matters |
|---|---|---|
| No row ID | all rows | Entries can't be told apart or traced. |
| Rows identical in all 15 columns (extra copies) | 26,455 (25%) | Totals of MRP_FCST_QTY are inflated by about a quarter. |
| Same 6-column key, different content | 712 extra rows, 691 groups, 1,403 rows | Two forecasts for the same slot. A sum or average mixes both. |
| Of those: quantity differs | 664 groups | Real disagreement about the forecast. |
| Of those: same quantity, only scheduler or supply chain type differs | 27 groups (28 extra rows with the one larger group) | Labeling inconsistency, not a forecast conflict. |
| DW_PKG_UPD_DTS holds only minutes and seconds (e.g. 51:30.0) | 106,367 rows | Can't be used to order, date or identify an entry. |
| MRP_FCST_QTY = 0 | 56,198 (53%) | Unclear whether 0 means no demand or no forecast. Averages are pulled down. |
| Forecast version later than the demand week | 1,937 rows | A forecast made after the week it covers. Not a true forecast. |
| Dates stored as text (dd-mm-yyyy) | all rows | No date sorting or grouping by month until converted. |
| 4,028 PRODUCT_ID vs 4,029 PRODUCT_NAME values | | One more name than ID. Not yet investigated. |

## Three questions the data can answer
1. Which commodities have the largest total forecast quantity? (COMMODITY, MRP_FCST_QTY, after removing exact copies)
2. How does total forecast quantity change by demand week? (DMND_WEEK_STRT_DATE as a real date, MRP_FCST_QTY)
3. How does forecast quantity differ by supply chain type? (SUPPLY_CHAIN_TYPE, MRP_FCST_QTY)

## One question it cannot answer
How accurate the forecasts were. The file has the forecast quantity but no actual demand, sales or consumption column to compare it with.

## Decisions and limits
- Remove: the 26,455 exact copies. They add no information.
- Flag, don't remove: the 712 key conflicts. Deciding which row is right needs to know which forecast version is final, and this file doesn't say.
- Flag: the 1,937 late-version rows and the zeros, until their meaning is known.
- Don't use: DW_PKG_UPD_DTS. It can't be repaired from this file.
- Limit: with no row ID, I can't prove that every exact copy was a data-entry error and not a real repeated entry.