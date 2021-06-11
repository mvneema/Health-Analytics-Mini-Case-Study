# Health-Analytics-Mini-Case-Study

### This is a mini case study project from the course Serious SQL
<img alt="alternate text goes here" src="SQL.png" width="60%" />

We’ve just received an urgent request from the General Manager of Analytics at Health Co requesting our assistance with their analysis of the health.user_logs dataset.

The Health Co analytics team have shared with us their SQL script - they unfortunately ran into a few bugs that they couldn’t fix!

We’ve been asked to quickly debug their SQL script and use the resulting query outputs to quickly answer a few questions that the GM has requested for a board meeting about their active users.



### Debug SQL Script
<img alt="alternate text goes here" src="debug.jpg" width="60%" />

### 1. How many unique users exist in the logs dataset?

```sql
SELECT
  COUNT DISTINCT user_id
FROM health.user_logs;
```

When you execute the SQL query above, we get the following errors:
1. **we get an error syntax error at or near "DISTINCT"**
2. **Also  no column name user_id** 

**Correction:** Put brackets after **COUNT** & Replace the **user_id** column to **id**

**SOLUTION:**
```sql
SELECT
  COUNT(DISTINCT id)
FROM health.user_logs;
```

| COUNT(DISTINCT id) |
| ------------------ |
| 554                | 

### for questions 2-8 we created a temporary table
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_cout
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1; 
  ```

### 2. How many unique users exist in the logs dataset?
```sql
SELECT
  ROUND(MEAN(measure_count))
FROM user_measure_count;
```
**ERROR:** Error in creating temp table "syntax error at or near "Select""

**Correction:** **AS** is missed and table name is not properly named
**SOULTION:**
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count AS
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1;
```
**Error for question no. 2** "function mean(bigint) does not exist"

**Correction:** Replace **MEAN** with **AVG** in SQL query
 **SOLUTION:**
 ```sql
  SELECT
  ROUND(AVG(measure_count)) AS average_value
FROM user_measure_count;
```
| average_value |
| ------------- |
| 79            | 
### 3. What about the median number of measurements per user?
```sql
SELECT
  PERCENTILE_CONTINUOUS(0.5) WITHIN GROUP (ORDER BY id) AS median_value
FROM user_measure_count;
```
**ERROR:** function percentile_continuous(numeric, character varying) does not exist

**CORRECTION:** Use a numeric column in percentile function
**Solution: 2**
 ```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_value
FROM user_measure_count;
```
| median_value |
| ------------ |
| 2            | 
### 4. How many users have 3 or more measurements?
```sql
SELECT
  COUNT(*)
FROM user_measure_count
HAVING measure >= 3;
```
**ERROR:** column "measure" does not exist
**CORRECTION:** Replace the **HAVING** clause with **WHERE** and give the correct column name
**Solution: 209**
```SQL
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;
```

### 5. How many users have 1,000 or more measurements?
```sql
SELECT
  SUM(id)
FROM user_measure_count
WHERE measure_count >= 1000;
```
**ERROR:** "Syntax error at or near "SUM"

**CORRECTION:** We have to replace **SUM**to **COUNT**
**Solution: 5**
```sql
SELECT
  count(id)
FROM user_measure_count
WHERE measure_count >= 1000;
```
| count(id) |
| --------- |
| 5         | 
### 6. Have logged blood glucose measurements?
```sql 
SELECT
  COUNT DISTINCT id
FROM health.user_logs
WHERE measure is 'blood_sugar';
```
**ERROR:** syntax error at or near "DISTINCT"

**CORRECTION:** Put barckets in distinct keyword and WHERE clause needs an "=" sign instead of 'is' and column value to be searched is "blood_glucose" instead of "blood_sugar"
**Solution: 325**
```sql
SELECT
  COUNT(DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';
```
| COUNT(DISTINCT id) |
| ------------------ |
| 325                | 
### 7. Have at least 2 types of measurements?
```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE COUNT(DISTINCT measures) >= 2;
```
**ERROR:** Column "measures" does not exist

**CORRECTION:** Replace column with correct column value **"unique_measures"**
**SOLUTION : 204**
```sql
SELECT
  COUNT(*) as count_measure
FROM user_measure_count
WHERE unique_measures >= 2;
```
| count_measure |
| ------------- |
| 204           | 
### 8. Have all 3 measures - blood glucose, weight and blood pressure?
```sql
SELECT
  COUNT(*)
FROM usr_measure_count
WHERE unique_measures = 3;
```
**ERROR:** relation "usr_measure_count" does not exist

**CORRECTION:** replace with correct table name
**SOLUTION: 50**
```sql
SELECT
  COUNT(*) as count_measure
FROM user_measure_count
WHERE unique_measures = 3;
```
| count_measure |
| ------------- |
| 50            | 
### 9.  What is the median systolic/diastolic blood pressure values?
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;
```
**ERROR:** syntax error at or near "(" 

**CORRECTION:** Seems like there is a comma missing between the median columns and **GROUP** keyword and where clause should use "=" instead of "is" for value 'blood_pressure'
**Solution:**
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure = 'blood_pressure';
```
|  median_systolic  | median_diastolic  | 
| ----------------- |:-----------------:|
| 126               | 79                |

