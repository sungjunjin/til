# Singleton
어떤 클래스에서 만들 수 있는 인스턴스 수를 하나로 제한하는 페턴. 주로 리소스를 많이 차지하는 객체를 메모리 절약을 위해 기존에 한번만 생성된 인스턴스를 가져와 다시 활용하는 기법이다.

싱글턴 패턴을 충족하기 위해서는 다음과 같은 조건을 충족해야 한다.
- 프로그램 실행 중에 최대 하나만 있어야 한다
- 싱글턴 객체에 전역적으로 접근이 가능해야 한다

## 기본 구현
1. Private 생성자를 가진다. 따라서 외부에서 객체를 생성할 수 없다.
2. Static 메소드를 통해서만 객체를 얻어올 수 있다.
    - 아직 객체가 없는 경우
        - 객체를 생성 후 static 변수에 저장
        - static 변수에 저장된 객체를 반환 
    - 이미 객체가 있는 경우
        - static 변수에 저장되어 있는 객체를 반환


## 싱글톤 구현 기법

### Eager Intialization
싱글톤 클래스의 static final 변수를 두고 객체를 미리 생성해두는는 방법이다. 
```java
public class EagerInit {
    private static final EagerInit INSTANCE = new EagerInit();

    private EagerInit() {

    }

    public static EagerInit getInstance() {
        return INSTANCE;
    }
}
```
클래스 변수가 Static final이므로 수정의 여지가 없다. 하지만 싱글톤 객체를 생성하는 시점에 발생할 수 있는 예외를 처리할 방법이 없고, 객체를 사용하지 않아도 메모리에 미리 생성되기 때문에 리소스의 낭비가 발생한다.

### Static Block Initialization
Static 블록 내부에 객체를 초기화 하는 방식이다.
```java
public class StaticBlockInit {
    private static StaticBlockInit INSTANCE;

    private StaticBlockInit() {

    }

    static {
        try {
            INSTANCE = new StaticBlockInit();
        } catch (Exception e) {
            throw new RuntimeException("Singleton 초기화 실패");
        }
    }

    public static StaticBlockInit getInstance() {
        return INSTANCE;
    }
}
```
싱글톤 객체를 생성하는 시점에 발생할 수 있는 예외를 처리할 수 있다는 점에서 Eager Initialization 방식보다 개선되었다고 할 수 있다. 하지만 여전히 객체를 사용하지 않아도 메모리에 미리 생성된다는 단점이 있다.

### Lazy Initialization
싱글톤 예시로 가장 많이 나오는 방식이며, 객체를 가져오는 메소드를 호출했을때 객체의 null 여부에 따라서 싱글톤 객체를 객체를 생성하여 반환하는 방식이다.

```java
public class LazyInit {
    private static LazyInit instance;

    private LazyInit() {
    }

    public static LazyInit getInstance() {
        if(instance == null) {
            instance = new LazyInit();
        }

        return instance;
    }
}
```
싱글톤 객체를 실제로 사용할 시점에 객체를 생성하고 가져오는 메소드를 호출하여 위 Eager Initialization과 Static Block Initialization 방식보다는 메모리 관리가 효율적이라는 점이 있지만 여러 쓰레드가 동시에 접근했을때 여러개의 객체가 생성될 수 있다는 치명적인 단점이 존재한다.

아래 예시를 들어 LazyInit 클래스에 싱글턴 객체를 가져오는 3개의 쓰레드를 실행시켰을때 반환되는 싱글턴 객체를 출력해보자.

```java
public class TestThread extends Thread {
    @Override
    public void run() {
        LazyInit singleton = LazyInit.getInstance();
        System.out.println(singleton);
    }
}
```

```java
public static void main(String[] args) {
    TestThread t1 = new TestThread();
    TestThread t2 = new TestThread();
    TestThread t3 = new TestThread();

    t1.start();
    t2.start();
    t3.start();
}
```

실행 결과 맨 아래 두번째 싱글턴 객체가 생성되었다.
```
> Task :Java.main()
com.study.sj.singleton.LazyInit@2a428578
com.study.sj.singleton.LazyInit@2a428578
com.study.sj.singleton.LazyInit@37470dd0 --> 두번쨰 싱글턴 객체
```

### Thread Safe Initialization
위 Lazy Initialization 방식에서 발생한 thread unsafety 문제를 해결하기 위해 싱글톤 객체를 생성하고 반환하는 메소드를 synchronized 메소드로 수정할 수 있다.

```java
public class ThreadSafeInit {
    private static ThreadSafeInit instance;

    private ThreadSafeInit() {
    }

    public static synchronized ThreadSafeInit getInstance() {
        if(instance == null) {
            instance = new ThreadSafeInit();
        }

        return instance;
    }
}
```

위와 같이 수정했을 경우 다시 여러 쓰레드에서 싱글턴 객체를 가져오는 메소드를 테스트 해보자.

```java
public class TestThread extends Thread {
    @Override
    public void run() {
        ThreadSafeInit singleton = ThreadSafeInit.getInstance();
        System.out.println(singleton);
    }
}
```

```java
public static void main(String[] args) {
    TestThread t1 = new TestThread();
    TestThread t2 = new TestThread();
    TestThread t3 = new TestThread();

    t1.start();
    t2.start();
    t3.start();
}
```

실행 결과 최초로 생성된 싱글톤 객체가 일관되게 반횐되는 모습을 확인할 수 있다.
```
com.study.sj.singleton.ThreadSafeInit@37470dd0
com.study.sj.singleton.ThreadSafeInit@37470dd0
com.study.sj.singleton.ThreadSafeInit@37470dd0
```

결론적으로 Thread Safe Initialization 방식은 thread unsafety 문제를 해결했지만 싱글톤 객체를 가져오는 메소드를 호출할때마다 synchronized 메소드로 인해 대기 쓰레드가 발생하여 성능이 느려진다는 단점이 있다.

### Double Checked Locking
Thread Safe Initialization 방식에서 synchronized 블록을 최초로 객체를 생성하는 시점에만 적용해 임계 구역을 축소하여 성능을 개선시킨 방법이다. 싱글톤 객체의 null 체크 구문이 이중으로 되어있어 Double Checked Locking이라고 부른다. 밖에서 하는 체크는 이미 객체가 존재할 경우 빠르게 반환하기 위한 목적이며 안에서 하는 체크는 객체가 최초로 생성할때 단 한개의 인스턴스만 생성하도록 보장하기 위함이다.

또한 `instance == null`을 확인하는 구문이 synchronized 블록 밖에 있기 때문에 volatile 키워드를 사용해 메인 메모리에서 변수를 동기화하여 캐시와 메인 메모리의 데이터가 불일치하는 경우를 예방했다.

```java
public class DoubleCheckLockInit {
    private static volatile DoubleCheckLockInit instance;

    private DoubleCheckLockInit() {
    }

    public static DoubleCheckLockInit getInstance() {
        if (instance == null) {
            synchronized (DoubleCheckLockInit.class) {
                if (instance == null) {
                    instance = new DoubleCheckLockInit();
                }
            }
        }

        return instance;
    }
}
```
위 방법은 제대로 작동하려면 volatile 키워드가 필요하다 하지만 volatile 키워드는 자바 1.5 버전 이후에 등장되었다. 따라서 하위 버전과 호환이 되지 않는다는 문제점이 있다.

### Bill Pugh Solution (LazyHolder) 
클래스안에 static inner class(Holder)를 두어 싱글톤 객체를 생성하는 방법이다. 내부 클래스를 static으로 선언함으로써 바깥 클래스가 참조되어도 내부 클래스는 메모리에 로드되지 않는다.

외부 클래스의 getInstance 메소드를 호출할때 내부 클래스가 로딩되면서 올라가면서 싱글톤 객체의 lazy loading이 가능하다는 장점이 있다.

여러개의 쓰레드가 인스턴스를 생성해도 클래스 로딩는 한번만 일어난다는 특징을 활용해 thread-safe하다는 성질을 가지고 있다.
```java
public class LazyHolder {
    private LazyHolder() {

    }

    private static class Holder {
        private static final LazyHolder INSTANCE = new LazyHolder();
    }

    public static LazyHolder getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Enum
컴파일러에 의해서 한번 실행되는 enum 클래스 생성자의 특성을 이용해 싱글톤 패턴을 구현한 기법이다. 따라서 thread safe하다.
```java
public enum EnumSingleton {
    INSTANCE;

    public static EnumSingleton getInstance() {
        return INSTANCE;
    }
}
```

`java.lang.Enum` 클래스 외에 상속을 허용하지 않는다는 단점과, 싱글톤 객체를 사용하지 않아도 메모리에 미리 생성되기 때문에 리소스의 낭비가 발생한다는 단점이 있다.