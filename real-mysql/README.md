# 실험
  
update시 where 절에 index 컬럼으로 특정 지으면 해당 값에 해당하는 레코드들이 락이 걸리기도 하고 update에 해당하는 레코드와 동일한 보조 인덱스의 값들을 가지는 로우들도 모두 락이 걸린다.  
테이블에 인덱스가 없으면 모든 로우가 락이 걸린다.  
락은 최대한 넓게 걸리는 것 같다..? where 절에 포함된 최소한의 범위로 락을 잡으려고 한다.  
클러스터 인덱스와 보조 인덱스를 같이 where 절에 추가한다면 클러스터 인덱스 기준으로 락을 건다.  
[Locks Set by Different SQL Statements in InnoDB](https://dev.mysql.com/doc/refman/8.0/en/innodb-locks-set.html)  
[Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)  
[보조 인덱스의 경우 InnoDB가 더 많은 레코드를 차단하는 이유는 무엇입니까?](https://stackoverflow.com/questions/60007863/why-does-innodb-block-more-records-in-case-of-a-secondary-index)  