# Hikari Connection Pool

> Hikari Connection Pool은 스프링이 채택하고 있는 JDBC전용 커넥션 풀이다

- `HikariConfig` 클래스에 DB Connection을 위한 설정을 저장 후 Connection 관리 클래스인 `HikariDataSource`를 통해 커넥션을 가져와서 connectionPool을 생성, 관리한다
  - `fastPathPool`필드를 통해 connection Pool을 관리하며, 스레드 풀 처럼 생성에 많은 코스트가 드는 커넥션을 생성 후 Pool로 관리한다
  - Postgre에서 추천하는 적정 커넥션 수는 `Runtime.getRuntime().availableProcessors()` *2 + harddisk 수(일반적으로 1)이며, Spring 기본 값은 10이다
- HikariCP는 이름처럼(일본어 빛) 빠른 커넥션이 되는 것을 목표로 많은 최적화가 들어가 있다

## HikariCP의 최적화들

### ThreadLocal을 이용한 Lock-Free 방식의 최적화

- `HikariPool`의 필드인 `ConcurrentBag<PoolEntry> connectionBag`필드를 이용한다
- `ConcurrencyBag`클래스의 `ThreadLocal<List<Object>> threadLocalList`를 사용해 이전에 사용된 스레드를 캐싱한다
  - 이 리스트를 역으로 순회하면서 기록된 최신 순으로 사용했던 Connection의 상태를 조사해 사용 가능한 Connection이 있으면 바로 사용한다
  - `connectionBag`에서 사용 가능한 Connection을 찾을 수 없다면 `CopyOnWriteArrayList<T> sharedList`를 사용해 `threadLocalList`필드에서 사용가능한 Connection이 없을 경우 현재 사용가능한 Connection을 찾는다
  - 사용한 Connection의 참조를 ThreadLocal에 넣어놓는 방식으로 Connection선택시 가장 많은 비용을 치루는 Lock의 경합을 회피한다

### 클래스의 필드 값 복사

- 클래스의 필드값을 복사해 JVM의 최적화를 돕는다
  - 필드의 값을 사용할 Compiler는 인라인화 결정을 하기 힘들다
  - 필드의 값을 final로 복사해 사용할 경우 Compiler는 값의 불변의 인식이 쉬워져 인라인화를 시도한다
    - 해당 메서드 스택 프레임 안에서만 존재하며, 값이 고정됨이 확실해진다. 즉 컴파일러는 이 값을 CPU 레지스터에 직접 올려두고 재사용할 수 있으며, 값이 변하지 않으니 이 값을 사용하는 메서드 호출 등을 인라인화할 수 있다
  - 알면 쓸모가 많은 최적화 방식 중 하나이다