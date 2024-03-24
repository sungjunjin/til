# User mode와 Kernal mode
![user-kernal](./img/user_kernal.png)

컴퓨터는 소프트웨어와 하드웨어로 구성된다. 그 중에서 소프트웨어는 user mode와 kernal mode로 나뉘어진다. 

유저 모드에서 각각의 어플리케이션은 OS로부터 독립적인 메모리 공간을 할당받은 프로세스로써 동작한다. 어플리케이션은 System Call Interface를 활용해 Kernal mode 내부의 소프트웨어와 통신한다.

