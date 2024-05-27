# MySQL-DataCleaning

-- SQL Project - Data Cleaning

-- Data source: https://www.kaggle.com/datasets/swaptr/layoffs-2022

SELECT * FROM world_layoffs.layoffs;

-- Initial step is to establish a staging table for working on and cleansing the data, ensuring the original data remains intact for backup purposes.
CREATE TABLE world_layoffs.layoffs_staging LIKE world_layoffs.layoffs;

INSERT INTO layoffs_staging SELECT * FROM world_layoffs.layoffs;

-- Data cleaning involves several key steps:
-- 1. Identify and eliminate duplicate records
-- 2. Normalize and correct data inconsistencies
-- 3. Evaluate and address missing values
-- 4. Discard unnecessary columns and rows

-- Step 1: Removing Duplicates

-- Checking for duplicates in the data

SELECT * FROM world_layoffs.layoffs_staging;

-- Identifying duplicates based on key fields

SELECT * FROM (SELECT company, industry, total_laid_off, `date`, ROW_NUMBER() OVER (PARTITION BY company, industry, total_laid_off, `date`) AS row_num FROM world_layoffs.layoffs_staging) duplicates WHERE row_num > 1;

-- Verify entries for specific companies to ensure accuracy before deletion

SELECT * FROM world_layoffs.layoffs_staging WHERE company = 'Oda';

-- Identifying absolute duplicates across more detailed attributes

SELECT * FROM (SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions, ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num FROM world_layoffs.layoffs_staging) duplicates WHERE row_num > 1;

-- Deleting confirmed duplicates using a common table expression (CTE)

WITH DELETE_CTE AS (SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions, ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num FROM world_layoffs.layoffs_staging) DELETE FROM world_layoffs.layoffs_staging WHERE (company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions, row_num) IN (SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions, row_num FROM DELETE_CTE) AND row_num > 1;

-- Adding a row number column for easier management of duplicates

ALTER TABLE world_layoffs.layoffs_staging ADD row_num INT;

-- Step 2: Standardizing Data

-- Identifying unique industry entries and correcting any inconsistencies or blank entries

UPDATE world_layoffs.layoffs_staging2 SET industry = NULL WHERE industry = '';

-- Standardizing entries in the industry field to maintain uniformity across records

UPDATE layoffs_staging2 SET industry = 'Crypto' WHERE industry IN ('Crypto Currency', 'CryptoCurrency');

-- Removing trailing periods from country names to ensure consistency

UPDATE layoffs_staging2 SET country = TRIM(TRAILING '.' FROM country);

-- Updating date format for consistency across the dataset

UPDATE layoffs_staging2 SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

-- Step 3: Addressing Null Values

-- Maintaining null values in certain fields for easier data analysis later

-- Step 4: Removing Redundant Data

-- Deleting rows where specific data points like total_laid_off and percentage_laid_off are both missing, as they offer no analytical value

DELETE FROM world_layoffs.layoffs_staging2 WHERE total_laid_off IS NULL AND percentage_laid_off IS NULL;

-- Cleaning up by removing the temporary row_num column

ALTER TABLE layoffs_staging2 DROP COLUMN row_num;
