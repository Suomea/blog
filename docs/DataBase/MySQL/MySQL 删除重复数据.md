先备份整表，防止数据误删。


```sql
select * from alarm_remote_check where id not in (
	select min(id) from alarm_remote_check
	group by license, detectdate, dsid, detectsn
)
```