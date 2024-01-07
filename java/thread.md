# Thread

## Thread 란
쓰레드는 프로세스안에서 실행되는 작업의 단위이다. 여러개의 작업을 동시에 수행하여 빠른 처리로 프로그램의 성능을 향상시키고자 고안되었다. 자바 프로세스가 점유하는 물리적인 메모리는 적어도 32 ~ 64MB 정도가 된다. 반면에 쓰레드는 1MB 이내의 메모리만 필요해서 "경량 프로세스"라고도 부른다.

## 자바에서 Thread를 생성하는 방법

자바에서 Thread를 생성하는 방법은 Runnable 인터페이스 구현과 Thread 클래스 상속이 있다.

### Runnable 인터페이스 구현
        
```java
public class MyRunnable implements Runnable{
    @Override
    public void run() {
        System.out.println("This is MyRunnable run() method");
    }
}
```
`java.lang` 패키지의 Runnable 인터페이스를 구현한다. run 메소드 내부에 작업에 실행할 코드를 작성 해준다.

```java
public static void main(String[] args) {
    MyRunnable runnable = new MyRunnable();
    Thread myRunnableThread = new Thread(runnable);

    myRunnableThread.start();
}
```
Thread 클래스의 생성자에 Runnable 객체를 매개변수로 Thread 객체를 생성한다.
Thread 객체의 start 메소드 호출하면 쓰레드가 실행된다.

### Thread 클래스 상속
        
```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("This is MyThread: run() method");
    }
}
```
java.lang 패키지의 Thread 클래스를 상속받고 run 메소드 내부에 작업에 실행할 코드를 오버라이딩 해준다.

```java
public static void main(String[] args) {
    MyThread myThread = new MyThread();
    myThread.start();
}
```
객체를 생성하고 start 메소드 호출한다.

### 정리
두 클래스 공통적으로 run 메소드 내부에서 쓰레드에서 실행될 코드를 작성해야 한다. 또 run 메소드에 코드를 작성해도 쓰레드를 시작하기 위해서는 start 메소드를 호출해야 한다. 

자바는 다중상속이 불가능 하므로 Thread를 상속한 클래스에서 다른 부모 클래스를 상속받아야 하는 상황이면 Runnable 인터페이스를 구현하면 된다.

## 쓰레드 예시

### 쓰레드는 서로 독립적으로 실행된다
main 메소드에서 위에 예시로 들었던 쓰레드들을 각각 5개씩 생성하고 실행해보자.
```java
public static void main(String[] args) {
    for(int i=0;i<5;i++) {
        MyThread myThread = new MyThread();
        MyRunnable myRunnable = new MyRunnable();

        myThread.start();
        new Thread(myRunnable).start();
    }

    System.out.println("Main Method Finished");
}
```

main 메소드를 실행시켜보면 결과는 다음과 같다.
```
This is MyThread: run() method
This is MyThread: run() method
This is MyRunnable run() method
This is MyRunnable run() method
This is MyThread: run() method
This is MyRunnable run() method
This is MyThread: run() method
This is MyRunnable run() method
This is MyThread: run() method
Main Method Finished
This is MyRunnable run() method
```

코드 상으로는 생성된 myThread, MyRunnable 객체가 for 루프를 돌면서 번갈아 가면서 run 메소드가 실행되도록 작성했지만 main 메소드를 반복 실행해보면 결과가 그때마다 다르다 심지어 메인 메소드가 종료되어도 마지막 남은 MyRunnable 스레드가 실행되는것도 확인할 수 있다. 

start 메소드를 호출하면 jvm은 하나의 쓰레드를 추가하고 run 메소드 내부에 구현되어 있는 코드를 실행한다. **각각의 스레드는 서로의 종료여부에 상관없이 독립적으로 실행된다**. 


### 모든 쓰레드가 종료되어야 JVM이 종료된다
무한 반복문을 돌면서 1초마다 현재 밀리초를 출력하는 클래스를 만들어보자. 이 클래스는 Thread 클래스를 상속받아 별도의 쓰레드로 동작한다.
```java
public class EndlessThread extends Thread{
    @Override
    public void run() {
        while(true) {
            try {
                System.out.println(System.currentTimeMillis());
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```
```java
public static void main(String[] args) {
    EndlessThread endlessThread = new EndlessThread();
    endlessThread.start();
}
```
```
> Task :Java.main()
1704618636724
1704618637725
1704618638726
1704618639729
.
.
.
```
main 메소드로 쓰레드를 생성해 실행시켜보면 강제로 자바 프로세스를 종료하지 않는 이상은 무한으로 1초마다 밀리초가 출력되는 것을 확인할 수 있다. 


예외로 데몬 쓰레드라는 것이 있다. 쓰레드가 실행되기전에 setDaemon 메소드를 통해 데몬 쓰레드 여부를 지정할 수 있다
```java
public static void main(String[] args) {
    EndlessThread endlessThread = new EndlessThread();
    endlessThread.setDaemon(true);
    endlessThread.start();
}
```

위 main 메소드를 실행해보면 아까와 다르게 jvm 프로세스가 바로 종료되는 모습을 확인 할 수 있다.

데몬 쓰레드로 지정해두면 자바 프로세스가 해당 쓰레드의 종료 여부와 상관없이 종료 될 수 있다. 데몬 쓰레드는 자신을 제외한 모든 일반 쓰레드가 종료되면 그제서야 종료된다. 이런 특성 때문에 데몬 쓰레드는 모니터링, 자동 저장과 같은 보조적인 역할을 하는 쓰레드에 사용된다.

## Thread를 통제하는 메소드들