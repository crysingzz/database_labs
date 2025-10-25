LAB 1

block 1 - 2nd task SELECT *
запустил бд в терминале: psql -U postgres -d demo
посмотрел структуру таблицы: \d
посмотрел структуру таблицы flights: SELECT * FROM flights LIMIT 10;
написал запрос, соблюдая все условия из задания:
SELECT * FROM flights WHERE arrival_airport = 'SVO' AND status = 'Arrived' AND actual_arrival > '2017-07-22 16:00:00+03' AND actual_arrival < '2017-07-22 19:00:00+03';

получаем: 12 строк

block 2 - 3d task
прописываем запрос к таблице ticket_flights, где меняем amount на 1000 больше при том условии, что полет с таким айдишником запланирован на интервалы из условия
UPDATE ticket_flights SET amount = amount + 1000 FROM flights WHERE ticket_flights.flight_id = flights.flight_id AND ( (flights.scheduled_departure::time >= '07:00:00' AND flights.scheduled_departure::time < '10:00:00') OR (flights.scheduled_departure::time >= '17:00:00' AND flights.scheduled_departure::time < '20:00:00') );

получаем: UPDATE 360234

block 3 - 2nd task
распределение количества перелетов из DME  будет выглядеть как таблица, где в левом столбце часы, в которые вообще были перелеты - это часы из столбца с датой и временем,
а во втором столбце количество строчек, которые подходят нам по условию. будем учитыватть именно по actual_departure, чтобы считать именно вылетевшие самолеты
SELECT EXTRACT(HOUR FROM actual_departure) AS hour_of_day, count(*) AS flight_count FROM flights WHERE departure_airport = 'DME' GROUP BY hour_of_day ORDER BY hour_of_day;

получаем:
 hour_of_day | flight_count 
-------------+--------------
           0 |            1
           9 |          230
          10 |          151
          11 |          209
          12 |          130
          13 |           69
          14 |          143
          15 |          170
          16 |           95
          17 |           79
          18 |          150
          19 |          114
          20 |           67
          21 |            6
          22 |            7
          23 |            5
             |         1591
(17 строк)

