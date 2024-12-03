## 主从形式

- 一主一从。

  ```mermaid
  graph LR
  M --> S
  ```

- 一主多从。

  ```mermaid
  graph LR
  M --> S1
  M --> S2
  ```

- 多主一从。

  ```mermaid
  graph LR
  M1 --> S
  M2 --> S
  ```

- 双主复制。

  ```mermaid
  graph LR
  M1 <--> M2
  ```

- 级联复制。

  ```mermaid
  graph LR
  M --> S1
  S1 --> S2
  S1 --> S3
  S3 --> S4
  ```

## 主从配置

主从同步不会创建库和表，在同步前需要保证同步的数据库和表存在。

主从服务器数据库版本必须一致。

#### 一主一从配置

###### 主配置

1. 配置 `/etc/mysql/mysql.conf.d/mysqld.cnf` 文件。

   ```ini
   [mysqld]
   log_bin                      # 开启二进制日志
   server_id    = 1             # 服务器id，不能和其他服务器id冲突
   bind-address = 0.0.0.0       # 允许远程访问
   ```

2. 创建从服务器使用的账户。

   ```mysql
   create user 'slave'@'%' identified by '12345678';
   grant replication slave, replication client on *.* to 'slave'@'%';
   ```

3. 备份数据库，手动同步初始化从数据库。

   ```shell
   sudo mysqldump -uroot -p --all-databases --single-transaction --source-data --flush-logs > bk.sql
   ```

4. 查看主配置状态。

   ```mysql
   show master status;

   +----------------------------+----------+--------------+------------------+-------------------+
   | File                       | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
   +----------------------------+----------+--------------+------------------+-------------------+
   | LAPTOP-A0EHGSSN-bin.000001 |  1078441 |              |                  |                   |
   +----------------------------+----------+--------------+------------------+-------------------+
   ```

###### 从配置

1. 配置 `/etc/mysql/mysql.conf.d/mysqld.cnf` 文件。

   ```ini
   [mysqld]
   server_id=2
   ```

2. 手动同步数据。

   ```shell
   sudo mysql -uroot -p < bk.sql
   ```

3. 设置主从关系。

   ```mysql
   change master to
   master_host='xxx.xxx.xxx.xxx',
   master_user='slave',
   master_password='12345678',
   master_log_file='LAPTOP-A0EHGSSN-bin.000001',
   master_log_pos=1078441;

   start slave;
   ```

4. 查看从配置状态。

   ```mysql
   show slave status;
   ```

## sqlite 迁移

1. 导出 sqlite 数据。

   ```shell
   sqlite3 RobotDatabase.db .dump > db.sql
   ```

2. 适配：

   - 表名不需要 `"` 包裹。

## qt mysql 驱动

#### apt 安装

```shell
sudo apt install libqt5sql5-mysql
```

#### 源码安装

`/usr/lib/x86_64-linux-gnu/qt5/plugins/sqldrivers` 存放驱动动态库。
