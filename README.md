### Config master

```text
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog_do_db = db
```

```sql
CREATE DATABASE IF NOT EXISTS db;

CREATE TABLE test
(
    id int auto_increment,
    constraint test_pk
        primary key (id)
);

GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY 'password';

FLUSH PRIVILEGES;
```

### Transfer data

```sql
USE db; 

FLUSH TABLES WITH READ LOCK;

SHOW MASTER STATUS;
```

| File | Position | Binlog\_Do\_DB | Binlog\_Ignore\_DB | Executed\_Gtid\_Set |
| :--- | :--- | :--- | :--- | :--- |
| mysql-bin.000003 | 4498 | db |  |  |

```shell
mysqldump -u root -p db > db.sql
```

```sql
UNLOCK TABLES;
```

### Config slave

```text
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
log-bin = mysql-bin
binlog_do_db = db
```

```sql
CREATE DATABASE IF NOT EXISTS db;
```

```shell
mysql -u root -p db < db.sql
```

```sql
CHANGE 
    MASTER TO MASTER_HOST='mysql-m', 
    MASTER_USER='slave', 
    MASTER_PASSWORD='password',
    MASTER_LOG_FILE = 'mysql-bin.000003', 
    MASTER_LOG_POS = 4498;
    
START SLAVE;

SHOW SLAVE STATUS;
```

### Modifications

```sql
ALTER TABLE test ADD test_1 varchar(255) null;
```

Run on slave
```sql
ALTER TABLE test DROP COLUMN test_1;
```