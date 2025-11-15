--1. Daily Sales Price per Unit
SELECT
    DATE,
    SALES,
    QUANTITY_SOLD,
    ROUND(SALES / NULLIF(QUANTITY_SOLD, 0), 2) AS DAILY_SALES_PRICE_PER_UNIT
FROM SALES.CASE.SALES_CASE_STUDY;

---------------------------------------------------------------------------------------------------------------------------------------------------------

--2. Average Unit Sales Price of the Product
SELECT
    ROUND(SUM(SALES) / NULLIF(SUM(QUANTITY_SOLD), 0), 2) AS AVG_UNIT_SALES_PRICE
FROM SALES.CASE.SALES_CASE_STUDY;

---------------------------------------------------------------------------------------------------------------------------------------------------------
--3. Daily % Gross Profit
SELECT
    DATE,
    SALES,
    COST_OF_SALES,
    ROUND(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100, 2) AS DAILY_GROSS_PROFIT_PERCENT
FROM SALES.CASE.SALES_CASE_STUDY;

----------------------------------------------------------------------------------------------------------------------------------------------------------
--4. Daily % Gross Profit per Unit
SELECT
    DATE,
    ROUND((((SALES - COST_OF_SALES) / NULLIF(QUANTITY_SOLD, 0)) /
          (SALES / NULLIF(QUANTITY_SOLD, 0))) * 100, 2) AS DAILY_GROSS_PROFIT_PER_UNIT_PERCENT
FROM SALES.CASE.SALES_CASE_STUDY;

------------------------------------------------------------------------------------------------------------------------------------------------------------
--5. Price Elasticity of Demand (for 3 promotion periods)
WITH price_qty AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2013-12-24' AND '2013-12-30' THEN 'PRE_PROMO_1'
        END AS PERIOD,
        AVG(SALES / NULLIF(QUANTITY_SOLD, 0)) AS AVG_PRICE,
        AVG(QUANTITY_SOLD) AS AVG_QTY
    FROM SALES.CASE.SALES_CASE_STUDY
    WHERE DATE BETWEEN '2013-12-24' AND '2014-01-07'
    GROUP BY 1
)
SELECT
    'PROMO_1' AS PROMOTION,
    ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) / ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
FROM price_qty p
JOIN price_qty n
  ON p.PERIOD = 'PROMO_1' AND n.PERIOD = 'PRE_PROMO_1';


-----------------------------------------------------------------------------------------------------------------------------------------------------------

--Insight Query (Performance Comparison during Promotions)
SELECT
    CASE 
        WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
        WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
        WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
        ELSE 'NORMAL'
    END AS PERIOD,
    AVG(SALES / NULLIF(QUANTITY_SOLD, 0)) AS AVG_UNIT_PRICE,
    AVG(QUANTITY_SOLD) AS AVG_QTY_SOLD,
    AVG(((SALES - COST_OF_SALES) / SALES) * 100) AS AVG_GROSS_PROFIT_PCT
FROM SALES.CASE.SALES_CASE_STUDY
GROUP BY 1
ORDER BY 1;

------------------------------------------------------------------------------------------------------------------------------------------------------------
--Combined sql queries
-- =====================================================
-- 1️⃣ DAILY METRICS: PRICE, GROSS PROFIT %, ETC.
-- =====================================================

CREATE OR REPLACE TABLE SALES.CASE.SALES_METRICS AS
SELECT
    DATE,
    SALES,
    COST_OF_SALES,
    QUANTITY_SOLD,
    ROUND(SALES / NULLIF(QUANTITY_SOLD, 0), 2) AS DAILY_UNIT_PRICE,
    ROUND(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100, 2) AS DAILY_GROSS_PROFIT_PCT,
    ROUND((((SALES - COST_OF_SALES) / NULLIF(QUANTITY_SOLD, 0)) /
          (SALES / NULLIF(QUANTITY_SOLD, 0))) * 100, 2) AS DAILY_GROSS_PROFIT_PER_UNIT_PCT
FROM SALES.CASE.SALES_CASE_STUDY;

-- =====================================================
-- 2️⃣ AVERAGE UNIT SALES PRICE OVER ENTIRE PERIOD
-- =====================================================
SELECT
    ROUND(SUM(SALES) / NULLIF(SUM(QUANTITY_SOLD), 0), 2) AS AVG_UNIT_SALES_PRICE
FROM SALES.CASE.SALES_CASE_STUDY;

-- =====================================================
-- 3️⃣ DEFINE PROMOTION PERIODS
--    (adjust these date ranges to match real promo periods)
-- =====================================================
-- PROMO 1: 2014-01-01 to 2014-01-07
-- PROMO 2: 2014-02-01 to 2014-02-07
-- PROMO 3: 2014-03-01 to 2014-03-07

WITH price_qty AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2013-12-24' AND '2013-12-31' THEN 'PRE_PROMO_1'
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2014-01-24' AND '2014-01-31' THEN 'PRE_PROMO_2'
            WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN DATE BETWEEN '2014-02-24' AND '2014-02-28' THEN 'PRE_PROMO_3'
            WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
        END AS PERIOD,
        AVG(SALES / NULLIF(QUANTITY_SOLD, 0)) AS AVG_PRICE,
        AVG(QUANTITY_SOLD) AS AVG_QTY
    FROM SALES.CASE.SALES_CASE_STUDY
    WHERE DATE BETWEEN '2013-12-24' AND '2014-03-07'
    GROUP BY 1
),

-- =====================================================
-- 4️⃣ CALCULATE PRICE ELASTICITY FOR EACH PROMO PERIOD
-- =====================================================

promo_elasticity AS (
    SELECT
        'PROMO_1' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) / ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_1' AND n.PERIOD = 'PRE_PROMO_1'

    UNION ALL

    SELECT
        'PROMO_2' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) / ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_2' AND n.PERIOD = 'PRE_PROMO_2'

    UNION ALL

    SELECT
        'PROMO_3' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) / ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_3' AND n.PERIOD = 'PRE_PROMO_3'
)

SELECT * FROM promo_elasticity;

-- =====================================================
-- 5️⃣ PERFORMANCE COMPARISON: NORMAL VS PROMO PERIODS
-- =====================================================

SELECT
    CASE 
        WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
        WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
        WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
        ELSE 'NORMAL'
    END AS PERIOD,
    ROUND(AVG(SALES / NULLIF(QUANTITY_SOLD, 0)), 2) AS AVG_UNIT_PRICE,
    ROUND(AVG(QUANTITY_SOLD), 2) AS AVG_QTY_SOLD,
    ROUND(AVG(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100), 2) AS AVG_GROSS_PROFIT_PCT
FROM SALES.CASE.SALES_CASE_STUDY
GROUP BY 1
ORDER BY 1;

--------------------------------------------------------------------------------------------------------------------------------------------------------
--Complete SQL Script (with Performance Flags)

-- =====================================================
-- 1️⃣ DAILY METRICS TABLE
-- =====================================================

CREATE OR REPLACE TABLE SALES.CASE.SALES_METRICS AS
SELECT
    DATE,
    SALES,
    COST_OF_SALES,
    QUANTITY_SOLD,
    ROUND(SALES / NULLIF(QUANTITY_SOLD, 0), 2) AS DAILY_UNIT_PRICE,
    ROUND(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100, 2) AS DAILY_GROSS_PROFIT_PCT,
    ROUND((((SALES - COST_OF_SALES) / NULLIF(QUANTITY_SOLD, 0)) /
          (SALES / NULLIF(QUANTITY_SOLD, 0))) * 100, 2) AS DAILY_GROSS_PROFIT_PER_UNIT_PCT
FROM SALES.CASE.SALES_CASE_STUDY;

-- =====================================================
-- 2️⃣ AVERAGE UNIT SALES PRICE OVER ENTIRE PERIOD
-- =====================================================
SELECT
    ROUND(SUM(SALES) / NULLIF(SUM(QUANTITY_SOLD), 0), 2) AS AVG_UNIT_SALES_PRICE
FROM SALES.CASE.SALES_CASE_STUDY;

-- =====================================================
-- 3️⃣ DEFINE PROMOTION PERIODS
-- =====================================================
WITH price_qty AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2013-12-24' AND '2013-12-31' THEN 'PRE_PROMO_1'
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2014-01-24' AND '2014-01-31' THEN 'PRE_PROMO_2'
            WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN DATE BETWEEN '2014-02-24' AND '2014-02-28' THEN 'PRE_PROMO_3'
            WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
        END AS PERIOD,
        AVG(SALES / NULLIF(QUANTITY_SOLD, 0)) AS AVG_PRICE,
        AVG(QUANTITY_SOLD) AS AVG_QTY
    FROM SALES.CASE.SALES_CASE_STUDY
    WHERE DATE BETWEEN '2013-12-24' AND '2014-03-07'
    GROUP BY 1
),

-- =====================================================
-- 4️⃣ PRICE ELASTICITY CALCULATION
-- =====================================================

promo_elasticity AS (
    SELECT
        'PROMO_1' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) /
        ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_1' AND n.PERIOD = 'PRE_PROMO_1'

    UNION ALL

    SELECT
        'PROMO_2' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) /
        ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_2' AND n.PERIOD = 'PRE_PROMO_2'

    UNION ALL

    SELECT
        'PROMO_3' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) /
        ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_3' AND n.PERIOD = 'PRE_PROMO_3'
)

SELECT * FROM promo_elasticity;

-- =====================================================
-- 5️⃣ PERFORMANCE COMPARISON (NORMAL VS PROMO)
-- =====================================================

WITH period_summary AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
            ELSE 'NORMAL'
        END AS PERIOD,
        ROUND(AVG(SALES / NULLIF(QUANTITY_SOLD, 0)), 2) AS AVG_UNIT_PRICE,
        ROUND(AVG(QUANTITY_SOLD), 2) AS AVG_QTY_SOLD,
        ROUND(AVG(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100), 2) AS AVG_GROSS_PROFIT_PCT
    FROM SALES.CASE.SALES_CASE_STUDY
    GROUP BY 1
)

SELECT
    ps.PERIOD,
    ps.AVG_UNIT_PRICE,
    ps.AVG_QTY_SOLD,
    ps.AVG_GROSS_PROFIT_PCT,
    CASE 
        WHEN ps.PERIOD LIKE 'PROMO%' THEN
            CASE 
                WHEN ps.AVG_QTY_SOLD > (SELECT AVG(AVG_QTY_SOLD) FROM period_summary WHERE PERIOD = 'NORMAL')
                     AND ps.AVG_GROSS_PROFIT_PCT >= (SELECT AVG(AVG_GROSS_PROFIT_PCT) FROM period_summary WHERE PERIOD = 'NORMAL')
                THEN 'Better: Higher Sales & Profit'
                WHEN ps.AVG_QTY_SOLD > (SELECT AVG(AVG_QTY_SOLD) FROM period_summary WHERE PERIOD = 'NORMAL')
                     AND ps.AVG_GROSS_PROFIT_PCT < (SELECT AVG(AVG_GROSS_PROFIT_PCT) FROM period_summary WHERE PERIOD = 'NORMAL')
                THEN 'Mixed: Higher Sales, Lower Margin'
                ELSE 'Worse: Lower Sales or Profit'
            END
        ELSE 'NORMAL PERIOD (Baseline)'
    END AS PERFORMANCE_FLAG
FROM period_summary ps
ORDER BY 
    CASE 
        WHEN ps.PERIOD = 'NORMAL' THEN 0
        WHEN ps.PERIOD = 'PROMO_1' THEN 1
        WHEN ps.PERIOD = 'PROMO_2' THEN 2
        WHEN ps.PERIOD = 'PROMO_3' THEN 3
    END;

-----------------------------------------------------------------------------------------------------------------------------------------------------------
--Combined above queries to produce final putput table:

WITH base_data AS (
    SELECT
        DATE,
        SALES,
        COST_OF_SALES,
        QUANTITY_SOLD,
        ROUND(SALES / NULLIF(QUANTITY_SOLD, 0), 2) AS DAILY_SALES_PRICE_PER_UNIT,
        ROUND(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100, 2) AS DAILY_GROSS_PROFIT_PERCENT,
        ROUND((((SALES - COST_OF_SALES) / NULLIF(QUANTITY_SOLD, 0)) /
              (SALES / NULLIF(QUANTITY_SOLD, 0))) * 100, 2) AS DAILY_GROSS_PROFIT_PER_UNIT_PERCENT
    FROM SALES.CASE.SALES_CASE_STUDY
),

avg_price AS (
    SELECT
        ROUND(SUM(SALES) / NULLIF(SUM(QUANTITY_SOLD), 0), 2) AS AVG_UNIT_SALES_PRICE
    FROM SALES.CASE.SALES_CASE_STUDY
),

price_qty AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2013-12-24' AND '2013-12-31' THEN 'PRE_PROMO_1'
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2014-01-24' AND '2014-01-31' THEN 'PRE_PROMO_2'
            WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN DATE BETWEEN '2014-02-24' AND '2014-02-28' THEN 'PRE_PROMO_3'
            WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
        END AS PERIOD,
        AVG(SALES / NULLIF(QUANTITY_SOLD, 0)) AS AVG_PRICE,
        AVG(QUANTITY_SOLD) AS AVG_QTY
    FROM SALES.CASE.SALES_CASE_STUDY
    WHERE DATE BETWEEN '2013-12-24' AND '2014-03-07'
    GROUP BY 1
),

promo_elasticity AS (
    SELECT
        'PROMO_1' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) /
        ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_1' AND n.PERIOD = 'PRE_PROMO_1'

    UNION ALL

    SELECT
        'PROMO_2' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) /
        ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_2' AND n.PERIOD = 'PRE_PROMO_2'

    UNION ALL

    SELECT
        'PROMO_3' AS PROMOTION,
        ((p.AVG_QTY - n.AVG_QTY) / n.AVG_QTY) /
        ((p.AVG_PRICE - n.AVG_PRICE) / n.AVG_PRICE) AS PRICE_ELASTICITY
    FROM price_qty p
    JOIN price_qty n ON p.PERIOD = 'PROMO_3' AND n.PERIOD = 'PRE_PROMO_3'
),

period_summary AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
            ELSE 'NORMAL'
        END AS PERIOD,
        ROUND(AVG(SALES / NULLIF(QUANTITY_SOLD, 0)), 2) AS AVG_UNIT_PRICE,
        ROUND(AVG(QUANTITY_SOLD), 2) AS AVG_QTY_SOLD,
        ROUND(AVG(((SALES - COST_OF_SALES) / NULLIF(SALES, 0)) * 100), 2) AS AVG_GROSS_PROFIT_PCT
    FROM SALES.CASE.SALES_CASE_STUDY
    GROUP BY 1
),

performance AS (
    SELECT
        ps.PERIOD,
        ps.AVG_UNIT_PRICE,
        ps.AVG_QTY_SOLD,
        ps.AVG_GROSS_PROFIT_PCT,
        CASE 
            WHEN ps.PERIOD LIKE 'PROMO%' THEN
                CASE 
                    WHEN ps.AVG_QTY_SOLD > (SELECT AVG(AVG_QTY_SOLD) FROM period_summary WHERE PERIOD = 'NORMAL')
                         AND ps.AVG_GROSS_PROFIT_PCT >= (SELECT AVG(AVG_GROSS_PROFIT_PCT) FROM period_summary WHERE PERIOD = 'NORMAL')
                    THEN 'Better: Higher Sales & Profit'
                    WHEN ps.AVG_QTY_SOLD > (SELECT AVG(AVG_QTY_SOLD) FROM period_summary WHERE PERIOD = 'NORMAL')
                         AND ps.AVG_GROSS_PROFIT_PCT < (SELECT AVG(AVG_GROSS_PROFIT_PCT) FROM period_summary WHERE PERIOD = 'NORMAL')
                    THEN 'Mixed: Higher Sales, Lower Margin'
                    ELSE 'Worse: Lower Sales or Profit'
                END
            ELSE 'NORMAL PERIOD (Baseline)'
        END AS PERFORMANCE_FLAG
    FROM period_summary ps
),

summary_combined AS (
    SELECT
        b.DATE,
        b.SALES,
        b.COST_OF_SALES,
        b.QUANTITY_SOLD,
        b.DAILY_SALES_PRICE_PER_UNIT,
        b.DAILY_GROSS_PROFIT_PERCENT,
        b.DAILY_GROSS_PROFIT_PER_UNIT_PERCENT,
        (SELECT AVG_UNIT_SALES_PRICE FROM avg_price) AS AVG_UNIT_SALES_PRICE,
        p.PROMOTION,
        p.PRICE_ELASTICITY,
        s.PERIOD AS PERFORMANCE_PERIOD,
        s.AVG_UNIT_PRICE,
        s.AVG_QTY_SOLD,
        s.AVG_GROSS_PROFIT_PCT,
        s.PERFORMANCE_FLAG
    FROM base_data b
    LEFT JOIN promo_elasticity p
        ON CASE 
            WHEN b.DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN b.DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN b.DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
        END = p.PROMOTION
    LEFT JOIN performance s
        ON CASE 
            WHEN b.DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN b.DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN b.DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
            ELSE 'NORMAL'
        END = s.PERIOD
)

SELECT * FROM summary_combined
ORDER BY DATE;

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------6. Additional Insights
--6A. Daily KPI Dashboard Table
SELECT
    DATE,
    SALES,
    QUANTITY_SOLD,
    ROUND(SALES / NULLIF(QUANTITY_SOLD,0), 2) AS UNIT_PRICE,
    ROUND(((SALES - COST_OF_SALES) / NULLIF(SALES,0)) * 100, 2) AS GROSS_MARGIN_PCT
FROM SALES.CASE.SALES_CASE_STUDY
ORDER BY DATE;

---------------------------------------------------------------------------------------------------------------------------------------------------------------------
--6B. Revenue & Volume Trend
SELECT
    DATE,
    SALES,
    QUANTITY_SOLD
FROM SALES.CASE.SALES_CASE_STUDY
ORDER BY DATE;

---------------------------------------------------------------------------------------------------------------------------------------------------------------------
--6C. Price vs Quantity Relationship
SELECT
    ROUND(SALES / NULLIF(QUANTITY_SOLD, 0), 2) AS UNIT_PRICE,
    QUANTITY_SOLD
FROM SALES.CASE.SALES_CASE_STUDY;

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------6D. Profitability Analysis
SELECT
    DATE,
    SALES,
    COST_OF_SALES,
    (SALES - COST_OF_SALES) AS GROSS_PROFIT,
    ROUND(((SALES - COST_OF_SALES) / NULLIF(SALES,0)) * 100,2) AS GROSS_MARGIN_PCT
FROM SALES.CASE.SALES_CASE_STUDY
ORDER BY DATE;

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------6E. Pre/Post Promotion Comparative Summary
WITH grouped AS (
    SELECT
        CASE 
            WHEN DATE BETWEEN '2014-01-01' AND '2014-01-07' THEN 'PROMO_1'
            WHEN DATE BETWEEN '2014-02-01' AND '2014-02-07' THEN 'PROMO_2'
            WHEN DATE BETWEEN '2014-03-01' AND '2014-03-07' THEN 'PROMO_3'
            ELSE 'NORMAL'
        END AS PERIOD,
        SALES,
        QUANTITY_SOLD,
        (SALES - COST_OF_SALES) AS GROSS_PROFIT
    FROM SALES.CASE.SALES_CASE_STUDY
)
SELECT
    PERIOD,
    ROUND(AVG(SALES),2) AS AVG_DAILY_REVENUE,
    ROUND(AVG(QUANTITY_SOLD),2) AS AVG_DAILY_UNITS,
    ROUND(AVG(GROSS_PROFIT),2) AS AVG_DAILY_PROFIT
FROM grouped
GROUP BY PERIOD
ORDER BY PERIOD;



