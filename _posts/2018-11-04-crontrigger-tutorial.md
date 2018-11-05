---
layout: post
title: CronTrigger教程
---

<div class="message">
  这篇文章翻译自Quartz官方文档。
</div>

## 介绍
Cron是一个存在了很长时间的UNIX工具，它的调度功能很强大而且经过了验证。CronTrigger类基于cron的调度功能。
CronTrigger使用“cron表达式”，它能够创建类似这样的触发策略：“每周一至周五上午8:00”或“每月最后一个星期五凌晨1:30”
Cron表达式很强大，但（规则）可能令人很困惑。 本教程旨在揭示撰写Cron表达式的一些谜团，为用户提供在论坛或邮件列表咨询之前可以访问的资源。

## 格式

Cron表达式是由空格分隔的6或7个字段组成的字符串。 字段可以包含任何允许的值，以及该字段允许的特殊字符的各种组合。 字段如下：

字段       |是否必须       |允许值  |允许特殊字符  
------------|-----------|-----------|--------
Seconds       |是        |0-59|, - * /
Minutes       |是        |0-59|, - * /
Hours       |是        |0-23|, - * /
Day of month       |是        |1-31|, - * ? / L W
Month       |是        |1-12 or JAN-DEC	|, - * /
Day of week       |是        |1-7 or SUN-SAT|, - * ? / L #
Year       |否        |empty, 1970-2099|, - * /

所以cron表达式可以像这样简单：* * * * ? *
或者更复杂一些，像这样：0/5 14,18,3-39,52 * ? JAN,MAR,SEP MON-FRI 2002-2010

## 特殊字符
- **\*** (所有值) - 表示选择字段中所有的值。例如，“*” 在Minutes 字段表示“每分钟”
- **?**（没有具体的值）- 当您需要在允许该字符的两个字段之一中指定某些内容时很有用。 例如，如果我希望我的触发器在该月的某个特定日期（例如，第10天）触发，但不关心在一周的哪一天，我会在day-of-month字段放置“10”，以及day-of-week字段放置“？”。 请参阅以下示例以获得说明。
> 这里有点拗口，其实很好理解，就是day-of-month和day-of-week2个字段是互斥的，因为它们都表示的天。如果两者中有一个指定了值，另一个必须设置成"?"，用来区分。译者注。

- **-** - 用于指定范围。 例如，Hours字段中的“10-12”表示“小时10,11和12”。
- **,** - 用于指定具体值。 例如，day-of-week字段中的“MON，WED，FRI”表示“星期一，星期三和星期五”。
- **/** - 用于指定增量。 例如，Seconds字段中的“0/15”表示“秒0,15,30和45”。 秒字段中的“5/15”表示“秒5,20,35和50”。 你也可以在''字符后指定'/' - 在这种情况下''相当于在'/'之前有'0'。 日期字段中的“1/3”表示“从该月的第一天开始每3天触发一次”。
> 即 /10 相当于 0/10。译者注

- **L**(LAST) - 在允许它的两个字段中的每个字段中具有不同的含义。 例如，day-of-month字段中的值“L”表示“月份的最后一天” - 1月31日，非闰年2月28日。 如果在day-of-week字段中，则表示“7”或“SAT”。 但是如果在day-of-week字段中用在另一个值后，则表示“该月的最后一个xxx日” - 例如“6L”表示“该月的最后一个星期五”。 您还可以指定从该月的最后一天开始的偏移量，例如“L-3”，这意味着该日历月的倒数第三天。 使用“L”选项时，重要的是不要指定列表或值范围，因为您会得到令人困惑/意外的结果。
- **W**(weekday) - 用于指定最接近给定日期的工作日（周一至周五）。 例如，如果您指定“15W”作为day-of-month字段的值，则含义为：“最近的工作日到该月的15日”。 因此，如果15日是星期六，触发器将在14日星期五触发。 如果15日是星期日，触发器将在16日星期一触发。 如果15日是星期二，那么它将在15日星期二触发。 但是，如果您指定“1W”作为day-of-month的值，并且第1日是星期六，则触发器将在星期一（3日）触发，因为它不会“跳过”一个月的边界。 只有当day-of-month是一个单个的天，而不是范围或天数列表时，才能使用“W”字符。

> 'L'和'W'字符也可以在day-of-month字段中组合以产生'LW'，这转换为*“最后一个工作日”*。

- **#** - 用于指定当月的“第n个”XXX天。 例如，day-of-week字段中的“6#3”的值表示“该月的第三个星期五”（第6天=星期五，“#3”=该月份的第3个星期五）。 其他例子：“2#1”=本月的第一个星期一，“4#5”=该月的第五个星期三。 请注意，如果您指定“#5”并且该月中没有给定星期几的5，则该月不会发生任何触发。

> 法定字符以及一周中几个月和几天的名称不区分大小写。 MON与mon相同。

## 例子
这是几个完整的例子：

表达式       |含义 
------------|---------------
0 0 12 * * ?       |每天12:00点触发
0 15 10 ? * *       |每天10:15触发
0 15 10 * * ?       |每天10:15触发
0 15 10 * * ? *       |每天10:15触发
0 15 10 * * ? 2005       |在2015年，每天上午10:15触发
0 * 14 * * ?       |每天，从14:00开始，到14:59结束，每1分钟触发
0 0/5 14 * * ?       |每天，从14:00开始，到14:55结束，每5分钟触发
0 0/5 14,18 * * ?       |每天，在14:00-14:55,18:00-18:55时间段内，每5分钟触发
0 0-5 14 * * ?       |每天，在14:00-14:05，每分钟触发
0 10,44 14 ? 3 WED       |每年3月的每个星期3，14:10和14:44触发
0 15 10 ? * MON-FRI       |每个星期一到星期五的10:15触发
0 15 10 15 * ?       |每月15日，10:15触发
0 15 10 L * ?       |每月最后一天，10:15触发
0 15 10 L-2 * ?       |每月距最后1天的前2天，10:15触发
0 15 10 ? * 6L       |每个月最后一个星期五的10:15触发
0 15 10 ? * 6L 2002-2005       |在2002，2004，2004，2005年，每个月最后一个星期五的10:15触发
0 15 10 ? * 6#3    |每个月第3个星期五的10:15触发
0 0 12 1/5 * ?       |从1日开始，每隔5天的12:00触发
0 11 11 11 11 ?       |每个11月的11:11触发
      
> 注意'?' 和'*' 在day-of-week和day-of-month字段中的用法！

## 说明
- 同时指定day-of-week和day-of-month的支持尚未完成（您必须在其中一个字段中使用'？'字符）。
- 在您的语言环境中发生“夏令时”更改的早上几小时之间设置触发时间时要小心（对于美国语言环境，这通常是凌晨2:00之前和之后的小时)，因为时移会导致跳过或重复执行，这取决于时间向后移动还是向前跳跃。您可以参考此维基百科条目这有助于确定您的语言环境的细节：https://secure.wikimedia.org/wikipedia/en/wiki/Daylight_saving_time_around_the_world
