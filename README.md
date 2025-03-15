# Data Cleaning with SQL - Alex Freberg's Tutorial

This project follows **Alex Freberg's YouTube tutorial** on data cleaning using SQL. I applied various SQL techniques to clean and structure the dataset.

## Project Overview
- Performed data cleaning using **SQL** in **MySQL Workbench**.
- Faced an issue where only **564 rows** were imported instead of the full dataset.
- Resolved the issue by manually importing the file and using 'LOAD DATA LOCAL INFILE'.

## Dataset
- The dataset used in this project is the **same dataset from Alex Freberg's tutorial**.
- To follow along, you can check it on his channel [here](https://www.youtube.com/@AlexTheAnalyst).

## Tools Used
- **SQL (MySQL)**
- **MySQL Workbench**

## Issues & Solutions

### **Problem:**
The dataset has 2361 rows (without the column names), but when importing the dataset into MySQL, only **564 rows** were imported instead of the full dataset.

<img width="455" alt="Image" src="https://github.com/user-attachments/assets/d9c90f51-d422-4618-8d02-214781af40a2" />

### **Solution:**
After researching the issue, I found that using 'LOAD DATA LOCAL INFILE' was an effective way to import the full dataset. Before using this command, I **truncated the table** to avoid duplicate entries.

```sql
LOAD DATA LOCAL INFILE '/my/file/path.csv'
INTO TABLE my_table
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

```

<img width="455" alt="Image" src="https://github.com/user-attachments/assets/93959d6e-dcc5-4549-b1d9-0db1c65843ba" />

## Steps Taken
1. **Truncated the table** before re-importing locally:
```sql
TRUNCATE TABLE my_table;
```

2. Used LOAD DATA LOCAL INFILE to import the dataset successfully.
3. Verified the row count:
```sql
SELECT COUNT(*) FROM my_table;
```

## Data Cleaning Steps
### 1. Remove Duplicates:
To ensure the data is clean and unique, duplicates were removed by using the ROW_NUMBER() function along with the PARTITION BY clause. This allowed us to identify rows with identical values for key columns and delete the duplicates.

```sql
-- Remove duplicates by partitioning the data based on key columns (e.g., company, industry, etc.)
WITH duplicate_cte AS (
  SELECT *,
  ROW_NUMBER() OVER(
  PARTITION BY company, location, 
  industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
  FROM world_layoffs.layoffs_staging
)
-- Delete rows where the row number is greater than 1 (i.e., duplicates)
DELETE FROM duplicate_cte
WHERE row_num > 1;
```

### 2. Standardize Text Formatting:
Text formatting inconsistencies were corrected by trimming leading and trailing spaces using TRIM(). Additionally, we standardized specific columns like industry by updating values to ensure uniformity.

```sql
-- Trim leading and trailing whitespace from the 'company' column
UPDATE layoffs_staging2
SET company = TRIM(company);

-- Standardize 'industry' values, e.g., change 'Crypto%' to 'Crypto'
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
### 3. Fix Inconsistent Date Formats:
To ensure all dates follow a consistent format, we used STR_TO_DATE() to convert the date strings into a standard DATE type format. The column type was then modified to ensure proper date handling.

```sql
-- Convert the 'date' column to a consistent DATE format using STR_TO_DATE()
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

-- Modify the 'date' column to ensure it's stored as a DATE type
ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```
### 4. Handle Missing Values:
Missing or blank values were handled by setting empty fields to NULL using UPDATE statements. Additionally, rows with NULL values in key columns were deleted to maintain data integrity.

```sql
-- Set blank values in the 'industry' column to NULL
UPDATE layoffs_staging2
SET industry = NULL
WHERE industry = '';

-- Delete rows where both 'total_laid_off' and 'percentage_laid_off' are NULL
DELETE FROM layoffs_staging2
WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;
```
### 5. Remove Unneccessary Columns:
Temporary or unnecessary columns, such as the row_num column used for identifying duplicates, were dropped after they had served their purpose. This helps in cleaning up the table for final use.

```sql
-- Drop the 'row_num' column after processing
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

## After Cleaning

After cleaning the dataset. I used the below command to show the cleaned data COUNT().

<img width="455" alt="Image" src="https://github.com/user-attachments/assets/d378732e-446f-494d-ae34-b425d0c33523" />

## Key Learnings
- How to troubleshoot SQL import issues in MySQL Workbench.
- Hands-on experience with SQL **SQL sata cleaning techniques**.
- Undersatnding **proper data formatting** before analysis

## How to Use This Project
1. You can use to dataset named layoff.csv or you can download the dataset from [**Alex Freberg's tutorial**](https://youtu.be/4UltKCnnnTA?si=yubPoghpmCJOzFyD).
2. Create a table in MySQL matching the dataset structure.
3. Use LOAD DATA LOCAL INFILE to import the dataset.
4. Apply the data cleaning SQL queries to transform the data.
