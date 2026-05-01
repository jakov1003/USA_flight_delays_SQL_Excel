## Project & dataset introduction

I had been wanting to do another data, SQL, and Excel-themed project for several months now. After in-depth dataset research, I settled on one covering commerical aviation flight delays in the US from 2023.

The aforementioned dataset is a subset of this [broader Kaggle dataset](https://www.kaggle.com/datasets/bordanova/2023-us-civil-flights-delay-meteo-and-aircraft?select=maj+us+flight+-+january+2024.csv). The file is called maj us flight - january 2024. It does not cover every single US commerical airline, flight and delay, however, most delayed flights from major US commerical airlines are included. 

I split the CSV into multiple PostgreSQL database tables, which I then queried. I've also uploaded the raw unsplit CSV file, the post-split CSV files, and the .SQL dump of the database.

**Click the dropdown triangle for the section you wish to see. Sections 2. and 3. have graphs, sections 1. and 4. have tables.** Graphs are more pleasing on the eye, while tables allow you to showcase more data (which may sometimes be necessary).

<details>
<summary> USA Flight Delays 2023, Entity Relationship Diagram of the database </summary>
<img width="1093" height="686" alt="schema_5" src="https://github.com/user-attachments/assets/37cf721b-83f7-4fbc-83b8-d1cd6b377ca1" />
The dep_airport_id and arr_airport_id mapping onto airport_id in the airports table simply means that every airport has its own unique ID, which can act as both a departure and arrival airport ID. There are no duplicates and errouneous applications of primary and foreign keys.
</details>

<details>
<summary> <b>1.Do American citizens fly with old aircraft?</b> </summary>


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

**American Civilan Aircraft by Age and Number of Flights, 2023**
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

**Insight**
The oldest aircraft are at the bottom of the frequency pile. However, the answer to the section title question is not a resounding NO, as aircrafts aged 20-24 and 15-19 are/have been more commonly used than those aged 5-9 and 0-4.

</details>

<details>
<summary><b>2.Flight delays by airline</b></summary>

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

**Insight**
It's clear that more flights does not always correlate with the amount of delay hours.

</details>

<details>
<summary><b>3.Flight delays by aircraft type</b></summary>

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

**Insight**
I purposefully haven't shown everything from the query on the graph. It seems complicated, but it actually isn't. 

You can also look at this graph as "How many flights would aircraft type have to make to achieve 1000 delay hours over the course of a year?". The answer would be, for example, 5740 flights for Boeing 717 and 1130 flights for Boeing 737.

</details>

<details>
<summary><b>4.Flight delays by the part of day</b></summary>

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

**What part of the day was the most problematic for US commercial aviation in 2023?**
Larger ratio = better, smaller ratio = worse.

| dep_part_of_day | no_of_flights | hours_of_total_delays | flights_to_delay_hours_ratio |
|:---------------:|:-------------:|:---------------------:|:----------------------------:|
| Evening         | 118,329       | 57,474                | 2.06                         |
| Afternoon       | 188,313       | 74,727                | 2.52                         |
| Morning         | 205,409       | 45,851                | 4.48                         |
| Night           | 15,146        | 3,100                 | 4.89                         |

**Insight**
Evening is/was most problematic, but the difference between evening and night is stark, regardless of whether the hours map perfectly or not.

</details>

<details>
<summary><b>5.Tools</b></summary>

**(Postgre)SQL**

**pgAdmin 4**

For this project, I did not use VS Code due to time-related reasons. The connections and uploads would have taken a significant amount of time to set up. I therefore queried the data in pgAdmin 4.

**GitHub** 

I wrote the entire README file on GitHub itself. What I said above about omitting VS Code applies to GitHub too. 

**Excel**

Import, export, copy, and paste operations.

**AI - ChatGPT, Perplexity, NotebookLM**

I used AI tools to split the raw CSV file into multpile PostgreSQL database tables and fill those tables with data. I also used them to create the two graphs found within this project. The data queries and this README file were both entirely written by myself. I did not just blindly accept the AI tools' responses - I further prompted them to both correct and clarify specific matters.

</details>

