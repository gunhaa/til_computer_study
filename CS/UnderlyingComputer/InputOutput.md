# Chapter 6. 입출력이 없는 컴퓨터가 있을까?

## Section 6.1. CPU는 어떻게 입출력 작업을 처리할까?

- 각각의 하드웨어들은 데이터를 저장하는 register(data, status)와 device controller(작은 프로세서로써 CPU와 통신을 전담, interrupt를 발생 시킴)를 가지고 있다
  - device controller가 cpu에 interrupt를 보내고, cpu는 data register에서 값을 복사해간다(interrupt 기반 I/O 방식이며, 저속/소량에 사용한다 e.g. 키보드, 마우스)
  - cpu가 복사하는 것이 비효율적이라고 판단되면 DMA Controller(Direct Memory Access Controller)라는 별도의 하드웨어를 사용해 약속된 ram영역에 data register의 데이터를 복사한다(대용량 데이터에 사용한다 e.g. SSD에서 데이터 로드)
- 하드웨어와 통신에는 메모리 사상 입출력(Memory-Mapped I/O, MMIO) 방식을 사용한다
  - MMIO는 RAM에 특정 메모리 주소(e.g. 0-900 메모리로 사용, 900-950 키보드, 951-1000 마우스)을 매핑시키는 방법이다
  - 지정된 구역은 하드웨어의 data register로 직접 매핑이 되어있으며, 이 매핑이 MMIO로 불리며 하드웨어의 data register의 값을 특정 주소의 메모리 접근으로 바로 읽을 수 있다

### CPU는 어떻게 하드웨어와 데이터를 주고받을까?

1. 명령 (MMIO)
  - CPU가 MMIO 주소를 통해 device controller의 제어 레지스터에 `디스크에서 파일 읽어`라고 명령한다
2. 준비 (레지스터)
  - 하드웨어가 동작하여 데이터를 가져오고, 장치 컨트롤러 내부의 Data Register에 데이터를 채운다
3. 전송 (DMA 또는 인터럽트)
  - 소량 데이터: 장치 컨트롤러가 인터럽트를 보내면, CPU가 MMIO 주소를 통해 Data Register의 값을 RAM 버퍼로 직접 복사한다
  - 대용량 데이터: DMA 컨트롤러가 CPU 대신 Data Register의 데이터를 진짜 RAM 버퍼로 알아서 다 복사한 뒤, 완료되면 CPU에 인터럽트를 보낸다
4. 반복 (스트리밍)
  - 데이터가 레지스터나 버퍼 크기보다 크다면, 위의 복사-채우기 과정을 끝날 때까지 반복한다

### 저장된 데이터는 어떻게(어떤 시점에) 가져올까?

1. 폴링 방식(동기 대기)
  - 가장 생각하기 쉬운 방식이다
  - spin 대기로써 CPU에 부하가 가는 동기적인 방식이다
  - 이를 자연스럽게 동기 -> 비동기로 바꾸는 것은 컴퓨터 과학에서 매우 일반적인 최적화 방법이다
    - 소프트웨어, 하드웨어 할 것 없이 적용 가능한 방식이다
2. 인터럽트 기반 처리(비동기)
  - 하드웨어의 device controller가 interrupt를 보내면 CPU는 작업의 우선순위를 판단해 interrupt가 높다면 우선 처리하고, 현재 작업으로 돌아온다
  - 현재 진행중인 작업보다 interrupt의 우선순위가 높다면 진행 중인 작업의 context를 저장한 뒤 interrupt를 수행 후 context를 복구한다

## Section 6.4. 높은 동시성의 비결: 입출력 다중화

### 입출력은 file을 통해서 실행된다

- 프로그래머가 입출력을 실행하는 코드를 작성한다면 결국 file이라는 개념을 벗어나는 것이 불가능하다
  - UNIX/Linux세계에서 file은 매우 간단한 개념으로, 프로그래머는 파일을 N바이트의 수열(sequence)로 이해하면 된다
  - 모든 입출력 장치는 file이라는 이름으로 추상화된다
    - 이는 `everything is file`이라는 개념이다
    - 디스크, 네트워크 데이터, 터미널, 프로세스 간 통신 도구인 파이프까지 모두 파일로 취급된다
  - 프로그래머는 file이라는 인터페이스로 모든 외부 장치를 사용할 수 있다
    - open, write로 파읽을 읽고 쓴다
    - seek로 file일을 읽고 쓰는 위치를 변경 할 수 있다
    - close로 file을 닫을 수 있다
    - 이것이 file이라는 인터페이스가 발휘하는 힘이다
- UNIX/Linux을 사용하기 위해서는 file을 식별하기 위한 식별자가 필수이며, 이를 `file descriptor`이라고 부른다
  - 외부 장치라는 것이 천차 만별이고 커널에서 이 장치들을 표현하거나 처리하는 방법도 모르두 다르지만 이것은 프로그래머가 알 필요가없으며, 프로그래머가 알아야할 것은 `file descriptor`라는 숫자 하나뿐이다
  - 이 file interface의 함수들을 채우는 것이 `hardware driver`이다

### 입출력 다중화

- file을 다루는 함수는 동기적, 블로킹 입출력이다
- 이를 극복하기 위해 입출력 다중화(I/O multiplexing)이 사용된다
- I/O multiplexing은 다음과 같은 과정을 의미한다
  1. file descriptor를 획득한다
  2. 특정 함수를 호출해 커널에 알린다: `이 함수를 먼저 반환하는 대신, 이 file descriptor를 감시하다가 읽거나 쓸 수 있을 때 반환해 줘` = 특정 시스템 콜(select, epoll, kqueue, IOCP, io_uring)
  3. 해당 함수가 반환되면 사용 가능한 file descriptor를 획득할 수 있다
- 위 방식은 현대 async/event-driven architecture의 핵심 근간이 된다
  - 하부 구조 (OS & 런타임): 이 I/O multiplexing 기반으로 대규모 요청을 대기 없이 처리하는 event loop 엔진(e.g., nodejs의 libuv, java의 Netty 등)이 돌아간다
  - 상부 구조 (프로그래머가 쓰는 추상화 도구): event loop 위에서 개발자가 동기식 코드처럼 편하게 비동기를 제어할 수 있도록 제공되는 스펙이 바로 Promise(Node.js), Publisher/Subscriber(WebFlux), Coroutine(Kotlin)이다

### mmap

- file을 다루는 함수는 동기적, 블로킹 입출력이다
- read()의 동작은 커널의 특정 영역(buffer)에 데이터를 가지고 온 뒤, user가 요청한 메모리 영역에 kernel buffer의 데이터를 복사해서 전달한다
  - 결국 파일을 한번 read()하기 위해 두 번의 복사가 일어나는 문제가 발생한다
- 이를 해결하기 위한 system call이 mmap이다
  - mmap는 메모리의 특정 영역과 disk의 영역을 같이 사용한다(kernel에서 유저 영역으로의 '추가 복사'를 하지 않고 kernel의 메모리 영역을 사용할 수 있게 해준다, 이를 zero-copy라고 한다)
  - os scheduling 상태에 따라 disk의 원본 파일과 동기화 시킨다
  - flush()를 사용해 강제로 동기화 시키는 방법도 있다(성능 상 문제 생김)
    - java에서는 flush()이고, 정확한 system call은 msync이다