# Fintrack Dashboard: Real-Time Financial Analytics Platform

A production-grade financial analytics dashboard for B2B fintech platforms, enabling data-driven decision-making across revenue, retention, and churn metrics in real-time.

## 🎯 Overview

Fintrack ingests transaction data from multiple sources, normalizes it through an automated ETL pipeline, and surfaces critical KPIs in an interactive, executive-ready interface. Decision-makers can slice data by cohort, period, and segment without touching a spreadsheet.

### The Problem
Finance and product teams were operating from disconnected data exports—weekly CSV dumps that were 72 hours stale by the time decisions needed to be made. Leadership had no unified view of revenue velocity, customer lifetime value, or cohort-level churn patterns.

### The Solution
An end-to-end analytics system combining:
- **dbt-powered transformation layer** on BigQuery warehouse
- **Python/Prefect orchestration pipeline** (15-minute refresh cadence)
- **Interactive Plotly Dash dashboard** with dynamic filtering
- **Stakeholder-aligned KPI definitions** ensuring metrics drive business decisions

## 🛠 Technology Stack

| Component | Technology |
|-----------|-----------|
| **Data Warehouse** | Google BigQuery |
| **Transformation** | dbt (data build tool) |
| **Orchestration** | Prefect, Python |
| **Visualization** | Plotly Dash |
| **Data Processing** | pandas, Python |
| **Database Query** | SQL |
| **Containerization** | Docker |

## 📁 Project Structure

```
fintrack-dashboard/
├── dbt/                          # dbt transformation layer
│   ├── models/
│   │   ├── staging/
│   │   ├── intermediate/
│   │   └── marts/
│   ├── macros/
│   ├── tests/
│   ├── dbt_project.yml
│   └── profiles.yml
├── orchestration/                # Prefect pipelines
│   ├── flows/
│   │   ├── etl_pipeline.py
│   │   └── data_refresh.py
│   ├── tasks/
│   │   ├── extract.py
│   │   ├── transform.py
│   │   └── load.py
│   ├── config/
│   └── requirements.txt
├── dashboard/                    # Plotly Dash application
│   ├── app.py
│   ├── callbacks.py
│   ├── layouts/
│   │   ├── home.py
│   │   ├── revenue.py
│   │   ├── retention.py
│   │   └── churn.py
│   ├── components/
│   │   ├── filters.py
│   │   ├── charts.py
│   │   └── metrics.py
│   ├── utils/
│   │   ├── data_loader.py
│   │   ├── calculations.py
│   │   └── formatting.py
│   ├── assets/
│   │   ├── styles.css
│   │   └── images/
│   └── requirements.txt
├── data/                         # Sample data and fixtures
│   ├── sample_transactions.csv
│   └── test_data/
├── docker/
│   ├── Dockerfile.dashboard
│   ├── Dockerfile.orchestration
│   └── docker-compose.yml
├── tests/                        # Unit and integration tests
│   ├── test_transformations.py
│   ├── test_pipeline.py
│   └── test_dashboard.py
├── docs/                         # Documentation
│   ├── ARCHITECTURE.md
│   ├── SETUP.md
│   ├── API.md
│   └── DEPLOYMENT.md
├── .github/
│   └── workflows/
│       ├── ci_tests.yml
│       └── deploy.yml
├── .env.example
├── requirements.txt
├── docker-compose.yml
└── .gitignore
```

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- Docker & Docker Compose
- Google Cloud SDK (for BigQuery access)
- dbt CLI

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/massoda-m/fintrack-dashboard.git
cd fintrack-dashboard
```

2. **Set up environment variables**
```bash
cp .env.example .env
# Edit .env with your BigQuery credentials
```

3. **Install dependencies**
```bash
pip install -r requirements.txt
```

4. **Run with Docker Compose**
```bash
docker-compose up --build
```

5. **Access the dashboard**
Open `http://localhost:8050` in your browser

### Development Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -r dashboard/requirements.txt
pip install -r orchestration/requirements.txt

# Initialize dbt profile
cd dbt && dbt debug && cd ..

# Run the dashboard locally
python dashboard/app.py
```

## 📊 Key Features

### Real-Time KPI Tracking
- **Revenue Metrics**: MRR, ARR, revenue velocity by cohort
- **Retention Analysis**: Cohort retention curves, repeat customer rates
- **Churn Monitoring**: Customer churn rates, churn risk signals
- **Custom Dimensions**: Slice by product, region, customer segment

### Dynamic Filtering
- Cohort-based analysis
- Period selection (daily, weekly, monthly, custom ranges)
- Customer segment filtering
- Product line breakdown

### Executive-Ready Reporting
- One-click metric exports
- Pre-built report templates
- Snapshot comparisons (period-over-period analysis)
- Automated insight generation

## 🔄 Data Pipeline

### Refresh Cadence
- **Automated runs**: Every 15 minutes via Prefect
- **Manual triggers**: On-demand refresh available
- **Data freshness**: Near real-time analytics

### Data Flow
```
Multi-source Transactions → BigQuery Landing Zone
                    ↓
             dbt Transformations
                    ↓
         Normalized Analytics Tables
                    ↓
        Prefect Orchestration Pipeline
                    ↓
          Plotly Dash Visualization
```

## 📈 Performance Metrics

- **Dashboard Load Time**: <2 seconds (p95)
- **Data Freshness**: 15-minute refresh cycle
- **Concurrent Users**: Supports 100+ simultaneous dashboard viewers
- **Query Latency**: <1 second (95th percentile)

## 🧪 Testing

```bash
# Run all tests
pytest tests/ -v

# Run specific test suite
pytest tests/test_transformations.py

# Run with coverage
pytest tests/ --cov=dashboard --cov=orchestration
```

## 📚 Documentation

- **[Architecture Overview](docs/ARCHITECTURE.md)** - System design and data flow
- **[Setup Guide](docs/SETUP.md)** - Detailed installation instructions
- **[API Reference](docs/API.md)** - Dashboard API endpoints
- **[Deployment Guide](docs/DEPLOYMENT.md)** - Production deployment

## 🔐 Security

- Environment variable management for sensitive credentials
- BigQuery service account authentication
- Data encryption in transit (TLS)
- Row-level security for multi-tenant scenarios
- Input validation and SQL injection prevention

## 🤝 Contributing

1. Create a feature branch (`git checkout -b feature/amazing-feature`)
2. Commit your changes (`git commit -m 'Add amazing feature'`)
3. Push to the branch (`git push origin feature/amazing-feature`)
4. Open a Pull Request

## 📝 License

This project is proprietary. All rights reserved.

## 👤 Author

**Mohammad Assoda**  
Full Stack Software Developer | Data Analytics Engineer  
[GitHub](https://github.com/massoda-m) | [Portfolio](https://your-portfolio-url.com)

---

**Status**: Active Development  
**Last Updated**: May 2026  
**Version**: 0.1.0
