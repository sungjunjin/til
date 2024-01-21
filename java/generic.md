# Generic

제네릭은 클래스, 인터페이스나 메소드를 사용할 때 내부 데이터에 대한 타입을 외부(사용하는 입장)에서 지정하는 방법이다. 제네릭을 활용하면 실행 환경에서 타입과 관련된 실수를 방지할 수 있다. 

## 제네릭을 사용하기 전
아래와 같이 CastingDto 클래스는 Object 타입의 멤버 변수를 getter / setter와 함께 가지고 있다. 
```java
public class CastingDto {
    private Object object;

    public Object getObject() {
        return object;
    }

    public void setObject(Object object) {
        this.object = object;
    }
}
```

CastingDto를 메인 메소드에서 활용해보자. 멤버 변수의 타입이 Object이므로 아무 타입이든 저장할 수 있다. 간단한 예시로 String, StringBuilder 타입의 데이터를 setObject 메소드로 초기화하고 getObject 메소드로 가져와보자.

```java
public static void main(String[] args) {
    CastingDto stringDto = new CastingDto();
    stringDto.setObject("Hi");
    String str = (String) stringDto.getObject();

    CastingDto stringBuilderDto = new CastingDto();
    stringBuilderDto.setObject(new StringBuilder());
    StringBuilder sb = (StringBuilder) stringBuilderDto.getObject();
}
```

CastingDto 클래스의 멤버 변수는 Object 타입이라 setObject 메소드의 매개변수에는 아무 타입이나 들어갈 수 있다. 하지만 getObject 메소드를 사용할때는 Object 타입을 반환한다. 따라서 반환되는 객체에 대한 타입을 정확히 알고 형변환을 일일이 해줘야 하는 번거로움이 있다. 

CastingDto를 사용하는 프로그래머가 내부 멤버변수 object의 타입을 직접 결정할 수 없을까? 제네릭을 사용하면 가능하다.

## 제네릭 클래스

제네릭 문법을 적용하고 하는 클래스 이름 옆에 `<T>`를 사용한다. 여기서 T는 **타입 파라미터**라고 한다. "내부 멤버 변수 object의 타입은 사용자가 정해라" 라는 의미를 가진다고 볼 수 있다. 

제네릭에서 <>안에 들어 갈 수 있는 자료형은 **참조 자료형**뿐이다 (Integer, Double, Float등의 wrapper 클래스가 존재하는 이유 증 하나다).

```java
public class CastingGenericDto<T> {
    private T object;

    public T getObject() {
        return this.object;
    }

    public void setObject(T object) {
        this.object = object;
    }
}
```

T외에도 E, K, V 등과 같이 내부 데이터의 속성에 따라 자바에서 따르고 있는 컨벤션을 따른다.
- E : 요소 자바 컬렉션에서 주로 사용된다
- K : 키
- N : 숫자
- T : 타입
- V : 값
- S,U,V : 두 번째, 세 번째, 네 번쨰에 선언된 타입

### 타입 파라미터와 구체화 과정
String 타입으로 GenericDto를 선언하면 타입 파라미터로 String 타입이 전달되고 내부에서 T 타입을 활용하는 모든 멤버 변수와 메소드에 전파가 된다. 이를 구체화 (Specialization) 이라고 한다.

예를들어 아래와 같이 타입 변수 T에 String 타입을 전달하면
```java
GenericDto<String> stringGenericDto = new GenericDto<>();
```

다음과 같이 GenericDto 클래스 내부에 타입이 구체화된다.
```java
public class CastingGenericDto<String> {
    private String object;

    public String getObject() {
        return this.object;
    }

    public void setObject(String object) {
        this.object = object;
    }
}
```

작성한 제네릭 클래스를 메인 메소드에서 다시 활용해보자.
```java
public static void main(String[] args) {
    GenericDto<String> stringGenericDto = new GenericDto<>();
    stringGenericDto.setObject("Hi");
    String str = stringGenericDto.getObject();

    GenericDto<StringBuilder> stringBuilderGenericDto = new GenericDto<>();
    stringBuilderGenericDto.setObject(new StringBuilder());
    StringBuilder sb = stringBuilderGenericDto.getObject();
}
```

GenericDto의 <>안에 내부 멤버 변수로 활용될 object의 타입을 명시해주면 내부 멤버 변수는 해당 타입으로 지정된다. 따라서 getter 메소드를 사용할때 해당 타입으로 반환이 되므로 별도의 형변환 과정이 필요가 없다.

## 복수 타입 파라미터
복수 타입 파라미터를 가진 제네릭 클래스도 선언할 수 있다. 위 예시로 들었던 GenericDto에 타입 파라미터를 2개 받는 제네릭 클래스로 변경하여 String, StringBuilder 타입을 가지는 제네릭 클래스를 선언해보자.

```java
public class GenericDto<T, U> {
    private T firstObject;
    private U secondObject;

    public T getFirstObject() {
        return this.firstObject;
    }

    public U getSecondObject() {
        return this.secondObject;
    }

    public void setFirstObject(T firstObject) {
        this.firstObject = firstObject;
    }

    public void setSecondObject(U secondObject) {
        this.secondObject = secondObject;
    }
}
```

이를 메인 메소드에서 활용하면 다음과 같다.
```java
public static void main(String[] args) {
    GenericDto<String, StringBuilder> genericDto = new GenericDto<>();

    genericDto.setFirstObject("Hi");
    genericDto.setSecondObject(new StringBuilder());

    String str = genericDto.getFirstObject();
    StringBuilder sb = genericDto.getSecondObject();
}
```

## 제네릭 메소드
제네릭 메소드는 메소드의 **선언부**에 타입 파라미터가 들어간 형식으로 선언할 수 있다.
```java
public class GenericDto<T> {
    private T object;

    public T getObject() {
        return this.object;
    }

    public void setObject(T object) {
        this.object = object;
    }
    
    // 제네릭 메소드
    public <T> Class getObjectClass(T object) {
        return object.getClass();
    }
}
```

제네릭 메소드에 사용되는 타입 파라미터는 별도로 동작을 한다. 따라서 위 코드에서 사용하는 타입 파라미터는 모두 T이지만, 제네릭 메소드 getObjectClass의 타입 파라미터는 클래스에서 사용하는 타입 파라미터가 아니라 제네릭 메소드를 호출할때 전달받은 타입 파라미터로 동작한다. 아래 예시로 확인해보자.

```java
public static void main(String[] args) {
    GenericDto<String> genericDto = new GenericDto<>();

    // 메소드 타입 파라미터에 Integer 전달, 타입 파라미터는 메소드 매개변수의 타입으로 컴파일러가 추론이 가능하므로 생략할 수 있다.
    Class result = genericDto.getObjectClass(Integer.valueOf(10));
    System.out.println(result);
}
```

실행 결과
```
> Task :Java.main()
class java.lang.Integer
```

위 예시처럼 클래스 타입 파라미어와 메소드 타입 파라미터는 별도로 동작하는 모습을 확인할 수 있다.

## 제네릭 타입 범위 한정하기
제네릭 클래스나 메소드에 전달되는 타입 파라미터의 범위를 한정할 수 있다. 예를들어 Car 클래스를 상속받은 Bus 클래스가 있다고 가정해보자.

```java
public class Car {
    protected String name;

    public Car(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Car name = " + this.name;
    }
}

```

```java
public class Bus extends Car{
    public Bus(String name) {
        super(name);
    }

    @Override
    public String toString() {
        return "Bus name = " + this.name;
    }
}
```

 제네릭 클래스의 <> 안에 extends 키워드를 사용하여 타입 파라미터를 해당 클래스와 하위 타입의 클래스 타입으로 제한할 수 있다. 이를 제한된 타입 매개변수 (Bounded Type Parameter)라고 한다.
 ```java
 public class GenericDto<T extends Car> {
    private T object;

    public T getObject() {
        return this.object;
    }

    public void setObject(T object) {
        this.object = object;
    }
}
```

메인 메소드에서 String 타입 파라미터를 전달해 위 제네릭 클래스의 객체를 생성하려고 하면 다음과 같은 컴파일러 에러가 발생한다.
```java
public static void main(String[] args) {
    // 가능
    GenericDto<Car> carGenericDto = new GenericDto<>();
    GenericDto<Bus> busGenericDto = new GenericDto<>();

    // 불가능
    GenericDto<String> stringGenericDto = new GenericDto<>();
}
```

실행 결과
```
error: type argument String is not within bounds of type-variable T
        GenericDto<String> stringGenericDto = new GenericDto<String>();
```

## 제네릭의 형변환과 와일드카드
제네릭은 타입 매개변수로 구체화된 내부 요소들간의 형변환 문제를 일으키지 않기 위해 하위 타입간의 형변환을 금지하고 있다. 

```java
public static void main(String[] args) {
    // 가능
    Object[] objects = new String[10];

    for(Object e: objects) {
        System.out.println(e);
    }

    // 불가능
    List<Objects> objectsList = new ArrayList<String>();

    for(Object e: objectsList) {
        System.out.println(e);
    }
}
```

제네릭간의 형변환이 가능하게 하려면 와일드카드 문법을 사용해야 한다. 와일드 카드 타입이란 어떤 타입이 제네릭 타입이 되더라고 상관 없다는 의미를 가지고 있다. <>안에 ?을 적어주면 와일드 카드 타입을 타입 매개변수로 사용한다는 뜻이다.

```java
public static void main(String[] args) {
    // 이제 가능
    List<?> objectsList = new ArrayList<String>();

    for(Object e: objectsList) {
        System.out.println(e);
    }
}
```

와일드카드 타입 또한 extends, super 키워드를 사용해 와일드카드 타입의 매개변수형을 제한할 수 있다.

- ? extends 타입 : 해당 타입이나 **하위** 타입만 가능
- ? super 타입 : 해당 타입이나 **상위** 타입만 가능

```java
List<? extends Object> list = new ArrayList<String>(); // Object 클래스나 Object 클래스의 하위 타입만 가능
List<? super String> list2 = new ArrayList<Object>(); // String 클래스나 String 클래스의 상위 타입만 가능
```

## 제네릭의 제한사항

제네릭 타입의 객체는 생성이 불가능하다.
```java
public <T> void makeClass() {
    T t = new T(); // 불가능
}
```

static 멤버에 제네릭 타입이 올 수 없다. 객체를 생성하기도 전에 static 멤버의 타입을 확정할수는 없기 때문이다.
```java
public class Car {
    public static T staticGeneric; // 불가능

    protected String name;

    public Car(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Car name = " + this.name;
    }
}
``` 