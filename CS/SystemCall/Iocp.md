# iocp, epoll, kqueue

- iocp는 window, epoll은 linux, kqueue는 UNIX의 Network I/O 처리 kernel interface이다
- I/O를 위해 시스템이 할당해둔 버퍼를 이용하며, User mode에서 I/O요청을 보내면 OS Buffer 공간에서 완료 메시지를 보내는 상태를 위한 kernel interface이다
- epoll/kqueue
  - OS에서 준비(Readiness)를 통지해서 시스템 콜을 준비한다
  - 준비를 통지받을때 한번(kernel -> user), read()를 통지받을떄 한번(user -> kernel), 총 두번의 mode change가 발생한다
- IOCP
  - OS에서 완료(Completion)요청을 걸어둔다
    - 요청을 담아둘 버퍼를 지정하기 때문에 완료 통지를 받을때의 mode change 밖에 없다
- 중요한 것은 두 OS의 concern이 다르다는 것이다
  - IOCP는 작업의 completion에 관심이 있다(결과: 발생한 이벤트의 결과값을 요청한 buffer에 담는다)
  - epoll/kqueue는 file descriptor(socket)의 상태에 관심이 있다(결과: 이벤트 발생 통지 전달)

## 전통적 Thread model과의 차이

- 전통적 Thread model은 요청을 보낸후 I/O가 일어나는 동안 user mode에서 kernel mode로 바뀌어 Thread가 Blocking(Sleep)의 상태로 변하는데, 이 것은 CPU가 예상할 수 없고 코어의 캐시가 무효화되는 비용이 큰 작업이다
  - 사용 스레드가 바뀌며 context switcing이 일어나고 cpu의 prefetch가 실패하고 캐시가 무효화되고 scheduler의 예측이 실패하고 새로운 schedule을 만들어야 되므로 작업이 굉장히 무거워진다
- async는 이 I/O 작업을 user mode 에서 kernel mode의 전환 후 요청 한번으로 해결한다(system call)
  - context switching이 일어나지 않는 것이 가장 큰 장점이다
    - 캐시 히트율이 비약적으로 높아지고, CPU와 Scheduler의 예측을 높은 효율로 사용할 수 있다
  - 문제가 있다면, fair하거나 세부 비즈니스 로직이 있는 경우 다루기가 어렵다는 것이다
  - 코드가 장황해질 가능성도 존재한다, 로직의 예외가 많이 발생해 코드 자체가 읽기 어려워지는 트레이드 오프가 있다
- 결국 컴퓨터 시스템 중 가장 병목지점이 큰 I/O를 해결하기 위한 모델이다
  - 모드 체인지를 이용한 시스템콜은 비용이 적어서 I/O 비용 대신 이 비용을 지불한다