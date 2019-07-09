# pt-archiver

## 名称

pt-archiver - 将MySQL数据库中表的记录归档到另外一个表或者文件

## 简介

### 用法
```
pt-archiver [OPTIONS] --source DSN --dest DSN --where WHERE
```
-- source 和 --dest 选项使用 DSN（数据源名称）语法。如果 COPY 值是 yes, –dest 的值在未指定的情况下，默认等于 –source 给定的值.
pt-archiver数据归档使用大全
-where 'id<3000'	设置操作条件
--limit 10000	每次取1000行数据给pt-archive处理
--txn-size 1000	设置1000行为一个事务提交一次
--progress 5000	每处理5000行输出一次处理信息
--statistics	结束的时候给出统计信息：开始的时间点，结束的时间点，查询的行数，归档的行数，删除的行数，以及各个阶段消耗的总的时间和比例，便于以此进行优化。只要不加上--quiet，默认情况下pt-archive都会输出执行过程的
--charset=UTF8	指定字符集为UTF8，字符集需要对应当前库的字符集来操作
--no-delete	表示不删除原来的数据，注意：如果不指定此参数，所有处理完成后，都会清理原表中的数据
--bulk-delete	批量删除source上的旧数据
--bulk-insert	批量插入数据到dest主机 (看dest的general log发现它是通过在dest主机上LOAD DATA LOCAL INFILE插入数据的)
--purge	删除source数据库的相关匹配记录
--local	不把optimize或analyze操作写入到binlog里面（防止造成主从延迟巨大）
--analyze=ds	操作结束后，优化表空间（d表示dest，s表示source）
默认情况下，pt-archiver操作结束后，不会对source、dest表执行analyze或optimize操作，因为这种操作费时间，并且需要你提前预估有足够的磁盘空间用于拷贝表。一般建议也是pt-archiver操作结束后，在业务低谷手动执行analyze table用以回收表空间。

1）注意--source后的DSN之间不能空格出现，否则会出错。 --where条件的值，有字符串的，要用引号括起来。 
2) --limit表示，每组一次删除多少条数据（注意：如果数据比较多时，也可以设置大一些，减少循环次数），最终的清理操作，还是通过Where pK=xx来处理的；

### 示例

#### 创建示例表
```
CREATE DATABASE IF NOT EXISTS percona;
USE percona;
CREATE TABLE pt_archiver (
  id INT         NOT NULL AUTO_INCREMENT,
  c1 VARCHAR(50) NOT NULL DEFAULT '',
  c2 VARCHAR(50) NOT NULL DEFAULT '',
  c3 VARCHAR(50) NOT NULL DEFAULT '',
  PRIMARY KEY (id)
);
INSERT INTO pt_archiver (c1, c2, c3) VALUES ('fake_data', rand(), rand());
```
#### 命令示例
```
1、全表归档，不删除原表数据，非批量插入----批量插入
pt-archiver --source h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --dest h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --charset=UTF8 --where '1=1' --progress 10000 --limit=10000 --txn-size 10000 --statistics --no-delete ##

pt-archiver --source h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --dest h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --charset=UTF8 --where '1=1' --progress 10000 --limit=10000 --txn-size 10000 --bulk-insert --bulk-delete --statistics --no-delete

2、数据归档，删除原表数据，非批量插入、非批量删除--批量插入、批量删除
pt-archiver --source h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --dest h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --charset=UTF8 --where '1=1' --progress 10000 --limit=10000 --txn-size 10000 --statistics --purge

pt-archiver --source h=172.16.1.10,P=3306,u=backup_user,p='xxx',,D=test123,t=c1 --dest h=172.16.1.10,P=3306,u=backup_user,p='xxx',D=test123,t=c1 --charset=UTF8 --where '1=1' --progress 10000 --limit=10000 --txn-size 10000 --bulk-insert --bulk-delete --statistics --purge


3、用于把数据导出文件，不用删除原表中数据
pt-archiver --source h=127.0.1.1,P=3306,D=test,t=test --charset=UTF8 --where 'itemID>100' --progress 1000 --file "/tmp/aa.txt" --limit=10000 --no-delete

4、强制指定索引，通过参数i来指定索引名字，默认按PRIMARY走，数据大的时候非常慢

 pt-archiver --source h='xx',P='3306',u='xx',p='xx',D='db_order',t='xx' --dest h='xx',P='3306',u='xx',p='xx',D='xx',t='xx' --charset=utf8mb4,i=index_createTime  --where 'createTime<20180201000000' --progress 10000 --limit 10000 --statistics

5、主键冲突数据归档，通过replace来解决

 pt-archiver --source h='xx',P='3306',u='xx',p='xx',D='db_order',t='xx' --dest h='xx',P='3306',u='xx',p='xx',D='xx',t='xx' --charset=utf8mb4  --replace  --where 'createTime<20180201000000' --progress 10000 --limit 10000 --statistics

6、通过dry-run来查看PT的执行计划，数据查询使用的索引

pt-archiver --source h='xx',P='3306',u='xx',p='xx',D='db_order',t='xx' --dest h='xx',P='3306',u='xx',p='xx',D='xx',t='xx' --charset=utf8mb4,i=index_createTime --replace  --where 'createTime<20180201000000' --progress 10000 --limit 10000 --statistics --dry-run

7、source 与dest的字符集不统解决方式

pt-archiver --source h='xx',P='3306',u='xx',p='xx',D='db_order',t='xx',A=utf8mb4 --dest h='xx',P='3306',u='xx',p='xx',D='xx',t='xx' --charset=UTF8 --where 'createTime<20180201000000' --progress 10000 --limit 10000 --statistics

```
