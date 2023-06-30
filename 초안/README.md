# Item

1. `Collections.synchronizedMap`과 `ConcurrentHashMap` 차이
   - mutex로 한 개씩 락을 거는 방법과 Lock Striping에 대한 이해
2. [LXC 및 LXD: Linux 컨테이너 설명](https://www.sumologickorea.com/blog/lxc-lxd-linux-containers/)
   - [`RedHat` Linux 컨테이너란?](https://www.redhat.com/ko/topics/containers/whats-a-linux-container)
   - Docker의 차이
3. `@ConfigurationPropertiesScan`, `@EnableConfigurationProperties`, `@EnableAutoConfiguration`
4. `@Transactional` 샅샅이 파헤쳐보기
5. JSON 직렬화 시 표준에 맞지 않는 네이밍이면 ??
   1. `rName` -> `rname`으로 변환된다.
   2. [참고](https://unhosted.tistory.com/82)
   3. kotlin에서 필드에 `@JsonProperty`가 왜 안먹힐까?
   4. 그리고 왜 `rname`으로 변환되는지 확인하기