---
layout: post
title: jmeter plugins 学习
excerpt: Jmeter plugin可以扩展原生jmeter的功能，现在越来越多的软件涉及都趋向于插件式，maven，jenkins/hudson,trc等
comments: true
---

1 下载地址
http://code.google.com/p/jmeter-plugins/
作者为俄罗斯人，俄文看不懂。作者的更新频率还是比较高的，在项目主页上可以给作者留言。
 
2 安装
下载解压之后，将JMeterPlugins.jar拷贝到lib/ext目录下面。
如果需要用服务器资源监控的，需要将serverAgent目录上传到测试服务器上。serverAgent的默认端口是4444，./startAgent.sh 9998，这样可以自定义端口。
serverAgent是作者使用了SIGAR作为资源监控的工具，这个是另一个开源工具，详情访问
http://support.hyperic.com/display/SIGAR/Home#Home-overview

3 环境需求
JMeter 2.4 以上
JRE 1.6 以上 

4 插件功能列表
4.1 Stepping Thread Group
<img src="/images/stepping-thread-group.jpg" class="alignmiddle">
默认的thread group功能只能实现并发和平均的请求数，还不足以模拟真实的一些业务场景。
Stepping Thread Group类似与loadrunner里面的scenario schedule功能，熟悉loadrunner的人这个功能应该不陌生。 

4.2 Ultimate Thread Group
"Ultimate" means there will be no need in further Thread Group plugins. The features that everyone needed in JMeter and they finally available:
1 infinite number of schedule records
2 separate ramp-up time, shutdown time, flight time for each shedule record
3 and, of course, trustworthy load preview graph
以上是作者给ultimate thread group的定义，试验之后，这个功能确实比较实用。在ultimate thread group下，可以根据实际的业务场景来设置虚拟用户的请求情况，可添加的场景计划数没有限制。如果能增加调度器的功能，那么这个功能就比较完美了。
<img src="/images/active-threads-over-time.jpg" class="alignmiddle">
<img src="/images/ultimate-thread-group.jpg" class="alignmiddle">
从两张虚拟用户的曲线来看，计划和实际运行的结果是一致的。
4.3 Parameterized Controller
4.4 Variables From CSV File
4.5 Dummy Sampler
可以用来模拟response data，方便自己写beanshell脚本的调试或者正则匹配的调试。
4.6 Active Threads Over Time
Active Threads Over Time is a simple listener showing how many active threads are there in each thread group during test run.
配合Ultimate Thread Group一起使用。