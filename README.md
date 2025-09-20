# 📌 SQL Project: World Layoffs Data (2020–2023)

Dataset: Kaggle – Layoffs 2022(https://www.kaggle.com/datasets/swaptr/layoffs-2022)

🔹 Project Objective

The goal of this project was to clean and standardize a real-world dataset on global layoffs from 2020–2023. The cleaned data was later used for Exploratory Data Analysis (EDA) to identify industry trends, country-wise layoffs, and patterns over time.

🔹 Skills Demonstrated

* Creating staging tables for safe data cleaning
* Detecting and removing duplicates using ROW_NUMBER()

## Standardizing data:
* Fixing inconsistent industry labels (e.g., “Crypto Currency” → “Crypto”)
* Cleaning country names (United States. → United States)
* Converting string dates → SQL DATE format
* Handling NULL values and choosing whether to impute or retain
* Removing unusable rows with missing key values
* Writing update queries with JOIN to propagate missing values
* Designing a reproducible workflow for data cleaning in SQL

🔹 SQL Highlights

Examples of key queries:
-- Detect duplicates using ROW_NUMBER
SELECT company, location, industry, total_laid_off, `date`,
       ROW_NUMBER() OVER (
         PARTITION BY company, location, industry, total_laid_off, `date`
       ) AS row_num
FROM layoffs_staging;

-- Standardize inconsistent industry labels
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');

-- Convert string dates into DATE format
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

## Results
* Cleaned dataset with consistent values and correct data types
* Ready for analysis of industry-wise layoffs, country-level trends, and funding correlations

# 📊 Exploratory Data Analysis (EDA)

After cleaning, I performed exploratory analysis to identify patterns, trends, and anomalies in global layoffs.

## 🔹 Key Findings
* Largest Layoffs (single events):
SELECT company, total_laid_off
FROM layoffs_staging2
ORDER BY total_laid_off DESC
LIMIT 5;
→ Revealed the top 5 biggest single layoff events.

* Companies with the Most Layoffs (overall):
  SELECT company, SUM(total_laid_off) AS Total_laid_offs
FROM layoffs_staging2
GROUP BY company
ORDER BY 2 DESC
LIMIT 10;
→ Shows which companies were most impacted across 2020–2023.

* Layoffs by Location and Country:
SELECT country, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY country
ORDER BY 2 DESC;
→ Countries like the United States dominate total layoffs.

* Trends by Year:

SELECT YEAR(date), SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY YEAR(date)
ORDER BY 1 ASC;


→ Sharp increases in layoffs visible in 2022–2023.

* Industry Breakdown:

SELECT industry, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY industry
ORDER BY 2 DESC;


→ Tech-driven industries like Consumer, Retail, and Crypto had the highest layoffs.

* Funding Stage Impact:

SELECT stage, SUM(total_laid_off)
FROM layoffs_staging2
GROUP BY stage
ORDER BY 2 DESC;


→ Later-stage companies and unicorns faced larger layoffs compared to early-stage startups.

## 🔹 Advanced Queries

* Top 3 Companies with Most Layoffs per Year
Using CTEs + window functions:

WITH Company_Year AS (
  SELECT company, YEAR(date) AS years, SUM(total_laid_off) AS total_laid_off
  FROM layoffs_staging2
  GROUP BY company, YEAR(date)
),
Company_Year_Rank AS (
  SELECT company, years, total_laid_off,
         DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
  FROM Company_Year
)
SELECT company, years, total_laid_off, ranking
FROM Company_Year_Rank
WHERE ranking <= 3
AND years IS NOT NULL
ORDER BY years ASC, total_laid_off DESC;


* Rolling Monthly Layoffs
Shows cumulative layoffs over time:

WITH DATE_CTE AS (
  SELECT SUBSTRING(date,1,7) as dates, SUM(total_laid_off) AS total_laid_off
  FROM layoffs_staging2
  GROUP BY dates
  ORDER BY dates ASC
)
SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) as rolling_total_layoffs
FROM DATE_CTE
ORDER BY dates ASC;

## 📌 Insights

* Many startups had 100% layoffs, essentially shutting down.
* 2022–2023 saw the largest surge in global layoffs.
* Countries like the United States led both in total and percentage layoffs.
* The Tech industry, especially Crypto, was disproportionately affected.
* Later-stage companies (Series C and above) had the largest layoff numbers
