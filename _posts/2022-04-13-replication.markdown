---
title: "Database Replication (1) - Docker MariaDB Replication"
excerpt:

header:
  overlay_image: https://images.unsplash.com/photo-1501785888041-af3ef285b470?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=1350&q=80
  overlay_filter: 0.5

tags:
  - Docker
  - Replication
  - MariaDB

toc: true
toc_label: "table of content"
toc_icon: "bars"
toc_sticky: true
---

# Replication

백업 및 성능 향상을 위해서 데이터베이스를 여러 대의 서버에 복제하는 행위를 Replication 이라고 함.

# Read Replica

RDS DB 인스턴스의 읽기전용 인스턴스.

서비스에 읽기 위주의 작업이 많은 경우 Read Replica를 여러 개 만들어서 부하를 분산시킬 수 있음. 즉, 쓰기 작업은 마스터 DB 인스턴스에하고 읽기 작업은 Read Replica 에 할당하면 마스터 DB 인스턴스의 부하를 줄일 수 있다.

단, 마스터 DB 인스턴스에 update/write 작업이 일어난 경우 자동으로 read replica DB 인스턴스로 데이터가 복제되나 바로 되는 것은 아니고 약간의 시간차가 있긴 함.

생성하는 방법 블로그 링크: [[AWS] RDS DB 인스턴스 Read Replica 생성하기](https://minholee93.tistory.com/entry/AWS-RDS-DB-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-Read-Replica-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0)

<br/>

# Master / Slave

원본 데이터가 위치하는 서버를 마스터, 복제한 서버를 슬레이브라고 한다.

<br/>

# 도커로 MariaDB Replicate

## MariaDB Image Pull

```sh
$ docker pull mariadb
```

간단하게 latest 의 mariadb 이미지를 내려받는다.

<br/>

## Master / Slave 컨테이너 생성

mariadb 이미지를 가지고 2개의 컨테이너를 생성하여 master/slave 관계를 가지도록 하려면 설정이 필요함. `~/Docker/mysql` 디렉토리를 생성하고 아래에 `master`, `slave` 디렉토리를 생성한다. 그리고 각 디렉토리에 `config_file.cnf` 를 아래와 같이 입력 후 생성.

- `~/Docker/mysql/master/config_file.cnf`

```sh
[mysqld]
log-bin=mysql-bin
server-id=1
```

- `~/Docker/mysql/slave/config_file.cnf`

```sh
[mysqld]
server-id=2
```

이제 컨테이너를 실행

```sh
$ docker run --name maria-master -v ~/Docker/mysql/master:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=password -p 3306:3306 -d mariadb
$ docker run --name maria-slave -v ~/Docker/mysql/slave:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=password --link mysql-master -d mariadb
```

<br/>

## Master 서버에 User 생성

slave 서버에서 master 서버에 접근이 가능한 replication slave 권한을 가지고 있는 사용자를 생성한다.

```bash
$ docker exec -it mysql-master /bin/bash
root:/# mysql -u root -p
mysql> CREATE USER 'repluser'@'%" IDENTIFIED BY 'password';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repluser'@'%';
```

유저를 생성했으면 replication 테스트를 위한 데이터베이스, 테이블을 하나씩 생성.

```bash
mysql> CREATE DATABASE testdb;
mysql> USE testdb;
mysql> CREATE TABLE test ( no INT(8), PRIMARY KEY (no) );
```

<br/>

## DB dump

master 서버의 db 상태를 slave 에 그대로 반영하기 위해서 dump 하자.

```bash
$ docker exec -it mysql-master /bin/bash
root:/# mysqldump -u root -p testdb > dump.sql
```

dump 한 파일을 로컬로 복사하고 slave 서버로 보내주자.

```bash
$ docker cp mysql-master:dump.sql .
$ docker cp dump.sql mysql-slave:.
```

`dump.sql` 를 이용하여 DB를 복사한다.

```bash
$ docker exec -it mysql-slave /bin/bash
root:/# mysql -u root -p
mysql> CREATE DATABASE testdb;
mysql> exit
root:/# mysql -u root -p testdb < dump.sql
```

다시 slave 서버에 접속하여 테이블을 조회했을 때 `test` 테이블이 생성되어 있다면 잘 복사된 것임.

<br/>

## slave 서버와 master 서버 연결

연결하기 위해서 우선 master 서버에 접속하여 바이너리 로그 파일의 현재 상태를 확인해야 한다.

```bash
$ docker exec -it mysql-master /bin/bash
root:/# mysql -u root -p
mysql> SHOW MASTER STATUS\G
```

```
MariaDB [(none)]> show master status\g
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000002 |     1002 |              |                  |
+------------------+----------+--------------+------------------+
```

위와 같은 결과를 얻을 수 있다. 메모해두고 다시 slave 서버로 접속하여 마스터 유저를 변경하는 쿼리를 작성하자.

```bash
$ docker exec -it mysql-slave /bin/bash
root:/# mysql -u root -p
mysql> CHANGE MASTER TO MASTER_HOST='mysql-matser', MASTER_USER='repluser', MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000002', MASTER_LOG_POS=1002;
mysql> START SLAVE;
```

- MASTER_HOST: master 서버의 호스트명. docker 실행 할 때 `--link` 옵션을 이용하여 mysql-master 컨테이너와 호스트를 연결하였기 때문에 `mysql-master` 를 입력해줄 수 있음
- MASTER_USER: master 서버에서 `REPLICATION SLAVE` 권한을 가지고 있는 사용자 계정의 이름
- MASTER_PASSWORD: 그 사용자를 만들 때 지정한 password
- MASTER_LOG_FILE: master 서버의 바이너리 로그 파일명
- MASTER_LOG_POS: master 서버의 현재 로그의 위치

마지막으로 아래 명령을 실행하여 결과를 확인해본다.

```
mysql> SHOW SLAVE STATUS\G
```

```bash
MariaDB [(none)]> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: mysql-master
                   Master_User: repluser
                   Master_Port: 3306
                 Connect_Retry: 60
               Master_Log_File: mysql-bin.000002
           Read_Master_Log_Pos: 1002
                Relay_Log_File: mysqld-relay-bin.000002
                 Relay_Log_Pos: 555
         Relay_Master_Log_File: mysql-bin.000002
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table:
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 1002
               Relay_Log_Space: 865
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: 0
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 1
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: optimistic
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Slave_DDL_Groups: 0
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 0
1 row in set (0.000 sec)
```

여기서 `Last_Error` 와 `Last_IO_Error` 가 비어있다면 성공한 것임.

<br/>

## Replication 테스트

Replication 은 끝났음. 이제 Master 서버에 데이터를 새로 입력하면 slave 에도 똑같이 반영되어야 함.

```bash
$ docker exec -it mysql-master /bin/bash
root:/# mysql -u root -p
mysql> USE testdb;
mysql> INSERT INTO test VALUES (1);
```

```bash
$ docker exec -it mysql-slave /bin/bash
root:/# mysql -u root -p
mysql> USE testdb;
mysql> SELECT * FROM test;
```

```
MariaDB [cmx_cloud_v2_region_db]> select * from test;
+----+
| no |
+----+
|  1 |
+----+
```

잘 되었네요.

<br/>

# 갈무리

master/slave DB 생성했으니 프로젝트에 적용해보자. Spring Boot 에서 여러 데이터베이스를 적용하는 것과 Transaction 성격에 따라 (readOnly 인지? write, update 등 수정 작업인지?) 다른 데이터베이스에서 쿼리를 적용하는 걸 할거임.
