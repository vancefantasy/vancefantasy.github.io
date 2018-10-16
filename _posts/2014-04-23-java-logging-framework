---
layout: post
title: Java日志框架
---

<div class="message">
  这篇文章写于2014年，适当做了微调。
</div>

## 名次解释
>**Log4j **(The original apache logging framework for java)，很早以来，使用最为广泛的日志框架。 
>**Commons Logging**，Apache基金会所属的项目，是一套Java日志接口，之前叫Jakarta Commons Logging，后更名为Commons Logging。
>**SLF4J** (Simple logging facade for java)，类似于Commons Logging，是一套Java日志接口。
>**Logback**，一套日志组件的实现(slf4j阵营)。
>**JUL**(Java util logging)，自Java1.4以来的官方日志实现。
 
是不是觉得很混乱？先看一段历史八卦吧：

>早在1996年，E.U. SEMPER项目组(Secure Electronic Marketplace for Europe Research，欧洲安全电子交易研究所，欧盟建立的一个通过Internet进行货币支付的项目)就决定开发自己的tracing API，这套API就是Log4j的前身。其作者是Ceki Gülcü。后来这套api经过各种变种，最终成为Apache基金会项目中的一员。 期间Log4j近乎成了Java社区的日志标准。据说Apache基金会还曾建议sun引入log4j到java的标准库中，但Sun拒绝了（未求证）。
 
>2002年Java1.4发布，Sun推出了自己的日志库，JUL(Java Util Logging)，其实现基本模仿了Log4j的实现。
 
>接着，Apache推出了Jakarta  Commons Logging项目，JCL只是定义了一套日志接口（其内部也提供一个Simple Log的简单实现），支持运行时动态加载日志组件的实现，也就是说，在你应用代码里，只需调用Commons Logging的接口，底层实现可以是Log4j，也可以是Java Util Logging。即便Sun推出了官方的Log API，但实际上很少有人用。
>2006年，Ceki Gülcü不适应Apache的工作方式，离开了Apache（据说，未求证）。然后先后创建了SLF4J（日志门面接口，类似于Commons Logging）和Logback（SLF4J的实现）两个项目，并回瑞典创建了QOS公司（Quality Open Software，based in Lausanne, witzerland.），以在线销售Log4j Manual文档、SLF4J/Logback技术支持和为期2天的SLF4J/Logback技术培训为主营业务。

QOS官网上是这样描述Logback的： 
>一个通用，可靠，快速且灵活的日志框架。
 
Logback还声称：
>某些关键操作，比如判定是否记录一条日志语句的操作，其性能得到了显著的提高。这个操作在Logback中需要3纳秒，而在Log4J中则需要30纳秒。LogBack创建记录器（logger）的速度也更快：13毫秒，而在Log4J中需要23毫秒。更重要的是，它获取已存在的记录器只需94纳秒，而Log4J需要2234纳秒，时间减少到了1/23。跟JUL相比的性能提高也是显著的。
 
此后一直持续到现今，Java日志领域被划分为两大阵营：Commons Logging和SLF4J。Commons Logging在Apache大树的笼罩下，有很大的用户基数。但有证据表明，形式正在发生变化。有人[分析了GitHub上30000个项目](http://www.takipiblog.com/2013/11/20/we-analyzed-30000-github-projects-here-are-the-top-100-libraries-in-java-js-and-ruby/ )，统计出了最流行的100个Libraries，可以看出一些端倪。
 
八卦完毕。是否Log4j，commons-logging，SLF4J，Logback之间的关系一下子就清晰了？

SLF4J/Logback对比commons logging/log4j的一些优势：
1. 性能方面的优势
2. Commons logging，为了减少构建日志信息的开销，通常的做法是：

		if(log.isDebugEnabled()) {
			log.debug("User name： " +    user.getName() + " buy goods id：" + good.getId());
		}
在slf4j阵营，你只需这么做：

	log.debug("User name：{}, buy goods id ：{}", user.getName(), good.getId());
		
也就是说，slf4j把构建日志的开销放在了它确认需要显示这条日志之后，减少了内存和cpu的开销，代码也更为简洁。但计算参数的开销并没有被推迟。比如：
		
	log.debug("User name：{}, user’s password：{}", user.getName(), crypt(password));

这也是SLF4J仍保留isDebugEnabled方法的原因。

3. Commons logging是通过动态查找机制，在程序运行时，使用自己的ClassLoader寻找和载入本地具体的实现。详细策略可以看commons-logging-*.jar包中的org.apache.commons.logging.impl.LogFactoryImpl.java文件。由于OSGi不同的插件使用独立的的classLoader，其机制限制了commons logging在OSGi中的正常使用。Slf4j在编译期间，静态绑定本地的LOG库，因此可以在OSGi中正常使。 
4. SLF4J/Logback的文档是免费的，Log4j是收费的。
5. 丰富的Appender（当然你可以继承Logback提供的接口，定制自己的Appender）。
 
## 做出选择
对于一个新项目，不要犹豫，使用SLF4J+Logback吧。
仅仅是这样么？答案是NO。 
去看一看，你项目中使用的第三方jar吧。Spring依赖的是Commons logging，xSocket则使用Java Util Logging记录日志，如果只配置了Logback，你可能会丢失一部分日志甚至出错
有没有好的办法呢？当然有：使用桥接器！
桥接器是一个伪造的日志实现，它允许你将Commos logging收集的日志，重定向至SLF4J。
 
你唯一需要做的，就是将以下jar包引入你的工程：

	log4j-over-slf4j-xx.jar //log4j to slf4j
	jcl-over-slf4j-xx.jar  //commos logging
	jul-to-slf4j-xx.jar  //java Util Logging
 
注意，如果你的工程中同时存在log4j-over-slf4j.jar，slf4j-log4j12.jar，你的日志会陷入死循环进而可能导致内存溢出。

## Logback的配置
参考[logback.xml](https://github.com/vancefantasy/flyer-springmvc-rest/blob/master/src/main/resources/prod/logback.xml)

## Logback加载配置流程：
1. 尝试在classpath下查找logback-test.xml文件
2. 如果文件不存在，查找logback.xml文件
3. 如果还不存在，logback用BasicConfigurator自动对其进行配置。

##  选择规则
日志记录请求级别为p，其logger的有效级别为q，只有则当p>=q时，该请求才会被执行。
该规则是logback的核心，其他各日志框架也遵循此规则。
级别排序为：TRACE < DEBUG < INFO < WARN < ERROR。

## Logger的有效Level
Logger L可以手动分配级别。如果L未被分配，它将从父层次等级中第一个非Null继承。所以为了确保每一个logger都持有一个level，根logger需要持有一个level。
例如com.flyer，如果手动指定了，其有效Level即为手动指定。若未指定，从com继承。若com为null，从root logger继承。

## Logback的Appender
列举一些常见的appender：
>**ConsoleAppender**，输出至控制台
>**FileAppender**，输出至文件
>**RollingFileAppender**，输出至可以滚动的文件。TriggeringPolicy，决定是否以及何时进行滚动，RollingPolicy，负责滚动。TimeBasedRollingPolicy 它根据时间来制定滚动策略
>**SocketAppender**，它通过序列化LoggingEven 将日志记录输出到远程。如果远程服务是关闭的，日志会被丢弃，其后台通过一个线程定时去尝试连接远程
>**JMSTopicAppender和JMSQueueAppender**，它允许将日志输出至JMS。
>**SMTPAppender**，输出至邮件服务器。
>**DBAppender**，输出至DB，支持DB2,MySQL,Oracle,SQL Server，HSQL,PostgreSQL等。
>**SyslogAppender**，输出至*nix的syslog守护线程。
 
## Logback的过滤器
基于三值逻辑（ternary logic），允许把它们组装成链，从而组成任意的复合过滤策略。过滤器很大程度上受到Linux的iptables启发。
>**DENY**，立即被抛弃。
>**NEUTRAL**，交给下一个过滤器处理。
>**ACCEPT**，立即处理。

## 其他特性
Logback的其他特性，例如排版规则，MDC(Mapped Diagnostic Context)，JMX注册等可以参考[Logback的官方文档](http://logback.qos.ch/manual/index.html)，讲的很详细。



