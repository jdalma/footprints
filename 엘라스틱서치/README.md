
엘라스틱 서치는 아파치의 루씬 라이브러리를 기반으로 만든 **분산 검색 엔진** 이다  
특징으로는  
1. 분산 처리를 고려하여 설계됐기 때문에 데이터를 여러 노드에 분산 저장하며 검색이나 집계 작업 등을 수행할 때도 분산 처리를 지원한다.  
2. 클러스터를 구성하고 있는 일부 노드에 장애가 발생해도 복제본 데이터를 이용해 중단 없이 서비스를 지속할 수 있다.  
3. 요청 수나 데이터의 양이 증가함을 대비하여 수평적 확장을 지원한다. 새로운 노드에 ES를 설치하여 클러스터에 참여시키기만 하면 된다. 새 노드에 데이터를 복제하거나 옮기는 작업도 다종 수행한다.
4. **준실시간 검색** 을 지원한다. 데이터를 색인하자마자 조회하는 것은 가능하지만, 색인 직후의 검색 요청은 성공하지 못할 가능성이 높다.
   1. ES가 역색인을 구성하고 이 역색인으로부터 검색이 가능해지기까지 시간이 걸리기 때문이다. 기본 설정으로 최대 1초 정도 시간이 걸린다.
5. **트랜잭션을 지원하지 않는다.**
6. **사실상 조인을 지원하지 않는다.**
   1. 조인을 염두에 두고 설계되지 않았다
   2. `join`이라는 특별한 데이터 타입이 있지만 굉장히 제한적인 상황을 위한 기능이며 성능도 떨어진다.
   3. **RDBMS와는 다르게 데이터를 비정규화해야 한다** ⭐️


> 루씬은 데이터를 색인하고 검색하는 기능을 제공하는 검색 엔진의 코어 라이브러리이다.

1. 엘라스틱서치의 데이터를 손쉽게 시각화하는 **키바나**
2. 엘라스틱서치에 색인할 데이터를 수집하고 변환하는 **로그 스태시**
3. 경량 데이터 수집 플랫폼 **비츠**

ELK 스택을 이용하여 데이터의 수집부터 변환, 저장, 색인, 검색, 시각화까지 데이터 생명 주기 전체를 모두 엘라스틱의 솔루션으로 해결할 수 있어 큰 주목을 받았다.  

## 엘라스틱서치 구조

![](./imgs/structure.webp)
[출처](https://nidhig631.medium.com/primary-shards-replica-shards-in-elasticsearch-269343324f86)

- **문서** : ES가 저장하고 색인을 생성하는 JSON 문서를 뜻한다
- **인덱스** : **문서를 모아놓은 단위** 가 인덱스다. 클라이언트는 이 인덱스 단위로 ES에 검색을 요청하게 된다.
- **샤드** : 인덱스는 그 내용을 여러 샤드로 분리하여 분산 저장한다.
  - 또한 **고가용성을 제공하기 위해 샤드의 내용을 복제해둔다.**
  - 원본 역할을 담당하는 샤드를 primary shard, 복제본을 replication shard라고 한다.
- **_id** : 인덱스 내 문서에 부여되는 고유한 구분자다.
  - 인덱스의 이름과 _id의 조합은 ES 클러스터 내에서 고유하다.
- **타입** : 과거에 하나의 인덱스 안에 여러 문서를 묶어서 타입이라는 논리 단위로 나눴었지만, 이 개념은 폐기됐으며 논리적으로 구분해야 할 필요가 있다면 **별도의 인덱스로 독립** 시켜야 한다.
  - ES6 버전부터는 인덱스 하나에 타입 하나만 둘 수 있도록 제한됐다.
  - ES7 버전부터는 여러 API에서 타입을 명시하는 부분들이 지원 중단 됐다.
  - 타입 이름이 들어가야 할 자리에는 기본값인 `_doc`이 들어간 API를 사용해야 한다.
- **노드** : **ES 프로세스 하나가 노드 하나를 구성한다.**
  - ES 노드 하나는 여러 개의 샤드를 가진다.
  - 고가용성을 제공하기 위해 같은 종류의 샤드를 같은 노드에 배치하지 않는다.
  - 위의 그림을 보면 주 샤드와 복제 샤드 번호가 다른것을 확인할 수 있다.
- **클러스터** : 위의 ES 노드가 여러 개가 모여 하나의 클러스터를 구성한다.

> **노드의 역할**  
> - ES의 노드는 **데이터 노드** , **마스터 노드** , **조정 노드** 등 여러 역할 중 하나 이상의 역할을 맡아 수행한다.  
> - **데이터 노드** : 샤드를 보유하고 샤드에 실제 읽기와 쓰기 작업을 수행하는 노드
> - **마스터 노드** : 클러스터를 관리하는 중요한 역할을 수행하는 노드, 마스터 후보 노드 중에서 1대가 선출된다.
> - **조정 노드** : 클라이언트의 요청을 받아서 데이터 노트에 요청을 분배하고 클라이언트에게 응답을 돌려주는 노드

## 루씬 flush

문서 색인 요청이 들어오면 루씬은 문서를 분석해서 **역색인** 을 생성한다.  
최초 생성 자체는 메모리 버퍼에 들어간다. 문서 색인, 업데이트, 삭제 등의 작업이 수행되면 루씬은 이러한 변경들을 메모리에 들고 있다가 주기적으로 디스크에 flush 한다.  
  
**루씬은 색인한 정보를 파일로 저장** 하기 때문에 루씬에서 검색을 하려면 먼저 파일을 열어야 한다.  
파일을 연 시점에 색인이 완료된 문서만 검색할 수 있다.  
이후 색인에 변경사항이 발생했고, 그 내용을 검색 결과에 반영하고 싶다면 파일을 새로 열어야 한다.  

- [InternalEngine#createReaderManager](https://github.com/elastic/elasticsearch/blob/7e24080fb26a88d7b1a0b897ef425317251747d5/server/src/main/java/org/elasticsearch/index/engine/InternalEngine.java#L760)  
- [ElasticsearchReaderManager#refreshIfNeeded](https://github.com/elastic/elasticsearch/blob/7e24080fb26a88d7b1a0b897ef425317251747d5/server/src/main/java/org/elasticsearch/index/engine/ElasticsearchReaderManager.java#L47)  
- [DirectoryReader](https://lucene.apache.org/core/6_6_0/core/org/apache/lucene/index/DirectoryReader.html)  
  
```java
@Override
protected ElasticsearchDirectoryReader refreshIfNeeded(ElasticsearchDirectoryReader referenceToRefresh) throws IOException {
    return (ElasticsearchDirectoryReader) DirectoryReader.openIfChanged(referenceToRefresh);
}
```

1. ES는 내부적으로 루씬의 `DirectoryReader`라는 클래스를 이용해 파일을 연다
2. 루씬의 색인에 접근할 수 있는 `IndexReader` 객체를 얻는다.
3. ES는 변경 내용을 검색에 반영하기 위해 루씬의 `DirectoryReader#openIfChanged`를 호출해 변경 사항이 적용된 새 `IndexReader`를 열어 준 뒤 기존 IndexReader를 안전하게 닫는다.

위의 과정을 ES에서는 **refresh** 라고 하며, 이 단계까지 온 데이터가 검색 대상이 된다.  
이 작업은 비용이 있는 작업이기 때문에 적절한 간격마다 주기적으로 실행되며 또는 refersh API를 호출할 수 있다.  

## 루씬 commit

> 리눅스는 파일 I/O의 성능 향상을 위해 페이지 캐시라는 메모리 영역을 만들어서 사용한다.  
> 디스크로의 접근을 최소화하기 위해 한 번 읽은 파일의 내용을 페이지 캐시라는 영역에 저장시켜 놨다가 다시 한 번 동일한 파일 접근이 일어나면 디스크에서 읽지 않고 페이지 캐시에서 읽어서 제공해주는 방식이다.  
> [출처](https://brunch.co.kr/@alden/25)

루씬의 flush는 시스템의 페이지 캐시에 데이터를 넘겨주는 것까지만 보장할 뿐 디스크에 파일이 실제로 안전하게 기록되는 것까지 보장하지는 않는다.  
`fsync` 시스템 콜을 통해 주기적으로 커널 시스템의 페이지 캐시의 내용과 실제로 디스크에 기록된 내용의 싱크를 맞추는 작업을 수행하며 이를 루씬 commit이라고 한다.  
  
> 루씬의 flush와 엘라스틱 서치의 flush는 서로 다른 개념이며, 엘라스틱 서치의 flush 작업은 내부적으로 이 루씬 commit을 거친다.  

ES에 색인된 문서들은 루씬 commit까지 완료되어야 디스크에 안전하게 기록된다.  
commit은 비용이 많이 드는 작업이기 때문에 매번 할 수 없다. 그렇다고 변경사항을 모아서 commit한다면 장애 발생 시 데이터가 유실될 우려가 있다.  
**이런 문제를 해결하기 위해 엘라스틱서치 샤드는 모든 작업마다 translog라는 이름의 작업 로그를 남긴다.**  

## translog

색인 또는 삭제 작업이 루씬 인덱스 수행된 직후에 기록되며 translog에 기록이 끝난 이후에야 작업 요청이 성공으로 승인된다.  
ES 장애가 발생한 경우 샤드 복구 단계에서 translog를 읽으며, translog 기록은 성공했지만 루씬 commit에 포함되지 못했던 작업 내용이 있다면 샤드 복구 단계에서 복구된다.  
  
translog의 크기를 적절히 유지해줘야 하는데 **엘라스틱서치 flush는 루씬 commit을 수행하고 새로운 translog를 만드는 작업이다.**  
이 엘라스틱서치 flush가 주기적으로 수행되며 translog의 크기를 적절한 수준으로 유지한다.  
translog에는 디스크에 `fsync`된 데이터만 보존된다.  
  
클라이언트가 색인, 삭제, 업데이트 등의 요청을 보냈을 때 엘라스틱 서치는 translog에 성공적으로 `fsync`됐을 때에만 성공을 보고한다.  

## 세그먼트

![](./imgs/elasticSearchIndex.png)

위의 작업을 거쳐 디스크에 기록된 파일들이 모이면 **세그먼트** 라는 단위가 된다.  
이 **세그먼트가 루씬 검색의 대상** 이 되며, **루씬의 검색은 모든 세그먼트를 대상으로 수행된다.**  
세그먼트 자체는 불변으로 구성돼 있으며, 새로운 문서가 들어오면 새 세그먼트가 생성된다. 기존 문서를 삭제하는 경우 삭제 플래그만 표시해둔다.  
중간 중간 **병합 단계** 를 수행하여 삭제 플래그가 표시된 데이터를 실제로 삭제한다.  
병합 자체는 비싼 작업이지만 **검색 성능의 향상을 기대할 수 있다.**  
  
여러 세그먼트가 모이면 **하나의 루씬 인덱스가 된다.**  
**엘라스틱 서치 샤드는 이 루씬 인덱스 하나를 래핑한 단위다.** 그리고 **엘라스틱서치 샤드 여러 개가 모이면 엘라스틱서치 인덱스가 된다.**  
- `[ES 인덱스 1] : [N ES 샤드 1] : [1 루씬 인덱스]`
  
엘라스틱서치 레벨에서는 여러 샤드에 있는 문서를 모두 검색할 수 있다.  
**새 문서가 들어오면 해당 내용응 라우팅하여 여러 샤드에 분산시켜 저장, 색인한다.**  
이후 클라이언트가 엘라스틱서치에 검색 요청을 보내면 **엘라스틱서치는 해당하는 갹 사드를 대상으로 검색한 뒤 그 결과를 모아 병합하여 최종 응답을 만든다.**  
  
중요한 점은 **루씬 검색 대상으로는 루씬 인덱스 이며, 엘라스틱서치 검색 대상으로는 엘라스틱 서치 인덱스이다.**  

# 데이터 분산 처리 과정

데이터 읽기와 쓰기 작업 요청이 들어왔을 때 어떤 단계를 거치는지, 작업은 어떤 샤드에 배정되는지, 쓰기 작업의 결과가 어떻게 복제되는제, 어떻게 요청의 동시성 제어를 하는지, 샤드에 문제가 생겼을 때는 어떻게 복구가 되는지 알아보자.  
  
## 쓰기 작업 시 엘라스틱서치 동작과 동시성 제어

색인 API 등의 쓰기 요청이 들어올 때 쓰기 작업은 

1. **조정 단계 (coordination stage)** : 작업할 주 샤드를 선정
2. **주 샤드 단계 (primary stage)** : 주 샤드가 요청에 문제가 있는지 검증하고 요청을 처리
3. **복제 단계 (replication stage)** : 주 샤드의 작업이 끝나면 각 복제본 샤드로 요청을 넘김
  
클라이언트가 조정 노드에게 쓰기 작업을 요청하면 라우팅을 통해 어느 샤드에 작업을 해야할지 파악한다.  
몇 번 샤드에 작업해야 하는지가 확인되면 해당 번호의 샤드 중에서 현재 주 샤드를 찾아 작업을 넘겨준다.  
주 샤드는 요청을 검증하고 로컬에 쓰기 작업을 진행한다. 작업이 끝나면 주 샤드는 `in-sync` 복제본에 병렬적으로 복제 요청을 넘긴다.  
(마스터 노드는 작업을 복제받을 샤드 목록을 관리하고 있는데 이 목록을 `in-sync` 복제본이라고 한다.)  
복제 샤드들의 모든 응답을 주 샤드가 받고 난 이후 최초 요청을 수신했던 조정 노드에게 작업 결과를 응답하고, 조정 노드는 클라이언트에게 응답을 돌려준다.  
  
주 샤드의 변경 내용을 복제 샤드에게 병렬로 요청하게 되면 **메시지 순서의 역전이 일어날 수 있다.**  
어떻게 해결하는지 **낙관적 동시성 제어** 에 대해 알아보자.  
  
### 낙관적 동시성 제어

주 샤드의 변경 내용을 복제본 샤드에게 병렬로 복제된다. 만약 한 문서의 views 필드 값을 1로 색인했다면 잠시 후 복제본에도 이 내용이 적용된다.  
이때 이 1의 값이 복제본 샤드에 색인되기 전에 주 샤드에 같은 문서의 views의 값이 2로 색인하는 변경이 발생한다면 어떻게 처리될까?  
값을 2로 색인하는 요청도 복제본 샤드에 전달 될 텐데 **분산시스템 특성 상 1의 값 요청과 2의 값 요청 중 어떤 요청이 먼저 복제본 샤드로 들어올지 보장할 수 없다.**  
  
이 문제를 해결하기 위해 `_seq_no`가 존재한다.  
**이 숫자값은 각 주 샤드마다 들고 있는 시퀀스 숫자값이며 매 작업마다 1씩 증가한다.**  
엘라스틱서치는 문서를 색인할 때 이 값을 함께 저장하여 요청 순서의 역전 적용을 방지한다.  
  
주 샤드가 `_seq_no`를 각 요청에 할당하며 복제본 샤드는 동일한 문서의 동일한 필드가 다른 값으로 색인하는 요청이 순서가 역전되더라도 기존에 작업한 `_seq_no` 보다 작다면 순서가 역전되었다고 판단하고 해당 요청은 처리하지 않는다.  
  
주 샤드를 들고 있는 노드에 문제가 발생하여 해당 노드가 클러스터에서 빠진다면 엘락스틱 서치는 복제본 샤드 중 하나를 주 샤드로 지정한다.  
엘라스틱서치는 이전 주 샤드에서 수행했던 작업과 새로 임명된 주 샤드에서 수행했던 작업을 구분하기 위해 `_primary_term`이라는 값도 도입했다.  
이 값은 주 샤드가 새로 지정될 때 1씩 증가한다.  
  
```json
{
  "_index": "my_index5",
  "_id": "10",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 5
}
```

`_seq_no`는 같은 문서를 새로 색인하면 증가하며 주 샤드마다 따로 매긴다. 만약 주 샤드가 한 개라면 `_seq_no`를 공유하게 된다.  

### 버전

`_seq_no`를 확인하면서 `_version`필드도 증가하는 것을 확인할 수 있다.  
이 버전 필드도 동시성을 제어하기 위한 메타데이터로 모든 문서마다 붙으며 문서의 버전을 표현한다.  
**기본적으로는 1부터 시작해서 업데이트나 삭제 작업을 수행할 때 마다 1씩 증가한다.**  
다른 필드와 다른점은 클라이언트가 문서의 `_version` 값을 직접 지정할 수 있다는 점이다.  
  
```
PUT concurrency_test2/_doc/A?version=31&version_type=external
{
  "views": 31
}

# res
{
  "_index": "concurrency_test2",
  "_id": "A",
  "_version": 31,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 1,
    "failed": 0
  },
  "_seq_no": 2,
  "_primary_term": 1
}
```

더 낮은 값으로 `_version` 을 색인하면 예외가 발생한다.  
**external 타입의 버전을 명시하면 더 높은 버전으로만 업데이트가 가능하다.**  

```
PUT concurrency_test2/_doc/A?version=30&version_type=external
{
  "views": 30
}

# res
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[A]: version conflict, current version [31] is higher or equal to the one provided [30]",
        "index_uuid": "N8DMuuodThqkxHOoN00jEQ",
        "shard": "0",
        "index": "concurrency_test2"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[A]: version conflict, current version [31] is higher or equal to the one provided [30]",
    "index_uuid": "N8DMuuodThqkxHOoN00jEQ",
    "shard": "0",
    "index": "concurrency_test2"
  },
  "status": 409
}
```

## 읽기 작업 시 엘라스틱 서치 동작

클라이언트로부터 읽기 작업을 요청받은 조정 노드는 라우팅을 통해 적절한 샤드를 찾아 요청을 넘긴다.  
이때 쓰기 작업과는 다르게 주 샤드가 아니라 복제본 샤드로 요청이 넘어갈 수 있다.  
문서 단건 조회라면 하나의 샤드에 요청이 넘어가겠지만 검색 등 다수의 샤드에 요청을 넘겨줄 수도 있다.  
요청을 넘겨받은 각 샤드는 로컬에서 읽기 작업을 수행한 뒤 그 결과를 조정 노드로 돌려준다.  
조정 노드는 이 결과를 모아서 클라이언트에게 응답한다.  
  
읽기 요청이 주 샤드, 복제본 샤드 구분 없이 분산되어 독립적으로 동작하는 이 흐름에서는 주 샤드에 색인이 완료됐지만 특정 복제본에는 반영이 완료되지 않은 상태의 데이터를 읽을 수도 있다는 점을 염두에 둬야 한다.  
  
## 체크포인트와 샤드 복구 과정

만약 문제가 생겨서 특정 노드가 재기동되었다면 그 노드가 들고 있던 샤드에 복구 작업이 진행된다.  
이 과정에서 복구 중인 샤드가 현재 주 샤드의 내용과 일치하는지를 파악할 필요가 있다.  
재기동되는 동안 주 샤드에 색인 작업이 있었다면 그 내용을 복구 중인 샤드에도 반영해야 할 것이다.  
만약 문제가 발생했던 노드가 주 샤드를 들고 있었다면 다운타임 중에 선출된 새 주 샤드가 미처 반영하지 못한 작업 내용이 복구 중인 샤드에 포함돼 있을 수도 있다.  
이런 문제는 엘라스틱서치가 어떻게 해결할까?  
  
쓰기 작업에서 동시성 제어를 설명하면서 `_primary_term`과 `_seq_no`를 설명했다.  
**이 두 필드를 조합하면 샤드와 샤드 사이에 어떤 반영 차이점이 있는지를 알 수 있다.**  
  
각 샤드는 로컬에 작업을 수행하고 로컬 체크포인트로 기록한다.  
예를 들어, `_seq_no`가 각각 1,2,3,5,7인 작업을 수행한 샤드는 완료되는 대로 로컬 체크포인트에 기록한다.  
복제본 샤드는 로컬 체크 포인트가 업데이트되면 이를 주 샤드에 보고한다.  
이후에 이 복제본 샤드가 4번 작업을 수신하여 반영을 완료하면 로컬 체크 포인트를 5로 업데이트한다.  
주 샤드는 각 복제본 샤드로부터 받은 로컬 체크포인트를 비교하여 가장 낮은 값을 글로벌 체크포인트 값으로 기록한다.  
해당 작업 까지는 그 내용이 모든 샤드에 반영 완료됐다는 것을 나타낸다.  
글로벌 체크포인트가 업데이트되면 다음 색인 작업 시 그 내용을 주 샤드가 복제본 샤드로 전달한다.  
  
**문제가 발생해 샤드를 복구해야 할 경우가 생기면 엘라스틱서치는 샤드 간에 글로벌 체크포인트를 비교한다.**  
주 샤드와 복제본 샤드의 글로벌 체크포인트가 같다면 이 샤드는 추가 복구 작업이 필요 없다.  
글로벌 체크포인트가 차이 난다면 두 샤드 간 체크포인트를 확인해서 필요한 작업만 재처리하여 복구한다.  
  
루씬 레벨에서 엘라스틱서치가 수행하는 쓰기 작업은 새 문서의 색인과 기존 문서의 삭제 작업 두 가지 뿐이다.  
(업데이트 작업은 문서를 삭제하고 새 문서를 색인하는 작업에 불과하다.)  
체크포인트를 비교하여 샤드를 복구하는 과정에서 작업을 재처리할 때 문서 색인 작업은 루씬에 색인된 문서가 재처리에 필요한 정보를 모두 들고 있기 때문에 문제가 되지 않는다.  
그러나 데이터를 삭제하는 삭제 작업은 그렇지 않다. 최근 삭제한 문서를 일정 기간 보존해 두고 작업 재처리에 활용하는
**논리적 삭제** 를 도입하였다.  
  
재처리할 내용을 추척하는 이러한 메커니즘을 엘라스틱서치에서는 **샤드 이력 보존 (shard history retention leases)** 이라고 부른다.  
루씬의 세그먼트가 병합되는 도중에도 샤드 이력은 지정한 기간 동안은 보존된다. (기본값은 12시간이다.)  
이 기간을 넘겨서 샤드 이력이 만료된 이후 수행되는 복구 작업에는 작업 재처리를 이용하지 않는다.  
세그먼트 파일을 통째로 복사하는 방법을 사용한다.  
  

## 확인 목록

1. scroll api 보다는 search_after를 사용하라
2. 페이지네이션으로 전부 순회하며 집계를 하려고 한다면 size를 무작정 계속 높이는 것 보다는 composite 집계를 사용하라
3. vm.max_map_count 값 조회, 일반적으로 262144가 권고된다고 한다. `sysctl vm.max_map_count` 조회, `sudo sysctl -w vm.max_map_count=262144` 수정
4. 파일 기술자 값 조회 `ulimit -a | grep "open files"`, 65535 이상으로 지정하도록 가이드한다고 한다.
5. `cluster.routing.allocation.same_shard.host` 옵션으로 샤드 할당 시 같은 샤드의 주 샤드와 복제 샤드가 한 서버에 몰리지 않도록 조정할 수 있다. 기본값은 false이기 때문에 한 서버에 여러 프로세스를 띄우는 경우에만 수정하면 된다.


```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.2-darwin-x86_64.tar.gz
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.4.2-darwin-x86_64.tar.gz
wget https://github.com/lmenezes/cerebro/releases/download/v0.9.4/cerebro-0.9.4.zip

bin/elasticsearch -d -p {pid}
kill -SIGTERM `cat {pid file}`

bin/kibana
bin/cerebro
```