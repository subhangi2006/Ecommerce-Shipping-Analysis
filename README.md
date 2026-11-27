# E-Commerce Order Fulfillment & Shipping Performance Analysis

## Project overview

This project analyzes real e-commerce order and delivery records to measure order-to-door performance, on-time delivery, weekday/weekend outcomes, seasonal patterns, and geographic bottlenecks. The reproducible analysis is in [`notebooks/shipping_analysis.ipynb`](notebooks/shipping_analysis.ipynb); high-resolution figures are saved under [`outputs/charts/`](outputs/charts/).

## Business problem

Late deliveries can reduce customer satisfaction and increase support costs. Operations teams need to quantify service performance, understand when and where delays occur, and distinguish observed patterns from causes that cannot be established from the available data.

## Objectives

- Inspect and clean real order and customer records.
- Calculate delivery KPIs with explicit cohort and denominator definitions.
- Compare delivery outcomes by weekday/weekend and purchase-month cohort.
- Evaluate seasonal and November–December peak-season patterns without claiming a verified holiday effect.
- Compare state and Brazilian macroregion performance.
- Export clear, print-quality visualizations.
- Identify unavailable shipping mode, carrier, and warehouse fields rather than fabricate comparisons.

## Dataset description

The analysis uses two tables from the **Brazilian E-Commerce Public Dataset by Olist**:

- `data/raw/olist_orders_dataset.csv`: one order per row, including purchase, carrier-handoff, actual customer-delivery, and estimated-delivery timestamps and order status.
- `data/raw/olist_customers_dataset.csv`: customer identifiers, city, and Brazilian state (UF).

The selected CSVs were downloaded from the public mirror [spdrio/Brazilian-E-Commerce-Public-Dataset-by-Olist](https://github.com/spdrio/Brazilian-E-Commerce-Public-Dataset-by-Olist), commit `fa0cc6d1aff713d3c8b9a1e633b56889881848bf`. The original dataset is published by Olist on [Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce), which lists the license as **CC BY-NC-SA 4.0**. Attribute Olist, follow the license terms, and note the non-commercial restriction before reusing or redistributing the data.

The source describes approximately 100,000 anonymized orders from 2016–2018. The two tables loaded here each contain 99,441 rows. The order table has no shipping-mode, carrier-ID, or warehouse-ID field. The customer table provides state codes but no geographic boundaries or coordinates. Therefore, this project provides state and macroregion comparisons, not carrier/warehouse analysis or a choropleth.

## Technologies

- Python
- Pandas and NumPy
- Matplotlib and Seaborn
- Jupyter Notebook

## KPI definitions and assumptions

The analysis counts unique `order_id` values across all statuses for **Total orders**. Delivery performance uses a separate eligible cohort: rows with `order_status == delivered`, valid purchase, actual-delivery, and estimated-delivery timestamps, and a non-negative elapsed duration.

| KPI | Definition |
|---|---|
| Average delivery duration | Mean elapsed days from `order_purchase_timestamp` to `order_delivered_customer_date` among eligible delivered orders. |
| On-time delivery rate | Eligible orders whose actual delivery calendar date is on or before the estimated delivery calendar date, divided by eligible delivered orders. |
| Delayed delivery percentage | Eligible orders whose actual delivery calendar date is after the estimated delivery calendar date, divided by eligible delivered orders. |
| Average shipping delay | Mean positive calendar-day difference between actual and estimated delivery dates, calculated only among delayed eligible orders. |
| Total orders | Distinct order IDs across all order statuses. |
| Total on-time orders | Eligible delivered orders meeting the on-time date-level definition. |
| Total delayed orders | Eligible delivered orders arriving after the estimated delivery date. |

Estimated delivery values are date-only in practice, so on-time status compares calendar dates rather than treating an arrival later in the day as late. Rates exclude non-delivered orders and delivered rows with invalid required dates or negative duration. Counts for eligible and excluded orders are shown and reconciled. Purchase month is used for monthly cohorts; delivery weekday is used for weekday/weekend comparisons. November–December is a seasonal proxy only, not a dataset-provided holiday flag. Seasons follow Southern Hemisphere meteorological seasons.

## Key findings from the included data

The notebook was executed against the included CSVs. Results below are calculated from the eligible cohort of **96,470 delivered orders**:

- **99,441 total orders**; **2,971** are outside the eligible delivery-KPI cohort (2,963 are not marked delivered and 8 delivered rows lack valid required timing data or have an invalid duration).
- **12.56 days** average purchase-to-door duration.
- **89,936 on-time orders** and **6,534 delayed orders**; on-time rate **93.23%**, delayed rate **6.77%**.
- Average delay among late orders is **10.62 calendar days**.
- Weekend deliveries average **13.06 days** and are **89.58% on time**, versus **12.52 days** and **93.50% on time** for weekday deliveries. Weekend volume is smaller (6,669 vs 89,801).
- November–December purchase cohorts have **15.26 average delivery days** and **89.73% on time**, versus **12.14 days** and **93.76% on time** in other months. This is observational and is not proof that holidays caused the difference.
- The **Northeast** has the highest macroregion delay rate (**12.72%**, 9,044 orders); the **South** has the lowest (**5.90%**, 13,813 orders). The Northeast average duration is **20.01 days**. **Alagoas (AL)** has the highest state delay rate (**21.41%**, 397 orders); validate sample size and local operating context before treating it as a priority.
- Carrier, shipping-mode, and warehouse comparisons cannot be produced from the loaded tables. No regional cause is attributed without those operational fields.

Very small monthly cohorts occur at the beginning of the dataset window; extreme monthly percentages with tiny denominators should not be interpreted as stable performance. See the notebook tables for order counts alongside rates.

## Business recommendations

1. Investigate Northeast delivery lanes and handoff timelines first, using actual carrier/warehouse scans and lane-level service commitments to establish root cause.
2. Review weekend delivery coverage and handoff schedules. Compare adjusted cohorts before changing staffing, since weekend volume is much smaller and this dataset does not identify the responsible carrier.
3. Use November–December as a planning signal for capacity monitoring, but validate a holiday effect with a longer time series and a calendar of relevant sales events.
4. Track late-delivery rate and delay duration together: the 6.77% of eligible deliveries that were late averaged 10.62 calendar days beyond the estimate.
5. Add carrier IDs, shipping service/mode, warehouse or fulfillment-center IDs, and geographic coordinates/boundaries to future reporting so regional patterns can be diagnosed rather than merely observed.

## Project structure

```text
Ecommerce-Shipping-Analysis/
├── data/
│   └── raw/
│       ├── olist_customers_dataset.csv
│       └── olist_orders_dataset.csv
├── notebooks/
│   └── shipping_analysis.ipynb
├── outputs/
│   └── charts/
├── README.md
└── requirements.txt
```

## Generated charts

The notebook exports PNG files at **300 DPI** and an interactive HTML choropleth:

- `kpi_summary.png`
- `duration_and_delivery_outcomes.png`
- `weekday_weekend_performance.png`
- `monthly_delivery_trends.png`
- `seasonal_delivery_performance.png`
- `state_average_delivery_duration.png`
- `regional_delivery_delay.png`
- `state_delivery_choropleth.html` (interactive state map; generated by the final notebook section)

Shipping-mode and carrier charts are intentionally omitted because their fields are absent from these tables.

## Run the project

From the project directory, create and activate a virtual environment, install dependencies, and launch the notebook:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
jupyter notebook notebooks/shipping_analysis.ipynb
```

Run the notebook from top to bottom. The notebook resolves data and output paths relative to the project root or its `notebooks` folder, calculates findings from the included CSVs, validates reconciliations, recreates the PNG charts in `outputs/charts/`, and builds an interactive Brazil state-level delivery-duration map. The map downloads a public GeoJSON boundary file on first run, so an internet connection is required for that section.


## Geographic visualization note

The interactive choropleth colors Brazilian states by average purchase-to-door delivery duration and exposes order volume, delay rate, and on-time rate on hover. It uses public state boundaries from [click_that_hood](https://github.com/codeforgermany/click_that_hood). This is a state-level visualization, not a carrier route map. The source tables still do not identify individual carriers or warehouses, so those comparisons remain out of scope until such fields are added.
