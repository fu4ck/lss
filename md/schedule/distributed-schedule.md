

<!-- TOC -->

- [1、调度框架](#1调度框架)
    - [1、Quartz](#1quartz)
        - [1、使用样例](#1使用样例)
        - [2、介绍](#2介绍)
    - [2、elastic-job](#2elastic-job)
    - [3、XXL-JOB](#3xxl-job)
    - [4、LTS](#4lts)
    - [5、Uncode-Schedule](#5uncode-schedule)
    - [6、Antares](#6antares)
    - [7、niubi-job](#7niubi-job)
- [参考](#参考)

<!-- /TOC -->




分布式定时调度

把定时任务通过集群的方式进行管理调度，并采用分布式部署，保证系统的高可用，提高了容错。

> 问题：

- 1、那么如何保证定时任务只在集群的某一个节点上执行？

- 2、一个任务如何拆分为多个独立的任务项，由分布式的机器去分别执行？

- 3、众多的定时任务如何统一管理？


# 1、调度框架

## 1、Quartz

### 1、使用样例

```java
public static void refreshTask() {
	try {
		//1、jobDetail 任务详情描述
		JobDetail jobDetail = JobBuilder.newJob(RefreshTokenJob.class).withIdentity("cronJob","group1").build();
		//2、cronTrigger 定时调度规则
		CronTrigger cronTrigger = TriggerBuilder.newTrigger().withIdentity("cronTrigger")
				.withSchedule(CronScheduleBuilder.cronSchedule("0 0 0/1 * * ?"))//整点执行
				.build();
		//3、Scheduler实例
		StdSchedulerFactory stdSchedulerFactory = new StdSchedulerFactory();
		Scheduler scheduler = stdSchedulerFactory.getScheduler();
		scheduler.start();
        //4、设置任务和调度规则
		scheduler.scheduleJob(jobDetail,cronTrigger); 

	} catch (SchedulerException e) {
		e.printStackTrace();
	} finally {

	}
}


//封装业务逻辑
public class RefreshTokenJob implements Job {
    private static final Logger logger = LoggerFactory.getLogger(RefreshTokenJob.class);

    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        //打印当前的执行时间 例如 2017-11-23 00:00:00
        Date date = new Date();
        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println("现在的时间是："+ sf.format(date));
        //具体的业务逻辑
        runBusiness();
        System.out.println("Hello Quartz");
    }
}
```

### 2、介绍

Quartz集群中每个节点都是一个单独的Quartz应用，它又管理着其他的节点。这个集群需要每个节点单独的启动或停止；和我们的应用服务器集群不同，独立的Quratz节点之间是不需要 通信的。不同节点之间是通过数据库表来感知另一个应用。只有使用持久的JobStore才能完成Quartz集群。

![](../../pic/2020-04-13-10-27-10.png)



- 1、Job：表示一个工作，要执行的具体内容。此接口中只有一个方法void execute(JobExecutionContext context)

- 2、JobDetail：JobDetail表示一个具体的可执行的调度程序，Job是这个可执行程调度程序所要执行的内容，另外JobDetail还包含了这个任务调度的方案和策略。

- 3、Trigger代表一个调度参数的配置，什么时候去调。

- 4、Scheduler代表一个调度容器，一个调度容器中可以注册多个JobDetail和Trigger。当Trigger与JobDetail组合，就可以被Scheduler容器调度了。

![](../../pic/2020-04-13-10-29-08.png)

上图三个节点在数据库中都拥有同一份Job定义，如果某一个节点失效，那么Job会在其他节点上执行。由于三个节点上的Job执行代码是一样的，那么怎么保证只有在一台机器上触发呢？答案是使用了数据库锁。在quartz的集群解决方案里有张表scheduler_locks，quartz采用了悲观锁的方式对triggers表进行行加锁，以保证任务同步的正确性。一旦某一个节点上面的线程获取了该锁，那么这个Job就会在这台机器上被执行，同时这个锁就会被这台机器占用。同时另外一台机器也会想要触发这个任务，但是锁已经被占用了，就只能等待，直到这个锁被释放。之后会看trigger状态，如果已经被执行了，则不会执行了。

> 缺点

但是对于大量的短任务，各个节点都会抢占数据库锁，这样就出现大量的线程等待资源。这种情况随着节点的增加会越来越严重。quartz的分布式只是解决了高可用的问题，并没有解决任务分片的问题，还是会有单机处理的极限。




## 2、elastic-job

[elastic-job-lite源码](https://github.com/elasticjob/elastic-job-lite)

Elastic-Job is a distributed scheduled job framework, based on Quartz and Zookeeper.

Elastic-Job是一个分布式调度解决方案，由两个相互独立的子项目Elastic-Job-Lite和Elastic-Job-Cloud组成。

Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务；Elastic-Job-Cloud采用自研Mesos Framework的解决方案，额外提供资源治理、应用分发以及进程隔离等功能。

elastic-job主要的设计理念是无中心化的分布式定时调度框架，思路来源于Quartz的基于数据库的高可用方案。但数据库没有分布式协调功能，所以在高可用方案的基础上增加了弹性扩容和数据分片的思路，以便于更大限度的利用分布式服务器的资源。


![](../../pic/2020-04-13-14-34-25.png)


## 3、XXL-JOB

[xxl-job源码](https://github.com/xuxueli/xxl-job)

XXL-JOB是一个轻量级分布式任务调度平台,调度采用中心式设计，“调度中心”基于集群Quartz实现并支持集群部署。任务分布式执行，任务"执行器"支持集群部署。

> 设计思想

将调度行为抽象形成“调度中心”公共平台，而平台自身并不承担业务逻辑，“调度中心”负责发起调度请求。将任务抽象成分散的JobHandler，交由“执行器”统一管理，“执行器”负责接收调度请求并执行对应的JobHandler中业务逻辑。因此，“调度”和“任务”两部分可以相互解耦，提高系统整体稳定性和扩展性；



## 4、LTS 

- [light-task-scheduler源码（两年没有更新）](https://github.com/ltsopensource/light-task-scheduler)

LTS，light-task-scheduler，是一款分布式任务调度框架, 支持实时任务、定时任务和 Cron 任务。有较好的伸缩性和扩展性，提供对 Spring 的支持（包括 Xml 和注解），提供业务日志记录器。支持节点监控、任务执行监、JVM 监控，支持动态提交、更改、停止任务。 


## 5、Uncode-Schedule 

- [源码（4年没有更新）](https://github.com/uncodecn/uncode-schedule)

Uncode-Schedule 是基于 ZooKeeper + Quartz / spring task 的分布式任务调度组件，确保每个任务在集群中不同节点上不重复的执行。支持动态添加和删除任务，支持添加 ip 黑名单，过滤不需要执行任务的节点。 

![](../../pic/2020-04-13-14-22-13.png)



## 6、Antares 

- [源码（3年没有更新）](https://github.com/ihaolin/antares)

Antares 是一款基于 Quartz 机制的分布式任务调度管理平台，内部重写执行逻辑，一个任务仅会被服务器集群中的某个节点调度。用户可通过对任务预分片，有效提升任务执行效率；也可通过控制台 antares-tower 对任务进行基本操作，如触发，暂停，监控等。 

![](../../pic/2020-04-13-14-23-17.png)


## 7、niubi-job

一个高可用的，专门针对定时任务的分布式任务调度框架

- [niubi-job源码（4年没有更新）](https://github.com/xiaolongzuo/niubi-job)






# 参考

- [分布式定时任务调度框架](https://www.jianshu.com/p/ab438d944669)

- [互联网分布式任务调度工具](https://www.jianshu.com/p/5718b0f13d99)

- [【niubi-job——一个分布式的任务调度框架】----框架设计原理以及实现](https://www.cnblogs.com/zuoxiaolong/p/niubi-job-3.html)

- [这些优秀的国产分布式任务调度系统，你用过几个？](https://www.jianshu.com/p/39515916a63d)

- [分布式定时任务（一）](https://www.jianshu.com/p/e0e7e8494d96)

- [定时任务的分布式调度](https://www.cnblogs.com/haoxinyue/p/6886196.html)