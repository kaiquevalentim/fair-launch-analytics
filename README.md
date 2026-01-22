# fair-launch-analytics
A data analysis framework for evaluating Liquidity Bootstrapping Pool (LBP) performance, price discovery efficiency, and optimal token launch configurations using on-chain data.

## Final table schema:

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

| Column Name | Source | Calculation / Logic |
| --- | --- | --- |
| `pool_address` | Raw | Foreign Key. |
| `volume_usd` | Calc | Total USD value of all swaps during the LBP. |
| `unique_buyers` | Calc | Count of unique wallet addresses that executed a BUY. |
| `price_retention` | Calc | `(Avg Price Last 5 Trades) / (Avg Price First 5 Trades)`. |
| `volatility_score` | Calc | `StdDev(Price) / Mean(Price)`. (0.0 if single trade). |
| `bot_tx_ratio` | Calc | `%` of total trades that occurred in the first 3 blocks. |
| `dump_pressure` | Calc | `Sell Volume USD / Buy Volume USD`. |
| `final_success_score` | Calc | **Target Variable.** Weighted sum of Retention (40%), Volatility, Unique Buyers, and Dump Pressure. |

### **Dataset Statistics**

* **Raw Input (`table_a_complete.csv`):** 2,456 rows.
* *Contains every configuration update, pause, and test event.*


* **Final Output (`table_a_final_enriched.csv`):** 961 rows.
* *Contains only valid, unique LBP launches (Duration > 6h, deduplicated).*


* **Target Labels (`table_b_advanced.csv`):** 961 rows.
* *Financial performance metrics for the exact same pools.*


* **Training Set (`training_dataset.csv`):** 961 rows.
* *Merged dataset (Features + Targets) ready for ML.*