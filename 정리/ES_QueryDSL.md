
[ES 공식문서 `v2.2`](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/)

# QueryDSL

쿼리를 정의하기 위해 JSON 기반의 쿼리 DSL을 제공한다.  

1. Leaf Query clauses
   - `match`, `term`, `range` 필드를 참조한다.
   - 이 쿼리는 자체적으로 사용 가능하다.
2. Compound Query clauses
   - 다른 리프 또는 복합 쿼리를 래핑하며 여러 쿼리를 논리적 방식으로 결합하거나 동작을 변경하는데 사용된다.

## Query and filter context

쿼리 절의 동작은 **쿼리 컨텍스트**에서 사용되는지 **필터 컨텍스트**에서 사용되는지 달라진다.  

1. **쿼리 컨텍스트**
   - 다큐먼트에서 얼마나 일치하는지 찾는다
2. **필터 컨텍스트**


```
다큐먼트 → 인덱스 → 샤드 → 노드 → 클러스터
```

## Full text Queries

1. **Match Query**
   1. `match`
   2. `match_phrase`, type : `phrase`
   3. `match_phrase_prefix`, `max_expansions`, type : `phrase_prefix`
2. **Multi Match Query**
   1. `multi_match`
   2. [multi match types](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-multi-match-query.html#multi-match-types)
3. **Common Terms Query**
4. **Query String Query**
5. **Simple Query String Query**

## Compound Queries
1. **[Bool Query](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/query-dsl-bool-query.html)**
   1. `must`
   2. `filter`
   3. `should`
   4. `must_not`

***

# Node

연결된 노드의 모음을 [Cluster](https://www.elastic.co/guide/en/elasticsearch/reference/2.2/modules-cluster.html)라고 한다.  
전송 계층에서는 노드 간 노드와 Java TransportClient간 통신에만 사용된다.  
HTTP 계층은 외부 REST 클라이언트에서만 사용된다.  
2개의 네트워크 통신을 사용한다.  
  
1. Master-eligible node `default`
   - 클러스터를 제어하는 마스터 노드로 선출될 수 있는 노드
2. Data node `default`
   - 데이터 노드는 데이터를 보유하고 CRUD, 검색 및 집계와 같은 데이터 관련 작업을 수행
3. Client node
   - 데이터를 보유할 수 없고 마스터 노드가 될 수 없는 노드
   - "스마트 라우터"로 작동하며 클러스터 수준 요청을 마스터 노드로 전달하고 데이터 관련 요청을 적절한 데이터 노드로 전달하는데 사용
4. Tribe node
   - 여러 클러스터에 연결하고 연결된 모든 클러스터에서 검색 및 기타 작업을 수행할 수 있는 특수항 유형의 클라이언트 노드

***

# Aggregations

검색 쿼리를 기반으로 집계된 데이터를 제공  
데이터의 복잡한 요약을 작성하기 위해 구성할 수 있는 Aggregations라고 하는 간단한 빌딩 블록을 기반으로 한다  
일련의 문서에 대한 분석 정보를 작성하는 작업 단위로 볼 수 있다.  
- 예를 들어 최상위 aggregations는 검색 요청의 실행된 쿼리/필터 컨텍스트 내에서 실행된다

