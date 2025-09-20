# 📊 SQL Project: World Layoffs Data (2020–2023)

**Dataset:** [Kaggle – Layoffs 2022](https://www.kaggle.com/datasets/swaptr/layoffs-2022)

This project focuses on **cleaning and analyzing global layoffs data** using MySQL. The raw dataset (2020–2023) contained duplicates, inconsistencies, and missing values. I built a structured SQL workflow to **clean the dataset** and then performed **exploratory analysis** to extract insights about industries, companies, countries, and time trends.

---

## 🔹 Project Objectives

* Practice **SQL data cleaning techniques** (duplicates, standardization, null handling).
* Prepare a cleaned dataset for **exploratory data analysis (EDA)**.
* Extract **insights and trends** about global layoffs.
* Demonstrate **SQL query writing and problem-solving skills**.

---

## 🛠️ Data Cleaning Steps

1. **Create Staging Table**
   Work in a copy of the raw dataset to preserve original data.

2. **Remove Duplicates**

   * Used `ROW_NUMBER()` with `PARTITION BY` to detect duplicate rows.
   * Deleted exact duplicates while retaining legitimate entries.

   ```sql
   WITH DELETE_CTE AS (
     SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
            ROW_NUMBER() OVER (
              PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
            ) AS row_num
     FROM layoffs_staging
   )
   DELETE FROM layoffs_staging
   WHERE row_num > 1;
   ```

3. **Standardize Data**

   * Fixed inconsistent **industry labels** (`Crypto Currency` → `Crypto`).
   * Removed trailing punctuation from **country names** (`United States.` → `United States`).
   * Converted **string dates → DATE type** using `STR_TO_DATE()`.

4. **Handle Nulls**

   * Replaced blanks with `NULL`.
   * Populated missing `industry` values where possible by joining on company names.
   * Left some nulls as-is for accurate EDA.

5. **Remove Useless Data**

   * Dropped rows where both `total_laid_off` and `percentage_laid_off` were `NULL`.
   * Dropped helper columns (e.g., `row_num`) after cleaning.

✅ Result: A **clean, analysis-ready dataset** stored in `layoffs_staging2`.

---

## 📊 Exploratory Data Analysis (EDA)

After cleaning, I explored the dataset to identify patterns, trends, and outliers.

### 🔹 Basic Insights

* **Largest Single Layoff Events**

  ```sql
  SELECT company, total_laid_off
  FROM layoffs_staging2
  ORDER BY total_laid_off DESC
  LIMIT 5;
  ```

* **Companies with the Most Total Layoffs**

  ```sql
  SELECT company, SUM(total_laid_off) AS Total_laid_offs
  FROM layoffs_staging2
  GROUP BY company
  ORDER BY 2 DESC
  LIMIT 10;
  ```

* **Layoffs by Country**

  ```sql
  SELECT country, SUM(total_laid_off)
  FROM layoffs_staging2
  GROUP BY country
  ORDER BY 2 DESC;
  ```

* **Layoffs by Year**

  ```sql
  SELECT YEAR(date), SUM(total_laid_off)
  FROM layoffs_staging2
  GROUP BY YEAR(date)
  ORDER BY 1 ASC;
  ```

* **Layoffs by Industry**

  ```sql
  SELECT industry, SUM(total_laid_off)
  FROM layoffs_staging2
  GROUP BY industry
  ORDER BY 2 DESC;
  ```

---

### 🔹 Advanced Insights

* **Top 3 Companies with Most Layoffs per Year**

  ```sql
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
  ```

* **Rolling Monthly Layoffs**

  ```sql
  WITH DATE_CTE AS (
    SELECT SUBSTRING(date,1,7) as dates, SUM(total_laid_off) AS total_laid_off
    FROM layoffs_staging2
    GROUP BY dates
    ORDER BY dates ASC
  )
  SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) as rolling_total_layoffs
  FROM DATE_CTE
  ORDER BY dates ASC;
  ```

---

## 📌 Key Findings

* Several startups experienced **100% layoffs**, effectively shutting down.
* **2022–2023** saw a sharp spike in global layoffs.
* The **United States** had the highest total layoffs.
* **Tech industries** (Consumer, Retail, and Crypto) were most affected.
* Later-stage companies (Series C and beyond) faced larger layoffs compared to early-stage startups.

---

## 🔹 Repository Structure

```
world-layoffs-sql-project/
├── data/ 
│   └── layoffs.csv                 # (optional) dataset
├── scripts/
│   ├── data_cleaning.sql           # data cleaning queries
│   └── exploratory_analysis.sql    # EDA queries
├── diagrams/                       # (optional) schema or ERD
└── README.md                       # project documentation
```
✅ **This project demonstrates my ability to clean, transform, and analyze real-world data using SQL.**
🚀 Next Steps

# To extend this project, I plan to:

* Connect MySQL to Python using pandas and SQLAlchemy for seamless querying.
* Visualize trends with libraries like matplotlib and seaborn (e.g., layoffs by year, industry, or country).
* Build an interactive dashboard using Tableau or Power BI for business storytelling.
* Experiment with predictive analysis (e.g., forecasting layoffs by industry) using machine learning models.

