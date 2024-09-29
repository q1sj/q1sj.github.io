---
layout: post
title: "生产环境定时任务不执行问题排查过程"
date: 2023-06-27
tags: [java]
---
定时任务采用的Quartz开源框架,生产环境服务运行一段时间后,定时任务全部不执行,重启服务后恢复

## 可能导致原因

### 时钟回拨

假设

正确时间:2023-01-01 00:00:00

系统时间:2024-01-01 00:00:00

cron:5分钟一次

在2024-01-01 00:00:00执行了第一次定时任务,那么下次执行时间应是在2024-01-01 00:05:00,此时服务器回拨2023-01-01 00:00:
00,那么下次执行就是在一年后

#### 问题定位

查看NEXT_FIRE_TIME字段 定时任务下次执行时间是否正确

```sql
SELECT * FROM `qrtz_triggers`
```

| SCHED_NAME     | TRIGGER_NAME | TRIGGER_GROUP | JOB_NAME | JOB_GROUP | DESCRIPTION | NEXT_FIRE_TIME | PREV_FIRE_TIME | PRIORITY | TRIGGER_STATE | TRIGGER_TYPE | START_TIME     | END_TIME | CALENDAR_NAME | MISFIRE_INSTR | JOB_DATA       |
|----------------|--------------|---------------|----------|-----------|-------------|----------------|----------------|----------|---------------|--------------|----------------|----------|---------------|---------------|----------------|
| DemoScheduler	 | TASK_22	     | DEFAULT       | 	TASK_22 | 	DEFAULT	 |             | 1687856760000	 | 1687856700000	 | 5	       | BLOCKED	      | CRON	        | 1687856674000	 | 0        | 		            | 2	            | (BLOB) 0 bytes |
| DemoScheduler	 | TASK_23	     | DEFAULT       | 	TASK_23 | 	DEFAULT	 |             | 1687856940000  | 	-1            | 	5       | 	WAITING	     | CRON	        | 1687856674000	 | 0        | 	             | 2	            | (BLOB) 0 bytes |

### 线程池耗尽

定时任务耗时过久,死锁或死循环导致线程池耗尽无法执行新的任务

#### 问题定位

查询执行中的任务数量,比较SCHED_TIME是否耗时过久

```sql
SELECT *
FROM `qrtz_fired_triggers`
```

使用arthas thread命令查看线程池运行情况

```shell
thread --all |grep Worker
#35       Scheduler_Worker-1                             main                      5                 TIMED_WAITING    0.0               0.000            0:0.000           false            false            
#36       Scheduler_Worker-2                             main                      5                 TIMED_WAITING    0.0               0.000            0:0.000           false            false            
#37       Scheduler_Worker-3                             main                      5                 TIMED_WAITING    0.0               0.000            0:0.000           false            false
```

查看线程堆栈信息定位问题代码

```shell
thread 35
#"DemoScheduler_Worker-1" Id=35 TIMED_WAITING
#    at java.lang.Thread.sleep(Native Method)
#    at java.lang.Thread.sleep(Thread.java:342)
#    at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386)
#    at com.xsy.modules.job.task.TestTask.run(TestTask.java:29)
#    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
#    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
#    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
#    at java.lang.reflect.Method.invoke(Method.java:498)
#    at com.xsy.modules.job.utils.ScheduleJob.executeInternal(ScheduleJob.java:63)
#    at org.springframework.scheduling.quartz.QuartzJobBean.execute(QuartzJobBean.java:75)
#    at org.quartz.core.JobRunShell.run(JobRunShell.java:202)
#    at org.quartz.simpl.SimpleThreadPool$WorkerThread.run(SimpleThreadPool.java:573)
```