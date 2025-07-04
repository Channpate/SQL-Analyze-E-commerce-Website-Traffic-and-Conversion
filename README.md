# [SQL] Analyze Traffic and Conversion on E-commerce Website 

  **Author:** Mai Ngoc Chau (Chann)
  
  **Date:** 2025-03-12
  
  **Tool used:** SQL (Google BigQuery)
  
  ## 📑 Table of Contents
  - [📌 Background & Overview](#-background--overview)
  - [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)
  - [⚒️ Process](#process)
  - [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)
  
  
  ## 📌 Background & Overview

  ### **Objectives:**
  This project use SQL to analyze Google Analytics data of an E-commerce website to:
  * Evaluate website performance through key metrics such as visits, pageviews, transactions, and revenue over time.
  
  * Analyze user behavior and conversion funnel.
  
  * Identify marketing and product insights by comparing performance across channels and uncovering product bundling opportunities.
  
  ### **Who is this project for:**
  * Data analysts & business analysts
  * Marketing decision-makers, team leaders

  ## 📂 Dataset Description & Data Structure

  ### Data source

  ### Data Structure & Relationships
  
  ## ⚒️ Process
  ### Query 1: Analyze **total visit, pageview, transaction** for Q1/2017 (Jan,Feb,Mar) (order by month)
  #### Code:
  ``` sql
  SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) month
        , SUM(totals.visits) visits
        , SUM(totals.pageviews) pageviews
        , SUM(totals.transactions) transactions
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
  WHERE EXTRACT(month from PARSE_DATE('%Y%m%d', date)) <4
  GROUP BY FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date))
  ORDER BY month ASC
  ```
  #### Result:
  | month    | visits | pageviews | transactions |
  | -------- | ------- | -------- | ------- |
  | 201701  | 64694    | 257708 | 713 |
  | 201702 | 62192     | 233373 | 733 |
  | 201703   | 69931    | 259522 | 993 |
  
  #### Observations & Findings:
  * Transactions increased steadily over Q1, with a 39% rise from January (713) to March (993), indicating improved user conversion.
  * Visits and pageviews dipped slightly in February, then peaked in March, suggesting a possible campaign or seasonal effect.
  * March was the strongest month across all key metrics, marking the best performance in the quarter.
  
  ### Query 2: Analyze bounce rate per traffic source in July 2017 (order by total_visit DESC)
  
  Bounce_rate = num_bounce/ total_visit
  
  #### Code:
  ``` sql
  WITH group_by_source AS (
      SELECT trafficSource.source
        , SUM(totals.bounces) total_no_of_bounces
        , SUM(totals.visits) total_visits
      FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
      GROUP BY trafficSource.source
  )
  SELECT *
        , ROUND(total_no_of_bounces*100.0/total_visits,3) bounce_rate
  FROM group_by_source
  ORDER BY total_visits DESC
  ```
  #### Result:
  | source    | total_no_of_bounces | total_visits | bounce_rate |
  | -------- | ------- | -------- | ------- |
  | google    | 19798    | 38400 | 51.557 |
  | (direct) | 8606     | 19891 | 43.266|
  | youtube.com   |4238    | 6351 |66.73 |
  | analytics.google.com   | 1064    | 1972 | 53.955 |
  | Partners   | 936 | 1788 | 52.349 |
  | m.facebook.com   | 430    | 669 | 64.275 |
  | google.com   | 183  | 368 | 49.728 |
  #### Observations & Findings:
  * Google and (direct) were the top traffic sources by visit volume, with 38,400 and 19,891 visits respectively, contributing the most to site traffic.
  * YouTube.com and m.facebook.com had the highest bounce rates, at 66.73% and 64.28%, indicating low user engagement from social channels.
  * Despite high traffic, Google’s bounce rate (51.56%) was moderate, suggesting a fair level of user interaction compared to other sources.
  * Referral sources like analytics.google.com and Partners had smaller traffic volumes but relatively high bounce rates (~53%), which may warrant closer examination of user intent or landing page relevance.
 
  ### Query 3: Revenue by traffic source by week, by month in June 2017
  #### Code:
  ``` sql
  SELECT 'Month' time_type
        , FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) time
        , trafficSource.source source
        , ROUND(SUM(product.productRevenue)/1000000.0,4) revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
      , UNNEST(hits) hits
      , UNNEST(hits.product) product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date))
          ,trafficSource.source
    
  UNION ALL
    
  SELECT 'Week' time_type
        , FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d', date)) time
        , trafficSource.source source
        , SUM(product.productRevenue)/1000000.0 revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`
      , UNNEST(hits) hits
      , UNNEST(hits.product) product
  WHERE product.productRevenue IS NOT NULL
  GROUP BY FORMAT_DATE('%Y%W',PARSE_DATE('%Y%m%d', date))
          ,trafficSource.source
  ```
  #### Result:
  | Time Type | Time   | Source              | Revenue     |
  |-----------|--------|---------------------|-------------|
  | Month     | 201706 | (direct)            | 97,333.62   |
  | Month     | 201706 | google              | 18,757.18   |
  | Month     | 201706 | dfa                 | 8,862.23    |
  | Week      | 201724 | (direct)            | 30,908.91   |
  | Week      | 201723 | (direct)            | 17,325.68   |
  | Week      | 201724 | google              | 9,217.17    |
  | Week      | 201726 | google              | 5,330.57    |
  | Week      | 201725 | (direct)            | 27,295.32   |
  | Week      | 201726 | (direct)            | 14,914.81   |
  | Week      | 201722 | (direct)            | 6,888.90    |
  | Week      | 201726 | dfa                 | 3,704.74    |
  | Week      | 201724 | mail.google.com     | 2,486.86    |

  #### Observations & Findings:
  * **(direct) traffic was the dominant revenue driver** in June 2017, contributing over $97,000 in the month and consistently leading weekly revenues.
  * Google and dfa were the next top-performing channels, with notable weekly spikes, especially in week 24 and 26.
  * Revenue is heavily concentrated among a few top channels, while most others (e.g., social and referral sources like YouTube, Yahoo, Facebook) contributed minimally.
  * Week 24 showed the highest revenue peak, likely driven by both direct and Google traffic.
  ### Query 4: Average number of pageviews by purchaser type (purchasers vs non-purchasers) in June, July 2017.
  #### Code:
  ``` sql
  WITH purchaser_data AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
    SUM(totals.pageviews) / COUNT(DISTINCT fullVisitorId) AS avg_pageviews_purchase
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(product) AS product
  WHERE
    _TABLE_SUFFIX BETWEEN '0601' AND '0731'
    AND totals.transactions >= 1
    AND product.productRevenue IS NOT NULL
  GROUP BY
    month
),

non_purchaser_data AS (
  SELECT
    FORMAT_DATE('%Y%m', PARSE_DATE('%Y%m%d', date)) AS month,
    SUM(totals.pageviews) / COUNT(DISTINCT fullVisitorId) AS avg_pageviews_non_purchase
  FROM
    `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) AS hits,
    UNNEST(product) AS product
  WHERE
    _TABLE_SUFFIX BETWEEN '0601' AND '0731'
    AND totals.transactions IS NULL
    AND product.productRevenue IS NULL
  GROUP BY
    month
)

SELECT
  pd.month,
  pd.avg_pageviews_purchase,
  npd.avg_pageviews_non_purchase
FROM
  purchaser_data AS pd
FULL JOIN
  non_purchaser_data AS npd
  ON pd.month = npd.month
ORDER BY
  pd.month;

  ```
  #### Result:
  | month   | avg_pageviews_purchase | avg_pageviews_non_purchase|
|---------|------------------------------|----------------------------------|
| 201706  | 94.02                        | 316.87                           |
| 201707  | 124.24                       | 334.06                           |

  #### Observations & Findings:
  In both June and July 2017, non-purchasers had significantly higher average pageviews than purchasers (e.g., 316 vs. 94 in June; 334 vs. 124 in July)
  &rarr; Purchasers tend to navigate more efficiently, while non-purchasers browse more without converting.
  
  ### Query 5: Average number of transactions per user that purchased in July 2017
  #### Code:
  ``` sql
  SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) month
        , ROUND(
                SUM(totals.transactions)/COUNT(distinct fullVisitorId)
                , 3) AS Avg_total_transactions_per_user
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
      , UNNEST(hits) hits
      , UNNEST(hits.product) product
  WHERE totals.transactions >=1 AND product.productRevenue IS NOT NULL
  GROUP BY FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date))
  ```
  #### Result:
  | month    | Avg_total_transactions_per_user |
  | -------- | ------- |
  | 201707  | 4.164    |
  #### Observations & Findings:
  In July 2017, users who made purchases had an average of 4.16 transactions per user, indicating a high level of repeat purchasing behavior among active buyers during the month.
  
  ### Query 6: Show average amount of money spent per session. Only include purchaser data in July 2017
  #### Code:
  ``` sql
  SELECT FORMAT_DATE('%Y%m',PARSE_DATE('%Y%m%d', date)) month
    , ROUND(SUM(product.productRevenue)/1000000.0
            /COUNT(fullVisitorId), 2) avg_revenue_by_user_per_visit
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) hits
    , UNNEST(hits.product) product
  WHERE totals.transactions >=1 AND product.productRevenue IS NOT NULL
  GROUP BY month
  ```
  #### Result:
  | month    | avg_revenue_by_user_per_visit |
  | -------- | ------- |
  | 201707  | 43.86    |
  #### Observations & Findings:
  On average, each purchasing session in July 2017 generated $43.85 in revenue.
  
  ### Query 7: Other products purchased by customers who purchased product "YouTube Men's Vintage Henley" in July 2017. 
  Output will show product name and the quantity was ordered.
  #### Code:
  ``` sql
  WITH buyer_list AS (
    SELECT
        distinct fullVisitorId  
    FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
    , UNNEST(hits) AS hits
    , UNNEST(hits.product) as product
    WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions >= 1
    AND product.productRevenue is not null
)

SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`
, UNNEST(hits) AS hits
, UNNEST(hits.product) as product
JOIN buyer_list USING(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
 and product.productRevenue is not null
GROUP BY other_purchased_products
ORDER BY quantity DESC;
  ```
  #### Result (shortlisted):
  | other_purchased_products                                | quantity |
|---------------------------------------------------------|----------|
| Google Sunglasses                                       | 20       |
| Google Women's Vintage Hero Tee Black                   | 7        |
| SPF-15 Slim & Slender Lip Balm                          | 6        |
| Google Women's Short Sleeve Hero Tee Red Heather        | 4        |
| YouTube Men's Fleece Hoodie Black                       | 3        |
| Google Men's Short Sleeve Badge Tee Charcoal            | 3        |
| YouTube Twill Cap                                       | 2        |
| Red Shine 15 oz Mug                                     | 2        |
| Android Women's Fleece Hoodie                           | 2        |
| Crunch Noise Dog Toy                                    | 2        |
| Google Men's Long & Lean Tee Charcoal                   | 1        |

  #### Observations & Findings: 
  Among users who purchased YouTube Men's Vintage Henley, the most frequently co-purchased item was Google Sunglasses (20 units), followed by vintage tees and lip balm.
  
  ### Query 8: Create cohort map from product view to addtocart to purchase in Jan, Feb and March 2017. For example, 100% product view then 40% add_to_cart and 10% purchase.
  Add_to_cart_rate = number product  add to cart/number product view. 
  Purchase_rate = number product purchase/number product view. 
  #### Code:
  ``` sql
  WITH product_data AS (
SELECT 
    FORMAT_DATE('%Y%m', parse_date('%Y%m%d',date)) AS month,
    COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) AS num_product_view,
    COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) AS num_add_to_cart,
    COUNT(CASE WHEN eCommerceAction.action_type = '6' AND product.productRevenue is not null THEN product.v2ProductName END) AS num_purchase
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`
,UNNEST(hits) AS hits
,UNNEST (hits.product) AS product
WHERE _table_suffix BETWEEN '20170101' AND '20170331'
AND eCommerceAction.action_type IN ('2','3','6')
GROUP BY month
ORDER BY month
)

SELECT 
    *,
    ROUND(num_add_to_cart/num_product_view * 100, 2) AS add_to_cart_rate,
    ROUND(num_purchase/num_product_view * 100, 2) AS purchase_rate
FROM product_data;
  ```
  #### Result:
  | month   | num_product_view | num_add_to_cart | num_purchase | add_to_cart_rate |purchase_rate |
|---------|----------------|-------------|-----------|----------------------|--------------------|
| 201701  | 25,787         | 7,342       | 2,143     | 28.47                | 8.31               |
| 201702  | 21,489         | 7,360       | 2,060     | 34.25                | 9.59               |
| 201703  | 23,549         | 8,782       | 2,977     | 37.29                | 12.64              |

  #### Observations & Findings:
  * Both add-to-cart and purchase rates improved month over month, indicating a positive trend in conversion funnel efficiency from January to March.
  * March recorded the highest funnel performance, with 12.64% of product views resulting in purchases.

  ## 🔎 Final Conclusion & Recommendations
  From the eight analytical questions run against user behavior and performance data on an e-commerce website, key findings were identified:
  
  1. Traffic & Engagement Trends
  * Visits, pageviews, and transactions increased steadily across Q1/2017, with March being the highlight month, reflecting solid momentum across customer engagement and purchasing behavior.
  * Direct and Google sources drove most traffic and revenue, with social and referral sources showing high bounce and low conversion rates.
  
  2. User Behavior & Conversion Funnel
  * Buyers looked at fewer pages than non-buyers, indicating increased goal-oriented activity in converting users.
  * The average user who made a purchase during July placed over 4 purchases and spent $43.85 per session—indicating high involvement by buyers.
  * Funnel performance increased across the months: product view → add to cart → buy rates all increased from January to March.
  
  3. Product Performance
  * Certain items, such as "Google Sunglasses," tended to be purchased in conjunction with other items, and hence offered clear cross-sell opportunities.
  
  4. Recommendations
  *  Invest more marketing spend in Direct and Google channels, both of which produce high traffic and revenue with good bounce rates.
  *  Address YouTube, Facebook, and other referral channel high bounce rates with landing page optimization and targeting more intentful segments.
  *  Deploy product bundles or recommendation widgets on highly co-bought products to increase average order value.
  *  Segment user behavior deeper to target marketing efforts more specifically.
