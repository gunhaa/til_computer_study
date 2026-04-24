# Spring Transaction의 동작

- Spring의 `@Transaction` 어노테이션과 JPA를 사용해 Database 사용은 높은 수준의 추상화가 적용 되어있어서 직관적으로 알기가 힘들어 한눈에 알아보기 쉽도록 정리
- `@Transaction`과 JPA는 결국 JDBC를 통한 DB접근 방법의 추상화이며, 정확히 알아야 문제 없이 프로그램을 작성할 수 있다
  - JDBC와 비교하는게 추상화를 정확히 알 수 있을 것 같아, 비교하여 정리

## Transaction의 추상화와 JDBC 비교

1. Connection을 얻기
- JDBC: `DriverManager.getConnection()`을 이용해서 소켓을 사용해 일회성 세션을 얻는다(TCP)
- Spring: `HikariDataSource`에 요청해서 HikariPool의 Connection을 얻는다
  - 두 API 모두 `setAutoCommit(false)`로 설정 후 작업을 시작한다

2. Transaction 시작
- JDBC: 쿼리를 시작하며 `start transaction`으로 시작한다
- Spring: `@Transaction`/TransactionTemplate 등으로 추상화 되어있으며, AOP(Proxy)를 이용하여 트랜잭션을 삽입한다

3. Query
- JDBC: Statement, PrepareStatement를 이용해 쿼리를 실행한다
- Spring: JPA Layer에서 CGLIB을 통해 자동 생성된 쿼리가 추상화 된 메서드 혹은 Mapper를 사용해 쿼리를 실행한다
  - JPA는 Persistence Context를 사용해 메모리에 DB 데이터를 올려, 변화에 따른 쿼리를 자동으로 처리한다
  - CGLIB 프록시를 통해 호출된 메서드가 Hibernate 엔진을 거쳐 SQL로 변환된다

4. Commit
- JDBC: `commit`을 실행해 커밋시킨다
- Spring: `TransactionSynchronizationManager`를 통해 실행되고 있는 트랜잭션을 참여시키거나 독립적으로 실행시키는 판단을 진행 후 커밋 혹은 롤백한다
  - `isNewTransaction` / `isRollBack`을 통해 트랜잭션의 상태를 판단해 rollback/commit을 실행한다
  - `ThreadLocal`을 이용해 스레드 독립적으로 실행된다

5. End
- JDBC: 얻은 Connection을 종료시키며, OS Level에서 소켓 연결을 종료한다
- Spring: 세션을 초기 설정 상태로 초기화 시킨후 `HikariDataSource`에 커넥션을 반납한다 
  - `setAutoCommit(true)`로 초기화 한다