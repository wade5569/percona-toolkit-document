# pt-archiver

## 名称

pt-archiver - 将MySQL数据库中表的记录归档到另外一个表或者文件

## 简介

### 用法
```
pt-archiver [OPTIONS] --source DSN --dest DSN --where WHERE
```
-- source 和 --dest 选项使用 DSN（数据源名称）语法。如果 COPY 值是 yes, –dest 的值在未指定的情况下，默认等于 –source 给定的值.

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
