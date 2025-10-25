## LAB 2 TASK 2 ##
у нас всего один параметр, по которому мы можем сравнивать самолеты, и логично, что можно не использовать самолет, который может пролетать без дозаправки 11.000 км, 
на рейсах, где расстояние между аэропортами 1400 км, а, например, можно использовать самолет cr2, который может пролететь 2700, а самолет 773 можно отправить на более
длинные перелеты.
нужно рассмотреть таблицу flights, в ней забираем айди перелета, запланированный аэропорт отправления и прибытия и самолет, который запланирован. 
с помощью координат из таблицы airports, мы рассчитаем расстояние по формуле гаверсинусов ([https://en.wikipedia.org/wiki/Haversine_formula](https://en.wikipedia.org/wiki/Haversine_formula)),
и уже сравним реальное расстояние с аэропортами с расстояниями, которые могут пролететь самолеты без дозаправки и выберем тот, которыый минимально покрывает потребность расстояния

```
WITH airports_coords AS (
    SELECT 
        airport_code,
        (coordinates[0])::numeric AS x,
        (coordinates[1])::numeric AS y
    FROM airports
),
flight_distances AS (
    SELECT 
        f.flight_id,
        f.departure_airport,
        f.arrival_airport,
        f.aircraft_code,
        ac.range as aircraft_range,
        6371 * 2 * ASIN(
            SQRT(
                POWER(SIN(RADIANS(a2.y - a1.y) / 2), 2) +
                COS(RADIANS(a1.y)) * COS(RADIANS(a2.y)) *
                POWER(SIN(RADIANS(a2.x - a1.x) / 2), 2)
            )
        ) as real_range
    FROM flights f
    JOIN aircrafts ac ON f.aircraft_code = ac.aircraft_code
    JOIN airports_coords a1 ON f.departure_airport = a1.airport_code
    JOIN airports_coords a2 ON f.arrival_airport = a2.airport_code
    WHERE a1.x IS NOT NULL AND a1.y IS NOT NULL
      AND a2.x IS NOT NULL AND a2.y IS NOT NULL
),
recommended_aircrafts AS (
    SELECT 
        fd.*,
        (
            SELECT aircraft_code 
            FROM aircrafts 
            WHERE range >= fd.real_range 
            ORDER BY range
            LIMIT 1
        ) as recommended_aircraft,
        (
            SELECT range 
            FROM aircrafts 
            WHERE range >= fd.real_range 
            ORDER BY range
            LIMIT 1
        ) as recommended_aircraft_range
    FROM flight_distances fd
)
SELECT 
    flight_id,
    departure_airport,
    arrival_airport,
    aircraft_code,
    aircraft_range as real_aircraft_range,
    recommended_aircraft,
    recommended_aircraft_range,
    ROUND(aircraft_range - recommended_aircraft_range) as range_difference
FROM recommended_aircrafts
WHERE recommended_aircraft IS NOT NULL
  AND aircraft_code != recommended_aircraft
ORDER BY range_difference DESC;
```


# получаем (первые 5 строк): #
```
flight_id | departure_airport | arrival_airport | aircraft_code | real_aircraft_range | recommended_aircraft | recommended_aircraft_range | range_difference 
-----------+-------------------+-----------------+---------------+---------------------+----------------------+----------------------------+------------------
      3475 | VKO               | PEE             | 773           |               11100 | CN1                  |                       1200 |             9900
      3476 | VKO               | PEE             | 773           |               11100 | CN1                  |                       1200 |             9900
      3477 | VKO               | PEE             | 773           |               11100 | CN1                  |                       1200 |             9900
     14548 | PEE               | VKO             | 773           |               11100 | CN1                  |                       1200 |             9900
      3471 | VKO               | PEE             | 773           |               11100 | CN1                  |                       1200 |             9900
```

