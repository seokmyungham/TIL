# Mutual Exclusion

## Process Synchronization

여러 개의 프로세스들이 서로 독립적으로 동작하는 `다중 프로그래밍 시스템 환경`에서  
둘 이상의 프로세스가 동시에 공유 자원을 사용하려고 할 때 문제가 발생할 수 있다.  
  
이러한 문제를 방지하기 위해 프로세스 간 서로 정보를 공유하는 행위를 `동기화(Synchronization)`라고 한다.

> 공유 데이터(Shared data): 여러 프로세스들이 공유하는 데이터  
> 임계 영역(Critical section): 공유 데이터를 접근하는 코드 영역  
> 상호 배제(Mutual exclusion): 둘 이상의 프로세스가 동시에 critical section에 진입하는 것을 막는 것

## Mutual Exclusion Methods

프로세스간 동기화를 달성하기 위해서는 핵심 매커니즘 중 하나인 `상호 배제`를 보장할 수 있어야 하는데  
상호 배제를 구현하기 위한 기본 연산으로 `enterCS()`, `exitCS()` 두 가지를 생각해 볼 수 있다.  
  
`enterCS()`는 임계 영역 진입 전, 다른 프로세스가 임계 영역 안에 존재하는지 검사하는 행위를 말하고  
`exitCS()`는 임계 영역을 벗어날 때 후처리 하는 과정으로, 임계 영역을 벗어났음을 시스템에 알리는 행위를 말한다.  

- Mutual exclusion(상호 배제)
  - CS에 프로세스가 있으면, 다른 프로세스의 진입을 금지해야 한다.
  - > 공유 자원의 일관성과 안전성을 보장하기 위해서는  
    > 한 시점에 오직 한 프로세스만이 그 자원을 사용하도록 보장해야한다.
- Progress(진행)
  - CS안에 있는 프로세스 외에, 다른 프로세스가 CS에 진입하는 것을 방해하면 안된다.
  - > CS안에 존재하는 프로세스만 CS로의 진입을 막을 수 있다.
- Bounded waiting(한정 대기)
  - 프로세스의 CS진입은 유한시간 내에 허용되어야 한다.
  - > 한정 대기를 만족하지 못하면 무한 대기하는 프로세가 생길 수 있다.
 
기본 연산을 직접 구현하고자 할 때, 위 세 가지 요구 사항을 만족할 수 있어야 한다.

---

## Mutual Exclusion Solution 

상호 배제를 해결 하기위한 방법들은 매우 다양하다.  
해결 방법들은 `소프트웨어`, `하드웨어`, `OS`, `Language-Level` 수준에서 점점 발전되어 왔다.

초기에 등장한 Dekker's, Dijkstra\`s 알고리즘 같은 `SW 솔루션`들은 속도가 느리고 구현이 복잡하다는 문제점이 있다.  
또한 연산 실행중 선점될 수 있기 때문에 안전하지 않으며 Busy waiting이 발생한다.  
  
`하드웨어에서 지원`하는 TestAndSet 명령어는 원자성이 보장되어 실행 중 인터럽트를 받지 않는다는 장점이 있지만  
여전히 Busy waiting이 발생하고 비 효율적인 단점이 존재한다.  

`OS가 지원하는 SW 솔루션`은 Spinlock, Semaphore, Eventcount/sequencer등이 존재하고  
`Semaphore`는 Busy waiting 문제를 해소한 대표적인 상호 배제 기법이다.
  
`Language-Level` 솔루션은 High-level 매커니즘으로서 사용이 쉽다는 장점 및 Deadlock과 error 발생 가능성이 낮다는 장점이 있지만
지원하는 언어에서만 사용 가능하고 컴파일러가 OS를 이해하고 있어야 한다는 단점이 존재한다.

---

# Reference

- [https://hpclab.tistory.com/1?category=887083](https://hpclab.tistory.com/1?category=887083)
