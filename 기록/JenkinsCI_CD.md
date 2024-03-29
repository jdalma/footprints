
# MSA란?

MSA를 왜 쓸까? MSA란 정확히 어떤 구조와 기능을 뜻하는 것일까? 서비스끼리 분리하고 API Call을 한다면 그걸 MSA라고 부를 수 있나?  
명확히 알지 못하고 있다.  
MSA를 쓰면
1. 각 서비스 배포를 부담없이 할 수 있다
2. 예외를 격리 시킬 수 있다
밖에 떠오르지 않는다.  
  
`MSA를 왜 쓰나요?`, `어떨때 MSA라고 자신있게 말할 수 있을까요?`라는 질문을 들은적이 있는데 명확히 답한적이 없는 것 같다.  
  
마틴 파울러가 작성한 [마이크로서비스 가이드](https://martinfowler.com/microservices/), [마이크로서비스](https://martinfowler.com/articles/microservices.html) 그리고 [MSA 개념 정립하기 - MSA 아키텍처 패턴](https://www.icatpark.com/entry/MSA-%EA%B0%9C%EB%85%90-%EC%A0%95%EB%A6%BD%ED%95%98%EA%B8%B0-MSA-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%ED%8C%A8%ED%84%B4-Client-%EC%9A%B4%EC%98%81%EC%9E%90-%EA%B0%9C%EB%B0%9C%EC%9E%90-%EC%B8%A1%EB%A9%B4%EC%9D%98-%ED%9D%90%EB%A6%84)를 읽어보자
  
**장점**  
- **모듈식 구조** 마이크로서비스 아키텍처 스타일은 단일 애플리케이션을 작은 서비스 모음 으로 개발하는 접근 방식  
  - 각 서비스는 자체 프로세스에서 실행  
- **독립적인 배포** 비즈니스 기능을 중심으로 구축 되며 완전히 자동화된 배포 시스템을 통해 독립적으로 배포  
- **기술 다양성** 서로 다른 프로그래밍 언어로 작성되고 서로 다른 데이터 저장 기술을 사용  
  
**단점**
- **운영 복잡성** 인프라 자동화 구축과 관리 비용
- **분산** 원격 호출은 느리고 실패할 위험이 존재
- **최종 일관성** 분산시스템에서 일관성을 유지하기 위한 높은 난이도

# 클라우드 네이티브란?  

- [`AWS` 클라우드 네이티브란 무엇입니까?](https://aws.amazon.com/ko/what-is/cloud-native/)
  - Microservices + Containers + DevOps + CI/CD

# 임시

1. Java17
2. Docker
3. node
4. newman, newman-reporter
5. /var/jenkins_home/workspace/webhook-test/src/test
6. docker cp jenkins:/var/jenkins_home/workspace/webhook-test/newman /Desktop 


for collection in ~/Desktop/newman/test/*/*.json
do
  newman run $collection -e ~/Desktop/newman/environment/dev1.json 
                         -r htmlextra 
                         --reporter-htmlextra-export ~/Desktop/newman/report/"$(date '+%Y-%m-%d_%H-%M-%S')"_$(basename "$collection" .json)'_report'.html
done