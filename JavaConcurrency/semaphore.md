# Semaphore (세마포어)

![image](https://github.com/user-attachments/assets/4d873165-0f06-4553-813c-22178fa042ce)

`세마포어`는 공우 자원에 대한 접근을 제어하기 위해 사용되는 신호전달 매커니즘 동기화 도구이다.  
  
세마포어는 `정수형 변수 S`와 `P(Proberen: try)`, `V(Verhogen: increment)`의 두 가지 `원자적 함수`로 구성된다.  
P는 임계 영역을 사용하려는 스레드의 진입 여부를 결정하는 연산으로 `Wait` 연산이라고도 하고  
V는 대기 중인 프로세스를 깨우는 신호로 `Signal` 연산이라고 한다.
  
스레드가 임계영역에 진입하지 못할 경우 자발적으로 블록상태에 들어가고 임계영역을 빠져나오는 스레드가 블록상태의 스레드를 실행대기상태로 깨워준다.
자바에는 java.util.concurrent 패키지 하위에 세마포어 구현체가 존재한다.

## 작동방식

```c
do {
  P(S)
    critical section
  V(S)
    remainder section
} while(true)

P(Semaphore *S) {
  while (test_and_set(&lock)) == 1);
  S->value--;
  if(S->value < 0) {
    add this thread to m->list
    block();
  }
}

V(Semaphore *S) {
  while (test_and_set(&lock)) == 1);
  S->value++;
  if(S->value <= 0) {
    remove a thread T to m->list
    wakeup(t)
  }
}
```

세마포어 카운트(S)는 공유 자원의 개수로서 이 개수 만큼 스레드의 접근이 허용된다.  
  
임계 영역에 진입하기 전 Wait 연산을 통해 카운트를 1 감소시키고, 만약 S가 0보다 작으면 공유자원이 없는 것으로 스레드 접근을 거부하고 대기 큐로 진입시킨다.  
  
스레드가 작업을 마치고 임계 구역에서 빠져나올 때 Signal 연산을 통해 카운트를 1 증가시키고, 세마포어 카운트가 0 이하라면 대기중인 스레드가 존재한다고 판단하게된다. 대기 중인 스레드를 큐로부터 가져와 깨우고 실행대기 큐로 이동시킨다. 

## 이진 세마포어 (Binary Semaphore)

### 직접 구현

```java
public class BinarySemaphore implements Semaphore {

  private int signal = 1;

  public synchronized void accquired() {
    while(this.signal == 0) wait();
    this.signal = 0;
  }

  public synchronized void release() {
    this.signal = 1;
    this.notify();
  }
}
```

카운트 변수 S가 1인 세마포어를 이진 세마포어로 구분할 수 있다.  
  
이진 세마포어는 뮤텍스 처럼 락으로 사용할 수 있으며, 한 스레드만이 세마포어를 획득할 수 있기 때문에 다른 스레드들이 acquired() 호출하면 세마포어가 해제되기 전까지 블록된다.

## 카운팅 세마포어 (Counting Semaphore)

### 직접 구현 

```java
public class CountingSemaphore implements Semaphore {

  private int signal;
  private int permits;

  public CountingSemaphore(int permits) {
    this.permits = permits;
    this.signal = permits;
  }

  public synchronized void accquired() {
    while(this.signal == 0) wait();
    this.signal--;
  }

  public synchronized void release() {
    if (this.signal < permits) {
      this.signal++;
      this.notify()
  }
}
```

카운팅 세마포어는 카운트 변수를 설정해서 스레드가 공유할 수 있는 자원의 최대치를 한정하고 운용하는 방식이다.  
자원 풀이나 컬렉션의 크기에 제한을 두고자 할 때 유용하다. (DB ConnectionPool, 파일 다운로드 동시 실행 제한 등)
  
핵심은 락을 획득한 스레드와 락을 해제하는 스레드는 다를 수 있다는 것이다.  
스레드 간 락과 락 해제를 위한 신호를 전달함으로 동기화를 구현한다.

## 프로세스 순서화 (Ordering)

세마포어를 활용하면 스레드간 프로세스 흐름을 순서화할 수 있다.

```java
class WaitThread extends Thread {

  private Semaphore semaphore;

  public WaitThread(Semaphore semaphore){
    this.semaphore = semaphore;
  }

  public void run() {
    while(true){
      this.semaphore.acquired();
      // critical section
    }
  }
}

class SignalThread extends Thread {

  private Semaphore semaphore;

  public SignalThread(Semaphore semaphore){
    this.semaphore = semaphore;
  }

  public void run() {
    while(true){
    // do task;
    this.semaphore.release();
    }
  }
}
```

이진 세마포어의 카운트를 0으로 설정하면, 어떤 스레드가 먼저 수행되어도 WaitThread는 항상 임계구역에 진입하기 전 SignalThread의 release() 호출을 기다려야 한다.
  
만약 WaitThread 가 먼저 실행될 경우 WaitThread는 세마포어의 대기 큐에 진입하게 되고 SignalThread의 신호를 기다린다.  
혹은 SignalThread가 먼저 실행될 경우 임계영역에서 작업을 마치고 세마포어 카운트를 1 증가시켜 WaitThread의 임계영역 진입을 허용한다.
  
결과적으로 두 스레드중 어느 것을 먼저 실행하여도 항상 SignalThread -> WaitThread 순서로 작업을 수행하게 설계할 수 있게된다.

## 뮤텍스 VS 세마포어

### 뮤텍스

뮤텍스는 공유 자원에 대한 접근을 동시에 하나의 스레드만 가능하도록 보장하는 기술이다.  
즉 뮤텍스는 상호 배제를 위한 동기화 기법이다. 

뮤텍스는 소유권이 있어서 락을 획득한 스레드만이 락을 해제할 수 있다.  
즉 락을 획득한 스레드가 락을 해제하지 않으면 다른 스레드는 해당 뮤텍스에 접근할 수 없다.
  
뮤텍스는 기본적으로 잠겨있는 상태로 시작하고 한 스레드가 뮤텍스를 획득하여 자원에 접근하면 다른 스레드들은 해당 뮤텍스를 획득하기 위해 블로킹된다.
  
뮤텍스는 주로 상호 배제를 위해 사용되며 하나의 자원에 하나의 스레드만 접근하도록 보장해야 하는 경우에 사용된다.

### 세마포어

세마포어는 카운팅 기법으로, 특정 개수의 스레드가 동시에 공유 자원에 접근할 수 있도록 제어하는 기술이다.  
0또는 1의 값을 이진 세마포어는 뮤텍스와 유사한 역할을 하며, 카운팅 세마포어는 양수 값만큼 스레드의 동시 접근을 허용한다.
  
세마포어는 뮤텍스와 달리 소유권의 개념이 없다. 특정 개수의 스레드가 동시에 접근을 허용하는 카운팅 기법으로 작동한다.  
따라서 세마포어를 사용하는 스레드들이 모두 세마포어를 해제할 수 있다.
  
세마포어는 초기값을 설정할 수 있고 초기값에 따라서 처음부터 스레드가 자원에 접근할 수 있는지의 여부가 결정된다.  
초기값은 0일 수도 있다.
  
세마포어는 주로 리소스의 한정적인 사용을 제어하는 데 사용되며 특정 개수의 스레드만이 동시에 자원에 접근하도록 제한하고자 할 때 사용된다.

---

### Reference
- [자바 동시성 프로그래밍 \[리액티브 프로그래밍 Part.1\]](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%8F%99%EC%8B%9C%EC%84%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-part1/dashboard)
