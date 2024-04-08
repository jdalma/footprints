---
title: AWS DMS를 이용한 MySQL 마이그레이션 (작성중)
date: "2024-03-28"
tags:
   - mysql
   - aws dms
   - migration
---

단일 데이터베이스 환경에서 AWS Aurora 클러스터로 마이그레이션하면서 고민했었던 부분들 그리고 MySQL 5.7에서 8.0으로 버전업하면서 신경써야 했던 부분들을 정리해보려한다.  

# 직접 클러스터 구성하기

읽기 전용 DB(Secondary)들을 Primary DB의 Replication DB로 구성하여 **[ProxySQL](https://proxysql.com/)을 사용하여 RW를 분리**하고 **[Orchestrator](https://github.com/openark/orchestrator)를 통하여 페일백/페일오버**를 직접 구성하여 클러스터를 직접 구축하는 것을 고민했다.  

![](proxySQLstructure.png)

하지만 큰 단점이 있었다.  

1. Proxy SQL에 문제가 생긴다면 모든 DB를 사용할 수 없게 되는 **단일 장애 지점** 이 생긴다.
2. 애플리케이션 외부에서 사용될 DB가 결정되기 때문에 작업 단위(트랜잭션)에 대한 문제가 생긴다.

위의 두 문제가 있어 AWS Aurora 클러스터를 분석하기로 하였다.  
혹시 이 구성에 대해 더 자세히 알고 싶다면 [`인프런` 따라하며 배우는 MySQL on Docker](https://www.inflearn.com/course/mysql-docker/dashboard) 강의를 참고하길 바란다.

# AWS Aurora 클러스터 분석

Aurora 클러스터를 사용하기 위해서는 확인해야 할 부분들이 있었다.

<h3>Aurora 클러스터의 페일오버와 페일백은 어떻게 진행되는가?</h3>

기본 인스턴스에 문제가 발생하는 경우 읽기 인스턴스 중 하나가 기본 인스턴스 역할을 인수한다.  
Aurora는 데이터베이스 문제를 감지하고 필요한 경우 장애 조치 메커니즘을 자동으로 활성화하여 장애 조치 중에 쓰기 작업을 처리하지 못하는 시간이 최소화된다.  
Aurora는 다음 두 가지 방법 중 하나를 사용하여 자동으로 새 기본 인스턴스로 장애 조치한다.  

1. 우선순위가 높은 기존 Aurora 복제본을 새 기본 인스턴스로 승격시킴
2. 새로운 기본 인스턴스 만들기

DB 클러스터에 Aurora 복제본이 하나 이상인 경우에는 장애가 발생하더라도 Aurora 복제본이 기본 인스턴스로 승격되며 **이 실패 이벤트로 인해 예외적으로 실패하는 읽기 및 쓰기 작업 동안 짧은 중단이 발생한다.**  
하지만, **일반적인 서비스 복구 시간은 60초 미만이지만 대부분 30초 미만에 복원된다.** (read only connection 예외 발생)  
최소 하나 이상의 Aurora 복제본을 둘 이상의 다른 가용 영역에 생성하는 것이 바람직하다.  
  
DB 클러스터에 Aurora 복제본이 포함되어 있지 않으면 기본 인스턴스가 동일한 AZ에 다시 생성되기 때문에 보통 10분 미만의 시간이 걸리며 그 동안 예외적으로 실패하는 읽기 및 쓰기 작업 동안 중단이 발생한다.  
Aurora 복제본을 기본 인스턴스로 승격시키는 것이 기본 인스턴스를 새로 생성하는 것보다 훨씬 빠르다.  

[출처 : [Amazon Aurora의 고가용성](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html)]  
  
운영 중에 DB 스케일 업에 대한 트러블 슈팅 글도 읽어보면 도움이 될 것이다. [29CM 의 이굿위크 장애대응 기록](https://medium.com/29cm/29cm-%EC%9D%98-%EC%9D%B4%EA%B5%BF%EC%9C%84%ED%81%AC-%EC%9E%A5%EC%95%A0%EB%8C%80%EC%9D%91-%EA%B8%B0%EB%A1%9D-177b6b2f07a0)

<h3>Secondary의 동기화 지연율은 어느정도인가?</h3>

Aurora는 단일 AWS 리전에서 다중 가용 영역에 걸쳐 DB 클러스터에 데이터 복사본을 저장하며, DB 클러스터의 인스턴스가 여러 가용성 영역에 걸쳐 있는지 여부에 관계없이 이러한 복사본을 저장한다.  

![](./clusterTopology.png)

기본 DB 인스턴스에 데이터가 기록되면 Aurora에서 가용 영역의 데이터를 클러스터 볼륨과 연결된 6개의 스토리지 노드에 동기적으로 복제합니다. 이 방법은 데이터 중복을 제공하고, I/O 중지를 없애고, 시스템 백업 중에 지연 시간 스파이크를 최소화합니다. DB 인스턴스를 고가용성으로 실행하면 계획된 시스템 유지 관리 중 가용성을 향상시킬 수 있으며, 데이터베이스에서 오류 및 가용 영역 중단이 일어나는 것을 방지할 수 있습니다. 가용 영역에 대한 자세한 정보는 리전 및 가용 영역 단원을 참조하십시오.

[출처 : [Amazon Aurora MySQL을 사용한 복제](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.html), [AWS Aurora 아키텍처](https://medium.com/garimoo/aws-aurora-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-6ff87b0d48c5)]

<h3>Aurora 클러스터 스토리지는 어느것을 사용하나?</h3>

<h3>읽기 전용 인스턴스의 스펙을 지정할 수 있는가?</h3>



***

1. Aurora 클러스터 페일오버, 페일백
   1. AWS 인스턴스 장애 발생 시 진행되는 페일오버, 페일백이 진행되면 대략 30초 정도 read only connection 예외가 발생, 30초 정도의 시간은 감내 가능하다고 판단 [참고](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html#Aurora.Managing.FaultTolerance)
2. Aurora 클러스터 가용성 [참고](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.html)
   1. Aurora 복제본은 클러스터 볼륨의 읽기 작업 전용이므로 읽기 확장에 적합합니다. 쓰기 작업은 기본 인스턴스에서 관리합니다.
   2. 클러스터 볼륨은 Aurora MySQL DB 클러스터의 모든 인스턴스 간에 공유되므로 각 Aurora 복제본에 대한 데이터 사본을 복제하는 데 추가 작업이 필요하지 않습니다.
   3. 반면, MySQL 읽기 복제본은 소스 DB 인스턴스에서 로컬 데이터 저장소에 대한 모든 쓰기 작업을 단일 스레드에서 재생해야 합니다. 이러한 제한은 MySQL 읽기 복제본이 대량의 읽기 트래픽을 지원하는 기능에 영향을 미칠 수 있습니다.
   4. MySQL read replicas must replay, on a single thread, all write operations from the source DB instance to their local data store.
3. Aurora 클러스터 스토리지 조사
   1. [Aurora 클러스터 스토리지 구성](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html)
   2. [Aurora 클러스터 스토리지 요금](https://aws.amazon.com/ko/rds/aurora/pricing/)

## Aurora 인스턴스 스펙 선정을 위한 부하 테스트

1. mysqlslap
2. sysbench

# 스프링에서 읽기/쓰기 작업 분리

1. 스프링에서 RW 커넥션을 분리

# AWS DMS를 이용한 데이터 이관

4. AWS DMS를 사용하면서 주의해야 할점
   1. 엔드포인트, 복제 인스턴스, 마이그레이션 태스크
   2. 적재 모드 선택
   3. 지속적인 복제를 사용하려면 binary logging이 활성화되어 있어야함
   4. 마이그레이션 대상 DB에 파티션된 테이블이 있다면 해당 테이블은 대상 DB에 미리 생성 해놓아야하며, 대상 테이블 준비 모드는 "아무 작업 안함" 또는 "자르기"를 선택해야 한다.
   5. 동기화가 끝나면 대상 테이블의 AUTO_INCREMENT 값을 수동으로 동기화 시켜줘야 한다.
   6. 테이블에 LOB 형식의 컬럼이 Not null 제약 조건이 있다면 해당 제약 조건을 제거하고 마이그레이션이 끝난 후 제약 조건을 추가해야 한다.
5. AWS DMS
   1. 동시에 8개의 테이블을 10000 레코드를 복제함. 변경 감지 트랜잭션 활성화 시간은 기본값인 10분이지만 변경도 가능하긴 함

# MySQL 버전업을 진행하면서 고려해야할 점

1. [예약어](https://dev.mysql.com/doc/refman/8.0/en/keywords.html#keywords-8-0-detailed-C)
1. SSL에 대한 암호화 알고리즘 호환 확인
   1. 기존에는 useSSL=false로 사용하고 있었음
2. utf8mb3 -> utf8mb4
   1. utf8mb3을 utf8mb4로 변경하면서 컬럼 사이즈(VARCHAR는 최대 65535자를 저장)와 인덱스 사이즈(The DYNAMIC row format supports index key prefixes up to 3072 bytes.) 확인
      1. [DYNAMIC Row Format](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html#innodb-row-format-dynamic)
   2. MySQL 8.0.17부터 숫자 데이터 유형에 대한 ZEROFILL 속성과 정수 데이터 유형에 대한 표시 너비 속성은 더 이상 사용되지 않습니다. 향후 MySQL 버전에서는 정수 데이터 유형에 대한 ZEROFILL 및 표시 너비 지원이 제거될 것으로 예상해야 합니다. [참고](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-attributes.html)
3. 기존에 사용중이던 DB의 환경변수 (max-allow-packet, max-connections)
