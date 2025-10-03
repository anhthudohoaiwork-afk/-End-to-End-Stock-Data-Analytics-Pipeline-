# I. Project Overview
The project goal is to build a stock data processing pipeline from multiple sources (raw/historical):
(1) Clean and standardize data (data format, numbers, K/M notation) 
(2) Check data quality (missing values, duplicates, outliers, consistency) 
(3) Statistical analysis, trends, and relationships 
(4) Create a foundation for the closing price (close_price) prediction model

**Key   Questions:**
**1. Data Quality & Processing** 
*   How much preprocessing or standardization does raw data require before analysis?

        _ Check datatype, missing values, duplicates
        _ Standardize column names, add primary key, save clean file 

* Is the raw data consistent with the historical data based on % change? 

**2. Exploratory  Analysis**
*  Which trading days are outliers based on % change?
*  Volatility trends of each stock ticker over time?
  
         _ Stock-level trend (individual ticker performance)
         _ Sector-level trend (Bank vs Non-bank)
         _ Advanced analysis (rolling average & volatility)

**3. Relationship & Modeling**
*  Which factor affects the closing price the most? 
*  Does the predicted vs actual result of Linear Regression fit well? 


# II. Approach

![Flowchart](https://raw.githubusercontent.com/anhthudohoaiwork-afk/-End-to-End-Stock-Data-Analytics-Pipeline-/main/images/images/Flowchart.png)

# III. Tech Stack Used

*  **Spreadsheet (Google Sheets/ Excel):** Initially used for manual data cleaning (e.g., converting Volume from K/M notation to numeric values, formatting Pct_change with formulas.)
*  **SQLite:** Imported the cleaned dataset for running basic SQL queries
*  **Python (Pandas, Numpy):** Later replaced manual cleaning with Python scripts to automate preprocessing and handle data at scale (duplicates, missing values, outliers, data type conversion)
*  **MySQL:** Designed and created a dedicated database to store cleaned datasets and perform advanced queries
*  **Visualization:** Matplotlib and Seaborn for exploratory data analysis (EDA); Looker Studio for interactive dashboards
*  **Modeling:** Scikit-learn (Linear Regression, Pipeline) applied to validate relationships between input variables and closing price

# IV. Data Processing
**1. Data Collection & Overview**
*    *Dataset:* 6 files (3 raw, 3 historical) for VCB, VIB, VIC
*    *Key columns (file raw):* time, open, high, low, close, volume, ticker
*    *Key columns (file hist):* Ngày, Đóng cửa, Mở cửa, Cao nhất, Thấp nhất, Khối lượng, % Thay đổi

**2. Handling Duplicate Values**

*  **Check duplicates across all datasets:** Checked duplicates across all 6 files csv (raw/ historical) for all 3 stock tickers (VCB, VIB, VIC). No duplicate rows were found, however, when checking the specific column “Date”, all 3 raw files showed duplicate entries on Date “2022-12-27”, with Open/High/Low/Close values showing minor (insignificant) differences, but large volume discrepancies. 

*   **Solution:** Drop duplicates, keep the last row (prioritizing the row with higher volume). For the data to be joinable in both raw and historical files, duplicates must be removed, keeping the row with the larger volume (higher volume usually reflects the full and valid total trading volume (higher volume usually reflects the full and valid total trading volume) 

**3. Handling Missing Values**

_ Checked using df.isnull().sum() → no missing values detected

**4.  Error Correction & Outlier Handling**
*   **Error Correction:** Historical file – The Volume column has K, M notation in str/object format. To facilitate calculation queries, convert to numeric format (K corresponds to e3, M corresponds to e6). Example: 11.57M = 11570000, 996.80K = 996.8. Pct_change from % → float. Example: 0.33% = 0.0033.

*    **Outlier Detection:** Calculate the mean of pct change, stddev of pct change. Then find the threshold to filter out outlier days.

     _ mean(pct_change) + 2*std(pct_change) → threshold1
     _ mean(pct_change) -2*std(pct_change) → threshold2
     _ Pct change > threshold1 ~ Positive outlier
     _ Pct change < threshold 2 ~ Negative outlier 

*  **Result:** Most days  = Normal, some days = Outlier

*  **Visualization:** Bar chart of outlier days

![Distribution of Outlier Days](https://raw.githubusercontent.com/anhthudohoaiwork-afk/-End-to-End-Stock-Data-Analytics-Pipeline-/main/images/images/Distribution%20of%20Outlier%20Days.png)


#  V. Analysis and Results
**1. Section 1: Data Validation** 

*  **Business Question:** Is the % change in the raw and historical files consistent?
*  **Approach (Key Steps):**

   _ Calculate % change from raw data using LAG(close_price)
   _ Compare with the existing % change in the historical file
   _ Measure the difference to detect inconsistency

*  **SQL Snippet:** *“Full query available on Colab/ Appendix”*
*  **Output:** The distribution of the diff is close to 0 (for all 3 tickers, with VIC having almost no different) → data in raw and historical files is mostly consistent, with only a few minor (insignificant) discrepancies


**2. Section 2: Stock-level trend** 

*  **Business Question:** Which day did the stock increase/ decrease the most?
*  **Approach (Key steps):**

       _ Calculate % change and daily change using LAG(close_price)
       _ Use ROW_NUMBER() to rank up and rank down for each stock ticker

*  **SQL Snippet:** *“Full query available on Colab/ Appendix”*
*  **Output:** Top 5 days the stocks (VCB, VIB, VIC) increased/decreased the most

*  **Business Question:** Total trading volume monthly/quarterly?
*  **Approach (Key steps):**

           _ Calculate total volume by month using GROUP BY year, month
           _ Calculate total volume by quarter, use CASE WHEN and CONCAT(‘Q1-’, Year) to label the quarter. Sort by ticker, year and quarter_label 

*  **SQL Snippet:** *“Full query available on Colab/ Appendix”*

*  **Business Question:** How does the % change fluctuate across quarters?
*  **Approach (Key Steps):**

        _ Format to quarter, use FLOOR((month-1)/3) +1
        _ Use MAX to find the last trading day of the quarter
        _ Use JOIN to retrieve the closing price of that quarter’s last day
        _ Calculate % change using LAG(quarter_close_price)
        _ Use CASE WHEN to label the quarter 

*  **SQL Snippet:**  *“Full query available on Colab/ Appendix”*
*  **Output:** AVG of % change by quarter


**3. Section 3: Sector-level trend** 

 **Business Question:** Total trading volume of the sector, and the percentage contribution to the sector? 
 **Approach (Key steps):**


              _ JOIN the metadata table with the stock_prices_raw table to get the Sector, use GROUP BY sector
              to calculate the total trading volume of the industry  
              _ Create a CTE calculating volume by ticker and volume by sector 

*  **SQL Snippet:** *“Full query available on Colab/ Appendix”*
*  **Output:** VIB has the largest trading volume among the 2 bank sector stocks, with an insignificant difference compared to the leading ticker, VIC.  


 **Business Question:** Which sector has better performance monthly/quarterly?
**Approach (Key steps):**

               _ JOIN to retrieve the sector from the metadata table (stocks)
               _ Calculate daily change and % change using LAG(close_price)

 **SQL Snippet:** *“Full query available on Colab/ Appendix”*
 **Output:** 

              _ AVG of daily change by sector 
              _ AVG of % change by sector 


**4. Section 4:  Advanced Analysis**

 **Business Question:** Identify outlier days, volatility?
 **Approach (Key steps):**  

             _ Determine the threshold by calculating the mean and std of % change
             _ Use CASE WHEN to filter out outlier days 

 **SQL Snippet:** *“Full query available on Colab/ Appendix”*

**Output:** Most are normal days, with only a few Positive increase or Negative decrease days in the Q1-2020 period (for all 3 tickers).


**Descriptive Statistics**

**Business Question:** Basic statistics (Mean, Median, Std for Open/Close/Volume)? 
**Approach (Key steps):**

               _ Filter data by each stock ticker
               _ Use .describe() to get the basic statistical indicators

 
**Python  Snippet:** *“See Appendix or full script here for complete code”*

**Output:** Table of statistics including count, mean, std, min, max, quartiles for each stock ticker


**Correlation Analysis**

**Business Question:** Which factor has the strongest correlation with the closing price?   
**Approach (Key steps):** 

                _ Select independent vars: Open, High, Low, Volume, Pct_change
                _ Calculate correlation with Close price using .corrwith() and .corr()

**Python  Snippet:** *“See Appendix or full script here for complete code”*

**Output:** Open/ high/ low price have corr > 0.99 with the closing price, almost absolute


**Prediction Model**

**Business Question:** Can the closing price (Close Price) be predicted from the variables Open, High, Low? 
**Approach (Key steps):** 

                _ Select input variables (based on corr): open_price, high_price, low_price
                _ Use a Pipeline consisting of StandardScaler() for data standardization and LinearRegression()
                for training 
                _ Train for each stock ticker and evaluate using R^2 and MSE
                _ Compare Actual vs Predicted using a distribution chart 


**Python  Snippet:** *“See Appendix or full script here for complete code”*

**Output:** 

        *  VCB - R^2 ≈ 0.997, MSE ≈ 0.34
        *  VIB - R^2 ≈ 0,998, MSE ≈ 0.08
        *  VIC - R^2 ≈ 0.998, MSE ≈ 0.71 

**Result:** High R², the model explains Close Price well



# VI. Insights and Visualization

**1. Data Quality and Validation**


![Consistency Check](https://raw.githubusercontent.com/anhthudohoaiwork-afk/-End-to-End-Stock-Data-Analytics-Pipeline-/main/images/images/Consistency%20Check.png)

![Distribution of pct_change](https://raw.githubusercontent.com/anhthudohoaiwork-afk/-End-to-End-Stock-Data-Analytics-Pipeline-/main/images/images/Distribution%20of%20pct_change%20.png)

