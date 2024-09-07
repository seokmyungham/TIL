# ThreadLocal

![image](https://github.com/user-attachments/assets/fe554a28-a951-4fc1-b2f0-2ccebab6e90c)

자바에서 스레드는 오직 자신만이 접근해서 읽고 쓸 수 있는 로컬 변수 저장소 `ThreadLocal`을 제공한다.  
각 스레드는 고유한 `ThreadLocal` 객체를 속성으로 가지고 있으며 `ThreadLocal`은 스레드 간 격리되어 있다.  
  
스레드는 `ThreadLocal`에 저장된 값을 특정한 위치나 시점에 상관없이 어디에서나 전역변수처럼 접근해서 사용할 수 있다.  
주로 모든 스레드가 공통적으로 처리해야 하는 기능이나 객체를 제어해야 하는 상황에서 스레드마다 다른 값을 적용해야 하는 경우에 사용한다. (인증 주체 보관, 트랜잭션 전파, 로그 추적기 등)

#

```java
public class Thread implements Runnable {

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
}
```
스레드는 ThreadLocal에 있는 `ThreadLocalMap` 객체를 자신의 threadLocals 속성에 저장한다.  
스레드 생성 시 threadLocals 기본 값은 null이며 ThreadLocal에 값을 저장할 떄 ThreadLocalMap이 생성되고 threadLocals와 연결된다.

ThreadLocalMap은 스레드마다 항상 새롭게 생성되어 스레드 스택에 저장되기 때문에 근본적으로 스레드간 데이터 공유가 될 수 없고 동시성 문제가 발생하지 않는다.

그러나 스레드 풀을 사용해서 스레드를 운용한다면 반드시 ThreadLocal에 저장된 값을 삭제해 주어야 한다.  
스레드 풀은 스레드를 재사용하기 때문에 이전 스레드에서 ThreadLocal에 저장된 값을 삭제하지 않고 재사용하게 된다면 문제가 발생할 수 있기 때문이다.

#

```java
public class ThreadLocal<T> {

    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
        if (this instanceof TerminatingThreadLocal) {
            TerminatingThreadLocal.register((TerminatingThreadLocal<?>) this);
        }
        return value;
    }
}
```

ThreadLocal은 Thread와 ThreadLocalMap을 연결하여 스레드 전용 저장소를 구현하고 있는데 이것이 가능한 이유는 바로 `Thread.currentThread`를 참조할 수 있기 때문이다.

`Thread.currentThread()`는 현재 실행중의 스레드 객체를 참조하는 것으로서 CPU는 오직 하나의 스레드만 할당받아 처리하기 때문에 현재 실행 중인 스레드의 로컬 변수를 저장하거나 참조할 수 있게 된다.
ThreadLocal에서 현재 스레드를 참조할 수 있는 방법이 없다면 스레드를 식별할 수 없을 것이다. 때문에 `Thread.currentThread()`는 ThreadLocal의 중요한 식별 기준이 된다.

## InheritableThreadLocal

`InheritableThreadLocal`은 ThreadLocal의 확장 버전으로서 부모 스레드로부터 자식스레드로 값을 전달하는 것이 가능하다.  
  
부모 스레드가 `InheritableThreadLocal` 변수에 값을 설정하면, 해당 부모 스레드로부터 생성된 자식 스레드들은 부모의 값을 상속받게 된다.
자식 스레드가 상속받은 값을 변경하더라도 부모 스레드의 값에는 영향을 주지 않는다.

```java
class Thread implements Runnable {
    private Thread(ThreadGroup g, Runnable target, String name,
                   long stackSize, AccessControlContext acc,
                   boolean inheritThreadLocals) {
        ...
        Thread parent = currentThread(); // main Thread
        ...
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals); // main Thread의 ThreadLocalMap을 복사
        ...
    }
}

class ThreadLocal<T> {
    static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }
}
```


---

### Reference
- [자바 동시성 프로그래밍 \[리액티브 프로그래밍 Part.1\]](https://www.inflearn.com/course/%EC%9E%90%EB%B0%94-
