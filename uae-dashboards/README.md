# UAE industry dashboards

Four interactive dashboards on synthetic UAE data, plus the CSV files behind them.

| File | What it shows |
|---|---|
| `index.html` | Landing page linking all four dashboards |
| `real-estate.html` | Dubai residential sales: price per sq ft, rental yield, off-plan share, buyer origin |
| `ecommerce.html` | UAE online store: revenue, cash on delivery, returns, delivery speed by emirate |
| `asset-management.html` | DIFC fund house: AUM, net flows, performance vs benchmark, client call list, liquidity |
| `logistics.html` | UAE 3PL: on-time delivery by lane, causes of delay, cost per kg, transit times |
| `power-bi.html` | The Power BI version of the asset management dashboard: model, key DAX measures, download link |
| `power-bi/DIFC-Fund-House-PowerBI.zip` | The Power BI project itself (open the .pbip in Power BI Desktop) |
| `data/*.csv` | The datasets, for rebuilding each dashboard in Power BI |

Every page works on its own with no build step and no chart library. Charts are drawn in SVG, and the page follows the visitor's light or dark mode.

## Put it on your site

1. Unzip `uae-dashboards.zip`.
2. Open the repository behind your site on GitHub (the one named `nishadwankhede1.github.io`).
3. Click **Add file**, then **Upload files**, and drag the whole `uae-dashboards` folder onto the page.
4. Click **Commit changes**.
5. After a minute or two, open https://nishadwankhede1.github.io/uae-dashboards/

Then add a link from the Projects section of your homepage, for example:

```html
<a href="uae-dashboards/">UAE industry dashboards: real estate, e-commerce, asset management and logistics</a>
```

## After you open the Power BI project

Save it as `.pbix`, take a screenshot of each page, and upload them to `uae-dashboards/power-bi/` as `overview.png` and `clients.png`. The Power BI page shows them automatically. If you get a Publish to web link, paste it into `POWER_BI_EMBED_URL` at the bottom of `power-bi.html` to show the live report.

## Rebuild the others in Power BI before interviews

Interviewers will ask how you built these, so rebuild at least the asset management one in Power BI from the CSVs. Add a date column to each table first:

```DAX
Date = DATEVALUE ( 'table'[month] & "-01" )
```

Then mark a proper Date table and relate it to each fact table.

**Real estate** (`real_estate_transactions.csv`)

```DAX
Sales Value = SUM ( tx[price_aed] )
Price per Sq Ft = DIVIDE ( [Sales Value], SUM ( tx[size_sqft] ) )
Off-plan Share = DIVIDE ( CALCULATE ( COUNTROWS ( tx ), tx[status] = "Off-plan" ), COUNTROWS ( tx ) )
Price per Sq Ft PY = CALCULATE ( [Price per Sq Ft], DATEADD ( 'Date'[Date], -12, MONTH ) )
```

**E-commerce** (`ecommerce_orders_monthly.csv`)

```DAX
Return Rate = DIVIDE ( SUM ( f[returns] ), SUM ( f[orders] ) )
COD Return Rate = DIVIDE ( SUM ( f[cod_returns] ), SUM ( f[cod_orders] ) )
Prepaid Return Rate = DIVIDE ( SUM ( f[returns] ) - SUM ( f[cod_returns] ), SUM ( f[orders] ) - SUM ( f[cod_orders] ) )
Avg Delivery Days = DIVIDE ( SUM ( f[delivery_days_total] ), SUM ( f[orders] ) )
```

**Asset management** (`fund_monthly.csv`, `fund_investors.csv`)

```DAX
AUM End = CALCULATE ( SUM ( m[aum_end_usd_m] ), LASTDATE ( m[Date] ) )
Net Flows = SUM ( m[net_flow_usd_m] )
Monthly Return = DIVIDE ( SUMX ( m, m[aum_start_usd_m] * m[return_pct] / 100 ), SUM ( m[aum_start_usd_m] ) )
Period Return = PRODUCTX ( VALUES ( m[Date] ), 1 + [Monthly Return] ) - 1
Holding Change = DIVIDE ( SUM ( inv[holding_sep_2026_usd_m] ), SUM ( inv[holding_sep_2025_usd_m] ) ) - 1
```

**Logistics** (`logistics_shipments_monthly.csv`)

```DAX
On-time Rate = DIVIDE ( SUM ( f[on_time] ), SUM ( f[shipments] ) )
Cost per Kg = DIVIDE ( SUM ( f[cost_aed] ), SUM ( f[weight_kg] ) )
Uplift if Customs Fixed = DIVIDE ( SUM ( f[late_customs_hold] ), SUM ( f[shipments] ) )
```

## About the data

All figures are synthetic, generated in Python to follow the shape of each market (Ramadan and White Friday peaks, the summer slowdown, rising off-plan sales, slower cross-border lanes). None of it is real company or government data, and every page says so in its footer. Keep that note if you edit the pages.
