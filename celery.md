# celery

参考：[Celery基本使用及其在django项目中的应用 - the3times - 博客园 (cnblogs.com)](https://www.cnblogs.com/the3times/p/13377467.html)

​			[Celery - 分布式任务队列 — Celery 3.1.7 文档 (jinkan.org)](http://docs.jinkan.org/docs/celery/)



Celery 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。

它是一个专注于实时处理的任务队列，同时也支持任务调度。



**使用场景**

异步执行：解决耗时任务,将耗时操作任务提交给Celery去异步执行，比如发送短信/邮件、消息推送、音视频处理等等

延迟执行：解决延迟任务

定时执行：解决周期(周期)任务，比如每天数据统计



**Celery异步任务框架**

- 可以不依赖任何服务器，通过自身命令，启动服务(内部支持socket)
- celery服务为为其他项目服务提供异步解决任务需求的
- 会有两个服务同时运行，**一个是项目服务，一个是celery服务，项目服务将需要异步处理的任务交给celery服务，celery就会在需要时异步完成项目的需求**



架构图

![image-20210709120027385](C:\Users\cheny\AppData\Roaming\Typora\typora-user-images\image-20210709120027385.png)