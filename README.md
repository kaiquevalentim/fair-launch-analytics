# fair-launch-analytics
A data analysis framework for evaluating Liquidity Bootstrapping Pool (LBP) performance, price discovery efficiency, and optimal token launch configurations using on-chain data.

## Final table schema:

| Column Name                  | Source | Calculation / Logic |
|-----------------------------|--------|----------------------|
| pool_id                     | Raw    | Balancer Vault contract address + ID. |
| launch_date                 | Raw    | `evt_block_time` of the first `joinPool` or `swap`. |
| duration_h                  | Calc   | `(endTime - startTime) / 3600`. |
| start_weight_proj           | Raw    | Initial project token weight (e.g., `0.98`). |
| end_weight_proj             | Raw    | Target project token weight (e.g., `0.50`). |
| weight_slope                | Calc   | `(start_weight - end_weight) / duration_h`. Measures the weight decay "speed". |
| initial_fdv_usd             | Calc   | `Starting_Price * Total_Supply`. Starting Fully Diluted Valuation. |
| swap_fee_pct                | Raw    | The fee charged on trades (e.g., 0.02 for 2%). Controls bot friction. |
| initial_liquidity_usd       | Calc   | Collateral_Balance_Start × Collateral_Price. The “hard money” backing the pool. |
| collateral_is_stable        | Calc   | 1 if collateral is USDC/DAI/USDT, 0 if volatile (e.g., WETH). |
| is_weekend                  | Calc   | 1 if launch_date is Saturday or Sunday. Python: `dt.dayofweek >= 5`. |
| hour_of_day                 | Calc   | The hour (0–23) of the launch. Python: `dt.hour`. |
| total_swaps                 | Raw    | `count(*)` from `balancer_v2_ethereum.evt_Swap` for this pool. |
| unique_users                | Raw    | `count(distinct sender)`. |
| activity_entropy            | Calc   | Shannon entropy of trades over time. High = steady activity; low = single burst. |
| swaps_per_hour              | Calc   | `total_swaps / duration_h`. |
| peak_activity_time          | Calc   | Normalized time (`0–1`) when the highest number of trades occurred. |
| bot_volume_pct              | Calc   | Percentage of volume from addresses with `>10` transactions in the same LBP pool. |
| price_efficiency            | Calc   | `1 - (abs(Final_LBP_Price - Market_Price_24h_Post) / Final_LBP_Price)`. |
| max_drawdown                | Calc   | Largest percentage drop from the programmed price curve during the sale. |
| buy_pressure_avg            | Calc   | Average of `(Actual_Price_t / Theoretical_Price_t)`. |
| price_found_success         | Calc   | `abs(LBP_Final_Price - Market_Price_24h) < 15%`. |
| consistent_activity_success | Calc   | `activity_entropy > 0.7`. Indicates steady trading. |
| not_bot_success             | Calc   | `bot_volume_pct < 30%`. Prevents wallet concentration. |
| final_success               | Calc   | Logical AND of all success conditions above (“Goldilocks” launch). |