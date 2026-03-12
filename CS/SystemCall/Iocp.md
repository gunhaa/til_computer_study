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

- TODO
