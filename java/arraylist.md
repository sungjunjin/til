# ArrayList가 내부적으로 배열을 확장하는 방법
    
```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 나머지 생략

    /**
        * The array buffer into which the elements of the ArrayList are stored.
        * The capacity of the ArrayList is the length of this array buffer. Any
        * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
        * will be expanded to DEFAULT_CAPACITY when the first element is added.
        */
    transient Object[] elementData; // non-private to simplify nested class access
```
위 코드는 ArrayList의 일부이다. ArrayList는 내부에 Object 타입의 배열을 사용한다. 배열은 `elementData`라는 변수명으로 다음과 같이 선언되어 있다. 실제로 다루는 데이터들이 저장되는 영역이다.

## 배열의 초기화
ArrayList는 총 3개의 생성자를 가지고 있다. 내부 배열(elementData)의 용량 관점에서 각각 정리하자면 다음과 같다.

#### public ArrayList()
빈 배열로 내부 배열을 초기화 한다.

```java
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; // {}
}   
```

#### public ArrayList(int initalCapacity)
내부 배열을 매개변수로 넘어온 initialCapacity만큼의 공간으로 초기화 한다. initialCapacity가 0인 경우 빈 배열로 초기화 한다.

```java
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                            initialCapacity);
    }
}
```

#### public ArrayList(Collection<? extends E> c)
내부 배열을 매개변수로 넘어온 컬렉션의 길이만큼의 공간으로 초기화 한다. 컬렉션의 길이가 0인 경우 빈 배열로 초기화 한다.
```java
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}
```

## 배열의 확장
ArrayList는 배열에 데이터들이 들어갈때마다 동적으로 배열의 크기가 확장된다. 내부의 private으로 선언되어 있는 grow 메소드를 오버로딩하여 사용한다.

전체적인 과정을 요약하자면 다음과 같다.
1. 현재 배열의 크기 계산
2. 현재 배열의 크기를 기반으로 늘려야할 공간을 계산
3. 계산된 추가 공간만큼의 새로운 배열을 생성
4. 새로운 배열에 기존 데이터 복사
5. 새로운 배열에 데이터 추가

### 단건 추가
```java
private void add(E e, Object[] elementData, int s) {
    if (s == elementData.length)
        elementData = grow();
    
    // 나머지 생략
}
```
add 메소드 내부에서 `현재 데이터의 개수 == 기존 내부 배열의 길이` 인 경우에 grow 메소드를 호출한다.

#### grow()
```java
private Object[] grow() {
    return grow(size + 1);
}
```
minCapacity 매개변수에 `현재 데이터의 개수 + 1`의 값이 들어간다. 즉, 단건의 데이터가 추가될 때 배열이 가지고 있어야 할 최소 용량을 의미한다. grow 메소드 내부에 구현된 코드는 다음과 같다.

```java
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
```

oldCapacity에 내부 배열의 길이를 할당한다. 길이가 빈 배열(DEFAULTCAPACITY_EMPTY_ELEMENTDATA)이 아니거나, 현재 내부 배열의 길이가 0보다 클 경우 ArraySupport.newLength 메소드를 호출해 확장할 새로운 배열의 길이를 계산한다. 해당 메소드의 매개변수의 값들은 다음과 같다
- 기존 배열의 길이
- (현재 데이터 개수 + 1) - (기존 배열의 길이) = 1
- 기존 배열의 길이 / 2 

기존 내부 배열이 빈 배열일 경우 기본 용량(10)과 최소 용량의 최대값의 크기로 확장될 배열의 크기가 정해진다.

#### ArraysSupport.newLength

```java
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {
    // preconditions not checked because of inlining
    // assert oldLength >= 0
    // assert minGrowth > 0

    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // might overflow
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {
        return prefLength;
    } else {
        // put code cold in a separate method
        return hugeLength(oldLength, minGrowth);
    }
}
```

확장될 배열의 길이는 `prefLength = 기존 내부 배열의 길이 + (기존 내부 배열의 길이 / 2)` 가 된다. 최종적으로 기존 저장공간의 **1.5배** 정도의 새로운 배열 크기가 정해진다.

### 다건 추가
```java
public boolean addAll(Collection<? extends E> c) {
    // 일부 생략
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew);
}
```
addAll 메소드 내부에서 `추가하고자 하는 데이터의 수 > (기존 내부 배열의 길이 - 기존 배열의 데이터 수)` 인 경우에 grow 메소드를 호출한다.

#### grow()
minCapacity 매개변수에 `기존 배열의 데이터 수 + 추가할 데이터의 개수`의 값이 들어간다. 구현 코드는 다음과 같다.
```java
private Object[] grow(int minCapacity) {
    int oldCapacity = elementData.length;
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        int newCapacity = ArraysSupport.newLength(oldCapacity,
                minCapacity - oldCapacity, /* minimum growth */
                oldCapacity >> 1           /* preferred growth */);
        return elementData = Arrays.copyOf(elementData, newCapacity);
    } else {
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];
    }
}
```

oldCapacity에 내부 배열의 길이를 할당한다. 길이가 빈 배열(DEFAULTCAPACITY_EMPTY_ELEMENTDATA)이 아니거나 0보다 클 경우 ArraySupport.newLength 메소드를 호출해 확장할 새로운 배열의 길이를 계산한다. 해당 메소드의 매개변수의 값들은 다음과 같다

- 기존 배열의 길이
- (기존 배열의 데이터 수 + 추가할 데이터의 개수) - (기존 내부 배열의 길이) = 최소 필요한 용량
- 기존 배열의 길이 / 2

기존 내부 배열이 빈 배열일 경우 기본 용량(10)과 추가하고자 하는 데이터의 개수의 최대값의 크기로 확장될 배열의 크기가 정해진다.

#### ArraysSupport.newLength
```java
public static int newLength(int oldLength, int minGrowth, int prefGrowth) {
    // preconditions not checked because of inlining
    // assert oldLength >= 0
    // assert minGrowth > 0

    int prefLength = oldLength + Math.max(minGrowth, prefGrowth); // might overflow
    if (0 < prefLength && prefLength <= SOFT_MAX_ARRAY_LENGTH) {
        return prefLength;
    } else {
        // put code cold in a separate method
        return hugeLength(oldLength, minGrowth);
    }
}
```
newLength 메소드의 매개변수로 넘어온 인자를 정리하자면 최종적으로 확장될 배열의 길이는 추가할 데이터의 개수에 따라 다르다.
- 최소 필요한 용량이 기존 배열의 길이 / 2보다 크면 : 기존 배열의 길이 +  최소 필요한 용량
- 최소 필요한 용량이 기존 배열의 길이 / 2보다 작으면 : 기존 배열의 길이 + 기존 배열의 길이 / 2

### hugeLength
기존 배열을 확장할때 확장할 배열의 크기가 SOFT_MAX_ARRAY_LENGTH 보다 크거나 음수이면 내부 hugeLength 메소드가 호출된다.

```java
public static final int SOFT_MAX_ARRAY_LENGTH = Integer.MAX_VALUE - 8;
```
SOFT_MAX_ARRAY_LENGTH 상수는 배열이 가질수 있는 공간에 대한 최대값을 가진다. Integer.MAX_VALUE에 근접한 값으로 배열의 크기를 할당할 경우 일부 JVM에서 OOM을 발생시킬 수 있기 때문에 Integer.MAX_VALUE에서 8을뺀 값으로 설정되어 있다.

```java
private static int hugeLength(int oldLength, int minGrowth) {
    int minLength = oldLength + minGrowth;
    if (minLength < 0) { // overflow
        throw new OutOfMemoryError(
            "Required array length " + oldLength + " + " + minGrowth + " is too large");
    } else if (minLength <= SOFT_MAX_ARRAY_LENGTH) {
        return SOFT_MAX_ARRAY_LENGTH;
    } else {
        return minLength;
    }
}
```
(기존 배열의 길이 + 최소 필요한 용량)가 int의 표현범위를 넘어가 오버플로우가 발생하게 되면 OutOfMemoryError 예외를 발생시킨다. 그 외에 경우에는 SOFT_MAX_ARRAY_LENGTH 혹은 확장될 배열의 크기를 반환한다.
