# Monitor (모니터)

자바가 동기화를 지원하기 위해 사용하는 매커니즘은 `모니터 (Monitor)` 이며 뮤텍스나 세마포어보다 더 고수준의 동기화 기법이다.
  
모든 자바 객체는 기본적으로 모니터를 가지며(Object) 여러 스레드가 객체의 임계 영역에 진입하려고 할 때 JVM은 모니터를 사용하여
스레드 간 동기화를 제공한다. 자바 최상위 객체 `Object` 클래스는 `wait()`, `notify()`, `notifyAll()` 메서드를 포함하고 있기 때문에
그 하위의 모든 자바 객체들은 모니터를 가지고 동기화를 이룰 수 있다.

자바의 모니터는 `상호 배제(Mutual Exclusion)` 및 `협력(Cooperation)`이라는 동기화 기능을 제공하고 있으며
이를 위해 뮤텍스와 `조건 변수(Condition Variable)`를 사용한다.

## 상호 배제 (Mutex Exclusion)

객체가 가지고 있는 `모니터 Lock`을 통해 여러 스레드가 동시에 공유 자원에 접근하는 것을 막아 데이터의 일관성과 안전성을 보장하는 매커니즘이다.
  
JVM은 `synchronized` 키워드를 이용하여 뮤텍스 동기화를 암묵적으로 처리해 주고 있으며 `synchronized`는 메서드나 코드 블록에 적용할 수 있다.

## 협력 (Cooperation)

협력은 모니터의 `Condition Variable (조건 변수)`를 통해 상호협력으로 `작업의 효율성`을 높이고 데이터의 일관성과 안전성을 보장하는 동기화 매커니즘이다.
  
멀티스레드 환경에서 여러 스레드가 서로 협력하며 동시에 작업을 수행하므로, 각 스레드가 필요한 정보를 주고받으며 데드락을 방지하고 자원을 효율적으로 사용한다.
  
조건변수를 통해 스레드 간 `대기`와 `통지`를 서로 조절하면서 `경쟁 조건(race condition)`과 같은 문제를 방지할 수 있다. 
**모니터 내부에는 여러개의 조건 변수를 가질 수 있지만 자바 모니터는 오직 한 개의 조건 변수만 가질 수 있다.**

### Condition Variable (조건변수)

`Object` 클래스의 메소드인 `wait()`, `notify()`, `notifyAll()`과 함께 작용하며 특정 조건이 만족될 때까지 스레드를 대기시키는 기능을 제공한다.
  
스레드가 특정 조건에 부합하지 않을 때 `wait()` 메소드를 호출하면 조건변수의 `대기 셋(Wait Set)`에 들어가 대기한다.
다른 스레드가 특정 조건을 만족해서 `notify()` 또는 `notifyAll()` 메소드를 호출하면 해당 조건변수의 대기셋으로부터 스레드들을 깨워 실행시키게 된다.

## 모니터 구조

![image](https://github.com/user-attachments/assets/897096a4-1efc-4889-b9a9-b46211219016)

자바 모니터 내부에는 `EntrySet(진입셋)`과 `WaitSet(대기셋)`이라는 자료구조가 있으며 멀티스레드 환경에서 스레드들 간 상호작용을 조절하는 데 사용된다.

### EntrySet

`EntrySet`은 모니터 Lock을 획득하기 위해 대기중인 스레드들 모아 놓은 자료구조이다. 스레드가 Lock을 사용 중인 경우 그외 스레드는 EntrySet에 들어가게 된다.
  
`EntrySet`에 있는 스레드들은 Lock이 반납될 때까지 기다리며 락이 반납되면 EntrySet 중 하나의 스레드가 락을 획득하고 임계 영역으로 진입하게 된다.
 
### WaitSet

`WaitSet`은 모니터 조건 변수와 함께 사용하는 자료구조이며 스레드들이 특정한 조건을 만족할 때 까지 대기하고 있는 장소이다.
  
스레드는 `WaitSet`에 들어가 대기할 때 Lock을 해제하며 다른 스레드에 의해 깨어나게 되면 `EntrySet`으로 이동해서 Lock을 획득할 수 있다.

## 조건 변수 종류

조건변수를 통해 상호 협력하고 있는 두 스레드가 `wait()`과 `notify()` 메서드 실행 후에 하나의 모니터를 두고 두 스레드 모두 소유가 가능한 상황이 발생한다.
  
하나는 대기중인 스레드, 하나는 깨우는 스레드로서 어떤 스레드가 모니터를 먼저 소유할 것인가에 따라 두 종류의 조건변수 `Signal and Wait`, `Signal and Continue`로 나눌 수 있다.
  
이 개념들은 스레드 간에 자원의 접근을 제어하고 스레드 간의 협업을 가능하게 하는 방식이다.

### Signal and Wait

- 현재 모니터를 소유하고 있는 스레드가 `wait()`을 실행하면 모니터 내부에서 자신을 일시 중단하고 Lock을 해제한 후 대기셋에 들어간다.
- 깨우는 스레드가 `notify()` or `notifyAll()` 명령을 실행하면 대기셋에 있는 대기 스레드를 깨우고 깨우는 스레드는 Lock을 해제하고 대기한다.
- 대기에서 깨어난 스레드가 Lock을 획득한 후 모든 작업을 마치고 Lock을 해제하면 깨운 스레드가 Lock을 획득한 후 나머지 작업을 계속 작업을 진행한다.
- 대기 스레드와 깨운 스레드 사이에 다른 스레드가 모니터를 소유할 수 없도록 원자적 실행이 보장되어야 한다.

요약하면 하나의 스레드가 다른 스레드에게 신호를 보내고, `신호를 보낸 스레드는 스레드를 깨운 후 자신이 대기 상태에 들어가는 방식`이다. 
이 방식은 주로 하나의 자원을 공유할 때, 두 스레드가 교대로 해당 자원을 사용해야 하는 경우 사용된다.
  
이러한 협력 방식은 `생산자-소비자 패턴`에서 많이 활용된다.  
생산자는 데이터를 생산하고, 소비자는 데이터를 소비한다. 이떄 생산자는 버퍼가 가득찼을 때 생산을 멈추고 소비자가 버퍼를 비워줘야 생산을 재개할 수 있다.
마찬가지로 소비자는 버퍼에 데이터가 없을 때 소비할 수 없으므로 생산자가 데이터를 생산할 때 까지 기다려야한다. 이 협력 과정에서 신호를 주고받는 과정이 필요하다.

### Signal and Continue

- 현재 모니터를 소유하고 있는 스레드가 `wait()`을 실행하면 모니터 내부에서 자신을 일시 중단하고 Lock을 해제한 후 대기셋에 들어간다.
- 깨우는 스레드가 `notify()` or `notifyAll()` 명령을 실행하면 대기셋에 있는 대기 스레드를 깨운다. 이 때 일어난 스레드들은 진입셋에 이동한다.
- 깨우는 스레드는 Lock을 계속 유지하면서 모든 작업을 완료하고 Lock을 해제하면 진입셋에 대기하고 있는 모든 스레드가 Lock을 획득하기 위해 경쟁한다.

자바에서는 이 조건 변수 형식을 취하고 있다.  
  
하나의 스레드가 다른 스레드에게 신호를 보낸 후에도 신호를 보낸 스레드는 계속해서 자신의 작업을 수행하는 방식이다.
  
이러한 협력 방식은 신호를 보내는 것만으로 충분할 때, 예를 들어 어떤 작업이 완료되었음을 알리는 역할로 사용할 수 있다.
스레드들이 병렬로 작업을 진행할 때 유용하다. 생산자가 데이터를 생산하면서 다른 스레드가 그 데이터를 로그에 출력해야 한다면 생산자는 데이터를 생산하면서
로그 스레드에게 신호를 보내 로그 작업을 하도록 알리지만, 생산을 멈추지 않고 계속해서 데이터를 생성할 수 있다.

---

### Reference
- [자바 동시성 프로그래밍 \[리액티브 프로그래밍 Part.1\]](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-
