# ThreadLocal

- 자바/스프링에서 많이 활용하는 락프리를 위한 데이터 저장소이다
  - 대표적으로 SpringSecurity의 Context, Spring의 Transaction의 상태 관리에 사용된다

## 내부 구조

```java
// ThreadLocal 내부 클래스
// 이 구조를 사용해서 새로운 ThreadLocal이 생길때마다 서로 다른 해시값을 갖게한다
private final int threadLocalHashCode = nextHashCode();
private static final AtomicInteger nextHashCode = new AtomicInteger();
private static final int HASH_INCREMENT = 0x61c88647;
private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}

// get
private T get(Thread t) {
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // ThreadLocal.threadLocalHashCode를 사용해 배열을 탐색하고, Key와 일치하는 값을 찾는다
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T) e.value;
            return result;
        }
    }
    return setInitialValue(t);
}

private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.refersTo(key))
        return e;
    else
        return getEntryAfterMiss(key, i, e);
}

///

private void set(Thread t, T value) {
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // this를 넘기는 이유는 ThreadLocal.threadLocalHashCode를 사용하기 위함이다
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}
```

- ThreadLocal을 key로 사용한다
  - 정확히는 ThreadLocal.threadLocalHashCode를 key로 사용한다
  - ThreadLocal<?>을 공유 객체로 사용하면 해시 값은 모두 같지만, 스레드마다 ThreadLocalMap이 생성되어(Thread.currentThread()를 사용하는 로직)
- Thread마다 ThreadLocalMap이 존재하며, 저장하지 않는다면 null상태로 lazy loading된다
- threadLocalHashCode를 통해 ThreadLocal이 생성될때마다 고유 값이 생성된다
- ThreadLocalMap은 Thread클래스에 threadLocals필드로 존재한다
  - Thread.currentThread().threadLocals의 형태로 호출한다
  - ThreadLocalMap은 클래스 이름과 다르게 Entry[]구조이며, 해시충돌이 일어날 경우 배열을 탐색하여 옆으로 이동한다

- 간단한 시간화
  - ThreadLocalMap.table에 Entry[]이 담기고, 이 곳에 스레드별로 생성된 ThreadLocal들이 담긴다
```plaintext
[스레드 A (Thread)]
   └── threadLocals 필드
          └── [ThreadLocalMap]
                 └── table 변수 ──> [ Entry[] 배열 ]
                                       ├── Entry[0]: Key(ThreadLocal1) -> Value("홍길동")
                                       ├── Entry[1]: Key(ThreadLocal2) -> Value("{ role: user }")
                                       ├── Entry[2]: 빈 칸 (null)
                                       └── Entry[3]: Key(ThreadLocal3) -> Value("{ setRollbackOnly: false }")
```