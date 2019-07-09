1. pt-deadlock-logger

功能介绍：

提取和记录 mysql 死锁的相关信息

用法介绍：

pt-deadlock-logger [OPTION...] SOURCE_DSN
收集和保存 mysql 上最近的死锁信息，可以直接打印死锁信息和存储死锁信息到数据库中死锁信息包括发生死锁的服务器、最近发生死锁的时间、死锁线程id、死锁的事务 id、发生死锁时事务执行了多长时间等等非常多的信息。详情见下面的示例。

使用示例：

范例 1： 打印本地 mysql 的死锁信息

pt-deadlock-logger --user=root --password=hugnew h=localhost –print
范例 2：将本地的 mysql 死锁信息记录到数据库的表中，也打印出来

pt-deadlock-logger --user=root --password=hugnew h=localhost --print D=test,t=deadlocks
PS: 更多资料查看官方文档：https://www.percona.com/doc/percona-toolkit/2.2/pt-deadlock-logger.html

2. pt-fk-error-logger

功能介绍：

提取和记录 mysql 外键错误信息

用法介绍：

pt-fk-error-logger [OPTION...] SOURCE_DSN
通过 SHOW INNODB STATUS提取和保存mysql数据库最近发生的外键错误信息。可以通过参数控制直接打印错误信息或者将错误信息存储到数据库的表中。
