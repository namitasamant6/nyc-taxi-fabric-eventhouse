# NYC Taxi Real-Time Analytics - Microsoft Fabric Eventhouse

![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Eventhouse-7B68EE?style=for-the-badge&logo=microsoft)
![KQL](https://img.shields.io/badge/KQL-Query%20Language-4FC4A0?style=for-the-badge)
![Power BI](https://img.shields.io/badge/Power%20BI-Real--Time%20Dashboard-F2C811?style=for-the-badge&logo=powerbi)
![Data](https://img.shields.io/badge/Dataset-2.9M%20Rows-E05A7A?style=for-the-badge)

An end-to-end data engineering project built on Microsoft Fabric. I ingested 2.9 million NYC Yellow Taxi trip records from Kaggle into an Eventhouse KQL database, wrote five analytics queries, and built a real-time dashboard.

---

## Project Overview

I built this project to get hands-on with Microsoft Fabric's real-time analytics stack, specifically Eventhouse and KQL. The dataset is NYC Yellow Taxi trip records for January 2024 messy, real-world data with some genuinely interesting patterns once you clean it up.

### Architecture

```
Kaggle Dataset → Eventhouse (KQL Database) → KQL Queryset → Real-Time Dashboard
```

### Results

| Metric | Value |
|--------|-------|
| Total trips analysed | 2,926,266 |
| Total revenue | $80,341,990 |
| Average fare | $18.66 |
| Average trip distance | 3.66 miles |
| Average tip | $3.38 |
| Query speed | under 1 second on 2.9M rows |

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Microsoft Fabric Eventhouse | Real-time analytics store (KQL Database) |
| KQL (Kusto Query Language) | Analytics queries |
| Fabric Real-Time Dashboard | Visualisation layer |
| Kaggle / NYC TLC | Data source |

---

## Repository Structure

```
├── README.md
├── kql-queries/
│   ├── 01_create_table.kql
│   ├── 02_verify_ingestion.kql
│   ├── 03_hourly_trip_volume.kql
│   ├── 04_top_zones_revenue.kql
│   ├── 05_payment_analysis.kql
│   ├── 06_day_of_week.kql
│   └── 07_kpi_summary.kql
├── screenshots/
│   ├── 01_workspace.png
│   ├── 02_eventhouse.png
│   ├── 03_kql_queryset.png
│   └── 04_dashboard.png
```

---

## Getting Started

### Prerequisites
- Microsoft Fabric account free 60-day trial at [app.fabric.microsoft.com](https://app.fabric.microsoft.com)
- The dataset: search "NYC Yellow Taxi Trip Records January 2024" by Muhammad Ibrahim Qasmi on Kaggle, or download directly from the [NYC TLC website](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

I used just the January 2024 file (~150MB). One month is enough to get meaningful results and keeps ingestion time reasonable.

---

## Setup Steps

### Step 1 — Create a Fabric Workspace
1. Go to [app.fabric.microsoft.com](https://app.fabric.microsoft.com)
2. Click **Workspaces → + New workspace**
3. Name it `NYC-Taxi-Analytics`
4. Set license mode to **Trial**
5. Click **Apply**

### Step 2 — Create the Eventhouse
1. Inside the workspace, click **+ New → Eventhouse**
2. Name it `TaxiEventhouse`
3. Click **Create** and wait about 60 seconds

### Step 3 — Ingest the Data
1. Open the KQL Database (TaxiEventhouse)
2. Click **Get data → Local file**
3. Upload the CSV or parquet file
4. Set the table name to `TaxiTrips`
5. The wizard auto-maps columns — click **Finish**

If you prefer to create the table manually first:

```kql
.create table TaxiTrips (
    tpep_pickup_datetime:  datetime,
    tpep_dropoff_datetime: datetime,
    passenger_count:       int,
    trip_distance:         real,
    PULocationID:          int,
    DOLocationID:          int,
    fare_amount:           real,
    extra:                 real,
    tip_amount:            real,
    tolls_amount:          real,
    total_amount:          real,
    payment_type:          int
)
```

### Step 4 — Verify the Ingestion
```kql
TaxiTrips
| count
```
Should return around 2,964,624. If it does, you're good to move on.

### Step 5 — Create a KQL Queryset
1. Workspace → **+ New → KQL Queryset**
2. Name it `TaxiAnalytics`
3. Connect it to the `TaxiEventhouse` database

### Step 6 — Run the Queries
All five queries are in the `kql-queries/` folder. Run each one and use **Save to Dashboard** to pin the results.

### Step 7 — Build the Dashboard
1. Run each query in the Queryset
2. Click **Save to Dashboard** and create a new dashboard called `NYC Taxi Analytics Dashboard`
3. For each subsequent query, save to the same existing dashboard
4. Change visual types from the default table view to line chart, bar chart, and pie chart as appropriate

---

## KQL Queries

### Query 1 — Hourly Trip Volume

Shows how demand varies across the day and month. The daily cycle is clear once you plot it as a line chart.

```kql
TaxiTrips
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
| where fare_amount > 0
| summarize
    trip_count = count(),
    avg_fare   = round(avg(fare_amount), 2),
    total_rev  = round(sum(total_amount), 0)
  by pickup_hour = bin(tpep_pickup_datetime, 1h)
| order by pickup_hour asc
```

### Query 2 — Top 15 Zones by Revenue

```kql
TaxiTrips
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
| where fare_amount > 0
| where isnotempty(PULocationID)
| summarize
    total_revenue = round(sum(total_amount), 0),
    trip_count    = count(),
    avg_fare      = round(avg(fare_amount), 2)
  by PULocationID
| top 15 by total_revenue
```

### Query 3 — Payment Method Breakdown

Note: `payment_type` was ingested as a string in this dataset, hence the quoted comparisons.

```kql
TaxiTrips
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
| where fare_amount > 0
| summarize
    trips       = count(),
    total_tips  = round(sum(tip_amount), 0),
    avg_tip_pct = round(avg(tip_amount / fare_amount) * 100, 1)
  by payment_label = case(
    payment_type == "1", "Credit card",
    payment_type == "2", "Cash",
    payment_type == "3", "No charge",
    payment_type == "4", "Dispute",
    "Other")
| order by trips desc
```

### Query 4 — Day of Week Patterns

```kql
TaxiTrips
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
| where fare_amount > 0
| extend day_num = dayofweek(tpep_pickup_datetime) / 1d
| extend day_name = case(
    day_num == 0, "Sunday",
    day_num == 1, "Monday",
    day_num == 2, "Tuesday",
    day_num == 3, "Wednesday",
    day_num == 4, "Thursday",
    day_num == 5, "Friday",
    "Saturday")
| summarize
    trips     = count(),
    avg_fare  = round(avg(fare_amount), 2),
    total_rev = round(sum(total_amount), 0)
  by day_name, day_num
| order by day_num asc
```

### Query 5 — KPI Summary

```kql
TaxiTrips
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
| where fare_amount > 0
| summarize
    total_trips      = count(),
    total_revenue    = round(sum(total_amount), 0),
    avg_fare         = round(avg(fare_amount), 2),
    avg_trip_miles   = round(avg(trip_distance), 2),
    avg_tip_amount   = round(avg(tip_amount), 2),
    total_passengers = sum(passenger_count)
```

---

## What I Found

**Zone 132 is JFK Airport** and it completely dominates the revenue chart — $11.3M in a single month, nearly double the second-place zone. Zone 138 (LaGuardia) comes in second. Airport trips skew the average fare significantly because of flat-rate surcharges and longer distances.

**Credit card vs cash is almost no contest** — 97.8% of trips were paid by card. Worth noting that cash tips are not recorded in the system, so the tip percentage figures only reflect card payments. The actual tipping rate is likely higher.

**Wednesday is consistently the busiest day** across the month. Sunday is the quietest. The pattern holds even when you factor in New Year's weekend at the start of the dataset.

**The 1am hour on January 1st** had the highest single-hour trip count in the entire month — the New Year's Eve surge. It stands out clearly on the line chart.

---

## Data Quality Notes

The raw dataset had some records with pickup timestamps from 2002, which is clearly wrong — a known issue with NYC TLC source data, likely caused by GPS or meter system errors. I added a date filter to all queries to handle this:

```kql
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
```

The `PULocationID` and `payment_type` columns were also ingested as strings rather than integers, which required small adjustments in the queries — using quoted comparisons for `payment_type` and `isnotempty()` instead of `> 0` for location IDs. This is common with real-world datasets and worth knowing before you start writing queries.

---

## Why Eventhouse Instead of a Data Warehouse

| | Eventhouse (KQL) | Traditional SQL DW |
|--|--|--|
| Query speed on 3M rows | under 1 second | 30–60 seconds |
| Time-series aggregation | `bin()` built-in | requires DATE_TRUNC or equivalent |
| Real-time ingestion | native | needs a separate pipeline |
| Setup time | minutes | hours |

The `bin()` function in KQL does in one line what takes a subquery or CTE in SQL. For time-series work specifically, it makes a noticeable difference.

One thing to be aware of: Power BI has a 500k row limit when querying Eventhouse directly. You need to pre-aggregate in KQL before sending results to Power BI — which is better practice anyway, since it keeps your visuals fast regardless of how large the underlying table grows.

---

## Resources

- [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
- [KQL Quick Reference](https://aka.ms/KQLguide)
- [SQL to KQL Cheat Sheet](https://aka.ms/sqlcheatsheet)
- [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- [Eventhouse Documentation](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse)

---

Author
[Namita Samant] Data Analyst | Data Engineer | Power BI | PySpark | Microsoft Fabric | DAX | SQL LinkedIn: [www.linkedin.com/in/namita-samant-2706b3129] GitHub: [https://github.com/namitasamant6]

---

If this helped you build something similar, a star would be appreciated.
