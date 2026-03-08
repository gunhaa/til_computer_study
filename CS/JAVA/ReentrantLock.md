# ReentrantLock의 컨텍스트 스위칭

- ReentrantLock을 이용해 Ciritical Section이 생기게되면 컨텍스트 스위칭을 위한 많은 비용이 발생한다
- 비용이 발생하는 이유는 뭘까?
  - ReentrantLock은 내부의 AQS(Abstract Queued Synchronizer)에 대기 공간을 만든다
    - CPU는 lock의 존재여부를 모르고 pre-fetch, branch prediction을 시도하지만 이 예측이 무효화되며 비용이 발생한다
    - prefetcher는 미리 L1, L2캐시로 가져오지만 진행을 하던 스레드는 잠들게되고 그 시간을 이용할 다른 job을 계산해서 가지고 와야하며, 캐시를 비우고 다시 넣는 연산 비용이 추가로 발생한다
    - 또한 스레드를 sleep시켜야 하기에 kernel mode에 신호를 보내는 과정에서 큰 비용이 발생한다
- 즉, 요약하자면 Lock은
  1. CPU의 예측(Branch/Prefetch)을 완전히 빗나가게 만들기에 Warm up 데이터인 L1/L2캐시가 초기화된다
  2. OS가 개입하여 관리용 연산을 추가로 하게 만든다