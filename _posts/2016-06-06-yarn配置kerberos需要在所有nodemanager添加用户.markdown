---
layout: post
---

yarn添加kerberos认证需要在所有nodemanager上添加kerberos principal用户

### 发现问题

同事反应yarn提交任务有问题，发现yarn配置了kerberos认证，于是给新建的用户添加了principal，并且生成了keytab

```
添加principal
kadmin.local -q "addprinc -randkey spark_deploy@LOUGUANSTAR.COM"
生成keytab
kadmin.local -q "ktadd -k spark_deploy.keytab spark_deploy@LOUGUANSTAR.COM"

其他kerberos操作：
删除principal
kadmin.local -q "delprinc spark_deploy@LOUGUANSTAR.COM"
查看所有principal
./kadmin.local -q "listprincs" | grep spark
```

然后进行kerberos认证，尝试提交任务

```
kinit -kt spark_deploy.keytab spark_deploy
hadoop jar  /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples-2.6.0-cdh5.5.2.jar pi 10 100
```
任务很快结束，jobhistory中找不到日志，resourcemanager中的报错信息很简短，错误信息大致如下：

```
main : run as user is spark_deploy
main : requested yarn user is spark_deploy
User spark_deploy not found
```
### 解决问题

始终觉得问题在设置yarn.nodemanager.linux-container-executor的问题上，如果没有设置kerberos认证那么可以指定运行yarn.nodemanager.linux-container-executor；配置了kerberos之后yarn任务的用户为kerberos principal中的用户如：spark_deploy@LOUGUANSTAR.COM中即为spark_deploy
经历了多半天各种调试，最后发现：`配置kerberos之后必须在每个nodemanager上新建kerberos principal中的用户`，这样才能确保每个任务都可以在nodemanager上运行

在所有nodemanager上添加spark_deploy用户，问题成功解决





### 参考文档

> 1. [https://community.cloudera.com/t5/Batch-Processing-and-Workflow/YARN-force-nobody-user-on-all-jobs-and-so-they-fail/td-p/26050](https://community.cloudera.com/t5/Batch-Processing-and-Workflow/YARN-force-nobody-user-on-all-jobs-and-so-they-fail/td-p/26050)
