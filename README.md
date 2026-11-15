
# Financial Analytics: CFO Executive Dashboard

An executive finance dashboard built in Power BI as a Power BI Project (PBIP). It covers profitability, budget performance, cash, recurring revenue and customer metrics across four report pages and 14 KPIs.

| | |
|---|---|
| Tool | Power BI Desktop (PBIP / PBIR / TMDL) |
| Model | Star schema, 2 fact tables, 2 dimensions, 25 DAX measures |
| Report | 4 pages, 14 executive KPIs |
| Period | Actuals Jan 2021 - Jun 2026, budget through Dec 2026 |

---

## Dashboard Pages

| # | Page | Purpose |
|---|---|---|
| 1 | Executive Overview | Revenue, margins, operating profit, free cash flow, cash balance and ARR; monthly trends; revenue by region; summary P&L by year |
| 2 | Budget vs Actual | Revenue, cost and operating profit against budget; monthly variance |
| 3 | ARR & Customers | ARR trend, customer count, ARR per customer, churn |
| 4 | Country Performance | Region and country scorecard; drill-through target from the region visuals |

Global filters: Year, Quarter (all pages) and Region (pages 3 and 4).

## Executive KPIs (14)

| # | KPI | Measure |
|---|---|---|
| 1 | Revenue | `[Revenue]` |
| 2 | Revenue YoY % | `[Revenue YoY %]` |
| 3 | Gross Margin % | `[Gross Margin %]` |
| 4 | Operating Profit | `[Operating Profit]` |
| 5 | Operating Margin % | `[Operating Margin %]` |
| 6 | Free Cash Flow | `[Free Cash Flow]` |
| 7 | Cash Balance | `[Cash Balance]` |
| 8 | Revenue vs Budget % | `[Revenue vs Budget %]` |
| 9 | Costs vs Budget % | `[Costs vs Budget %]` |
| 10 | Operating Profit vs Budget | `[Operating Profit vs Budget]` |
| 11 | ARR | `[ARR]` |
| 12 | Customers | `[Customers]` |
| 13 | ARR per Customer | `[ARR per Customer]` |
| 14 | Monthly ARR Churn % | `[Monthly ARR Churn %]` |

---

## Semantic Model

### Tables

| Table | Type | Grain | Notes |
|---|---|---|---|
| `MonthlyFinancials` | Fact | Month | COGS, Opex, revenue and cost budgets, cash balance, free cash flow (company level) |
| `CountryPerformance` | Fact | Month x Country | Revenue, ARR, customers, churned ARR |
| `DimDate` | Dimension | Day | Calendar table, Year > Quarter > Month hierarchy |
| `DimGeography` | Dimension | Country | Region > Country hierarchy |
| `_Measures` | Measure table | - | All reporting measures, in display folders |

### Relationships

| From (many) | To (one) | Direction |
|---|---|---|
| `MonthlyFinancials[Month]` | `DimDate[DateKey]` | Single |
| `CountryPerformance[Month]` | `DimDate[DateKey]` | Single |
| `CountryPerformance[Country]` | `DimGeography[Country]` | Single |

`Month` in the source data is the Excel serial number of the first day of the month, which is why the date relationship uses an integer key.

### Measures

| Display folder | Measures |
|---|---|
| 1 Income Statement | Revenue, COGS, Operating Expenses, Total Costs, Gross Profit, Operating Profit, Gross Margin %, Operating Margin %, Revenue PY, Revenue YoY % |
| 2 Budget Performance | Revenue Budget, Cost Budget, Operating Profit Budget, Revenue vs Budget %, Costs vs Budget %, Operating Profit vs Budget |
| 3 Cash & Liquidity | Cash Balance, Free Cash Flow |
| 4 ARR & Customers | ARR, Customers, ARR per Customer, Churned ARR, Monthly ARR Churn % |
| Helpers (hidden) | Last Actual Month, Is Company Scope |

### Modeling Decisions

- **Point-in-time measures.** ARR, Customers and Cash Balance return the value at the last month in the selected period and are never summed across months.
- **Budget alignment.** Budget and prior-year comparisons stop at the last month with actuals, so a part-year is compared with the same part of the budget and prior year.
- **Geography guard.** COGS, Opex, budget and cash exist only at company level. Under a Region or Country filter, profit, budget and cash measures return blank instead of misleading values. Revenue, ARR, customers and churn still respond to geography.
- **Explicit measures only.** Fact columns are hidden; reports use measures from `_Measures`.
- **Calendar year** is used as the fiscal year.

### Validation

Measures were checked with DAX queries against source totals: revenue, cost, free cash flow, budget through the last actual month, and prior-year revenue for part-year periods. There are no orphan rows in any relationship, and the geography guard returns blank as intended.

---

## Repository Structure

```
Financial_Analytics.pbip
Financial_Analytics.SemanticModel/     TMDL model definition (tables, measures, relationships)
Financial_Analytics.Report/
  definition/                          PBIR: pages, visuals, bookmarks, report settings
  StaticResources/                     Custom theme (FinancialDashboardTheme.json)
README.md
```

## Getting Started

1. Install the latest Power BI Desktop.
2. Enable PBIR: File > Options and settings > Options > Preview features > "Store reports using enhanced metadata format (PBIR)". Restart Desktop if prompted.
3. Open `Financial_Analytics.pbip`.
4. In Table tools, mark `DimDate` as the date table on `DimDate[Date]` if it is not already marked.
5. Refresh the data.

## Design

| Item | Value |
|---|---|
| Canvas | 1280 x 720 |
| Font | Segoe UI |
| Primary | #0B3C5D |
| Highlight | #328CC1 |
| Comparison (budget, prior year) | #9AA5B1 |
| Favourable / unfavourable | #1E8E5A / #C0392B |
| Canvas background | #F4F6F9 |

Favourable and unfavourable colors are applied to variance KPIs. Cost variance is inverted, so costs above budget show as unfavourable.

## Assumptions and Limitations

- The currency is not specified in the data, so values are shown in M and K units with no symbol.
- `Cost_Budget` is treated as total costs (COGS plus Opex).
- Cost, budget and cash data are company level only; there is no country-level P&L.
- Monthly ARR Churn % is churned ARR divided by ARR, weighted across the months in the selected period.
- The report layer was generated programmatically as PBIR and validated against Microsoft's published schemas. Some visual formatting, such as slicer styling and navigation layout, is still being refined in Desktop.

## Tech Stack

Claude Code, Power BI, DAX, TMDL, PBIR, Power BI Project (PBIP)
