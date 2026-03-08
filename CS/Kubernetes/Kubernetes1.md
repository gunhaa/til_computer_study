# Kubernetes 세미나 1

## 쿠버네티스
- 컨테이너를 관리하는 오케스트레이션
- Unix Kernel의 격리 프로세스
- 1Pod = 1Container가 일반적이며, 보통 추가적으로 필요할 경우 sidecar를 사용한다
- 노드를 통해 물리적인 확장이 가능하다

## 주요 객체
- Pod
  - 컨테이너의 집합체
- Service
  - 파드에 접근 가능한 인터페이스
  - 존재이유: 리버스 프록시처럼 사용하며, 이를 통해서만 파드에 접근이 가능하다
- namespace
  - 하나의 cluster에서 논리적인 리소스 분리
- node
  - 하나의 물리적인 커널이며, 수평 확장이 가능하다
- volume
  - 파드는 임시이며 영구 저장이 불가능하기에, 영구적 보관을 위한 저장소이다
  - pod mount시 설정한다
- JOB
  - 일회성 작업, 파드는 작업후 죽는다
- CronJob
  - System cron과 같은 동작을하며, UTC를 사용하고 변경 불가함
- Daemonset
  - 단일 파드를 보장하며, 주로 로깅용 서브 프로그램을 사용한다
- Statefulset
  - 파드는 기본적으로 stateless이지만, stateful이 필요할 경우 사용한다(kafka, db 등)
- loadbalancer
  - 노출될 엔트리 포인트
- endpoint
  - 서비스를 만들면 자동생성되며, 파드의 virtual IP이다
- ingress(ingress nginx controller)
  - nginx proxy와 같은 역할을 한다
  - service가 있는데 사용하는 이유? 외부 접근을 위해서 사용한다
- 주요 설정 파일
  - pod.yml
    - 대소문자 구분 필수
  - deployment.yaml 
    - 죽은 후 살리기 위해 필요하다
  - replicaset.yaml
    - 롤링 업데이트시 필요하다
- 명령어
  - kubectl 
    - 주로 master에 설치한다
    - work node에도 설치 가능하다
    - binary이기에 설치만 하면 된다
