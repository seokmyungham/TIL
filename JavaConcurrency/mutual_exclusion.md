# Mutual Exclusion (상호 배제)

`뮤텍스(Mutual Exclusion)` 또는 `상호 배제`는 공유 자원에 대한 경쟁 상태를 방지하고 동시성 제어를 위한 락 매커니즘이다.
  
스레드가 임계 영역에서 Mutex 객체의 플래그(락)을 소유하고 있으면 다른 스레드가 액세스할 수 없으며 해당 임계영역에 액세스하려고
시도하는 모든 스레드는 차단된다. 이 매커니즘 덕분에 Mutex 락을 가진 오직 한개의 스레드만이 임계영역에 진입할 수 있으며 락을 획득한
스레드만이 락을 해제할 수 있다.

```c
do {
  acquired(mutex)
     // critical section
  release(mutex)
     // remainder section
} while (true);

m -> value = 1; lock = 0;
acquired(m) {
  while (test_and_set(&lock) == 1);  // 락 작업의 원자성을 보장
  if (m -> value == 0) {             // value가 0이면 이미 누군가가 락을 획득한 상태
    add this thread to m->list       // 스레드를 대기 큐에 추가
    block();                         // 스레드를 대기 큐에 넣고 대기 상태로 진입
  };
  m -> value = 0;                    // 다른 스레드가 진입하지 못하도록 value를 0으로 설정
  lock = 0;
}

realse(m) {
  while (test_and_set(&lock) == 1);  // 락 해제의 원자적 실행을 보장
  m -> value = 1;                    // 다른 스레드가 진입할 수 있도록 value를 1로 설정
  remove a thread T from -> list     // 대기 중인 스레드를 대기 큐로부터 가져옴
  wakeup();                          // 가져온 스레드를 깨우고 실행 대기큐로 이동
}
```

## Mutex Problems

### 데드락 (DeadLock)

데드락은 두 개 이상의 스레드가 서로가 가진 락을 기다리면서 상호적으로 블로킹되어 아무 작업도 수행할 수 없는 상태를 말한다.  
잘못된 뮤텍스 사용으로 인해 데드락이 발생할 수 있다.

### 우선 순위 역전 (Priority Inversion)

우선 순위 역전은 높은 우선 순위를 가진 스레드가 낮은 우선 순위를 가진 스레드가 보유한 락을 기다리는 동안 블록되는 현상이다.  
이로 인해 높은 우선 순위를 가진 스레드의 작업이 지연될 수 있다.

### 오버헤드

뮤텍스를 사용하면 여러 스레드가 경합하면서 락을 얻기 위해 스레드 스케줄링이 발생한다.  
이로 인해 오버헤드가 발생하고 성능이 저하될 수 있다.

### 성능 저하

뮤텍스를 사용하면 락을 얻기 위해 스레드가 대기하게 되고, 스레드의 실행 시간이 블록되면서 성능 저하가 발생한다.

---

### Reference
- [자바 동시성 프로그래밍 \[리액티브 프로그래밍 Part.1\]](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%8F%99%EC%8B%9C%EC%84%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-part1/dashboard)
