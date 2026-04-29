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

**Data refers to arrival and departure delays summed up as we are investigating it by cause in this section.**

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
