# NIO

JDK 1.4부터 NIO (New IO)가 추가되었다. 기존 I/O의 스트림을 사용하지 않고 채널(Channel)과 버퍼(Buffer)를 사용한다. 

Channel과 Buffer를 활용해 파일을 읽고 쓰는 예제는 다음과 같다
```java
public class NIO {
    public void write(String data, String fileName) {
        try (
            // FileOutputStream의 getChannel 메소드를 호출해 FileChannel 객체를 가져온다
            FileChannel channel = new FileOutputStream(fileName).getChannel();
        ) {
            // 데이터를 바이트 배열로 변환한다
            byte[] byteData = data.getBytes();

            // ByteBuffer의 wrap 메소드를 활용해 바이트 배열을 Buffer로 저장한다
            ByteBuffer buffer = ByteBuffer.wrap(byteData);

            // 채널의 write 메소드에 buffer 객체를 넘겨주면 buffer에 파일에 쓰게 된다
            channel.write(buffer);

        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    public void read(String fileName) {
        try (
            // FileInputStream의 getChannel 메소드를 호출해 FileChannel 객체를 가져온다
            FileChannel channel = new FileInputStream(fileName).getChannel()
        ) {
            // ByteBuffer의 allocate 메소드는 매개 변수만큼의 데이터가 저장되는 공간을 할당하고 Buffer 객체를 만든다
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            // 채널의 read 메소드에 buffer 객체를 넘겨주면 buffer에 데이터가 담기기 시작한다
            channel.read(buffer);

            // flip 메소드를 호출해 buffer에 맨 앞부분으로 이동한다
            buffer.flip();

            // get 메소드를 사용해 데이터가 남아있는지 확인하면서 한 바이트씩 데이터를 읽어 출력한다
            while (buffer.hasRemaining()) {
                System.out.println((char) buffer.get());
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```
main 메소드에서 간단한 문자열 파일을 저장하고 읽어보자.
```java
public static void main(String[] args) {
    String fileName = "./test";
    String data = "abcdefg";

    NIO nio = new NIO();
    nio.write(data, fileName);

    nio.read(fileName);
}
```

실행 결과
```
> Task :Java.main()
a
b
c
d
e
f
g
```

## Buffer
NIO에서는 데이터를 Buffer를 통해 처리한다. 데이터를 담는 메모리 영역으로 ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer 등이 존재한다.

버퍼는 데이터를 담거나 읽는 작업을 수행하면 현재의 "위치"가 이동한다. NIO는 데이터를 무조건 버퍼에 저장을 하기 때문에 버퍼를 사용하면 데이터의 위치를 이동해가면서 필요한 부분만 읽고 쓸 수 있다. 

Buffer의 속성이나 상태를 확인하기 위한 메소드의 목록은 다음과 같다.
| 메소드 | 설명 |
| --- | --- |
| int capacity() | 버퍼에 담을 수 있는 크기를 반환한다 |
| int limit() | 버퍼에서 읽거나 쓸 수 없는 첫 위치를 반환한다 |
| int position() | 현재 버퍼의 위치를 반환한다 |


Buffer의 위치를 변경할 수 있는 메소드의 목록은 다음과 같다.
| 메소드 | 설명 |
| --- | --- |
| Buffer flip() | limit 값을 현재 position으로 지정한 후, position을 0(가장 앞)으로 이동 |
| Buffer mark() | 현재 position을 mark |
| Buffer reset() | 버퍼의 position을 mark한 곳으로 이동 |
| Buffer rewind() | 현재 버퍼의 position을 0으로 이동 |
| int remaining() | limit - position 계산 결과를 리턴 |
| boolean hasRemaining() | position 과 limit 값에 차이가 있을 경우 true를 리턴 |
| Buffer clear() | 버퍼를 지우고 현재 position을 0으로 이동하며, limit 값을 버퍼의 크기로 변경 |

```java
public class NIO {
    public void checkBuffer() {
        try {
            IntBuffer buffer = IntBuffer.allocate(1024);

            for(int i=0;i<100;i++) {
                buffer.put(i);
            }

            System.out.println("Buffer capacity : " + buffer.capacity());
            System.out.println("Buffer limit : " + buffer.limit());
            System.out.println("Buffer position : " + buffer.position());

            buffer.flip();
            System.out.println("Buffer Flip ----------- ");

            System.out.println("Buffer capacity : " + buffer.capacity());
            System.out.println("Buffer limit : " + buffer.limit());
            System.out.println("Buffer position : " + buffer.position());

            for(int i=0;i<100;i++) {
                buffer.put(i);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
public static void main(String[] args) {
    NIO nio = new NIO();
    nio.checkBuffer();
}
```

실행 결과
```
> Task :Java.main()
Buffer capacity : 1024
Buffer limit : 1024
Buffer position : 100
Buffer Flip ----------- 
Buffer capacity : 1024
Buffer limit : 100
Buffer position : 0
```

