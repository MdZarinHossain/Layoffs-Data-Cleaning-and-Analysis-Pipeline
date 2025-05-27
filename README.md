# Layoffs Data Cleaning and Analysis Pipeline

This project contains SQL scripts to clean, deduplicate, standardize, and analyze a dataset of company layoffs. It is designed to ensure data quality before performing exploratory data analysis (EDA) on layoffs information.

---

## 🗂 Dataset Description

The dataset consists of a table named `layoffs` with the following columns:

- `company` — Name of the company
- `location` — Location of the layoff event
- `industry` — Industry sector of the company
- `total_laid_off` — Number of employees laid off
- `percentage_laid_off` — Percentage of workforce laid off
- `date` — Date of the layoff event
- `stage` — Company stage (e.g., seed, Series A, etc.)
- `country` — Country where the layoff occurred
- `funds_raised_millions` — Funding raised in millions (if applicable)

---

## ⚙️ Data Cleaning & Deduplication

1. **Create staging tables:**
   - Clone the structure of the original `layoffs` table to `layoffs_staging` and `layoffs_staging2` for intermediate cleaning steps.

2. **Populate staging table:**
   - Insert all data from `layoffs` into `layoffs_staging`.

3. **Identify duplicates:**
   - Use `ROW_NUMBER()` window function partitioned by key fields (company, location, industry, total layoffs, percentage laid off, date, stage, country, funds raised) to mark duplicates.

4. **Delete duplicates:**
   - Remove rows where the duplicate row number is greater than 1.

5. **Standardize text fields:**
   - Trim leading/trailing whitespace in `company`.
   - Normalize `industry` values, e.g., replace variants like 'Crypto%' with 'Crypto'.
   - Clean trailing periods from `country` names, especially for the 'United States'.
   - Convert `date` strings into proper SQL DATE format.

6. **Fill missing industries:**
   - For rows with missing or empty `industry`, update them with non-null industry values from the same company and location.

7. **Remove rows missing key numeric layoff info:**
   - Delete records where both `total_laid_off` and `percentage_laid_off` are NULL.

8. **Drop helper columns:**
   - Remove the `row_num` column after deduplication.

---

## 📊 Exploratory Data Analysis (EDA)

- Get max values for total layoffs and percentage laid off.
- Find companies with 100% layoffs sorted by funding.
- Aggregate total layoffs by company.
- Aggregate layoffs monthly and compute rolling totals.
- Rank companies by layoffs per year, displaying top 5 per year.

---

## 🧾 Example Queries

- Deduplication row number:

```sql
WITH duplicate_cte AS (
  SELECT *, ROW_NUMBER() OVER (
    PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
  ) AS row_num
  FROM layoffs_staging
)
SELECT * FROM duplicate_cte WHERE row_num > 1;

## Update industry for missing rows:

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
  ON t1.company = t2.company AND t1.location = t2.location
SET t1.industry = t2.industry
WHERE (t1.industry IS NULL OR t1.industry = '')
  AND t2.industry IS NOT NULL;

## Aggregate total layoffs monthly:

SELECT SUBSTRING(`date`, 1, 7) AS `MONTH`, SUM(total_laid_off)
FROM layoffs_staging2
WHERE SUBSTRING(`date`, 1, 7) IS NOT NULL
GROUP BY `MONTH`
ORDER BY `MONTH` ASC;


