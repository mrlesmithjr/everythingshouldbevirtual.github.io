---
  title: Purge Syslog and Trap alerts from Solarwinds DB
  date: 2014-02-21 22:28:57
---

I was getting overloaded on the number of syslog messages in my
Solarwinds DB and needed to purge them. So I wanted to share with
other's what MS SQL Server query to execute to do this. I purged
everything older than 2/15/2014 using this query. You can tailor to meet
your requirements.

```sql
Delete FROM [SolarWindsOrion].[dbo].[SysLog] Where datetime <= '2/15/2014'
Delete from [SolarWindsOrion].[dbo].[Traps] Where datetime <= '2/15/2014'
Delete from [SolarWindsOrion].[dbo].[TrapVarbinds] where TrapID not in (select TrapID from [SolarWindsOrion].[dbo].[Traps])
```

If you would like to use a query to delete everything older than for
example 30 days you can execute the following query. Of course you can
modify to meet your requirements again.

```sql
Delete FROM [SolarWindsOrion].[dbo].[SysLog] Where datetime < GETDATE() - 30
Delete from [SolarWindsOrion].[dbo].[Traps] Where datetime < GETDATE() - 30
Delete from [SolarWindsOrion].[dbo].[TrapVarbinds] where TrapID not in (select TrapID from [SolarWindsOrion].[dbo].[Traps])
```

If you want to delete all existing Syslog and Trap messages you can run
the following.

```sql
Truncate [SolarWindsOrion].[dbo].[SysLog]
Truncate [SolarWindsOrion].[dbo].[Traps]
Truncate [SolarWindsOrion].[dbo].[TrapVarbinds]
```

Enjoy!
