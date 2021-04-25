## Краткая теория

- каждая транзакция запускается со своим уникальным  XID
- XID это единый счетчик на весь инстанс потсгреса
- 32 бита - ограничение счетчика, т.е. примерно 2млрд транзакций, потом счетчик обнуляется
- периодически происходится заморозка с старых данных
- в случае если заморозка не происходила есть опасность ситуации истощения счетчика.
- при заморзке в таблице pg_class обновляется поле relfrozenxid, в котором прописывается XID, младше которого все записи в этой таблице считаются замороженными (старыми)
- age(relfrozenxid) - показывает дельту между этим значением и текущим состояние счетчика XID 
- заморозка происходит автоматически при достижении age(relfrozenxid)  значения параметра autovacuum_freeze_max_age
- при это могут быть заморожены только записи младше самого младшего XID активных транзакций

## Запросы для анализа ситуации

- смотрим xid текущих транзакций

```sql
select transactionid as XID, pid from pg_locks where locktype='transactionid';
```

- анализируем что за транзакции с наимеменьшим xid
```sql
select  * from pg_stat_activity where pid=<PID>
```

- проверяем на сколько транзакций хватает системы на текущий момент
```sql
WITH max_age AS ( 
    SELECT 2000000000 as max_old_xid
        , setting AS autovacuum_freeze_max_age 
        FROM pg_catalog.pg_settings 
        WHERE name = 'autovacuum_freeze_max_age' )
, per_database_stats AS ( 
    SELECT datname
        , m.max_old_xid::int
        , m.autovacuum_freeze_max_age::int
        , age(d.datfrozenxid) AS oldest_current_xid 
    FROM pg_catalog.pg_database d 
    JOIN max_age m ON (true) 
    WHERE d.datallowconn ) 
SELECT max(oldest_current_xid) AS oldest_current_xid
    , max(ROUND(100*(oldest_current_xid/max_old_xid::float))) AS percent_towards_wraparound
    , max(ROUND(100*(oldest_current_xid/autovacuum_freeze_max_age::float))) AS percent_towards_emergency_autovac 
FROM per_database_stats
```
- percent_towards_wraparound - если этот параметр приближается к 100
    - убиваем старые транзакции
    - проверяем что автовакуум или вакуум начал нормально фризить таблицы

- percent_towards_emergency_autovac если этот параметр больше 100 - значит автовакуум не отрабатывает фриз, что то ему мешает, вероятно старые транзакции


- смотрим топ 100 таблицы на которых последний  VACUUM проходил очень давно
- age - сколько транзакций назад происходил вакуум на данной таблице
```sql
select c.oid::regclass,
age(c.relfrozenxid),
c.relfrozenxid 
from pg_class c WHERE relkind IN ('r','p','i')
ORDER BY 2 DESC
limit 100 ;
```

- проверка конкретной таблицы

```sql
vacuum freeze _timescaledb_internal.compress_hyper_1043_487368_chunk;

select c.oid::regclass,
age(c.relfrozenxid),
c.relfrozenxid 
from pg_class c
where c.oid='_timescaledb_internal.compress_hyper_1043_487368_chunk'::regclass;
```


## Рекомендации по timescaledb

- timescaledb самостоятельно удаляет чанки
- поэтому лишний раз дергать VACUUM не имеет смысла
- лучше выставить  
    - autovacuum_freeze_max_age = 600000000
    - vacuum_freeze_table_age =  200000000
- настроить мониторинг за параметрами описанными выше:
    - percent_towards_wraparound
    - percent_towards_emergency_autovac
- настроить мониторинг за старыми XID у активных транзакций
- настроить мониторинг за старыми активными запросами в системе
