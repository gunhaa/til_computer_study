# MappedByteBuffer를 이용한 Write Ahead Logging

```java
public class StrictWal implements IWal<TransactionEvent> {
    private final MappedByteBuffer mappedByteBuffer;
    private final ExecutorService walExecutor = Executors.newSingleThreadExecutor();
    private static final int LOG_SIZE_DAY = 1024 * 1024 * 10;

    public StrictWal(int routerIdx){
        try (RandomAccessFile file = new RandomAccessFile("./logs/" + getWalFileName(routerIdx), "rw")) {
            file.setLength(LOG_SIZE_DAY);
            this.mappedByteBuffer = file.getChannel()
                    // mmap() system call 사용, 파일을 프로세스의 가상 메모리 주소 공간에 연결
                    .map(FileChannel.MapMode.READ_WRITE, 0, LOG_SIZE_DAY);
    // ...
```

- RandomAccessFile을 이용해 파일명 + mode를 인자로 받아 파일을 열고, 파일의 position을 MappedByteBuffer abstract class를 통해 매핑시킨다
  - 이후 넣는 접근(`mappedByteBuffer.put()`)에서 파일을 쓴 후 `position` 프로퍼티를 수정해서 O(1)으로 write(put)를 시킨다
    - Page Fault: 해당 주소에 처음 접근할 때 페이지 폴트가 발생하며, Page Cache를 할당
      - `file.setLength(LOG_SIZE_DAY)`를 통해 disk가 write할 연속적인 공간을 할당한다
      - Page Fault를 발생시켜 처음에 요청받은 사이즈()만큼 RAM에 공간을 할당해 disk I/O가 없도록 한다
      - Page단위(4KB+@)로 Page Fault가 발생해 문제가 발생할 수 있으므로, 어플리케이션이 시작될때 Warm up하는 것이 필요할 것 같다
      ```java
        private void warmUp() {
            for (int i = 0; i < LOG_SIZE_DAY; i += 4096) {
                mappedByteBuffer.get(i);
            }
            // get은 idx접근이라 position이 변하지 않지만 코드의 명확성을 위해서
            mappedByteBuffer.position(0);
        }
      ```
    - Dirty Page 생성: 데이터를 쓰면 해당 페이지 캐시의 데이터가 수정된다. 이렇게 수정되었으나 아직 디스크에 반영되지 않은 메모리 영역을 'Dirty Page'라고 부른다
  - `mappedByteBuffer.force()`를 호출하지 않으면 OS 스케쥴링에 맞춰 batch flush를 해서 I/O를 최소화해서 오버헤드를 줄인다
    - Dirty Page가 너무 많아지거나, 데이터가 메모리에 머문 지 일정 시간이 지나면 OS 커널 스레드가 백그라운드에서 디스크로 데이터를 쓴다
    - 이 방식은 비동기적이라 성능은 좋지만, 쓰기 직후 시스템 전원이 나가면 데이터가 소실될 위험이 있다
      - 이를 해결하기 위해 Queue를 이용한 Scheduling Batch flush가 필요하다
