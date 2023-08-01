# MongoDB Replication

```
경우에 따라 클라이언트가 읽기 작업을 다른 서버로 보낼 수 있으므로 복제를 통해 읽기 용량을 늘릴 수 있습니다.  
서로 다른 데이터 센터에서 데이터 복사본을 유지 관리하면 분산 응용 프로그램의 데이터 지역성과 가용성을 높일 수 있습니다.  
재해 복구, 보고 또는 백업과 같은 전용 목적을 위해 추가 복사본을 유지할 수도 있습니다.  
```

- Replica Set에서는 쓰기 작업을 처리할 수 있는 (`{ w: "majority" }`) Primary를 하나만 가질 수 있다. 
  - [Replica Set Primary](https://www.mongodb.com/docs/v4.4/core/replica-set-primary/)
  - [Replica Set Secondary Members](https://www.mongodb.com/docs/v4.4/core/replica-set-secondary/)
  - [Replica Set Arbiter](https://www.mongodb.com/docs/v4.4/core/replica-set-arbiter/)
- Primary는 Data Set에 대한 모든 변경 사항을 작업 로그, 즉 Operation Log (oplog)에 기록한다.
  - Secondary는 Primary의 oplog를 복제하고 Data Set에 비동기적으로 적용한다.
  - Secondary가 oplog를 다 읽어 가지 못한 상태에서 Primary에서 장애가 난다면, 읽지 못한 데이터는 유실된다.
  - 또한 Primary의 oplog가 가득 찼지만 슬레이브가 복제를 다 하지 못한 경우, 로그 기반 복제 대신 Primary의 데이터를 읽어 와서 Secondary에 반영한다. 이때에도 Primary에서 장애가 발생하면 데이터가 유실된다.
    - [Replica Set Oplog](https://www.mongodb.com/docs/v4.4/core/replica-set-oplog/)
    - [Replica Set Data Synchronization](https://www.mongodb.com/docs/v4.4/core/replica-set-sync/)
- Replica Set에서는 Primary를 사용할 수 없게되면 Secondary 중 Primary 서버가 될 서버를 선택한다. [Replica Set Elections](https://www.mongodb.com/docs/v4.4/core/replica-set-elections/#std-label-replica-set-elections)
  - **Primary 복제본 구성 설정을 가정할 때 클러스터가 새 Primary를 선택하기 전 평균 시간은 일반적으로 12초를 초과하지 않아야 합니다.**
  - `settings.electionTimeoutMillis` = 10초
  - 
- 클라이언트는 Secondary에 데이터를 쓸 수 없지만 읽을 순 있다.
- 클라이언트의 읽기 작업은 기본적으로 replica set의 Primary로 전달하지만 "읽기 기본 설정"을 지정하여 읽기 작업을 Secondary로 보낼 수 있다. [Read Preference](https://www.mongodb.com/docs/v4.4/core/read-preference/)
   1. **Tag Set Lists**
   2. **maxStalenessSeconds**
   3. **Hedged Read Option**
- Replica Set 간의 동기화 이슈를 제어하려면 ReadConcern 옵션을 확인해야 한다.
  - [Write Concern](https://www.mongodb.com/docs/v4.4/reference/write-concern/)
  - [Read Concern](https://www.mongodb.com/docs/v4.4/reference/read-concern/)
  


***

**?**
1. [ ] heartbeat의 주기는 어느정도인가?
2. [x] Secondary가 oplog를 전달받고 동기화 중이라면 Primary의 쓰기 작업은 블록되는가?
   1. 비동기 복제로 이루어지기 때문에 Primary의 성능 저하가 거의 없으며, Secondary를 추가해도 서비스 성능이 떨어지지 않는다.
   2. 하지만 데이터가 대부분 불일치하기 때문에 장애가 생기면 데이터 유실이 발생한다.
3. [ ] 투표는 어떤 기준으로 되는가?
4. [ ] 읽기 수준을 Secondary에게 책임을 준다고할 때 oplog를 통해 동기화가 되고 있을 때 읽기 요청이 들어오면 블록되는가?
5. [ ] 동기화 주기는 어떻게 되는가?
   1. Secondary 노드에서는 기본적으로 읽기 쿼리가 비활성화 되어 있으므로 `db.getMongo().setSecondaryOk()`로 읽기를 활성화시켜준다.


***

```
The server generated these startup warnings when booting:
        2023-07-31T04:33:47.954+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
        2023-07-31T04:33:48.067+00:00:
        2023-07-31T04:33:48.067+00:00: ** WARNING: This replica set has a Primary-Secondary-Arbiter architecture, but readConcern:majority is enabled
        2023-07-31T04:33:48.067+00:00: **          for this node. This is not a recommended configuration. Please see
        2023-07-31T04:33:48.067+00:00: **          https://dochub.mongodb.org/core/psa-disable-rc-majority
        2023-07-31T04:33:48.067+00:00:
```

- [[MongoDB] Multi-Document Transactions](https://medium.com/@marchpig/mongodb-multi-document-transactions-d51e047f811d)

***

# Write Concern

Primary에서 쓰기 작업이 끝났을 때 클라이언트에게 정상 응답을 반환한다.  
그 후 Secondary와의 동기화 되는 시점에(데이터의 일관성이 보장되지 않는 시점) Primary에 장애가 발생하여 Secondary가 승격되게 되면 클라이언트가 알고 있는 데이터와 DB의 데이터가 unmatch된다.  
  
응답 시점을 Primary와 Secondary가 동기화된 이후로 설정이 가능하다. 이것이 Write Concern의 핵심이다.  

> Primary가 데이터 쓰기를 처리한 이후 바로 Client에게 response를 보내는 것이 아니라 Secondary 쪽으로 데이터를 동기화 작업을 완료한 이후에 Client에게 response를 보내게 된다.

# Slow Oplog Application

Replica Set의 Secondary는 적용하는 데 작업 임계값보다 오래 걸리는 oplog 항목을 기록한다.
- 임계값 설정 방법
  1. [--slowms](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-option-mongod.--slowms)
  2. [https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs](https://www.mongodb.com/docs/manual/reference/configuration-options/#mongodb-setting-operationProfiling.slowOpThresholdMs)

```
2018-11-16T12:31:35.886-05:00 I REPL   [repl writer worker 13] applied op: command { ... }, took 112ms
```

# mongo를 사용한 이유

- originCache에 대한 내용을 중복 제거나 패밀리 그룹핑을 통해 재캐시되는 경우가 있다. 이때 2차,3차 결과물에 대한 내용을 삭제하게 되면 기존에 존재하던 캐시에 대해 수정이 필요하여 mongo를 사용
- 사용자 설정 (하이라이트 단어, ...)
  - 검색 설정도 mongo에 넣을 계획


# source

```
docker run --name mongodb -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=root -e MONGO_INITDB_ROOT_PASSWORD=root mongo:4.4
```

```yml
version: '3.7'

services:
  mongo1:
    image: mongo:4.4
    hostname: mongo1
    container_name: mongo1
    networks:
      - mongo-bridge
    ports:
      - 30001:27017
    volumes:
      - /Users/jeonghyeonjun/data/mongo/db-01:/data/db
    restart: always
    command: ["mongod", "--replSet", "mongo-repl"]
  mongo2:
    image: mongo:4.4
    hostname: mongo2
    container_name: mongo2
    networks:
      - mongo-bridge
    ports:
      - 30002:27017
    volumes:
      - /Users/jeonghyeonjun/data/mongo/db-02:/data/db
    restart: always
    command: ["mongod", "--replSet", "mongo-repl"]
  mongo3:
    image: mongo:4.4
    hostname: mongo3
    container_name: mongo3
    networks:
      - mongo-bridge
    ports:
      - 30003:27017
    volumes:
      - /Users/jeonghyeonjun/data/mongo/db-03:/data/db
    restart: always
    command: ["mongod", "--replSet", "mongo-repl"]
  # configure the mongo replica set
  mongosetup:
    build: ./mongosetup
    networks:
      - mongo-bridge
    volumes:
      - ./scripts:/scripts
    entrypoint: [ "/scripts/setup_init.sh" ]

networks:
  mongo-bridge:
    name: mongo-bridge
```

```sh
#!/usr/bin/env bash

echo "Waiting for MongoDB startup.."
until [ "$(mongo --host mongo1:27017 admin --eval "printjson(db.serverStatus())" | grep uptime | head -1)" ]; do
  printf '.'
  sleep 1
done

echo $(mongo --host mongo1:27017 admin --eval "printjson(db.serverStatus())" | grep uptime | head -1)
echo "MongoDB Started.."


echo SETUP.sh time now: `date +"%T" `
mongo --host mongo1:27017 <<EOF
   var cfg = {
        "_id": "mongo-repl",
        "members": [
            {
                "_id": 0,
                "host": "mongo1:27017",
                "priority": 1
            },
            {
                "_id": 1,
                "host": "mongo2:27017",
                "priority": 1
            },
            {
                "_id": 2,
                "host": "mongo3:27017",
                "priority": 0,
                "arbiterOnly": true
            }
        ]
    };
    rs.initiate(cfg, { force: true });
    rs.reconfig(cfg, { force: true });
    use admin;
    db.createUser(
    {
        user: "root",
        pwd: "$rootPassword",
        roles: [ { role: "root", db: "admin" } ]
    }
    );
    db.createUser(
    {
        user: "datadog",
        pwd: "datadog",
        roles: [ { role: "clusterMonitor", db: "admin" } ]
    }
    );
    use medusa;
    db.createUser(
    {
        user: "$userId",
        pwd: "$userPassword",
        roles: ["dbAdmin", "readWrite"]
    }
    );    
EOF
```