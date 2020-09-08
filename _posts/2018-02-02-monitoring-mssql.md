---
title: 'Monitoring MSSQL'
date: 2018-02-02 00:00:00
featured_image: '/images/post/Monitor_MSSQL.jpg'
excerpt: Monitoring a Microsoft SQL is no easy feet, here's a compiled approach ...
---

Monitoring a Microsoft SQL is no easy feet, as any seasoned DB admin  or if you are an accidental DBA.

Thankfully there are plenty of blogs and articles out there that offer guidance on monitoring this beast.

Here's a write-up with compilation of various guidance across the internet.

First 3 articles for your own reading as a reference on effective SQL Server monitoring

* [Microsoft Powershell Diagnose](https://blogs.technet.microsoft.com/heyscriptingguy/2013/05/07/use-powershell-to-discover-diagnose-and-document-sql-server/)
* [Apexsql How to Monitor](https://solutioncenter.apexsql.com/how-to-monitor-your-sql-server-instances-and-databases/)
* [MSSQLTips on monitoring](https://www.mssqltips.com/sql-server-tip-category/54/monitoring/)


### Why?

Why do we monitor our SQL Server ? There's plenty of reasons. Here's a summary of reason:

**Capacity Planning**: Unless we know the server’s resource limits (storage space, memory, I/O capacity and so on) and monitor current resource usage then we may not find out that we need to increase capacity until one of them “runs out”.

**Reaction time**: With effective monitoring, alerting and reporting, we can respond to an emergency before too many people are affected, and the crisis escalates.


### Key Areas of Monitoring

#### DISK ACTIVITY
* Physical Disk: % Disk Time: This counter monitors the portion of time the disk is busy with read/write activity. If the Physical Disk: % Disk Time counter is close to or over 90%, it indicates that too many system requests are waiting for disk access (check this via the Physical Disk: Current Disk Queue Length counter). The number of pending I/O requests should be sustained at no more than 1.5 to 2 times the number of spindles of the physical disk.
* Physical Disk: Average Disk Queue Length: number of I/O operations waiting (again, over 1.5 or 2 times the number of disk spindles is bad)
* SQL Server Buffer Manager – Page reads/sec and page writes/sec. If this counter rises above your baseline, it may indicate the need for more hardware power
* Physical Disk: Average Disk Read/Write per secThese two counters tell you how quickly your I/O subsystem is responding to requests for data from the operating system; in other words, latency. The latency values returned are valid regardless of the type of I/O subsystem you’re using, whether it’s local physical magnetic disk, SAN drives, NAS drives, or solid state drives. Your latency values should normally not be more than 20ms; if you’re using SSD, probably not more than 5ms. If you see latency values of a second or more, your I/O subsystem has issues that need to be addressed to keep performance at an acceptable level.

#### PROCESSOR UTILIZATION
* Processor: % Processor time: A consistent 80-90% is too high. Multiprocessor systems have a separate instance for each CPU.
* Processor(_Total): % Processor time:Percent Processor Time tells us how busy the server’s CPUs are. It’s a basic indicator to help us know that a server is running well within acceptable operating parameters. Normally I’d expect to see this counter in the 20 to 40 percent range. When it jumps above 80% I get very nervous, as that means that activity that requires the processor is probably waiting for resources, and thus is slowing down someone’s vital work.
* Processor: % Privileged time: indicates the time spent on Windows kernel commands (SQL Server I/O requests). If both this and Physical Disk counters are high, there might be a need for a faster disk or lower load for this server.
* Processor: % user time: the percentage of time the CPU spends on user processes (SQL Server)
* Processor: Queue Length: the number of threads waiting for processor time. A high number may indicate the need for faster or more processors.
The Processor Queue Length counter tells you the number of threads that are waiting for time on the system processor. If this number is greater than 0, that means that there are more requests per core than the system can handle, and this can be a cause for significant performance issues. I once had a client that had a month-end process that had to be run during the business day, which would take 2.5 to 3 hours to run; when it ran, performance for everyone else on that system would be horribly slow. I looked at the Processor Queue Length counter – normally it would get to no higher than 3 or 4 during the day, but During month-end it jumped to somewhere between 30 and 50. The client was running on a virtual machine with 4 processors, and I asked if they could double that. They did, and the next month-end completed in 45 minutes.


#### MEMORY
* Memory: Available MBs: indicates how much memory is available for new processes.
The Available MBytes Memory counter helps me know if server memory is an issue. I can set Max Server Memory settings in SQL Server, which will help SQL Server share the memory nicely with the Windows OS, but there may be other processes on the server besides SQL Server. Capturing this counter allows me to know if other processes are taking memory SQL Server needs to perform well.
* Memory: Pages/sec: this counter indicates how many times the virtual memory is getting accessed. A rule of thumb says that it should be lower than 20. Higher numbers might mean excessive paging. Using Memory: Page Faults/sec can further indicate whether SQL Server or some other process is causing it.
* Paging File(__Total)\% Usage:When Windows runs out of memory it takes large chunks of memory and swaps it out to disk, to the Paging File. Unfortunately, the slowest operation in all computing is writing to disk, regardless of the physical media involved, so swapping memory to disk is naturally going to slow down the performance of your system. Keeping an eye on this counter will help you know when you are encountering memory issues, and you can then take action to resolve the conflicts.


#### OTHERS
SQL Server works with objects and counters, with each object comprising one or more counters. For example, the SQL Server Locks object has counters called Number of Deadlocks/sec or Lock Timeouts/sec.

* Access Methods – Full scans/sec: higher numbers (> 1 or 2) may mean you are not using indexes and resorting to table scans instead.
* Buffer Manager – Buffer Cache hit ratio: This is the percentage of requests serviced by data cache. When cache is properly used, this should be over 90%. The counter can be improved by adding more RAM.
* Memory Manager – Target Server Memory (KB): indicates how much memory SQL Server “wants”. If this is the same as the SQL Server: Memory Manager — Total Server Memory (KB) counter, then you know SQL Server has all the memory it needs.
* Memory Manager — Total Server Memory (KB): much memory SQL Server is actually using. If this is the same as SQL Server: Memory Manager — Target Server Memory (KB), then SQL Server has all the memory it wants. If smaller, then SQL Server could benefit from more memory.
* Locks – Average Wait Time: This counter shows the average time needed to acquire a lock. This value needs to be as low as possible. If unusually high, you may need to look for processes blocking other processes. You may also need to examine your users’ T-SQL statements, and check for any other I/O bottlenecks.
* Access Methods\Forwarded Records/sec:   Forwarded Records/sec helps you understand how fragmented your heaps are. A heap is a SQL Server table without a clustered index, and SQL Server uses Row IDs to find the data it’s looking for. If it arrives at the page where the data should be, and that data has been moved, SQL Server leaves a pointer there called a forwarding pointer, and the process has to incur an additional I/O operation to get it from the new location. Each time a search for data encounters a forwarding pointer it increments this counter. This may be unavoidable, but if you track this over time and this number starts to increase, you should think about ways to defragment your table (or stop using a heap).
* Access Methods\Page Splits/sec:   Page Splits/sec also helps you understand how fragmented your tables are. In this case, even if your table is in good shape, when SQL Server adds pages to it, it’ll increment this counter. If, however, SQL Server needs to insert a row onto a page, and there isn’t room, SQL Server will split the page into multiple pages, move rows from one page to another to balance the pages out, and then insert the row. This is a very expensive and time-consuming process, and this counter will help you understand when this is happening a lot. Properly configuring the free space on each page will help minimize this activity, just note that there are “good” page splits and “bad” page splits, and this counter doesn’t differentiate (Jonathan Kehayias of SQLskills has an Extended Events session you can use instead.)
* SQL Statistics\Batch Requests/sec:   This counter is there to help you understand how busy your SQL Server system is. By capturing this counter, and using it in your baseline, you can identify variances easily – they might be reported by users, or it might just be extra load on the system because people are asking for more than they usually do.
* General Statistics\Processes blocked:   In any multi-user application you’re going to have blocked processes, and SQL Server has mechanisms to handle blocked processes well, but when this counter goes outside the normal range (for your system) you’ll want to investigate and see what might be causing the issue. There could be excessive blocking due to page escalation, for example, where entire tables are getting locked instead of individual rows or pages.
* SQL Statistics\SQL Compilations/sec  /SQL Recompilations/sec:  These counters will increment when SQL Server has to compile or recompile query plans because either the plan in cache is no longer valid, or there’s no plan in cache for this query. SQL Server uses a cost-based optimizer that relies on statistics to choose a good query plan, and when those statistics are out-of-date, additional compilations are done unnecessarily. It can be useful to understand the source of this problem, if it is a problem (this might be expected behavior, depending on the workload).



### Monitor through System Stored Procedure

The following stored procedures, built-in to SQL Server, provide a powerful alternative to many monitoring tasks:

|Stored Procedure|Description|
|----------------|-----------|
|[sp_who](https://technet.microsoft.com/en-us/library/ms174313(v=sql.110).aspx)|Reports snapshot information about current SQL Server users and processes, including the currently executing statement and whether the statement is blocked.|
|[sp_lock](https://technet.microsoft.com/en-us/library/ms187749(v=sql.110).aspx)|Reports snapshot information about locks, including the object ID, index ID, type of lock, and type or resource to which the lock applies.|
|[sp_spaceused](https://technet.microsoft.com/en-us/library/ms188776(v=sql.110).aspx)|Displays an estimate of the current amount of disk space used by a table (or a whole database).|
|[sp_monitor](https://technet.microsoft.com/en-us/library/ms188912(v=sql.110).aspx)|Displays statistics, including CPU usage, I/O usage, and the amount of time idle since sp_monitor was last executed.|

### SQL AGENT JOBS
Another ways to monitor is to ask yourself "Did all of your SQL AGENT Jobs run successfully ?"

This item can be checked with a fairly straightforward query of the msdb database. The first part of the query checks for any failed job steps and the second part is only concerned with the overall job status. This is also checked because a step could be set to continue even on failure, but should probably still be looked at in the morning. Also, if you are using the SQL Server Agent to backup your databases then this is also a good way to check if any backup jobs failed.

```
use msdb
go
select 'FAILED' as Status, cast(sj.name as varchar(100)) as "Job Name",
       cast(sjs.step_id as varchar(5)) as "Step ID",
       cast(sjs.step_name as varchar(30)) as "Step Name",
       cast(REPLACE(CONVERT(varchar,convert(datetime,convert(varchar,sjh.run_date)),102),'.','-')+' '+SUBSTRING(RIGHT('000000'+CONVERT(varchar,sjh.run_time),6),1,2)+':'+SUBSTRING(RIGHT('000000'+CONVERT(varchar,sjh.run_time),6),3,2)+':'+SUBSTRING(RIGHT('000000'+CONVERT(varchar,sjh.run_time),6),5,2) as varchar(30)) 'Start Date Time',
       sjh.message as "Message"
from sysjobs sj
join sysjobsteps sjs 
 on sj.job_id = sjs.job_id
join sysjobhistory sjh 
 on sj.job_id = sjh.job_id and sjs.step_id = sjh.step_id
where sjh.run_status <> 1
  and cast(sjh.run_date as float)*1000000+sjh.run_time > 
      cast(convert(varchar(8), getdate()-1, 112) as float)*1000000+70000 --yesterday at 7am
union
select 'FAILED',cast(sj.name as varchar(100)) as "Job Name",
       'MAIN' as "Step ID",
       'MAIN' as "Step Name",
       cast(REPLACE(CONVERT(varchar,convert(datetime,convert(varchar,sjh.run_date)),102),'.','-')+' '+SUBSTRING(RIGHT('000000'+CONVERT(varchar,sjh.run_time),6),1,2)+':'+SUBSTRING(RIGHT('000000'+CONVERT(varchar,sjh.run_time),6),3,2)+':'+SUBSTRING(RIGHT('000000'+CONVERT(varchar,sjh.run_time),6),5,2) as varchar(30)) 'Start Date Time',
       sjh.message as "Message"
from sysjobs sj
join sysjobhistory sjh 
 on sj.job_id = sjh.job_id
where sjh.run_status <> 1 and sjh.step_id=0
  and cast(sjh.run_date as float)*1000000+sjh.run_time >
      cast(convert(varchar(8), getdate()-1, 112) as float)*1000000+70000 --yesterday at 7am
```


### SQL SERVER LOGS

Looking through your sql server logs for error is a good practice.

In order to check the SQL Server Error Log we are going to use the undocumented extended stored procedure, xp_readerrorlog. This query will look at the current log and go back a maximum of 2 days looking for any errors during that time frame.

```
declare @Time_Start datetime;
declare @Time_End datetime;
set @Time_Start=getdate()-2;
set @Time_End=getdate();
-- Create the temporary table
CREATE TABLE #ErrorLog (logdate datetime
                      , processinfo varchar(255)
                      , Message varchar(500))
-- Populate the temporary table
INSERT #ErrorLog (logdate, processinfo, Message)
   EXEC master.dbo.xp_readerrorlog 0, 1, null, null , @Time_Start, @Time_End, N'desc';
-- Filter the temporary table
SELECT LogDate, Message FROM #ErrorLog
WHERE (Message LIKE '%error%' OR Message LIKE '%failed%') AND processinfo NOT LIKE 'logon'
ORDER BY logdate DESC
-- Drop the temporary table 
DROP TABLE #ErrorLog
```