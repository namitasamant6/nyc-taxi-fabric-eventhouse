# nyc-taxi-fabric-eventhouse
# 🚖 NYC Taxi Real-Time Analytics — Microsoft Fabric Eventhouse

![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Eventhouse-7B68EE?style=for-the-badge&logo=microsoft)
![KQL](https://img.shields.io/badge/KQL-Query%20Language-4FC4A0?style=for-the-badge)
![Power BI](https://img.shields.io/badge/Power%20BI-Real--Time%20Dashboard-F2C811?style=for-the-badge&logo=powerbi)
![Data](https://img.shields.io/badge/Dataset-2.9M%20Rows-E05A7A?style=for-the-badge)

> An end-to-end Data Engineering project built on Microsoft Fabric — from Kaggle data ingestion to a real-time analytics dashboard. Built in under 2 hours.

---

## 📊 Project Overview

This project demonstrates a complete real-time analytics pipeline using **Microsoft Fabric Eventhouse** on the NYC Yellow Taxi Trip dataset (January 2024, ~2.9 million rows).

### Architecture

```
Kaggle Dataset → Eventhouse (KQL Database) → KQL Queryset → Real-Time Dashboard
```

### Key Results
| Metric | Value |
|--------|-------|
| Total Trips Analysed | 2,926,266 |
| Total Revenue | $80,341,990 |
| Average Fare | $18.66 |
| Average Trip Distance | 3.66 miles |
| Average Tip | $3.38 |
| Query Speed | < 1 second on 2.9M rows |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| **Microsoft Fabric Eventhouse** | Real-time analytics store (KQL Database) |
| **KQL (Kusto Query Language)** | Analytics queries |
| **Fabric Real-Time Dashboard** | Visualisation layer |
| **Kaggle / NYC TLC** | Data source |

---

## 📁 Repository Structure

```
├── README.md                          # This file
├── kql-queries/
│   ├── 01_create_table.kql            # Table schema creation
│   ├── 02_verify_ingestion.kql        # Data verification queries
│   ├── 03_hourly_trip_volume.kql      # Hourly analytics
│   ├── 04_top_zones_revenue.kql       # Zone revenue analysis
│   ├── 05_payment_analysis.kql        # Payment method breakdown
│   ├── 06_day_of_week.kql             # Weekly patterns
│   └── 07_kpi_summary.kql            # KPI dashboard query
├── screenshots/
│   ├── 01_workspace.png               # Fabric workspace
│   ├── 02_eventhouse.png              # Eventhouse setup
│   ├── 03_kql_queryset.png            # KQL queries
│   └── 04_dashboard.png              # Final dashboard
└── docs/
    └── step-by-step-guide.md          # Detailed setup guide
```

---

## 🚀 Getting Started

### Prerequisites
- Microsoft Fabric account (free 60-day trial at [app.fabric.microsoft.com](https://app.fabric.microsoft.com))
- Kaggle account (free) or direct access to [NYC TLC data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)

### Dataset
- **Source:** NYC Yellow Taxi Trip Records — January 2024
- **Kaggle:** Search "NYC Yellow Taxi Trip Records January 2024" by Muhammad Ibrahim Qasmi
- **Direct:** Download `yellow_tripdata_2024-01.parquet` from NYC TLC website
- **Size:** ~150MB, ~2.9 million rows

---

## 📋 Step-by-Step Setup

### Step 1 — Create Fabric Workspace
1. Go to [app.fabric.microsoft.com](https://app.fabric.microsoft.com)
2. Click **Workspaces → + New workspace**
3. Name: `NYC-Taxi-Analytics`
4. License mode: **Trial**
5. Click **Apply**

### Step 2 — Create Eventhouse
1. Inside workspace → **+ New → Eventhouse**
2. Name: `TaxiEventhouse`
3. Click **Create** (wait ~60 seconds)

### Step 3 — Create KQL Table & Ingest Data
1. Open the KQL Database (TaxiEventhouse)
2. Click **Get data → Local file**
3. Upload your downloaded CSV/parquet file
4. New table name: `TaxiTrips`
5. Let the wizard auto-map columns
6. Click **Finish** and wait for ingestion (~10 min)

**Or create manually using KQL:**
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

### Step 4 — Verify Ingestion
```kql
// Should return ~2,964,624
TaxiTrips
| count
```

### Step 5 — Create KQL Queryset
1. Workspace → **+ New → KQL Queryset**
2. Name: `TaxiAnalytics`
3. Connect to `TaxiEventhouse` database

### Step 6 — Run Analytics Queries
See the `kql-queries/` folder for all queries.

### Step 7 — Build Real-Time Dashboard
1. In KQL Queryset, run each query
2. Click **"Save to Dashboard"** for each query result
3. Name the dashboard: `NYC Taxi Analytics Dashboard`
4. Change visual types (Table → Line chart, Bar chart, Pie chart)

---

## 🔍 KQL Queries

### Query 1 — Hourly Trip Volume
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

### Query 3 — Payment Method Analysis
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

## 📈 Key Insights Discovered

- 🏆 **Zone 132 (JFK Airport)** is the #1 revenue zone — $11.3M in one month alone
- ✈️ **Airport zones** have the highest avg fare ($42–62) due to long-distance trips
- 💳 **97.8% of trips** paid by credit card — cash tips are invisible in the data
- 📅 **Wednesday** is the busiest day (489k trips), **Sunday** is the quietest (334k)
- 🌙 **1am on Jan 1st** had the highest single-hour volume — New Year's Eve surge
- ⏰ **Early morning (6–8am)** has highest avg fare — airport runs before rush hour

---

## 💡 Why Eventhouse Over a Data Warehouse?

| Feature | Eventhouse (KQL) | Traditional SQL DW |
|---------|-----------------|-------------------|
| Query speed on 3M rows | < 1 second | 30–60 seconds |
| Time-series functions | Native (bin, ago) | Complex workarounds |
| Real-time ingestion | Built-in | Requires ETL pipeline |
| Setup time | Minutes | Hours |
| Cost for this project | Free trial | Paid compute |

---

## 🧹 Data Quality Notes

The raw dataset contained some records with incorrect timestamps (dates from 2002) — a known issue with NYC TLC source data caused by GPS/meter system errors. All queries include a date filter to exclude these:

```kql
| where tpep_pickup_datetime >= datetime(2024-01-01)
    and tpep_pickup_datetime < datetime(2024-02-01)
```

This reduced the dataset from 2,964,624 total records to ~2,926,266 valid January 2024 records.

---

## 📸 Screenshots

| Screenshot | Description |
|-----------|-------------|
| `screenshots/01_workspace.png` | Fabric workspace with all items |
| `screenshots/02_eventhouse.png` | Eventhouse with TaxiTrips table |
| `screenshots/03_kql_queryset.png` | KQL Queryset with all 5 query tabs |
| `screenshots/04_dashboard.png` | Final real-time dashboard |

---

## 🔗 Resources

- [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
- [KQL Quick Reference](https://aka.ms/KQLguide)
- [SQL to KQL Cheat Sheet](https://aka.ms/sqlcheatsheet)
- [NYC TLC Trip Record Data](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page)
- [Eventhouse Documentation](https://learn.microsoft.com/en-us/fabric/real-time-intelligence/eventhouse)

---

## 👤 Author

Built as a portfolio project to demonstrate Microsoft Fabric data engineering skills.

**Connect with me on LinkedIn:** [Your LinkedIn URL]

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

⭐ **If this project helped you, please give it a star!**
