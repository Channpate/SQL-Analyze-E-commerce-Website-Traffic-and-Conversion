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
  
  ### Query 2
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
  #### Observations & Findings:
  
  ### Query 3
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
  #### Observations & Findings:
  
  ### Query 4
  #### Code:
  ``` sql
  ```
  #### Result:
  #### Observations & Findings:
  
  ### Query 5
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
  #### Observations & Findings:
  
  ### Query 6
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
  #### Observations & Findings:
  
  ### Query 7
  #### Code:
  ``` sql
  ```
  #### Result:
  #### Observations & Findings:
  
  ### Query 8
  #### Code:
  ``` sql
  ```
  #### Result:
  #### Observations & Findings:
  
  ## 🔎 Final Conclusion & Recommendations
