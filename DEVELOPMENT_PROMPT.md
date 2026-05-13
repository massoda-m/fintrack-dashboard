# Fintrack Dashboard - Development Prompt for Claude Agent

You are an expert full-stack data engineer and software developer tasked with building a production-grade financial analytics dashboard called **Fintrack** for a B2B fintech platform.

## Project Context

**Repository**: `massoda-m/fintrack-dashboard`
**Tech Stack**: Python, dbt, BigQuery, Prefect, Plotly Dash, Docker, pandas, SQL
**Timeline**: End-to-end development from initialization to deployment and testing
**Business Goal**: Enable executives to make data-driven decisions without touching spreadsheets

## Phase 1: Project Initialization & Structure

### 1.1 Core Configuration Files
- [ ] Create `requirements.txt` (root level) with core dependencies:
  - plotly-dash==2.14.0
  - prefect==2.10.0
  - dbt-core==1.5.0
  - dbt-bigquery==1.5.0
  - pandas==2.0.0
  - google-cloud-bigquery==3.11.0
  - python-dotenv==1.0.0
  - pytest==7.3.0
  - docker and docker-compose

- [ ] Create `.env.example` with all required environment variables:
  ```
  # Google Cloud
  GCP_PROJECT_ID=your-project-id
  GCP_SERVICE_ACCOUNT_KEY=path/to/service-account-key.json
  
  # BigQuery
  BIGQUERY_DATASET=fintrack_analytics
  BIGQUERY_STAGING_DATASET=fintrack_staging
  
  # Prefect
  PREFECT_API_URL=http://localhost:4200/api
  PREFECT_API_KEY=your-key
  
  # Dash
  DASH_PORT=8050
  DASH_DEBUG=True
  
  # Data Refresh
  REFRESH_INTERVAL_MINUTES=15
  ```

- [ ] Update `.gitignore` (already created) to exclude:
  - Environment files, credentials, logs
  - Python cache, virtual environments
  - dbt artifacts, Prefect cache
  - Docker artifacts, test coverage

### 1.2 Directory Structure Creation
```
fintrack-dashboard/
├── dbt/
├── orchestration/
├── dashboard/
├── tests/
├── docs/
├── docker/
├── data/
└── .github/workflows/
```

---

## Phase 2: dbt Transformation Layer

### 2.1 dbt Project Setup
- [ ] Create `dbt/dbt_project.yml` with:
  - Project name: `fintrack`
  - Profile: `fintrack`
  - Model paths and macro paths
  - Testing and docs configuration
  - Seeds, analysis, snapshots setup

- [ ] Create `dbt/profiles.yml` (template in .env.example):
  - BigQuery adapter configuration
  - Service account authentication
  - Dataset references

### 2.2 Staging Models (`dbt/models/staging/`)
Build source-agnostic staging layer from multiple data sources:

- [ ] `stg_transactions.sql`
  - Clean and normalize transaction records
  - Handle null values, data type conversions
  - Add row hashes for deduplication
  - Sources: Stripe, PayPal, custom bank feeds

- [ ] `stg_customers.sql`
  - Customer master data normalization
  - Standardize customer attributes
  - Calculate customer age, tenure

- [ ] `stg_products.sql`
  - Product catalog standardization
  - Map product IDs across sources
  - Add product categories

- [ ] `stg_calendar.sql`
  - Date spine for time-series analysis
  - Fiscal periods, quarters, years
  - Business day indicators

### 2.3 Intermediate Models (`dbt/models/intermediate/`)
Transform staging data into business-logic-ready tables:

- [ ] `int_daily_revenue.sql`
  - Aggregate daily revenue by product, customer segment
  - Calculate transaction counts, avg order value
  - Cumulative revenue metrics

- [ ] `int_customer_cohorts.sql`
  - Assign customers to monthly cohorts (first purchase)
  - Calculate cohort size and attributes
  - Link to customer dimension

- [ ] `int_customer_activity.sql`
  - Monthly/quarterly activity window for each customer
  - Count transactions, sum revenue per period
  - Identify active vs. inactive customers

- [ ] `int_churn_signals.sql`
  - Detect customers who haven't transacted in 90 days
  - Calculate churn risk scores
  - Flag cohorts with declining activity

### 2.4 Marts (Analytics Tables) (`dbt/models/marts/`)
Business-facing dimensional models for dashboards:

- [ ] `fct_revenue_metrics.sql` (Fact table)
  - Grain: Date × Customer × Product
  - Metrics: Revenue, transaction count, AVG, LTV
  - Dimensions: Cohort, segment, region

- [ ] `fct_retention_analysis.sql` (Fact table)
  - Grain: Cohort × Month
  - Metrics: Cohort size, retention rates by month
  - Retention curves for visualization

- [ ] `fct_churn_tracking.sql` (Fact table)
  - Grain: Customer × Period
  - Metrics: Churn status, risk score, last transaction
  - Historical churn snapshots

- [ ] `dim_customers.sql` (Dimension)
  - Customer key, attributes, segments
  - Lifetime metrics (calculated at load time)

- [ ] `dim_products.sql` (Dimension)
  - Product key, category, pricing tier

- [ ] `dim_date.sql` (Dimension)
  - Date key, fiscal calendar, business flags

### 2.5 dbt Testing & Validation
- [ ] Create `dbt/tests/` directory with:
  - Generic tests: not_null, unique, relationships
  - Custom tests for business logic (e.g., retention sums to 100%)
  - Negative tests for data quality

- [ ] Add tests to YAML configs:
  ```yaml
  - name: fct_revenue_metrics
    columns:
      - name: revenue
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"
  ```

- [ ] Create dbt docs (`dbt/models/models.yml`) with descriptions and lineage

### 2.6 dbt Seeds & Sample Data
- [ ] Create `dbt/seeds/` with reference tables (if needed):
  - Customer segments mapping
  - Product categories
- [ ] Create sample data for local testing in `data/sample_transactions.csv`

---

## Phase 3: Data Orchestration Pipeline (Prefect)

### 3.1 Flow Structure (`orchestration/`)
- [ ] Create `orchestration/config/` directory:
  - `config.py` - Config management (environment, paths)
  - `logging.py` - Structured logging setup

- [ ] Directory structure:
  ```
  orchestration/
  ├── flows/
  │   ├── etl_pipeline.py (main orchestrator)
  │   ├── data_refresh.py (scheduled flow)
  │   └── backfill.py (historical data fill)
  ├── tasks/
  │   ├── extract.py (data extraction)
  │   ├── transform.py (dbt execution)
  │   └── load.py (BigQuery loading)
  ├── config/
  │   ├── config.py
  │   └── logging.py
  └── requirements.txt
  ```

### 3.2 Extract Tasks (`orchestration/tasks/extract.py`)
- [ ] Implement extractors for multiple data sources:
  ```python
  @task(name="extract_stripe_transactions")
  def extract_stripe(api_key: str, days_back: int = 1):
      # Fetch transactions from Stripe API
      # Handle pagination, rate limits
      # Return: pandas DataFrame
  
  @task(name="extract_salesforce_customers")
  def extract_salesforce(instance_url: str, client_id: str):
      # Query Salesforce API for customer data
      # Handle authentication, pagination
      # Return: pandas DataFrame
  
  @task(name="extract_bigquery_staging")
  def extract_staging_tables():
      # Query existing BigQuery staging tables
      # Validate row counts
      # Return: list of DataFrames
  
  @task(name="validate_extraction")
  def validate_extracted_data(df: pd.DataFrame, table_name: str):
      # Check for null counts, duplicates
      # Validate schema
      # Log warnings if anomalies detected
  ```

### 3.3 Transform Tasks (`orchestration/tasks/transform.py`)
- [ ] Orchestrate dbt execution:
  ```python
  @task(name="run_dbt_staging")
  def run_dbt_staging():
      # Execute: dbt run --select staging
      # Log output, capture warnings
      # Return: dbt run summary
  
  @task(name="run_dbt_tests")
  def run_dbt_tests():
      # Execute: dbt test
      # Fail if critical tests fail
      # Parse and log results
  
  @task(name="run_dbt_intermediate")
  def run_dbt_intermediate():
      # Execute: dbt run --select intermediate
      # Validate all dependencies ran
  
  @task(name="run_dbt_marts")
  def run_dbt_marts():
      # Execute: dbt run --select marts
      # Generate documentation
  ```

### 3.4 Load Tasks (`orchestration/tasks/load.py`)
- [ ] Implement BigQuery loading:
  ```python
  @task(name="upload_staging_to_bigquery")
  def upload_to_bigquery(df: pd.DataFrame, table_id: str):
      # Load DataFrame to BigQuery
      # Use write_disposition: WRITE_TRUNCATE for staging
      # Validate row counts match
      # Return: load job metadata
  
  @task(name="validate_marts_loaded")
  def validate_marts():
      # Query all marts tables
      # Verify row counts > 0
      # Check for nulls in key columns
  ```

### 3.5 Main Orchestration Flows
- [ ] Create `orchestration/flows/etl_pipeline.py`:
  ```python
  @flow(name="etl_main_pipeline", version="1.0")
  def main_etl_flow():
      # 1. EXTRACT: Parallel extraction from all sources
      stripe_data = extract_stripe()
      sf_customers = extract_salesforce()
      
      # 2. VALIDATE: Check extracted data
      validate_extracted_data.map(stripe_data, ["stripe"])
      
      # 3. TRANSFORM: Sequential dbt execution
      run_dbt_staging()
      run_dbt_tests()
      run_dbt_intermediate()
      run_dbt_marts()
      
      # 4. VALIDATE: Check mart tables
      validate_marts()
  ```

- [ ] Create `orchestration/flows/data_refresh.py`:
  ```python
  @flow(name="scheduled_refresh", version="1.0")
  def scheduled_refresh_flow():
      # 15-minute refresh cycle
      # Incremental updates only (last 24 hours)
      # Lightweight validation
      return main_etl_flow()
  
  # Register schedule: Every 15 minutes
  # Deployment configuration
  ```

### 3.6 Error Handling & Retries
- [ ] Add retry logic to all tasks:
  ```python
  @task(
      name="task_with_retry",
      retries=3,
      retry_delay_seconds=60,
      retry_jitter_factor=0.1
  )
  def resilient_task():
      pass
  ```

- [ ] Implement error notifications:
  - Slack integration on flow failure
  - Email alerts for critical errors
  - Log all errors to structured logging

### 3.7 Orchestration Requirements
- [ ] Create `orchestration/requirements.txt`:
  ```
  prefect==2.10.0
  prefect-gcp==0.2.0
  google-cloud-bigquery==3.11.0
  pandas==2.0.0
  stripe==5.4.0
  simple-salesforce==1.12.0
  python-dotenv==1.0.0
  pydantic==1.10.0
  ```

---

## Phase 4: Plotly Dash Dashboard Application

### 4.1 Dashboard Core (`dashboard/app.py`)
- [ ] Create main Dash application:
  ```python
  import dash
  from dash import dcc, html
  import plotly.graph_objects as go
  
  app = dash.Dash(__name__)
  
  # App layout structure:
  # - Header with company branding
  # - Navigation sidebar (Revenue, Retention, Churn)
  # - Filter section (date range, cohort, segment)
  # - KPI cards (MRR, ARR, retention rate, churn rate)
  # - Charts and detailed tables
  # - Last refresh timestamp
  
  app.layout = html.Div([
      # Header
      # Sidebar navigation
      # Main content area (page-dependent)
  ])
  
  if __name__ == '__main__':
      app.run_server(debug=True, port=8050)
  ```

### 4.2 Layout Components (`dashboard/layouts/`)

#### 4.2.1 Home Page (`home.py`)
- [ ] Executive dashboard landing page:
  - 4 KPI cards: MRR, ARR, Retention Rate (MTD), Churn Rate (MTD)
  - 30-day revenue trend chart (line chart)
  - Cohort retention heatmap
  - Top 10 customers by revenue
  - Key metrics summary table

#### 4.2.2 Revenue Page (`revenue.py`)
- [ ] Revenue analytics:
  - Daily revenue trend (last 90 days, interactive)
  - Revenue by product (stacked bar chart)
  - Revenue by customer segment (pie chart)
  - Customer LTV distribution (histogram)
  - Cohort revenue curves
  - Export to CSV functionality

#### 4.2.3 Retention Page (`retention.py`)
- [ ] Retention analysis:
  - Cohort retention heatmap (months 0-12)
  - Cohort size distribution
  - Month-over-month retention rate chart
  - Survival curve analysis
  - Segment comparison table

#### 4.2.4 Churn Page (`churn.py`)
- [ ] Churn monitoring:
  - Monthly churn rate trend
  - Churn by segment (bar chart)
  - Churn risk cohort highlighting
  - Customer at-risk list (with risk scores)
  - Churn prediction over next 30/60/90 days

### 4.3 Filter Components (`dashboard/components/filters.py`)
- [ ] Build reusable filter components:
  ```python
  def create_date_range_picker():
      # dcc.DatePickerRange component
      # Quick select buttons: Last 30, 90, 365 days
      
  def create_cohort_filter():
      # dcc.Dropdown with all cohorts
      # Multi-select enabled
      
  def create_segment_filter():
      # dcc.Checklist with customer segments
      
  def create_apply_button():
      # Button to trigger data updates
  ```

### 4.4 Chart Components (`dashboard/components/charts.py`)
- [ ] Create reusable chart factories:
  ```python
  def create_line_chart(df, x, y, title):
      # Interactive Plotly line chart
      # With hover info, zoom, pan
      
  def create_bar_chart(df, x, y, title):
      # Bar chart with annotations
      
  def create_heatmap(df, title):
      # Cohort retention heatmap
      
  def create_pie_chart(df, values, names, title):
      # Segment distribution pie
      
  def create_metric_card(value, title, subtitle):
      # KPI card with formatting
  ```

### 4.5 Callbacks (`dashboard/callbacks.py`)
- [ ] Implement interactive callbacks:
  ```python
  @app.callback(
      Output('revenue-chart', 'figure'),
      [Input('date-picker', 'start_date'),
       Input('date-picker', 'end_date'),
       Input('segment-filter', 'value')]
  )
  def update_revenue_chart(start_date, end_date, segments):
      # Query BigQuery
      # Filter and aggregate
      # Return updated chart
  ```

### 4.6 Data Loaders (`dashboard/utils/data_loader.py`)
- [ ] Create BigQuery data access layer:
  ```python
  class BigQueryClient:
      def __init__(self, project_id, credentials):
          self.client = bigquery.Client(...)
      
      def get_revenue_metrics(self, start_date, end_date, filters=None):
          # Query fct_revenue_metrics
          # Apply filters (cohort, segment)
          # Return cached DataFrame (TTL: 5 min)
      
      def get_retention_cohorts(self, start_cohort, end_cohort):
          # Query fct_retention_analysis
          # Pivot for heatmap format
          
      def get_churn_data(self, days_lookback=90):
          # Query fct_churn_tracking
          # Calculate risk scores
  ```

### 4.7 Utility Functions (`dashboard/utils/`)
- [ ] `calculations.py`: Business logic functions
  ```python
  def calculate_mrr(revenue_df):
      """Calculate Monthly Recurring Revenue"""
      
  def calculate_arr(revenue_df):
      """Calculate Annual Recurring Revenue"""
      
  def calculate_cohort_retention(cohort_df):
      """Transform cohort data to retention matrix"""
      
  def calculate_churn_rate(active_current, active_previous):
      """Calculate churn as (lost / previous)"""
  ```

- [ ] `formatting.py`: Display formatting
  ```python
  def format_currency(value):
      """Format as $XXX,XXX.XX"""
      
  def format_percentage(value):
      """Format as XX.X%"""
      
  def format_date(date):
      """Format date consistently"""
  ```

### 4.8 Styling (`dashboard/assets/styles.css`)
- [ ] Create professional styling:
  - Color scheme: Blue (primary), green (positive), red (negative)
  - Font: System fonts (Segoe UI, -apple-system)
  - Layout: Flexbox, responsive grid
  - Cards, tables, filters with Bootstrap/custom styles
  - Dark mode support

### 4.9 Dashboard Requirements
- [ ] Create `dashboard/requirements.txt`:
  ```
  plotly==5.14.0
  dash==2.14.0
  dash-bootstrap-components==1.4.0
  google-cloud-bigquery==3.11.0
  pandas==2.0.0
  python-dotenv==1.0.0
  gunicorn==20.1.0
  ```

---

## Phase 5: Docker Configuration

### 5.1 Dashboard Dockerfile (`docker/Dockerfile.dashboard`)
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY dashboard/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY dashboard/ ./dashboard/
COPY .env.example .env

EXPOSE 8050

CMD ["gunicorn", "--bind", "0.0.0.0:8050", "dashboard.app:server"]
```

### 5.2 Orchestration Dockerfile (`docker/Dockerfile.orchestration`)
```dockerfile
FROM python:3.9-slim

WORKDIR /app

COPY orchestration/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY orchestration/ ./orchestration/
COPY dbt/ ./dbt/
COPY .env.example .env

CMD ["prefect", "agent", "start", "-q", "default"]
```

### 5.3 Docker Compose (`docker-compose.yml`)
- [ ] Multi-container orchestration:
  ```yaml
  version: '3.8'
  services:
    dashboard:
      build:
        context: .
        dockerfile: docker/Dockerfile.dashboard
      ports:
        - "8050:8050"
      environment:
        - DASH_DEBUG=True
        - GCP_PROJECT_ID=${GCP_PROJECT_ID}
      volumes:
        - ./dashboard:/app/dashboard
    
    orchestration:
      build:
        context: .
        dockerfile: docker/Dockerfile.orchestration
      environment:
        - GCP_PROJECT_ID=${GCP_PROJECT_ID}
      volumes:
        - ./dbt:/app/dbt
        - ./orchestration:/app/orchestration
    
    prefect:
      image: prefecthq/prefect:2.10.0
      ports:
        - "4200:4200"
      command: prefect server start
  ```

---

## Phase 6: Testing Suite

### 6.1 Unit Tests (`tests/test_dashboard.py`)
- [ ] Test dashboard components and callbacks:
  ```python
  def test_kpi_card_renders():
      """Test metric card component"""
      
  def test_revenue_chart_callback():
      """Test revenue chart update callback"""
      
  def test_filter_interactions():
      """Test filter dropdown interactions"""
      
  def test_data_loader_bigquery_query():
      """Mock BigQuery, test query execution"""
      
  def test_calculations_mrr():
      """Test MRR calculation logic"""
      
  def test_calculations_retention_matrix():
      """Test cohort retention transformation"""
  ```

### 6.2 Integration Tests (`tests/test_pipeline.py`)
- [ ] Test end-to-end ETL flow:
  ```python
  @pytest.fixture
  def sample_transaction_data():
      """Provide sample transaction data"""
      
  def test_extract_task():
      """Test data extraction"""
      
  def test_transform_task():
      """Test dbt transformation execution"""
      
  def test_load_task():
      """Test BigQuery loading"""
      
  def test_full_pipeline_execution(sample_transaction_data):
      """Test complete ETL pipeline"""
      
  def test_pipeline_error_handling():
      """Test error handling and retries"""
  ```

### 6.3 dbt Tests (`dbt/tests/`)
- [ ] Add dbt-specific tests:
  ```yaml
  - name: fct_revenue_metrics
    tests:
      - dbt_utils.equality:
          compare_model: ref('fct_revenue_metrics_expected')
  ```

### 6.4 Data Quality Tests (`tests/test_data_quality.py`)
- [ ] Validate output data:
  ```python
  def test_revenue_metrics_no_nulls():
      """Check no nulls in key columns"""
      
  def test_retention_sums_to_100():
      """Validate cohort retention rates"""
      
  def test_no_future_dates():
      """Ensure no transactions dated in future"""
      
  def test_unique_customer_keys():
      """Check for duplicate customer records"""
  ```

### 6.5 Test Configuration (`pytest.ini`)
```ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short --cov=dashboard --cov=orchestration
```

### 6.6 Test Data Setup (`tests/conftest.py`)
```python
import pytest
import pandas as pd

@pytest.fixture
def sample_transactions():
    return pd.DataFrame({
        'transaction_id': [1, 2, 3],
        'customer_id': [101, 102, 101],
        'revenue': [100, 200, 150],
        'date': pd.date_range('2025-01-01', periods=3)
    })

@pytest.fixture
def bigquery_client_mock(monkeypatch):
    # Mock BigQuery client for testing
    pass
```

---

## Phase 7: Documentation

### 7.1 Architecture Documentation (`docs/ARCHITECTURE.md`)
- [ ] Create comprehensive architecture guide:
  - System diagram (data flow: Sources → ETL → Warehouse → Dashboard)
  - Component descriptions (dbt, Prefect, Dash)
  - Data model (star schema, dimensions, facts)
  - Metrics definitions (MRR, ARR, retention, churn)
  - Deployment architecture (Docker, Cloud Run, BigQuery)

### 7.2 Setup Guide (`docs/SETUP.md`)
- [ ] Detailed local development setup:
  - Prerequisites (Python 3.9+, Docker, GCP SDK)
  - Google Cloud project setup
  - BigQuery dataset and schema creation
  - dbt initialization and profiles
  - Prefect agent setup
  - Dash local development
  - Running with Docker Compose

### 7.3 API Documentation (`docs/API.md`)
- [ ] Document dashboard endpoints and data sources:
  - Available BigQuery tables
  - Filter parameters
  - Response formats
  - Rate limiting

### 7.4 Deployment Guide (`docs/DEPLOYMENT.md`)
- [ ] Production deployment instructions:
  - Containerization best practices
  - Cloud Run deployment
  - Prefect Cloud setup
  - Secret management
  - Monitoring and alerting
  - Scaling considerations

---

## Phase 8: GitHub Actions CI/CD

### 8.1 Test Workflow (`.github/workflows/ci_tests.yml`)
```yaml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install -r dashboard/requirements.txt
          pip install -r orchestration/requirements.txt
      - name: Run pytest
        run: pytest tests/ -v --cov
      - name: Run dbt tests
        run: cd dbt && dbt test
```

### 8.2 Deployment Workflow (`.github/workflows/deploy.yml`)
```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and push Docker images
        run: |
          docker build -f docker/Dockerfile.dashboard -t gcr.io/...
          docker push gcr.io/...
      - name: Deploy to Cloud Run
        run: gcloud run deploy fintrack-dashboard ...
```

---

## Phase 9: Quality Assurance & Testing

### 9.1 Manual Testing Checklist
- [ ] **Extraction**:
  - [ ] Verify all 3+ data sources connected
  - [ ] Check row counts increase appropriately
  - [ ] Validate null handling
  
- [ ] **Transformation**:
  - [ ] Run dbt and verify all models succeed
  - [ ] Check test coverage > 80%
  - [ ] Validate data integrity (no NaN in metrics)
  
- [ ] **Loading**:
  - [ ] Confirm data arrives in BigQuery
  - [ ] Validate row counts match dbt outputs
  - [ ] Test incremental and full refreshes
  
- [ ] **Dashboard**:
  - [ ] Test all 4 pages (Home, Revenue, Retention, Churn)
  - [ ] Verify filters work (date, cohort, segment)
  - [ ] Check calculations match expected values
  - [ ] Test responsiveness on mobile
  - [ ] Verify export to CSV works
  
- [ ] **Performance**:
  - [ ] Dashboard loads in <2s
  - [ ] Filters update in <500ms
  - [ ] BigQuery queries execute in <5s

### 9.2 Automated Test Execution
- [ ] Run full test suite before merge:
  ```bash
  pytest tests/ -v --cov=dashboard --cov=orchestration
  dbt test
  dbt run --select staging+
  ```

---

## Phase 10: Deployment & Production Ready

### 10.1 Pre-Production Checklist
- [ ] All tests passing (unit, integration, dbt)
- [ ] Code coverage > 80%
- [ ] Documentation complete and accurate
- [ ] Docker images built and tested
- [ ] Environment variables documented
- [ ] Error handling and retry logic verified
- [ ] Logging structured and comprehensive
- [ ] Performance benchmarks met
- [ ] Security review (no hardcoded secrets, input validation)

### 10.2 Production Deployment
- [ ] Set up Google Cloud project
- [ ] Create BigQuery datasets
- [ ] Configure service accounts and IAM roles
- [ ] Deploy Docker containers to Cloud Run
- [ ] Set up Prefect Cloud for orchestration
- [ ] Configure monitoring and alerting
- [ ] Set up backup and disaster recovery
- [ ] Document runbooks for common issues

### 10.3 Post-Deployment Monitoring
- [ ] Monitor ETL pipeline execution times
- [ ] Track dashboard query latencies
- [ ] Monitor BigQuery costs
- [ ] Set up alerts for pipeline failures
- [ ] Track user adoption and feature usage

---

## Summary of Deliverables

✅ **Configuration Files**: .gitignore, .env.example, requirements.txt
✅ **dbt Transformation**: Staging → Intermediate → Marts (30+ models with tests)
✅ **Prefect Orchestration**: Extract, Transform, Load tasks with error handling
✅ **Plotly Dash Dashboard**: 4 pages, 20+ interactive charts, filter system
✅ **Docker Setup**: Docker Compose for local development
✅ **Test Suite**: 50+ unit/integration tests, dbt tests, data quality checks
✅ **Documentation**: Architecture, setup, API, deployment guides
✅ **CI/CD**: GitHub Actions workflows for testing and deployment

---

## Success Criteria

By the end of this development effort:

1. **Code Quality**: All tests passing, >80% coverage
2. **Performance**: Dashboard <2s load time, queries <5s
3. **Reliability**: 99.9% pipeline uptime, automatic retries on failure
4. **Usability**: Executives can self-serve analytics without SQL knowledge
5. **Documentation**: Complete, up-to-date, easily searchable
6. **Production Ready**: Deployable to cloud with monitoring and alerting

---

## Next Steps for Claude Agent

1. Start with **Phase 1**: Create config files and directory structure
2. Move to **Phase 2**: Build dbt models incrementally, test each layer
3. Proceed to **Phase 3**: Implement Prefect tasks and flows
4. Develop **Phase 4**: Build dashboard pages and callbacks
5. Setup **Phase 5**: Docker configuration
6. Write **Phase 6**: Comprehensive tests
7. Document **Phase 7**: All components and deployment
8. Configure **Phase 8**: GitHub Actions workflows
9. Execute **Phase 9**: Full QA testing
10. Deploy **Phase 10**: To production

Ask for clarification on any business logic, specific metrics, or data sources. Request sample data if needed for testing.
