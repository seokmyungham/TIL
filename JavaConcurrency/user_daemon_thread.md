# 사용자 스레드, 데몬 스레드

![image](https://github.com/user-attachments/assets/3182e915-201e-490f-bc87-951b83993bdd)

자바에서는 스레드를 `사용자 스레드(user thread)`와 `데몬 스레드(daemon thread)`로 구분할 수 있다.  
사용자 스레드는 사용자 스레드를 낳고 데몬 스레드는 데몬 스레드를 낳는다. 즉 자식 스레드는 부모 스레드의 상태를 상속 받는다.  
자바 어플리케이션이 실행되면 JVM은 사용자 스레드인 메인 스레드와 나머지 데몬 스레드를 동시에 생성하고 시작한다.

## 메인 스레드

메인 스레드는 어플리케이션을 실행하는 최초의 스레드이자 마지막 스레드의 역할을 한다. 메인 스레드에서 여러 하위 스레드를 추가로 시작할 수 있고
하위 스레드는 또 여러 하위 스레드를 시작할 수 있다.
  
메인 스레드가 사용자 스레드이기 때문에 하위 스레드는 모두 사용자 스레드가 된다.

## 사용자 스레드

사용자 스레드는 메인 스레드에서 직접 생성한 스레드를 의미한다.  
모든 사용자 스레드가 스레드 작업을 완료하면 데몬 스레드 실행 여부와 상관없이 JVM이 강제로 데몬 스레드를 종료하고 어플리케이션이 종료된다.
  
사용자 스레드는 포그라운드에서 실행되는 높은 우선순위를 가지며 JVM은 사용자 스레드가 스스로 종료될 때 까지 어플리케이션을 강제 종료하지 않고 기다린다.
자바가 제공하는 스레드 풀인 `ThreadPoolExecutor`는 사용자 스레드를 생성한다.

## 데몬 스레드

데몬 스레드는 JVM에서 생성한 스레드거나 직접 데몬 스레드로 생성한 경우를 말한다.  
모든 사용자 스레드가 작업을 완료하면 데몬 스레드의 실행 여부에 관계없이 JVM이 데몬 스레드를 강제로 종료하고 어플리케이션이 종료된다.

데몬 스레드의 생명주기는 사용자 스레드에 따라 다르며 낮은 우선순위를 가지고 백그라운드에서 실행된다.  
자바가 제공하는 스레드 풀인 `ForkJoinPool`은 데몬 스레드를 생성한다.

`thread.setDaemon(true)`로 데몬 스레드를 지정하며 반드시 스레드가 시작되기 전에 호출되어야 한다.  
스레드가 실행중인 동안 `setDaemon()`을 호출하려고 하면 IllegalThreadStateException이 발생한다.
  
`isDaemon()`으로 데몬 스레드인지 확인할 수 있다.

---

### Reference
- [자바 동시성 프로그래밍 \[리액티브 프로그래밍 Part.1\]](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%8F%99%EC%8B%9C%EC%84%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-part1/dashboard)
