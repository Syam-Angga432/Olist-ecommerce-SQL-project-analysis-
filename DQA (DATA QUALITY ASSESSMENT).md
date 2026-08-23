# DQA (DATA QUALITY ASSESSMENT)
* DATA QUALITY ASSESSMENT (DQA) - OLIST E-COMMERCE DATASET
* Database : olist_project
* Tool     : PostgreSQL
  
  Tahapan DQA:
  1. Data Overview (Cek sampel & jumlah baris)
  2. Missing Values Check (Cek kolom ber-NULL)
  3. Duplication Check (Cek duplikasi pada Primary Key)
  4. Data Validity Check (Cek logika & keabsahan nilai)
  
## STRUKTUR QUERY UNTUK TAHAP DQA

*1. Overview*

SELECT * FROM <table_name> LIMIT 10;
SELECT COUNT(*) AS total_rows FROM <table_name>;

*-- 2. Missing Values (PostgreSQL Trick)*
SELECT 
    COUNT(*) AS total_rows,
    COUNT(*) - COUNT(<column_1>) AS <column_1>_null_count,
    COUNT(*) - COUNT(<column_2>) AS <column_2>_null_count
FROM <table_name>;

-- 3. Duplication Check
SELECT COUNT(*) - COUNT(DISTINCT <primary_key>) AS duplicate_pk_count
FROM <table_name>;

-- 4. Data Validity Check
SELECT COUNT(*) FILTER (WHERE <column_numeric> < 0) AS invalid_negative_values
FROM <table_name>;
