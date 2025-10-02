# I. Project Overview
The project goal is to build a stock data processing pipeline from multiple sources (raw/historical):
(1) Clean and standardize data (data format, numbers, K/M notation) 
(2) Check data quality (missing values, duplicates, outliers, consistency) 
(3) Statistical analysis, trends, and relationships 
(4) Create a foundation for the closing price (close_price) prediction model

Key Questions:
1. Data Quality & Processing 
*   How much preprocessing or standardization does raw data require before analysis?
        * Check datatype, missing values, duplicates
        * Standardize column names, add primary key, save clean file 

* Is the raw data consistent with the historical data based on % change? 

2. Exploratory Analysis
*  Which trading days are outliers based on % change?
*  Volatility trends of each stock ticker over time? 
         * Stock-level trend (individual ticker performance)
         * Sector-level trend (Bank vs Non-bank)
         * Advanced analysis (rolling average & volatility)

3. Relationship & Modeling 
*  Which factor affects the closing price the most? 
*  Does the predicted vs actual result of Linear Regression fit well? 


# II. Approach



