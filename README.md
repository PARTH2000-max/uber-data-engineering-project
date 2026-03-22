## Uber End-to-End Data Engineering Project

Architecture](./architecture.png)

### 🏗️ Project Architecture Flow

```
Uber App (trips/drivers/riders)
         ↓
    REST API (FastAPI)
         ↓
   Azure Event Hub (Kafka)
         ↓
    ADF Pipeline (trigger)
         ↓
   Databricks Streaming
    ↓          ↓
Bronze      (raw JSON)
    ↓
Silver      (cleaned + schema)
    ↓
Gold        (KPIs + aggregations)
         ↓
    ADLS Gen2 (Delta Lake)
         ↓
    Power BI / Reporting
```

---

## README.md Content

```markdown
# 🚗 Uber End-to-End Data Engineering Project

## 📌 Overview
A real-time data engineering pipeline that ingests live Uber trip events,
processes them through Bronze → Silver → Gold layers using Delta Lake,
and exposes business KPIs for reporting.

---

## 🏗️ Architecture

| Layer | Tool | Purpose |
|---|---|---|
| Ingestion | FastAPI + Azure Event Hub | Capture real-time Uber trip events |
| Orchestration | Azure Data Factory | Trigger and monitor pipelines |
| Processing | Databricks + PySpark | Stream processing & transformation |
| Storage | ADLS Gen2 + Delta Lake | Store raw and processed data |
| Reporting | Power BI | Business KPIs and dashboards |

---

## 📁 Project Structure

```
uber-data-engineering-project/
│
├── api/                          ← FastAPI source code
│   ├── main.py                   ← API endpoints (trips, drivers, riders)
│   └── models.py                 ← Pydantic data models
│
├── adf/                          ← Azure Data Factory
│   ├── pipeline/                 ← ADF pipeline JSON
│   └── linked_services/          ← ADF linked service configs
│
├── databricks/                   ← Databricks Asset Bundle (DAB)
│   ├── databricks.yml            ← Bundle root config
│   ├── resources/
│   │   ├── cluster.yml           ← Cluster definition
│   │   ├── job.yml               ← Job with 2 tasks
│   │   └── pipeline.yml          ← DLT pipeline with 3 notebooks
│   ├── transformations/
│   │   ├── ingestion.py          ← Bronze: Kafka → Delta
│   │   ├── silver.py             ← Silver: clean + schema
│   │   ├── model.py              ← Data models
│   │   └── sql_dbt.sql           ← SQL transformations
│   └── src/
│       └── notebooks/
│           ├── 01_bronze_ingestion.ipynb
│           └── 02_silver_gold_transform.ipynb
│
└── README.md
```

---

## 🔄 Data Flow

### 1️⃣ Ingestion Layer
- **FastAPI** exposes REST endpoints to simulate Uber trip events
- Events are pushed to **Azure Event Hub** (Kafka-compatible)
- Each event contains: `trip_id`, `driver_id`, `rider_id`, `fare_amount`, `distance_km`, `status`

### 2️⃣ Orchestration Layer
- **Azure Data Factory** pipeline triggers Databricks notebooks
- ADF monitors job status and handles retries

### 3️⃣ Processing Layer — Bronze
- Raw JSON events consumed from Kafka
- Written as-is to **Bronze Delta table** on ADLS Gen2
- Metadata added: `ingestion_time`, `kafka_offset`, `kafka_partition`

### 4️⃣ Processing Layer — Silver
- Bronze data parsed using defined schema
- Cleaning: null removal, deduplication, type casting
- Filtered by valid trip statuses: `COMPLETED`, `CANCELLED`, `IN_PROGRESS`
- Written to **Silver Delta table**

### 5️⃣ Processing Layer — Gold
- Silver data aggregated by `driver_id`
- KPIs computed:
  - `total_trips`
  - `total_fare`
  - `avg_distance_km`
  - `cancellation_rate`
- Written to **Gold Delta table**

---

## ⚙️ Tech Stack

| Technology | Version | Purpose |
|---|---|---|
| Python | 3.9+ | API + PySpark code |
| FastAPI | Latest | REST API for event ingestion |
| Apache Kafka | 2.0+ | Real-time event streaming |
| Azure Event Hub | - | Managed Kafka service |
| Azure Data Factory | - | Pipeline orchestration |
| Databricks | 13.3 LTS | Spark processing |
| Delta Lake | 2.4.0 | ACID storage layer |
| ADLS Gen2 | - | Cloud data storage |
| PySpark | 3.4+ | Distributed processing |

---

## 🚀 How to Deploy

### Prerequisites
- Azure subscription
- Databricks workspace
- Azure Event Hub namespace
- ADLS Gen2 storage account
- GitHub account

### 1. Clone the repo
```bash
git clone https://github.com/PARTH2000-max/uber-data-engineering-project.git
cd uber-data-engineering-project
```

### 2. Deploy API
```bash
cd api
pip install -r requirements.txt
uvicorn main:app --reload
```

### 3. Deploy Databricks Bundle
```bash
cd databricks
pip install databricks-cli
databricks configure --token
databricks bundle validate
databricks bundle deploy --target dev
databricks bundle run uber_streaming_job --target dev
```

### 4. Import ADF Pipeline
- Open Azure Data Factory
- Import JSON from `adf/pipeline/` folder
- Update linked services with your credentials

---

## 📊 Output — Gold Layer KPIs

| Metric | Description |
|---|---|
| `total_trips` | Total trips completed per driver |
| `total_fare` | Total fare earned per driver |
| `avg_distance_km` | Average trip distance per driver |
| `cancellation_rate` | % of cancelled trips per driver |

---

## 👨‍💻 Author
**Parth** — Azure Data Engineer
- GitHub: [@PARTH2000-max](https://github.com/PARTH2000-max)
```
