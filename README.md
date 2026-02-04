# fair-launch-analytics

A comprehensive data analysis framework for evaluating Liquidity Bootstrapping Pool (LBP) performance, price discovery efficiency, and optimal token launch configurations using on-chain data from Dune Analytics.

## Project Overview

This framework provides end-to-end analysis of LBP mechanics across Balancer V1 and V2 protocols, extracting key configuration and performance metrics to understand what makes successful token launches.

### Key Objectives
- **Configuration Analysis**: Extract launch parameters (weights, fees, duration, collateral type)
- **Performance Evaluation**: Calculate success metrics (volume, price discovery, buyer diversity)
- **Pattern Recognition**: Identify optimal launch configurations using machine learning
- **Historical Trends**: Analyze reverse LBPs (rLBPs) and historical performance patterns

---

## Project Structure

### **Data Sources & Downloads**
- `analytics/dune_api_downloads/` - Direct Dune API data extraction
  - `v1v2_download.ipynb` - Download Balancer V1/V2 pool data via API
  - `v1v2_eda_and_data_cleaning.ipynb` - Exploratory analysis and initial cleaning
  
- `analytics/dune_web_downloads/` - Web-based data processing and consolidation
  - `v1_csvs_formatting_and_merging.ipynb` - Format and merge Balancer V1 datasets
  - `v2_csvs_formatting_and_merging.ipynb` - Format and merge Balancer V2 datasets
  - `v1v2_eda_and_data_cleaning.ipynb` - Combined analysis and data validation
  - `LBPs_dataset_analysis.ipynb` - Comprehensive LBP dataset overview

### **Reverse LBPs (rLBPs) Analysis**
- `analytics/dune_web_downloads/rLBPs/`
  - `v1v2_download.ipynb` - Download and process reverse LBP data from Balancer V1 & V2
  - `v1v2_historical_analysis.ipynb` - Historical trends and patterns in reverse launches

### **Machine Learning**
- `model_development/logistic_regression_&_random_forest.ipynb` - Classification models for predicting LBP success factors

---

## Analytics Workflow

1. **Data Extraction** → Download raw pool data and trade history from Dune
2. **Data Cleaning** → Deduplicate, normalize, handle missing values
3. **Feature Engineering** → Calculate configuration metrics (weights, slopes, fees)
4. **Performance Calculation** → Compute success metrics from trade data
5. **Exploratory Analysis** → Identify patterns and outliers
6. **Model Development** → Train predictive models on merged feature-target dataset

---

## Output Datasets

### **Final table schema:

### **Table A: Configuration Features (`table_a_final_enriched.csv`)**

*Description: The "Physics" of the pool. Defines the parameters chosen by the creator at launch (t=0).*

| Column Name | Source | Calculation / Logic |
| --- | --- | --- |
| `pool_address` | Raw | Unique Contract Address (Primary Key). |
| `chain` | Raw | Blockchain network (Ethereum, Arbitrum, Polygon, etc.). |
| `version` | Raw | Balancer V1 (Legacy) or V2 (Standard). |
| `start_timestamp` | Raw | UTC Timestamp of the LBP creation/launch. |
| `duration_hours` | Calc | `(end_timestamp - start_timestamp) / 3600`. |
| `start_weight_proj` | Calc | Initial weight (0.0-1.0) of the Project Token (e.g., 0.99). |
| `end_weight_proj` | Calc | Final weight (0.0-1.0) of the Project Token (e.g., 0.20). |
| `start_weight_reserve` | Calc | Initial weight of the Collateral Token (e.g., 0.01). |
| `end_weight_reserve` | Calc | Final weight of the Collateral Token (e.g., 0.80). |
| `weight_slope` | Calc | `Abs(end_weight_proj - start_weight_proj) / duration_hours`. |
| `swap_fee_pct` | Raw | Trading fee charged to swappers (e.g., 0.01 for 1%). |
| `collateral_is_stable` | Calc | `1` if collateral is USDC/DAI/USDT, `0` if volatile (e.g., WETH). |
| `is_weekend` | Calc | `1` if launch day is Saturday or Sunday, `0` otherwise. |
| `weekend_pct` | Calc | Percentage of total duration that overlaps with a weekend. |

### **Table B: Success Targets (`table_b_advanced.csv`)**

*Description: The "Performance" of the pool. Calculated from trading history (Dune `dex.trades`).*

| Column Name                 | Source | Calculation / Logic                                                                    |
| --------------------------- | ------ | -------------------------------------------------------------------------------------- |
| `pool_address`              | Raw    | Foreign Key (Links to Table A).                                                        |
| `volume_usd`                | Calc   | Total USD value of all swaps during the LBP.                                           |
| `unique_buyers`             | Calc   | Count of unique wallet addresses that executed a BUY.                                  |
| `price_retention`           | Calc   | `(Avg Price Last 5 Blocks) / (Avg Price First 5 Blocks)`. (Measures if price held up). |
| `volatility_score`          | Calc   | `Mean Price / Standard Deviation of Price`. (Measures turbulence).                     |
| `dump_pressure`             | Calc   | `Total Buy Volume (USD) / Total Sell Volume (USD)`. (>1.0 means net selling).          |
| `volume_time_skew`          | New    | Time-weighted center of volume (0.0 = Start, 1.0 = End). Ideal is ~0.5.                |
| `whale_dominance_pct`       | New    | `Volume of Top 1% Trades / Total Volume`. (Measures centralization risk).              |
| `turnover_ratio`            | New    | `Total Volume / Initial Liquidity`. (Measures capital efficiency).                     |
| `bot_tx_ratio`              | New    | `Trades in First 5 Blocks / Total Trades`. (Measures sniper activity).                 |
| `bot_extraction_usd`        | New    | `Bot Sells - Bot Buys` (during first 5 blocks). Negative means bots are holding.       |
| `price_discovery_stability` | New    | Volatility calculated only on the last 10% of trades. (Did price settle?).             |

### **Dataset Statistics**

| Dataset | File | Rows | Description |
| --- | --- | --- | --- |
| **Raw Input** | `table_a_complete.csv` | 2,456 | Every configuration update, pause, and test event. |
| **Configuration** | `table_a_final_enriched.csv` | 961 | Valid, unique LBP launches (Duration > 6h, deduplicated). |
| **Performance** | `table_b_advanced.csv` | 961 | Financial performance metrics matched to pools. |
| **Training** | `training_dataset.csv` | 961 | Merged features + targets ready for ML models. |

---

## Key Features Analyzed

### Configuration Metrics
- Pool weight curves (start/end weights, slope steepness)
- Swap fees and collateral type (stable vs volatile)
- Launch timing and duration
- Temporal factors (weekends, holidays)

### Performance Indicators
- Trading volume and unique buyer count
- Price discovery efficiency (retention, volatility)
- Market participation patterns (bot activity, whale dominance)
- Capital efficiency (turnover ratio)

---

## Models & Analysis

### Machine Learning Approach
The project includes **Logistic Regression** and **Random Forest** classifiers to predict LBP success factors and identify configuration patterns that drive favorable outcomes.

### Data Processing Highlights
- **Deduplication Logic**: Identifies main LBP event by largest weight change delta
- **Decimal Conversion**: Handles token-specific decimal normalization (USDC, USDT, DAI, etc.)
- **Time-Series Analysis**: Computes rolling volatility and time-weighted metrics
- **Bot Detection**: Identifies sniper trades and extraction strategies in early blocks

---

## Getting Started

### Prerequisites
- Python 3.8+
- pandas, numpy, scikit-learn
- Dune API client library
- Jupyter Notebook

### Running the Analysis
1. Execute Dune API downloads to fetch raw pool data
2. Run data cleaning and formatting notebooks in sequence
3. Perform EDA to validate data quality
4. Train models on the final merged dataset

---

## File Structure

```
fair-launch-analytics/
├── README.md                                    # Project overview (this file)
├── analytics/
│   ├── dune_api_downloads/                     # Direct API extraction
│   │   ├── v1v2_download.ipynb
│   │   └── v1v2_eda_and_data_cleaning.ipynb
│   ├── dune_web_downloads/                     # Web-based data processing
│   │   ├── v1_csvs_formatting_and_merging.ipynb
│   │   ├── v2_csvs_formatting_and_merging.ipynb
│   │   ├── v1v2_eda_and_data_cleaning.ipynb
│   │   ├── LBPs_dataset_analysis.ipynb
│   │   └── rLBPs/                              # Reverse LBP analysis
│   │       ├── v1v2_download.ipynb
│   │       └── v1v2_historical_analysis.ipynb
│   └── media/                                  # Processed CSV outputs
├── model_development/
│   └── logistic_regression_&_random_forest.ipynb  # ML classification models
```

---

## Notes

- All datasets are deduplicated at the pool level (by `pool_address`)
- Duration filtering applies a 6-hour minimum threshold
- Token decimals are normalized based on token type and blockchain
- Price calculations use the Dune `dex.trades` table as source of truth
