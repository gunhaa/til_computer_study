# ReentrantLock의 컨텍스트 스위칭

- ReentrantLock을 이용해 Ciritical Section이 생기게되면 컨텍스트 스위칭을 위한 많은 비용이 발생한다
- 비용이 발생하는 이유는 뭘까?
  - ReentrantLock은 내부의 AQS(Abstract Queued Synchronizer)에 대기 공간을 만든다
    - CPU는 lock의 존재여부를 낙관적으로 예측하며 pre-fetch, branch prediction을 시도하지만 이 예측이 무효화되며 비용이 발생한다
    - prefetcher는 미리 L1, L2캐시로 가져오지만 진행을 하던 스레드는 잠들게되고 그 시간을 이용할 다른 job을 계산해서 가지고 와야하며, 캐시를 비우고 다시 넣는 연산 비용이 추가로 발생한다
    - 또한 스레드를 sleep시켜야 하기에 kernel mode에 신호를 보내는 과정에서 큰 비용이 발생한다
- 즉, 요약하자면 Lock은
  1. CPU의 예측(Branch/Prefetch)을 완전히 빗나가게 만들기에 Warm up 데이터인 L1/L2캐시가 초기화된다
  2. Kernel mode에 신호를 보내 Thread를 sleep시키는 비용이 큰 연산을 추가로 하게 만든다

## ReentrantLock의 fair/unfair모드

- fair모드와 unfair모드의 차이는 AQS에 넣기 전에 가져가는 것을 허가하는가, 허가하지 않는가의 차이이다
  - unfair모드에서는 락을 걸때 락을 확인하고 CAS를 시도하기에 더 빠를 수 있다
  - fair모드에서는 락을 걸때 락을 확인하지 않고 바로 Queue로 넣는다
    - 더 빠를수 있는 상황에서도 Queue에 넣는 것이 선행되기(hasQueuedPredecessors로 계산 후 add한다, first waiter확인)에 성능차이가 발생한다

## Recursive 상태 제어

- 일반적인 lock의 경우 recusive lock을 걸게 되면 deadlock에 빠진다
  - Recursive lock: 락을 획득한 상태에서 프로세스가 그 락을 해제하기 전에 다시 그 락을 획득하는 것
- Reentranlock은 recursive lock을 걸더라도 데드락이 발생하지 않는다
  - 내부적으로 lock을 count하기 때문에 데드락에 빠지지 않는다
  - Rust의 경우 가변 참조자가 1개 이상인 것을 절대 허용하지 않기에 재귀락을 허용하지 않는다
- 내부 구현 방식(동시성 프로그래밍 158p)
```C
// 재진입 가능한 뮤텍스용 타입
struct reent_lock {
   bool lock; // 락용 공유 변수
   int id; // 현재 락을 획득 중인 스레드 ID, 0이 아니라면 락 사용중
   int cnt; // 재귀락 카운트
}

// 재귀락 획득 함수
void reentlock_acquire(struct reent_lock *lock, int id) {
  // 락 획득 중이고 자신이 획득 중인지 판정
  if (lock -> lock && lock -> id == id) {
    lock -> cnt++;
  } else {
    // 어떤 스레드도 락을 획득하지 않았거나
    // 다른 스레드가 락 획득 중이면 락 획득
    spinlock_acquire(&lock -> lock);
    // 락을 획득하면 자신의 스레드 ID를 설정하고
    // 카운트 증가
    lock -> id = id;
    lock -> cnt++;
  }
}

// 재귀락 해제 함수
void reentlock_release(struct reent_lock *lock) {
  //카운트를 감소하고
  // 해당 카운트가 0이되면 락 해제
  lock -> cnt--;
  if (lock -> cnt == 0) {
    lock -> id = 0;
    spinlock_release(&lock -> lock);
  }
}
void spinlock_release(struct reent_lock *lock) {
    // Full Memory Barrier
    // java의 volatile과 다르게 c의 volatile은 compiler의 최적화를 막을 뿐이라
    // java처럼 스레드 가시성 보장을 하기 위해서 memory barrier를 호출하는 함수를 사용해야한다
    __sync_synchronize();

    lock->lock = false; 
}
```
- Memory Barrier는 java의 경우 volatile키워드로 실행되며, C에서는 `__sync_synchronize()`로 호출 가능하다
  - memory barrier는 cpu의 ordering을 강제한다
  - memory barrier 이전의 job의 완료를 보장하고, 이후가 실행된다
  - 실행되는 코어의 L1,L2캐시를 L3/RAM으로 전달하며, 다른 코어의 L1,L2캐시를 최신화하는 요청을 하드웨어 수준에서 동작시킨다
