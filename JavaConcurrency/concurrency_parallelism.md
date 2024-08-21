# Concurrency, Parallelism

`동시성`과 `병렬성`은 컴퓨터 과학에서 중요한 개념으로,
여러 작업을 동시에 처리한다는 개념에서 비슷한 부분이 있지만 동일하지 않다.

## Concurrency

![image](https://github.com/user-attachments/assets/338d90dc-f00c-4742-9560-376839a88dc1)

동시성은 CPU가 한번에 많은 일을 처리하는 것에 중점을 둔다. 
여러 작업이 동시에 실행되는 것 처럼 보이지만 많은 작업들을 아주 빠른 시간으로 교체하면서 작업을 처리한다.
그래서 작업 처리를 빠르게 하기 위한 목적이 아니라 CPU를 효율적으로 사용하는 것에 더 의미가 있다.

> 스레드가 작업을 처리하다 IO 블록에 걸렸을 경우 CPU는 다른 스레드로 전환해서 작업을 진행

작업해야 할 수가 CPU 코어 수 보다 많을 경우 해당되며 동시성이 없으면 작업을 순차적으로 진행해야 한다.

## Parallelism

![image](https://github.com/user-attachments/assets/79a6dba0-79cd-428f-9d4b-58e0680778ec)

병렬성은 CPU가 동시에 많은 일을 수행하는 것에 중점을 둔다. 즉 CPU가 놀지 않고 최대한 바쁘게 동작해야 한다.
실제로 여러 작업이 동시에 실행되는 것이며 멀티코어 CPU나 여러 CPU에서 각각의 코어가 독립적으로 작업을 수행할 때 가능하다.
작업을 여러 스레드로 분리하고, OS는 그 스레드를 여러 CPU 코어에 적절히 분배하여 동시적으로 실행되도록 하는 것이다.\

---

### Reference
- [자바 동시성 프로그래밍 \[리액티브 프로그래밍 Part.1\]](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-%EB%8F%99%EC%8B%9C%EC%84%B1-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%EB%A6%AC%EC%95%A1%ED%8B%B0%EB%B8%8C-part1/dashboard)
