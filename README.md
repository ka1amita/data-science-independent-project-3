# Project Tasks

## Basic Requirements
Let’s break this project down into a couple different parts.

- [X] Explore tables: Start off by exploring each table separately.

- [X] How many public high schools are in each zip code?
```sql
SELECT zip_code, count(*)
FROM public_hs_data
GROUP BY zip_code
ORDER BY count(*) DESC
```

- [X] ...in each state?

```sql
SELECT state_code, count(*)
FROM public_hs_data
GROUP BY state_code
ORDER BY count(*) DESC
;
```

The locale_code column in the high school data corresponds to various levels of urbanization as listed below. Use the CASE statement to display the corresponding locale_text and locale_size in your query result.[^1]
[^1]: Try taking a look at using the substr() function to help look at each part of the locale_code for determining locale_text and locale_size.
locale_text	locale_code (locale_size)
City	11 (Large), 12 (Midsize), 13 (Small)
Suburb	21 (Large), 22 (Midsize), 23 (Small)
Town	31 (Fringe), 32 (Distant), 33 (Remote)
Rural	41 (Fringe), 42 (Distant), 43 (Remote)

```sql
SELECT locale_code, SUBSTR(locale_code,1,1),
	CASE SUBSTR(locale_code,1,1)
	  WHEN '1' THEN 'City'
	  WHEN '2' THEN 'Suburb'
	  WHEN '3' THEN 'Town'
	  WHEN '4' THEN 'Rural'
	END AS locale_text
FROM public_hs_data
LIMIT 10
;
```
or

```sql
SELECT locale_code,
CASE
	WHEN locale_code like '1%' THEN 'City'
	WHEN locale_code like '2%' THEN 'Suburb'
	WHEN locale_code like '3%' THEN 'Town'
	WHEN locale_code like '4%' THEN 'Rural'
END AS 'locale_text',

CASE
	WHEN locale_code like '1%' or  locale_code like '2%' THEN 
		CASE substr(locale_code, 2, 1)
			WHEN '1' THEN 'Large'
			WHEN '2' THEN 'Midsize'
			WHEN '3' THEN 'Small'
		END
	WHEN locale_code like '3%' or  locale_code like '4%' THEN
		CASE substr(locale_code, 2, 1)
			WHEN '1' THEN 'Fringe'
			WHEN '2' THEN 'Distant'
			WHEN '3' THEN 'Remote'
		END 
END AS 'locale_size'
FROM public_hs_data
;
```

What is the minimum, maximum, and average median_household_income of the nation? for each state?
```sql
select state_code, max(median_household_income), avg(median_household_income), min(median_household_income)
from census_data
WHERE median_household_income != 'NULL'
group by state_code
```
- [X] Joint analysis: Join the tables together for even more analysis.

Do characteristics of the zip-code area, such as median household income, influence students’ performance in high school?
```sql
select round(avg(public_hs_data.pct_proficient_math)) as 'math avg', round(avg(public_hs_data.pct_proficient_reading)) as 'reading avg', 
	CASE
		WHEN median_household_income < 50000 THEN 'low'
		WHEN median_household_income  >=50000 and  median_household_income <  100000 THEN 'medium'		
		WHEN median_household_income >= 100000 THEN 'quite high'
		ELSE 'N/A'
	END AS 'income'
from census_data
JOIN public_hs_data
on census_data.zip_code = public_hs_data.zip_code
-- WHERE median_household_income != 'NULL'
GROUP  BY income
```
Hint: One option would be to use the CASE statement to divide the median_household_income into income ranges (e.g., <$50k, $50k-$100k, $100k+) and find the average exam scores for each.

## Additional Challenges

### Intermediate Challenge

- [X] On average, do students perform better on the math or reading exam? Find the number of states where students do better on the math exam, and vice versa.[^2]
[^2]: We can use the WITH clause to create a temporary table of average exam scores for each state, with an additional column for whether the average for math or reading is higher. (Note: Some states may not have standardized assessments, so be sure to also include an option for No Exam Data) Then, in your final SELECT statement, find the number of states fitting each condition.
```sql
select count(*) as 'number of states',
case
	WHEN public_hs_data.pct_proficient_math >= public_hs_data.pct_proficient_reading then 'math better'
	WHEN public_hs_data.pct_proficient_math <= public_hs_data.pct_proficient_reading then 'reading better'
-- 	ELSE 'N/A'
END as 'better'
from census_data
JOIN public_hs_data
on census_data.zip_code = public_hs_data.zip_code
where (pct_proficient_math != 'NULL' ) OR (pct_proficient_reading != 'NULL') 
GROUP BY better
```

### Advanced Challenge

- [X] What is the average proficiency on state assessment exams for each zip code, and how do they compare to other zip codes in the same state?
```sql
select census_data.zip_code, avg(public_hs_data.pct_proficient_math), avg(public_hs_data.pct_proficient_reading)
from census_data
JOIN public_hs_data
on census_data.zip_code = public_hs_data.zip_code
where (public_hs_data.pct_proficient_math != 'NULL' ) OR (public_hs_data.pct_proficient_reading != 'NULL') 
GROUP BY census_data.zip_code
```
- [X] Exam standards may vary by state, so limit comparison within states. Some states may not have exams. We can use the WITH clause to create a temporary table of exam score statistic for each state (e.g., min/max/avg) - then join it to each zip-code level data to compare.
```sql
with state_stats as (
select
	state_code,
	avg(pct_proficient_math) as 'state_math_avg',
	avg(pct_proficient_reading) as 'state_read_avg'
from public_hs_data
-- where (pct_proficient_math != 'NULL' ) OR (pct_proficient_reading != 'NULL') 
GROUP BY state_code
)

select
	public_hs_data.zip_code,
	public_hs_data.state_code,
	avg(public_hs_data.pct_proficient_math) as 'zip_math_avg',
	avg(public_hs_data.pct_proficient_reading) as 'zip_read_avg',
	state_stats.state_math_avg,
	state_stats.state_read_avg
from public_hs_data
JOIN state_stats
ON public_hs_data.state_code = state_stats.state_code
GROUP BY public_hs_data.zip_code
```
