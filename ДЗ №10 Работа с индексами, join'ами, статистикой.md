
## Домашнее задание

## Работа с индексами, join'ами, статистикой

### Цель:

-   знать и уметь применять основные виды индексов PostgreSQL
-   строить и анализировать план выполнения запроса
-   уметь оптимизировать запросы для с использованием индексов
-   знать и уметь применять различные виды join'ов
-   строить и анализировать план выполенения запроса
-   оптимизировать запрос
-   уметь собирать и анализировать статистику для таблицы

  

## Описание/Пошаговая инструкция выполнения домашнего задания:

Создать индексы на БД, которые ускорят доступ к данным.  
В данном задании тренируются навыки:

-   определения узких мест
-   написания запросов для создания индекса
-   оптимизации  
    Необходимо:

1.  Создать индекс к какой-либо из таблиц вашей БД
2.  Прислать текстом результат команды explain,  
    в которой используется данный индекс
3.  Реализовать индекс для полнотекстового поиска
4.  Реализовать индекс на часть таблицы или индекс  
    на поле с функцией
5.  Создать индекс на несколько полей
6.  Написать комментарии к каждому из индексов
7.  Описать что и как делали и с какими проблемами  
    столкнулись  
    2 вариант:  
    В результате выполнения ДЗ вы научитесь пользоваться  
    различными вариантами соединения таблиц.  
    В данном задании тренируются навыки:

-   написания запросов с различными типами соединений  
    Необходимо:

1.  Реализовать прямое соединение двух или более таблиц
2.  Реализовать левостороннее (или правостороннее)  
    соединение двух или более таблиц
3.  Реализовать кросс соединение двух или более таблиц
4.  Реализовать полное соединение двух или более таблиц
5.  Реализовать запрос, в котором будут использованы  
    разные типы соединений
6.  Сделать комментарии на каждый запрос
7.  К работе приложить структуру таблиц, для которых  
    выполнялись соединения




## Подготовка среды
- Выполним на существующей виритупльной машине с PostgreSQL 15.2 развертывание демо-базы

sudo wget https://edu.postgrespro.ru/demo-medium.zip -O /tmp/demo-medium.zip

sudo unzip /tmp/demo-medium.zip -d /tmp/

sudo psql -h localhost -U postgres < /tmp/demo-medium-20170815.sql



## Выполнение домашнего задания

### Использование индексов

- Для примера возьмем таблицу bookings.ticket_flights и выполним запрос, который получет все номера билетов
```
explain
SELECT ticket_no FROM bookings.ticket_flights where flight_id = 35763;

-- Посмотрим план выполнения:
Gather  (cost=1000.00..32971.71 rows=83 width=14)
  Workers Planned: 2
  ->  Parallel Seq Scan on ticket_flights  (cost=0.00..31963.41 rows=35 width=14)
        Filter: (flight_id = 35763)
```
- При запросе выполняется посл. сканирование таблицы. Создадим индекс на таблице  для ускорения поиска. Получаем значительное уменьшение времени выполнения запроса

```
create index idx_bookings_ticket_flights__flight_id  on bookings.ticket_flights (flight_id) include (ticket_no);

explain
SELECT ticket_no FROM bookings.ticket_flights where flight_id = 35763;

-- Посмотрим план выполнения:
Index Only Scan using idx_bookings_ticket_flights__flight_id on ticket_flights  (cost=0.43..5.88 rows=83 width=14) (actual time=0.029..0.035 rows=59 loops=1)
  Index Cond: (flight_id = 35763)
  Heap Fetches: 0
Planning Time: 0.088 ms
Execution Time: 0.055 ms
```
- Создадим индекс для полнотекстового поиска в таблице bookings.tickets для имени пассажира. 
```
-- Проверим работу запроса без индекса
explain analyze
select * from bookings.tickets where passenger_name @@ 'IVANOV';

-- Посмотрим план выполнения:
Gather  (cost=1000.00..212988.13 rows=10382 width=140) (actual time=3.104..2843.689 rows=17081 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  ->  Parallel Seq Scan on tickets  (cost=0.00..210949.93 rows=4326 width=140) (actual time=4.314..2766.251 rows=5694 loops=3)
        Filter: (passenger_name @@ to_tsquery('IVANOV'::text))
        Rows Removed by Filter: 270663
Planning Time: 0.609 ms
JIT:
  Functions: 6
  Options: Inlining false, Optimization false, Expressions true, Deforming true
  Timing: Generation 1.383 ms, Inlining 0.000 ms, Optimization 1.195 ms, Emission 9.370 ms, Total 11.948 ms
Execution Time: 2845.312 ms

-- Наблюдаем сканирование таблицы при выполнении
```
- Создадим создаем gin-индекс 
```
-- подключим расширение, создадим новое поле в таблице, заполняем его значениями, делаем gin-индекс по нему
CREATE EXTENSION btree_gin; --https://postgrespro.ru/docs/postgresql/14/btree-gin

alter table bookings.tickets add column passenger_name_tsv tsvector;

update bookings.tickets set passenger_name_tsv = to_tsvector(passenger_name);

create index gin_bookings_tickets_passenger_name on bookings.tickets using gin(passenger_name_tsv);



explain 
select * from bookings.tickets where passenger_name_tsv  @@ to_tsquery('IVANOV');

-- Посмотрим план выполнения:
Bitmap Heap Scan on tickets  (cost=176.77..34966.17 rows=18132 width=140) (actual time=4.007..147.654 rows=17081 loops=1)
  Recheck Cond: (passenger_name_tsv @@ to_tsquery('IVANOV'::text))
  Heap Blocks: exact=11026
  ->  Bitmap Index Scan on gin_bookings_tickets_passenger_name  (cost=0.00..172.24 rows=18132 width=0) (actual time=2.411..2.411 rows=17081 loops=1)
        Index Cond: (passenger_name_tsv @@ to_tsquery('IVANOV'::text))
Planning Time: 0.128 ms
Execution Time: 148.890 ms

-- Как видим, получили выигрыш в скорости выполнения запроса и использование gin-индекса. Использование индекса требует определенный систаксис команды, которым нужно уметь пользоваться
```

- Рассмотрим индекс на часть таблицы (фильтрованный) индекс  
```
-- Выполним запрос по вычислению рейсов для fare_conditions ='Comfort'
explain
SELECT count(distinct flight_id) FROM bookings.ticket_flights where fare_conditions ='Comfort'

-- Посмотрим план выполнения:
Aggregate  (cost=36834.32..36834.33 rows=1 width=8)
  ->  Gather  (cost=1000.00..36739.91 rows=37765 width=4)
        Workers Planned: 2
        ->  Parallel Seq Scan on ticket_flights  (cost=0.00..31963.41 rows=15735 width=4)
              Filter: ((fare_conditions)::text = 'Comfort'::text)

-- Как видим, выполняется сканирование таблицы. Создадим идекс с условием:
create index idx_bookings_ticket_flights__fare_conditions_Comfort on bookings.ticket_flights (flight_id) where fare_conditions ='Comfort';

-- Посмотрим план выполнения:
Aggregate  (cost=809.18..809.19 rows=1 width=8)
  ->  Index Only Scan using idx_bookings_ticket_flights__fare_conditions_comfort on ticket_flights  (cost=0.29..714.77 rows=37765 width=4)

-- Как видим, получили выигрыш в скорости выполнения запроса и использование индекса
```

- Рассмотрим создание индекса на несколько полей
```
-- Используем запрос с задачей регулярного отсулеживания статуса рейсов из а/э отправления по данному коду воздушного судна 

explain 
select departure_airport, aircraft_code, status  from bookings.flights where departure_airport = 'VKO' and aircraft_code= 'SU9';

-- Посмотрим план выполнения:
Seq Scan on flights  (cost=0.00..1776.96 rows=890 width=16)
  Filter: ((departure_airport = 'VKO'::bpchar) AND (aircraft_code = 'SU9'::bpchar))


-- Создадим индекс
create index idx_bookings_lights__flight_id  on bookings.flights (departure_airport, aircraft_code) include (status);

-- Повторно выполним запрос и посмотрим 
Index Only Scan using idx_bookings_flights__departure_airport__aircraft_code on flights  (cost=0.42..34.22 rows=890 width=16)
  Index Cond: ((departure_airport = 'VKO'::bpchar) AND (aircraft_code = 'SU9'::bpchar))
  
-- Как видим, получили выигрыш в скорости выполнения запроса и использование индекса.
При выборке всех полей (которые не входят в индекс)  таблицы мы будем наблюдать дополнительное сканирование 

Bitmap Heap Scan on flights  (cost=25.54..868.21 rows=890 width=63)
  Recheck Cond: ((departure_airport = 'VKO'::bpchar) AND (aircraft_code = 'SU9'::bpchar))
  ->  Bitmap Index Scan on idx_bookings_flights__departure_airport__aircraft_code  (cost=0.00..25.32 rows=890 width=0)
        Index Cond: ((departure_airport = 'VKO'::bpchar) AND (aircraft_code = 'SU9'::bpchar))
```


### Использование различных типов соединений

- Пример прямого соединение таблиц (бронирования с билетами)

```
select ticket_no
from bookings.tickets t
inner join bookings.bookings b on t.book_ref = b.book_ref 

```
- Пример левостороннего соединения таблиц (найдем все рейсы с билетами без посадочных талонов)
```
select tf.ticket_no, tf.flight_id 
from bookings.ticket_flights tf
left join bookings.boarding_passes bp  on bp.ticket_no = tf.ticket_no  and bp.flight_id = tf.flight_id 
where bp.boarding_no  is null

```
- Пример перекрестного соединения двух таблиц - получим все возможные комбинации аэропортов и воздушных судов
```
select a.airport_code, b.aircraft_code
from bookings.airports_data a
cross join bookings.aircrafts_data b;
```
- Пример полного соединения двух таблиц (похожий запрос с как левосторонним соединением)
```
select bp.flight_id 
from bookings.tickets t
full join bookings.boarding_passes bp  on bp.ticket_no = t.ticket_no  
where bp.boarding_no  is null
```
- Пример запроса с разными соединениями (получим развернутую информацию по а/э отправления, номеру рейса и свободным местам для запланированных рейсов )
```
select f.departure_airport , tf.ticket_no, f.flight_no 
from bookings.ticket_flights tf
inner join bookings.flights f on f.flight_id = tf.flight_id 
left join bookings.boarding_passes bp  on bp.ticket_no = tf.ticket_no  and bp.flight_id = tf.flight_id 
where bp.boarding_no  is null
and f.status ='Scheduled'
```

