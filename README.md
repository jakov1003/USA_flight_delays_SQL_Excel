**Flight delays by part of day and cause**

**Code**
```sql
SELECT 
	flights.dep_part_of_day as departure_part_of_day, 
	SUM(flight_delays.delay_carrier_mins) / 60 as hours_of_aircraft_or_airline_delays,
	SUM(flight_delays.delay_weather_mins) / 60 as hours_of_weather_delays,
	SUM(flight_delays.delay_air_traffic_control_mins) / 60 as hours_of_ATC_delays,
	SUM(flight_delays.delay_security_mins) / 60 as hours_of_security_delays,
	SUM(flight_delays.delay_last_aircraft_mins) / 60 as hours_of_last_aircraft_delays
FROM
	flights
INNER JOIN
	flight_delays on flight_delays.flight_id = flights.flight_id
GROUP BY
	dep_part_of_day;
```
**Output**

00:00–06:00 → Night

06:00–12:00 → Morning

12:00–18:00 → Afternoon

18:00–24:00 → Evening

**In this section, delays mean total amount of hours for both flight arrivals and departures. 
The same flight can, logically, be late on both departure and arrival.**

| dep_part_of_day | hours_of_aircraft_or_airline_delays | hours_of_weather_delays | hours_of_atc_delays |
|-----------------|--------------------------------------|-------------------------|---------------------|
| Afternoon       | 23924                                | 3494                    | 17321               |
| Evening         | 18404                                | 2978                    | 9999                |
| Morning         | 23823                                | 3481                    | 19171               |
| Night           | 2008                                 | 246                     | 1469                |

| dep_part_of_day | hours_of_security_delays | hours_of_last_aircraft_delays |
|-----------------|--------------------------|---------------------------------|
| Afternoon       | 165                      | 31617                           |
| Evening         | 167                      | 22380                           |
| Morning         | 162                      | 18762                           |
| Night           | 10                       | 883                             |

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

Larger ratio = better, smaller ratio = worse

| dep_part_of_day | no_of_flights | hours_of_total_delays | flights_to_delay_hours_ratio |
| --------------- | ------------- | --------------------- | ---------------------------- |
| Evening         | 118,329       | 57,474                | 2.06                         |
| Afternoon       | 188,313       | 74,727                | 2.52                         |
| Morning         | 205,409       | 45,851                | 4.48                         |
| Night           | 15,146        | 3,100                 | 4.89                         |
