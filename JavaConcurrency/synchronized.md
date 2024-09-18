# Synchronized

자바는 연산의 원자성을 보장하기 위해 `synchronized` 키워드를 제공하고 있으며 `synchronized` 구문을 통해 모니터 영역을 동기화할 수 있다.
  
`synchronized`는 명시적으로 락을 구현하는 것이 아닌 자바에 내장된 락으로서 이를 고유 락(Intrinsic Lock) 혹은 모니터 락(Monitor Lock)이라고 한다. 
동일한 모니터를 가진 객체에 대해 오직 하나의 스레드만 임계영역에 접근할 수 있도록 보장하며 조건 변수를 통해 스레드간 협력으로 동기화를 보장한다.
  
`synchronized`가 적용된 한 개의 메서드만 호출해도 같은 모니터의 모든 `synchronized` 메서드까지 락에 잠기게 된다. 스레드가 `synchronized` 블록에 들어가기 전에
락을 자동으로 확보하며 정상적이든 비정상적이든 예외가 발생해서든 해당 블록을 벗어날 때 자동으로 해제된다.

#

### synchronized method 

```java
public class MyClass {

  public synchronized void syncMethod1() {

  } 

  public synchronized void syncMethod2() {

  } 
}
```

인스턴스 단위로 모니터가 동작한다. 따라서 동일한 인스턴스 안에서 `synchronized`가 적용된 곳은 하나의 락을 공유한다. 
인스턴스가 여러개일 경우 인스턴스별로 모니터 객체를 가지므로 모니터 별로 락을 획득해서 동기화 영역에 진입하고 빠져나올 때 락을 해제할 수 있다.

### static synchronized method

```java
public class MyClass {

  public static synchronized void syncMethod1() {

  } 

  public static synchronized void syncMethod2() {

  } 
}
```

클래스 단위로 모니터가 동작한다. 따라서 `synchronized`가 적용된 곳은 하나의 락을 공유한다. 
인스턴스와는 별개의 모니터를 가지고 임계 영역을 동기화 하기 때문에 인스턴스 단위로 메서드를 호출할지라도 락은 클래스 단위로 스레드간 공유된다.
클래스는 메모리에 오직 하나만 존재하므로 하나의 모니터를 공유해서 동기화하고자 할 떄 유용하게 사용할 수 있다.

### synchronized block

```java
public class MyClass {

  private Object lock = new Object();

  public void syncMethod1() {
    synchronized(this) {

    }
  }

  public void syncMethod2() {
    synchronized(this) {

    }
  }

  public void syncMethod3() {
    synchronized(lock) {

    }
  }

  public void syncMethod4() {
    synchronized(lock) {

    }
  }
}
```



  
