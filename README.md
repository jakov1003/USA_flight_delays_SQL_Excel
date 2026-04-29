**1.Do American citizens fly with old aircraft?**

**Code**
```sql
SELECT
	CASE 
        WHEN age BETWEEN 0 AND 4 THEN '0-4'
        WHEN age BETWEEN 5 AND 9 THEN '5-9'
        WHEN age BETWEEN 10 AND 14 THEN '10-14'
        WHEN age BETWEEN 15 AND 19 THEN '15-19'
        WHEN age BETWEEN 20 AND 24 THEN '20-24'
        WHEN age BETWEEN 25 AND 29 THEN '25-29'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
		ELSE '40+'
    END AS aircraft_age,
	COUNT(flights.flight_id) as no_of_flights
FROM 
	aircrafts
INNER JOIN
	flights on flights.aircraft_id = aircrafts.aircraft_id 
GROUP BY
	aircraft_age
ORDER BY
	no_of_flights DESC;
```sql

**Output**
| aircraft_age | no_of_flights |
|:------------:|:-------------:|
|     5-9      |    145346     |
|    20-24     |    109214     |
|    15-19     |     95673     |
|    10-14     |     80731     |
|     0-4      |     51888     |
|    25-29     |     35107     |
|    30-39     |     8970      |
|     40+      |      268      |


**Flight delays by part of day**

**Code**

```sql

WITH hours_of_delays as (
SELECT
	dep_part_of_day,
	SUM(dep_delay_mins) / 60 as hours_of_departure_delays,
	SUM(arr_delay_mins) / 60 as hours_of_arrival_delays,
	COUNT(flight_id) as no_of_flights
FROM
	flights
GROUP BY
	dep_part_of_day
)

SELECT
	dep_part_of_day,
	no_of_flights,
	
	hours_of_departure_delays + hours_of_arrival_delays
	as hours_of_total_delays,
	
	ROUND(no_of_flights * 1.0 / (hours_of_departure_delays + hours_of_arrival_delays), 2)
	as flights_to_delay_hours_ratio
FROM
	hours_of_delays
ORDER BY
	flights_to_delay_hours_ratio ASC;
```
**Output**

Night = 00:00–06:00
Morning = 06:00–12:00
Afternoon = 12:00–18:00
Evening = 18:00–24:00

Larger ratio = better, smaller ratio = worse.

| dep_part_of_day | no_of_flights | hours_of_total_delays | flights_to_delay_hours_ratio |
| --------------- | ------------- | --------------------- | ---------------------------- |
| Evening         | 118,329       | 57,474                | 2.06                         |
| Afternoon       | 188,313       | 74,727                | 2.52                         |
| Morning         | 205,409       | 45,851                | 4.48                         |
| Night           | 15,146        | 3,100                 | 4.89                         |
