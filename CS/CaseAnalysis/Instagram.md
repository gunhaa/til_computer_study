# How Instagram Scaled Postgres to 2 Billion Users

> https://www.youtube.com/watch?v=YLoYcwnqVzM

- 초기 인스타그램은 복잡한 마이크로서비스 없이 단순한 Postgres 설정만으로 시작했으며, 이는 'boring technology'을 깊이 이해하는 것이 새로운 기술을 도입하는 것보다 강력하다
  - aws의 scale-up이 한계에 도달했을때 일반적인 NoSQL(Cassandra, MongoDB 등)로 변경 후 scale-out하는 방식이 아닌 기존 사용하는 postgre를 고도화 하는 방향을 선택했고, 이는 이후 많은 RDB를 다루는 기업들의 reference가 되었다
  - 2012년 당시 유저 수 2,700만 명, 데이터 용량 2TB에 도달하면서 aws 최대 스펙의 한계에 부딪혀 샤딩을 도입하게 되었다

## postgre connection 최적화

- instagram이 발전하면서 최초로 발생한 문제이다
- instagram은 database 단일 인스턴스를 scale-up해서 사용했다
- '단순 database connection 개수 증가로 인한 RAM 낭비'의 문제가 처음 발생했다
  - db connection 한개당 약 1.3mb의 memory가 소모된다
    - MySQL은 하나의 큰 프로세스 안에서 여러 thread를 생성해 커넥션을 처리한다
    - 반면, PostgreSQL은 클라이언트가 연결될 때마다 OS 레벨에서 완전히 독립된 backend process를 매번 fork하여 생성한다(connection이 문제가 생겨도 process를 지키기 위해)
    - 내부에 database에서 사용하는 metadata를 cache하고, 이것이 큰 용량을 차지한다
  - application은 scale-out상태로 50개가 존재하고 30개씩 connection pool을 유지하고 있다고 생각하면, 1500개의 connection이 존재하고, 이는 벌써 약2gb의 메모리를 소모한다
  - database의 메모리가 부족해져 문제가 생기게 된다
- PgBouncer를 이용해서 해결했다
  - PgBouncer는 PgBouncer는 애플리케이션으로부터 들어오는 수많은 연결 요청을 도맡아 받아준 뒤, 실제 PostgreSQL 데이터베이스 서버와는 훨씬 작은 크기의 실제 커넥션 풀만 맺고 이를 수많은 요청이 공유하여 재사용하도록 multiplexing 해준다
  - 기존 방식대로라면 1,500개의 커넥션이 2GB의 RAM을 무의미하게 낭비하고 있었겠지만, PgBouncer 덕분에 단 30개의 실제 커넥션 무게만 부담하면 된다
  - 규모가 커지는 PostgreSQL 환경에서 PgBouncer와 같은 커넥션 풀러는 가장 먼저 도입해야 할 레버리지가 높은 핵심 아키텍처이다

## postgre sharding

> 이 시기에(2012) scale-out은 일반적으로 NoSQL을 통해서 진행했지만, instagram은 해당 방식이 추상화 속에 가려질 뿐이지 본질적인 해결이 되지않는다고 생각해, RDB를 유지하고 자신들만의 방법으로 해결하기로 하였고 이것이 `논리-물리 분리 샤딩 패턴의 대표적인 레퍼런스`이며 가장 영향력있는 sharding architecture가 되었다

- 핵심 전략은 `논리적 샤드`와 `물리적 노드`의 분리이다
- 논리적 샤드는 schema를 이용해서 진행한다
  - postgre의 schema는 mysql의 database와 같다
  ```sql
  -- mysql의 경우 쿼리는 다음과 같이 나가야한다
  select u.id, u.name from shard0001.user where u.name = 'hgh';
  ```
- 처음에는 설정 값을 통해 shard번호가 어떤 물리적 위치(database)인지를 기록한다
- 추가적으로 확장이 필요하게 되면 shard를 다른 물리적 서버(database)로 옮기고 데이터도 옮기고 설정 값을 수정한다

### ID(고유값) shard 전략

- 64bit ID shard를 사용한다
  - 41 bits (Timestamp): ms 단위의 시간 정보이며, ID 자체만으로 시간순 정렬이 가능하게 한다
  - 13 bits (Logical Shard ID): 해당 데이터가 속한 논리 샤드의 고유 번호(최대 2^13)
  - 10 bits (Sequence Number): 하나의 샤드 안에서 동일한 밀리초에 생성되는 ID를 구분하기 위한 자동 증가 값(최대 2^10)