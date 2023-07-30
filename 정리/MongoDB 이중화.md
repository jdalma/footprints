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