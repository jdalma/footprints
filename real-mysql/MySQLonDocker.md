# Master와 Slave 컨테이너 실행

MySQL Container의 mysql error log를 보려면?  
MySQL Parameter를 변경하려면?  
  
Container에 접속하지 않고 Docker Host에서 로그 확인 및 parameter를 수정할 수 있도록 `log directory`와 `conf directory`를 생성햐고 호스트 볼륨을 공유할 수 있도록 추가설정하자.  

# Bridge Network 구성

Container는 언제든지 재시작될 수 있고 Container가 재시작되면 해당 Container IP가 변경될 수 있다.  
**MySQL의 Replication 설정이나 HA 설정에 IP를 사용하게 되면 Container가 재시작될 경우 변경된 IP 때문에 Replication이 깨질 수 있다.**  
이러한 문제를 방지하기 위해 Bridge Network를 구성하고 net alias를 사용하여 IP 변경에도 문제가 발생하지 않도록 할 수 있다.  
  
```
브릿지 생성

docker network ls
docker network create --driver bridge mybridge
```

## Master 컨테이너 생성

```
1. log폴더와 conf폴더를 만든다.
2. 권한 777 설정
3. Master cnf 파일 추가

[mysqld]
log_bin                     = mysql-bin
binlog_format               = ROW
gtid_mode                   = ON
enforce-gtid-consistency    = true
server-id                   = 100
log_slave_updates
datadir                     = /var/lib/mysql
socket                      = /var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links              = 0

log-error                   = /var/log/mysql/mysqld.log
pid-file                    = /var/run/mysqld/mysqld.pid

report_host                 = db001

[mysqld_safe]
pid-file                    = /var/run/mysqld/mysqld.pid
socket                      = /var/lib/mysql/mysql.sock
nice                        = 0

4. Master 컨테이너 생성

docker run -it --name db001 -p 3306:3306 \
--net mybridge --net-alias=db001 \
-v /Users/jeonghyeonjun/db/db001/data:/var/lib/mysql \
-v /Users/jeonghyeonjun/db/db001/log:/var/log/mysql \
-v /Users/jeonghyeonjun/db/db001/conf/my.cnf:/etc/my.cnf \
-e MYSQL_ROOT_PASSWORD="root" -d mysql
```

## Slave 컨테이너 생성

```
1. 마운트 폴더 생성
mkdir -p /Users/jeonghyeonjun/db/db002/data /Users/jeonghyeonjun/db/db003/data
chmod 777 /Users/jeonghyeonjun/db/db002 /Users/jeonghyeonjun/db/db002/data
chmod 777 /Users/jeonghyeonjun/db/db003 /Users/jeonghyeonjun/db/db003/data

mkdir -p /Users/jeonghyeonjun/db/db002/log /Users/jeonghyeonjun/db/db003/log
mkdir -p /Users/jeonghyeonjun/db/db002/conf /Users/jeonghyeonjun/db/db003/conf
chmod 777 /Users/jeonghyeonjun/db/db002/log /Users/jeonghyeonjun/db/db002/conf
chmod 777 /Users/jeonghyeonjun/db/db003/log /Users/jeonghyeonjun/db/db003/conf

2. Slave cnf 파일 추가

[mysqld]
log_bin                     = mysql-bin
binlog_format               = ROW
gtid_mode                   = ON
enforce-gtid-consistency    = true
server-id                   = 200
log_slave_updates
datadir                     = /var/lib/mysql
socket                      = /var/lib/mysql/mysql.sock
read_only

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links              = 0

log-error                   = /var/log/mysql/mysqld.log
pid-file                    = /var/run/mysqld/mysqld.pid

report_host                 = db002

[mysqld_safe]
pid-file                    = /var/run/mysqld/mysqld.pid
socket                      = /var/lib/mysql/mysql.sock
nice                        = 0

docker run -it --name db002 -p 3307:3306 \
--net mybridge --net-alias=db002 \
-v /Users/jeonghyeonjun/db/db002/data:/var/lib/mysql \
-v /Users/jeonghyeonjun/db/db002/log:/var/log/mysql \
-v /Users/jeonghyeonjun/db/db002/conf/my.cnf:/etc/my.cnf \
-e MYSQL_ROOT_PASSWORD="root" -d mysql

3. 두 번째 Slave도 (db004) 설정
4. docker ps --format "table {{.ID}} \t {{.Names}} \t {{.Status}}"
```

# 복제 구성하기

```
1. Master에서 복제에 사용할 user를 생성    `CREATE USER 'repl'@'%' IDENTIFIED BY 'repl';`
2. user 권한 부여   `GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';`
3. Master 컨테이너의 IP 확인 `docker inspect {container ID}`
    - "IPAddress": "172.17.0.2"
4. Slave 컨테이너 접속
5. `reset master;`
6. Master 변경하기

CHANGE MASTER TO MASTER_HOST='172.17.0.2', \
MASTER_USER='repl', MASTER_PASSWORD='repl', \
MASTER_AUTO_POSITION=1;

// 네트워크를 적용하였으니 컨테이너 이름으로도 가능
CHANGE MASTER TO MASTER_HOST='db001', \
MASTER_USER='repl', MASTER_PASSWORD='repl', \
MASTER_AUTO_POSITION=1;

7. `start slave;`
8. `show slave status\G`

*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: db001
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 730
               Relay_Log_File: 909502608ac7-relay-bin.000002
                Relay_Log_Pos: 420
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
        ....
                Last_IO_Errno: 2061
                Last_IO_Error: error connecting to master 'repl@172.17.0.2:3306' - retry-time: 60 retries: 1 message: Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.
        ....

9. Last_IO_Error가 존재한다면 Slave를 멈추고, CHANGE MASTER TO GET_MASTER_PUBLIC_KEY=1; 변경 후 다시 재시작하면 해결된다.
```

# Orchestrator Container

```
1. Orchestrator Container 실행

docker run -it --name orchestrator -h orchestrator \
--net mybridge --net-alias=orchestrator \
-p 3000:3000 -d openarkcode/orchestrator:latest

2. Orchestrator를 위한 Master DB에서 mysql user 생성

create user orc_client_user@`172.%` identified by 'orc_client_password';
GRANT SUPER, PROCESS, REPLICATION SLAVE, RELOAD ON *.* TO orc_client_user@`172.%`;

3. DB container들의 IP 대역 확인

docker inspect --format '{{.NetworkSettings.Networks.mybridge.IPAddress}}' db001

4. 웹 브라우저에서 Orchestrator 접속

http://{docker_host_ip}:3000/web/clusters
```

## Auto Fail Over 적용하기

```
Orchestrator 컨테이너에 접속

vi /etc/orchestrator.conf.json

"RecoverMasterClusterFilters": [  // Slave들을 자동으로 승격할 수 있게 설정
    "*"
],
"PromotionIgnoreHostnameFilters": ["db003"], // Master로 승격하지 못하게 설정

docker restart orchestrator
```

- http://localhost:3000/web/audit-recovery > Audit > Recovery > 장애 이력을 모두 ack처리 하여야함

# Proxy Layer 구성

MySQL FailOver 시 Application에서는 어떻게 접근해야할까?  
쓰기 작업은 DB001에 대한 DataSource를 사용하고, 읽기 작업은 DB002, DB003에 대한 DataSource를 사용할텐데,  
DB001이 죽어버리고, DB002가 Master로 승격된다면 Application에서는 쓰기 작업을 어떻게 처리할 수 있을까?  
  
이 문제를 해결하기 위해 Proxy Layer를 구축해야한다.  

```
1. ProxySQL Container 실행

docker run -it --name proxysql -h proxysql \
--net mybridge --net-alias=proxysql \
-p 16032:6032 -p 16033:6033 \
-v /Users/jeonghyeonjun/db/proxysql/data:/var/lib/proxysql \
-v /Users/jeonghyeonjun/db/proxysql/conf/proxysql.cnf:/etc/proxysql.cnf \
-d proxysql/proxysql

2. ProxySQL 접속

mysql -h127.0.0.1 -P16032 -uradmin -pradmin --prompt "ProxySQL Admin>" 안되면
mysql -uadmin -padmin -h 127.0.0.1 -P6032 --prompt='Admin> '

3. MasterDB 테스트 데이터베이스 및 User 생성
create database testdb default character set utf8;

create user appuser@'%' identified by 'apppass';
grant select, insert, update, delete on testdb.* to appuser@'%';

create user 'monitor'@'%' identified by 'monitor';
grant REPLICATION CLIENT on *.* to 'monitor'@'%';

flush privileges;

4. ProxySQL 환경 구성

4.1. hostgroup에 db 서버 정보 입력

// hostgroup_id를 10번,20번 등록
// 10번 = Write Transaction 처리
// 20번 = Read Transaction 처리
insert into mysql_servers(hostgroup_id, hostname, port) values (10, 'db001', 3306);
insert into mysql_servers(hostgroup_id, hostname, port) values (20, 'db001', 3306);
insert into mysql_servers(hostgroup_id, hostname, port) values (20, 'db002', 3306);
insert into mysql_servers(hostgroup_id, hostname, port) values (20, 'db003', 3306);

// 10번 = Write용 Host Group
// 20번 = Read용 Host Group
// 'read_only' = 등록된 서버가 Write, Read를 구분할 때 read_only 파라미터로 구분한다는 의미
insert into mysql_replication_hostgroups values (10, 20, 'read_only', '');

LOAD MYSQL SERVERS TO RUNTIME;
SAVE MYSQL SERVERS TO DISK;

4.2. 어플리케이션 User 정보 입력

// ProxySQL 서버가 트랜잭션 유형에 따라서 호스트 그룹별로 트랜잭션을 분기시켜주는 구조
insert into mysql_users (username,password,default_hostgroup,transaction_persistent) values ('appuser', 'apppass', 10, 0);

LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;

4.3. Query rule 정보 입력

// 모든 쿼리는 기본적으로 default host group인 10번으로 보낸다.
// SELECT FOR UPDATE는 10번으로 가도록, SELECT는 20번으로 가도록한다.
// Query RULE의 순서는 중요하다.
insert into mysql_query_rules (rule_id, active, match_pattern, destination_hostgroup) values (1, 1, '^SELECT.*FOR UPDATES', 10);
insert into mysql_query_rules (rule_id, active, match_pattern, destination_hostgroup) values (2, 1, '^SELECT', 20);

LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;
```

```
// db001 - 172.18.0.2
// db002 - 172.18.0.3
// db003 - 172.18.0.4
// orchestrator - 172.18.0.5
// proxysql - 172.18.0.6
```

```
set global read_only=1;
CHANGE MASTER TO MASTER_HOST='db002', \
MASTER_USER='repl', MASTER_PASSWORD='repl', \
MASTER_AUTO_POSITION=1;
```