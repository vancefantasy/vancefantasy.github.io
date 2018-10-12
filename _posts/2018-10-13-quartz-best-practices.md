---
layout: post
title: Quartz最佳实践
---

<div class="message">
  这篇文章翻译自Quartz<a href="http://www.quartz-scheduler.org/documentation/best-practices.html">官方文档<a>。
</div>
## 生产系统建议
#### 跳过更新检查
Quartz包含一个“更新检查”机制，这个机制会尝试连接远端服务器，来检查是否有新版本可供下载。该检查是异步运行的，不会影响到启动/初始化，并且如果连接失败检查的节奏会逐渐衰减。如果运行检查时发现有更新，Quartz会通过打印日志的方式报告出来。

你可以使用Quartz配置属性“org.quartz.scheduler.skipUpdateCheck：true”或系统属性
“org.terracotta.quartz.skipUpdateCheck = true”（你可以在系统环境中设置或在java命令中以 -D的形式) 来禁用更新检查。 建议你生产环境部署时禁用更新检查。

## JobDataMap建议
#### 在JobDataMap中只存储原始数据类型(包括String)
在JobDataMap中只存储原始数据类型(包括String)，短期和长期来看，这可以避免数据序列化问题。

#### 使用Merged JobDataMap
在Job执行时，可以很方便的从JobExecutionContext中获取JobDataMap使用。此JobDataMap是从JobDetail和Trigger获取的JobDataMap的合并，且前者中同名的值会被后者覆盖。

你在调度器上的一个Job，可供多个Triggers定期/重复使用，此时将JobDataMap值存储在Trigger上，每次独立触发时，你的Job就可以获取不同的输入。

鉴于以上所有，以下是我们推荐的最佳实践：在Job.execute(..)的代码中，应该从JobExecutionContext中的JobDataMap获取数据，而不是直接从JobDetail中获取。

## Trigger建议
#### 使用TriggerUtils
TriggerUtils:
* 提供一种更简单的方法去创建triggers(schedulers)
* 包含丰富的方法去用符合特定描述的方式去创建triggers和schedulers，而不是直接实例化具体的triggers(例如：SimpleTrigger, CronTrigger等)，然后调用setter方法去配置它们
* 提供一种更简单的方式创建Dates(for start/end dates)
* 给分析triggers提供帮助(例如：计算未来触发时间)

## JDBC建议
#### 不要对Quartz的表直接写入
直接想数据库写入调度数据（通过SQL）而不是使用scheduling API:

* 导致数据占用(删除数据、抢占数据)
* 导致trigger触发时间到了，job没有执行但数据莫名其妙“消失”
* 导致trigger触发时间到了，job没有执行执行
* 可能导致死锁
* 其他奇怪的问题、数据占用
#### 在非集群模式下，不要将两个调度器(scheduler)指向一个数据库。
如果你将多于一个scheduler实例指向同一组数据库表，且这些实例没有配置集群模式，可能产生以下结果：
* 导致数据占用(删除数据、抢占数据)
* 导致trigger触发时间到了，job没有执行但数据莫名其妙“消失”
* 导致trigger触发时间到了，job没有执行执行
* 可能导致死锁
* 其他奇怪的问题、数据占用

#### 确保充足的数据库连接
建议将数据源最大连接大小配置为至少为线程池中的工作线程数加3。如果您的应用程序也频繁调用调度程序API，则可能需要其他连接。 如果您使用的是JobStoreCMT，则“非托管”数据源的最大连接大小应至少为4。

## 夏令时(DST)
#### 避免在夏令时过渡时间附近调度job
注意: 过渡时间的细节和时钟向前或向后移动的时间因地点而异，详细参考：https://secure.wikimedia.org/wikipedia/en/wiki/Daylight\_saving\_time\_around\_the\_world.

SimpleTriggers不受夏令时影响，因为它们总是以精确的毫秒时间触发，并重复精确的毫秒数.

因为CronTriggers在给定的小时/分钟/秒时触发，所以当DST转换发生时它们会受到一些奇怪的影响.

作为可能出现问题的一个例子，在美国的TimeZones /位置观察夏令时，如果使用CronTrigger并在凌晨1:00和凌晨2:00安排开火时间，可能会出现以下问题：

* 1:05 AM 可能触发两次! - 可能在CronTrigger上重复触发
* 2:05 AM 可能会丢失触发! - 可能错过CronTrigger的触发
同样，时间和调整量的具体情况因地区而异。

其他基于沿日历滑动的触发类型（而不是精确的时间量），例如CalenderIntervalTrigger，也会受到类似的影响 - 但是不是错过了一次触发，或者两次触发，最终可能会让它的触发时间偏移一个小时。

## Jobs
#### 等待条件
长时间运行的job可能会阻止别的任务执行（如果线程池中所有的线程都是busy状态）
如果你需要在job执行线程中调用Thread.sleep()，这通常表明你的Job还没有准备好做接下来的工作，因为它需要等待一些其他条件（例如一些数据还没有准备好）。
一个更好的办法是，释放工作线程（exit the job），把资源留给其他的job。你可以让重新调度这个job，或者在退出前调度其他job。

#### 抛出异常
一个job的execute方法应该包含try-catch块，处理所有可能的异常。
如果一个job抛出异常，quartz典型的做法是重新执行它（当然它多半再次抛出同样的异常）。如果一个job 捕获可能遇到的所有异常，处理他们，重新调度它自己，或其他任务，就能解决这个问题。

#### 可恢复性和幂等性
进行中的job，如果标记为“可恢复”，在scheduler失效后会被重新调度。这意味着一些job做的工作会被执行两次。
这意味着job需要被编码成幂等的方式。

## Listeners(TiggerListener,JobListener,SchedulerListener)
#### 确保Listeners中的代码够简洁、高效。
不建议在Listener中做大量的工作，因为执行Job的线程和Listeners捆绑在一起。

#### 处理你的Exceptions
每个listener的方法都应该包含try-catch代码块，处理所有可能的exception。
如果一个listener抛出一个异常，可能导致别的listeners不能被通知到，或者会中止job的执行。

## 通过应用程序暴露Scheduler的方法
#### 小心安全！
一些用户会把quartz的Scheduler方法通过应用程序接口暴露出去。这看起来很有用，不过也极其危险。

你要确定不能让用户随意定义他们想要的Job类型。例如，quartz提供了一种预制的任务 org.quartz.jobs.NativeJob，这种Job可以执行任意的、他们事先定义好的原生系统命令（操作系统级别）。
同样，其他的任务例如SendEmailJob，包括其他任何有恶意意图的Job

__为了更高效率，允许用户定义任意的Job，你可能都将会面临与OWASP和MITER定义的命令行攻击 相当的威胁。__

