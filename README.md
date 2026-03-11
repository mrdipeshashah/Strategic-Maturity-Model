Phase 0: Data Integrity & Foundation
Goal: To ensure the raw data is trustworthy before any analysis begins.

0.1_data_validation.sql – Identifies nulls, blanks, and schema inconsistencies in the raw dataset.

0.2_duplicate_transactions.sql – Detects and removes double-counted orders to ensure revenue accuracy.

0.3_date_range_integrity.sql – Confirms all transaction timestamps fall within the expected 2023–2025 window.

0.4_day_0_validation.sql – Validates that "First Purchase" dates correctly align with the global dataset start.

Phase 1: Exploratory Cohort Analysis
Goal: Understanding the scale and basic behavior of our customer segments.

1.1_annual_active_customers.sql – Calculates the total unique customer count active in each calendar year.

1.2_cohort_size_analysis.sql – Groups customers by their "Birth Year" to establish the starting size of each cohort.

1.3_frequency_bucket_analysis.sql – Segments the database by how many times a customer has purchased (1x, 2x, 5x+).

1.4_profit_bucket_segments.sql – Ranks customers into profitability tiers to identify our highest-value users.

Phase 2: Matured LTV & Performance
Goal: Measuring the true value of customers using matured time-windows (Apples-to-Apples).

2.1_matured_ltv_summary.sql – Calculates the "true" average profit for customers who have hit 1-year and 2-year milestones.

2.2_yearly_sidebyside_performance.sql – Compares the 1-year LTV of the 2023 cohort vs. the 2024 cohort to track quality trends.

2.3_payback_velocity.sql – Measures how many days it takes for a new customer to reach specific profit thresholds (e.g., $40).

2.4_days_between_orders.sql – Analyzes the "Purchase Gap" to help predict when a customer is likely to churn.

Phase 3: Strategic Planning & Contribution
Goal: High-level financial context for budgeting and 2026 annual planning.

3.1_migration_retention_profit.sql – Tracks how many 2023/2024 customers are still active and profitable in 2025.

3.2_revenue_profit_contribution_pct.sql – Breaks down annual revenue into % piles (New vs. Returning) for "Annuity" reporting.

3.3_cac_payback_model.sql – Merges acquisition costs with LTV curves to find the exact day of profitability.

3.4_monthly_yearly_trend_summary.sql – Provides the "Big Picture" monthly view used for board-level financial reporting.
