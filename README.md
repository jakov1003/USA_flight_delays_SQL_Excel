## 0.Project & dataset introduction

I had been wanting to do another data, SQL, and Excel-themed project for several months now. After finally finding some time to do so, I settled on an aviation industry dataset, because free workable datasets are difficult to find. Kaggle's decent, but many datasets on there are either overly clean/messy or too large/small.

The dataset I used is a subset of this [broad Kaggle dataset](https://www.kaggle.com/datasets/bordanova/2023-us-civil-flights-delay-meteo-and-aircraft?select=maj+us+flight+-+january+2024.csv). The file is called maj us flight - january 2024. Even this sole file cotains plenty of data on flight delays from the United States of America from 2023.

I split the CSV into multiple PostgreSQL database tables, which I then queried. You can find the database schema below. I've also uploaded the raw unsplit CSV file, the post-split CSV files, and the .SQL dump of the database.

<img width="1093" height="686" alt="schema_5" src="https://github.com/user-attachments/assets/37cf721b-83f7-4fbc-83b8-d1cd6b377ca1" />


## 1.Do American citizens fly with old aircraft?

### 1.1.Code
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
```

### 1.2.Output
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


## 2.Flight delays by airline

### 2.1.Code
```sql

-- I have left gaps between certain lines of code
-- and written the "AS" part on another line intentionally.
-- In my opinion, the length of the line containing multiple
-- mathematical operations and one "AS" keyword would have
-- made the whole code look awkward.

SELECT
	airlines.airline_name,
	
	(SUM(flights.dep_delay_mins) / 60) + (SUM(flights.arr_delay_mins) / 60) 
	as hours_of_total_delays,
	
	COUNT(flight_id) as no_of_flights
FROM
	airlines
INNER JOIN
	flights on flights.airline_id = airlines.airline_id
GROUP BY
	airline_name
ORDER BY
	hours_of_total_delays DESC;
```
### 2.2.Output

<img width="1536" height="1024" alt="flight delays graph" src="https://github.com/user-attachments/assets/ede6aaa5-9028-4fc3-96ac-6b1d44c9df8e" />

## 3.Flight delays by aircraft type

### 3.1.Code

```sql

-- I am well aware that I should have probably
-- further inlined the code within the WITH statement below.
-- I would have fixed it if I were doing this as part of my job.
-- I hope to display critical thinking rather than sloppiness by leaving it.
-- The same applies to section 4.1 in addition to what I have already written there.

WITH hours_of_delays as (
SELECT
	aircrafts.manufacturer || ' ' || aircrafts.model AS aircraft_type,
	SUM(flights.dep_delay_mins) / 60 as hours_of_departure_delays,
	SUM(flights.arr_delay_mins) / 60 as hours_of_arrival_delays,
	COUNT(flights.flight_id) as no_of_flights
FROM
	aircrafts
INNER JOIN
	flights on flights.aircraft_id = aircrafts.aircraft_id
GROUP BY
	aircraft_type
)

SELECT
	aircraft_type,
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

### 3.2.Output

<img width="1935" height="840" alt="cropped_chart_v2" src="https://github.com/user-attachments/assets/caac6651-386f-4578-8118-7c102deaf277" />


## 4.Flight delays by part of day

### 4.1.Code

```sql

-- I have left gaps between certain lines of code
-- and written two "AS" keywords on another line intentionally.
-- In my opinion, the length of the lines containing multiple
-- mathematical operations and two "AS" keywords would have
-- made the whole code look awkward.

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
### 4.2.Output

Night = 00:00–06:00

Morning = 06:00–12:00

Afternoon = 12:00–18:00

Evening = 18:00–24:00

Larger ratio = better, smaller ratio = worse.


| dep_part_of_day | no_of_flights | hours_of_total_delays | flights_to_delay_hours_ratio |
|:---------------:|:-------------:|:---------------------:|:----------------------------:|
| Evening         | 118,329       | 57,474                | 2.06                         |
| Afternoon       | 188,313       | 74,727                | 2.52                         |
| Morning         | 205,409       | 45,851                | 4.48                         |
| Night           | 15,146        | 3,100                 | 4.89                         |

## 5.Tools

**(Postgre)SQL**

**pgAdmin 4**

For this project, I did not use VS Code due to time-related reasons. The connections and uploads would taken significantly more time. I therefore queried the data in pgAdmin.

**GitHub** 

I wrote the entire README file on GitHub itself. What I said about omitting VS Code applies to GitHub as well.

**Excel**

Import, export, copy, and paste operations.

**AI - ChatGPT, Perplexity, NotebookLM**

I had AI tools write me the code for splitting the raw CSV file from Kaggle into multpile PostgreSQL database tables and filling those those tables with data. I also used them to create the two graphs contained within this project. The data queries were all written by myself. I did not just blindly accept the AI tools' responses - I further prompted them to both correct and clarify specific matters.


