---
layout: post
title: "[OS] I/O Device Drivers"
author: "Sara Han"
categories : OperatingSystem
comments : true
---


```
[DISCLAIMER]
서울대학교 홍성수 교수님의 운영체제 강좌 필기입니다. 
본 PPT 캡처가 문제될 시 즉시 삭제하겠습니다. 
```

* [SNU etl 에서 수강 가능합니다](http://etl.snu.ac.kr/)

# Lecture 20 : IO Devices and Device Drivers 

* 기업체에서 새로운 장치를 만들면 회사만의 IO 를 잘 구동할 수 있도록 만든다. 
* 기본적으로 디바이스들이 어떻게 동작하고 운영체제가 그것을 어떻게 받아주는건지 
* Driver 들이 어떤 소프트웨어인지 공부한다. 



### I/O Hardware 

![{{site.baseurl}}/assets/img/oslec20/image-20200613144408092]({{site.baseurl}}/assets/img/oslec20/image-20200613144408092.png)



![{{site.baseurl}}/assets/img/oslec20/image-20200613144440117]({{site.baseurl}}/assets/img/oslec20/image-20200613144440117.png)

* 하드웨어를 배울 때 bus 를 중심으로 설명했었다. 
* 컴퓨터 시스템에는 bus 가 여러개로 나누어져 있다. IO 디바이스들을 위한 IO 버스도 있다. 
* PCI 같은 고급 사양의 버스로 교체되었다. 
* CPU도 물려있고, 여러가지 IO 카드들, 디바이스들이 몰려있다. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613144551572]({{site.baseurl}}/assets/img/oslec20/image-20200613144551572.png)

* CPU와 소통할 수 있도록 인터페이스를 갖고 있다. (표준 IO 버스 규격에 맞는 인터페이스)
* 외부 장치와, 사람과 interaction 하기 위한 외부 인터페이스가 있고 이는 장치의 목적에 따라 다르다. 
* 내부에는 IO 디바이스 혹은 컨트롤러에는 여러개의 레지스러들이 들어간다. 
  * Status Register : 장치의 상태를 보여주는 register 
  * Control Register : 장치의 동작을 결정하는 register (제어 flag 들을 갖고 있다)
  * Data Register : 입출력할 데이터를 잠시 보관하는 register 
  * 컴퓨터 CPU가 IO 디바이스에게 명령하는 명령어 레지스터들도 있음 ... 
* 레지스터가 있으면 어떤 operation 을 할 수 있어야 하는가? 
  * 응용 프로그램이 register 에 read write 를 할 수 있어야 한다. 
  * IO 디바이스에 있는 register 에 접근하는 방법은 2가지가 있다. 
    * Memory Mapped IO 
    * Port Mapped IO : 데이터 레지스터들을 port 라 부르기도 한다. 

1. 여기 레지스터들은 자기만의 어드레스를 갖고 있는데, memory mapped 에서는 physical 주소를 할당하게 된다. 

2. port mapped 에서는 별도의 IO 주소를 제공해준다.



![{{site.baseurl}}/assets/img/oslec20/image-20200613145036681]({{site.baseurl}}/assets/img/oslec20/image-20200613145036681.png)

결국 운영체제가 IO 디바이스를 컨트롤 한다는 것은 IO 디바이스의 레지스터들을 프로그램으로 컨트롤 하는 것이다. 

이것을 프로그래밍 하기 위해서 운영체제는 BUS 의 어드레스 라인에다가 특정 레지스터에 주소를 보내고 그곳에 read write 를 한다. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613145147711]({{site.baseurl}}/assets/img/oslec20/image-20200613145147711.png)

통상적으로 IO 디바이스는 여러가지 종류로 구분한다. 

* 캐릭터 : 입력과 출력을 캐릭터 단위로 한다 
  * 마우스, 터미널
* 블럭 : 입력돠 출력을 블럭 단위로 한다 (임의의 사이즈)
  * 블럭 사이즈는 페이지 사이즈와도 관련된다. 
  * 디스크, 플래시
* 네트워크 : 네트워크 인터페이스 카드 

이런 분류는 UNIX 운영체제에서 왔다. Unix 는 처음 만들 때 단순하게 만드는게 목적이었음. 따라서 File System 도 단순하고, File 도 단순하다. Unix 에서 File 은 named collection of byte stream 이다. 

Device modelling 을 단순화 하면서 위와 같이 세 가지로 특성을 나누게 되었다. 



**위의 대표적인 디바이스들을 어떻게 프로그래밍 하는지 알아보자.** 

* Mouse (Optical, USB 마우스 - 제조사 Microsoft)

![{{site.baseurl}}/assets/img/oslec20/image-20200613145555555]({{site.baseurl}}/assets/img/oslec20/image-20200613145555555.png)

* 운영체제가 잘 팔릴려면 마우스가 많아야 했는데, 아직 보편화되지 않은 상태라서 MS 가 만들기 시작했음 ... 
* 1999년 optical mouse 만들어짐. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613145826565]({{site.baseurl}}/assets/img/oslec20/image-20200613145826565.png)

* 옵티컬 마우스는 아래에 레이저와 센서를 갖고 있음. 초당 몇천개의 이미지를 캡처해서 DSP 로 보내고, motion estimation 을 함. 
* register 의 serial port 를 이용해서 3-4바이트의 정보를 보낸다. 움직인 coordinate 과 관련된 정보들을 보냄. 
* 이 data packet 이 CPU에 도착하면 인터럽트가 걸리고 마우스 디바이스 드라이버가 이 데이터를 읽어간다. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613145950329]({{site.baseurl}}/assets/img/oslec20/image-20200613145950329.png)

CMOS 센서가 모션 컨트롤러를 거쳐서 ... 디바이스 드라이버 구동, 애플리케이션이 정보를 취득. 

**마우스의 디바이스 드라이버가 패킷을 받았을 때 어떻게 운영체제가 작동하는지 알아보자**. 

![{{site.baseurl}}/assets/img/oslec20/image-20200613150032807]({{site.baseurl}}/assets/img/oslec20/image-20200613150032807.png)

* 백그라운드 어플리케이션이 마우스 동작을 기다리며 sleep 하고 있다. 
* 마우스가 이동하면, 이동한 상황을 packet 으로 드라이버에 보낸다. 
* 드라이버는 운영체제에게 마우스 이벤트(좌표 값들)의 존재를 알린다. 



**주목할 점 : 누군가 인터럽트를 EVENT 로 바꿔줘야 한다. -> 디바이스 드라이버가 함.** 

인터럽트에 user level semantics 를 부여하게 되는 것. 

* 이벤트와 된다면 OS 가 갖고 있는 이벤트 큐에 이것을 넣는다. 
  * mouse press, release ... 
* 이벤트에 해당하는 콜백 함수를 운영체제가 호출하게 됨. 
* 결국, 메뉴가 뜬다라고 하는 것은 콜백으로 인해 foreground application 으로 전해진다는 것. 



**Mouse Driver 의 역할 요약** 

* Mouse Interrupt 발생 시 Mouse 의 Data register 에서 값을 읽어오는 역할 
* Interrupt 를 Event 로 바꾸어 주는 역할 



![{{site.baseurl}}/assets/img/oslec20/image-20200613150544733]({{site.baseurl}}/assets/img/oslec20/image-20200613150544733.png)

두번째는 터미널에 대해서 보자. 

Mouse 는 인풋 디바이스, 터미널은 입출력 디바이스. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613150653622]({{site.baseurl}}/assets/img/oslec20/image-20200613150653622.png)

ref. 타이핑 하면 타입 한 캐릭터들이 디스플레이 되는 것. => 이것을 echo 라 한다. 

* 캐릭터 디바이스 : 캐릭터 단위로 입출력 된다. 
  * 키를 하나 누르면 인터럽트 발생, 캐릭터 값이 전송된다. 
  * 아스키라는 인코딩 값이 전달됨. 
  * 매우 느리다. 빠를 필요가 없음. 사람이 느리기 때문에... 



![{{site.baseurl}}/assets/img/oslec20/image-20200613150816478]({{site.baseurl}}/assets/img/oslec20/image-20200613150816478.png)

* 키보드 내부를 보면 두개의 레지스터가 있다. 
  * 키보드 데이터 레지스터 
  * 키보드 스테이터스 레지스터 

* 어떤 키가 프레스 되면 그 키가 데이터 레지스터에 (ready bit 이 켜져있을 때) 들어간다. 
* 인터럽트가 걸리면 디바이스 드라이버, 인터럽트 핸들러가 이 register 값을 읽어가고 
* ready bit 를 다시 0으로 바꾼다. 

![{{site.baseurl}}/assets/img/oslec20/image-20200613151007617]({{site.baseurl}}/assets/img/oslec20/image-20200613151007617.png)

Keyboard Driver 의 역할 요약 

* 키보드로부터 들어오는 각 Character 들을 Line Buffer 에 순서대로 저장 
* **End of Line Character** 가 들어오면 전체 Line Buffer 를 운영체제에게 전달 



디스플레이도 비슷함. 

* 데이터를 써놓고
* 디스플레이 
* ready bit 은 플로우 컨트롤 위해 사용 



![{{site.baseurl}}/assets/img/oslec20/image-20200613151126246]({{site.baseurl}}/assets/img/oslec20/image-20200613151126246.png)



## IO Hardware (2) 



### Disk 

블럭 디바이스 -> 데이터 전달이 블럭 단위로 이루어진다. 

![{{site.baseurl}}/assets/img/oslec20/image-20200613151328003]({{site.baseurl}}/assets/img/oslec20/image-20200613151328003.png)

* Disk 용어 
  * 하나의 회전축 - Spindle 이 있고, 여러장의 디스크가 스태킹 되어있다. 
  * 디스크위 위아래면은 다 자화 되어있어서 위 아래를 다 읽을 수 있다. 
  * 스핀들과 평행하게 arm 이 있는데, 앞뒤로 전진과 후퇴하는 모션 가능 
  * 헤드는 디스크 위 아래 한개씩, 두 개 있다. 
  * 회전될때 동심원들을 track 이라 한다. (스핀들로부터의 거리가 동일한 환형의 저장 공간)
  * 동일한 동심원들의 트랙들을 다 묶에서 cylinder 라 부른다. 
  * 각각의 트랙들은 sector 라 부른다. 



* 굳이 Track 들을 모아서 cylinder 라고 할 필요가 있을까? 
  * arm 이 움직이면 동일한 실린더에 접근하게 됨. 
* **Cylinder 단위의 데이터 접근이 중요한 이유** 
  * Arm Assembly 가 고정된 상태에서 여러 Platter 에 걸쳐 있는 한 Cylinder 의 track 들을 동시에 읽을 수 있기 때문이다. 

![{{site.baseurl}}/assets/img/oslec20/image-20200613151721502]({{site.baseurl}}/assets/img/oslec20/image-20200613151721502.png)

> 참고만 하자 



![{{site.baseurl}}/assets/img/oslec20/image-20200613151736479]({{site.baseurl}}/assets/img/oslec20/image-20200613151736479.png)

**디스크의 동작** 

특정 섹터를 읽고자 할 때 명령을 내릴 것. 섹터의 주소가 필요함. 

섹터의 주소는 어떻게 되어있을까? => 3차원 구조로 되어있다. 



**섹터 넘버**로 정보를 읽는다. 

1. 실린더 넘버 
2. 실린더 안의 어떤 트랙인지, 트랙 넘버 
3. 트랙 상의 몇번째 섹터인지, 섹터 넘버 



**섹터의 사이즈 : 대략 0.5 ~ 4KB 까지** => 섹터의 사이즈가 블럭의 사이즈이다. 데이터 transfer 이 일어나는 minimum Unit



**데이터 읽는 과정** 

1. 디스크는 arm 어셈블리를 알맞은 실린더로 옮긴다. (Seek Time - ms 단위의 시간이 걸린다)
2. 해당 섹터가 돌아올때까지 기다린다. (Spindle 의 회전 시간에 의존적. RPM - rotations per minute, Rotational Delay - 디스크가 회전하여 목표한 지점이 head 에 도착하는 시간)
3. 해당 섹터를 읽어서 전기 신호를 만들어서 보낸다. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613152300237]({{site.baseurl}}/assets/img/oslec20/image-20200613152300237.png)



최근들어서는 플래시 메모리를 많이 사용. 

점점 Solid state storage device 가 중요해지고 있다. (2013년 강의 기준)

이 위에 올리는 File System 은 기존의 3차원 디스크와 다를 것. 



**Flash 메모리의 특징** 

* NAND 플래시가 사용되는데, 이것의 메모리는 RAM과 액세스 패턴이 굉장히 다르다. 
* Flash 역시 블럭 디바이스로, 데이터 전달이 블럭 단위로 일어난다. 
* 그래서 read write 를 할 때 대상이 되는 데이터 블럭을 Page 라 부른다. 
  * 메모리 관리때 배웠던 페이지와는 다르다. 
* 페이지들을 몇장 묶어서 블럭이라 한다. 
* 플래시 메모리는 1차원적 배열로 구성되어 있지만 굳이 묶으면 블럭들의 배열로 생각할 수 있다. 
* 블럭들은 또 여러개의 페이지들로 구성되어 있다. 



페이지 안에는 데이터를 저장하는 공간과 spare 공간이 있다.



**왜 굳이 페이지들의 linear array 로 보지 않고, 블럭을 도입했는가?** 

![{{site.baseurl}}/assets/img/oslec20/image-20200613152715108]({{site.baseurl}}/assets/img/oslec20/image-20200613152715108.png)

플래시 연산에는 read, write 연산 존재. 

write 는 program 한다 - 즉 굽는다 라는 표현 사용. 

erase 라는 별도의 operation 도 존재. - 왜 필요한가? 

Flash 는 rewritable 하지 않다. ROM 이기 떄문에 다시 쓸 수 없다. 다시 쓰려면 그 영역을 초기화시켜야 함. 

**📌 "write 하기 위해서는 반드시 erase 해야 한다" 는 특징이 있음.** 

**📌 "worn - out" 이라는 특징 - 종이에 계속 쓰고 지울 수 없듯이 플래시도 마찬가지로 계속 지우면 너덜너덜 해짐.** 

📌 read > write > erase 순서대로 연산 속도가 다름. 또, read write 는 page 단위, erase 는 블럭 단위로 진행된다. 



그래서 flash device driver 는 데이터를 특정 영역에만 썼다가 지웠다 하지 않도록, 돌아가면서 썼다 지우는 메커니즘을 지원해준다. 



**Wear - leveling** 

NAND flash 의 한 영역에 erase 가 집중되어 worn - out 이 발생하는 것을 막기 위해 전체 영역을 고루 사용하도록 조절하는 기술. 



![{{site.baseurl}}/assets/img/oslec20/image-20200613153135856]({{site.baseurl}}/assets/img/oslec20/image-20200613153135856.png)

* Erase Before write 제약조건
  * 반드시 Log structure 를 가진 file system 이 필요함. 
* 페이지 1에 뭔가 쓰면 페이지 넘버는 1, Valid, ... 

![{{site.baseurl}}/assets/img/oslec20/image-20200613153229006]({{site.baseurl}}/assets/img/oslec20/image-20200613153229006.png)

* 페이지 1의 내용을 업데이트 하려면 전체 블럭을 다 지워야 하므로, 별도의 Log 를 남겨둔다. 
* 완전히 새로운 블럭에 업데이트 된 내용을 쓰고, 기존의 내용을 invalid 시킴. 

![{{site.baseurl}}/assets/img/oslec20/image-20200613153311446]({{site.baseurl}}/assets/img/oslec20/image-20200613153311446.png)

* Log Block 에다가 별도로 기록 

**=> Log Structured File System !** 

![{{site.baseurl}}/assets/img/oslec20/image-20200613153343017]({{site.baseurl}}/assets/img/oslec20/image-20200613153343017.png)

File 에 대한 인덱스들을 갖고 있고, 몇번 썼다 지우고 하면 mapping 이 복잡해짐. 

여러번 업데이트 하다보면 한 블럭 전체가 invalidate 될 수 있고, Garbage collection 의 필요성이 생기기도 함. 



### Device Driver 

디바이스 드라이버 : IO Device 를 통제하는, 운영하는 SW 

![{{site.baseurl}}/assets/img/oslec20/image-20200613153610694]({{site.baseurl}}/assets/img/oslec20/image-20200613153610694.png)



* 디바이스 드라이버가 있음으로 인해 
  * File System Storage 가 디스크인지, 플래시인지와 상관 없이 read write 를 할 수 있다. 
  * 추상화된 OS 인터페이스를 제공한다. 



* 디바이스 드라이버들의 핵심 구성 요소들은? 

  > SW system 의 구성을 이야기 할 때는 1. 정적인 구성요소 (Structure) 2. 동적인 구성요소(Behavior)를 이야기 한다. Program 과 Process (Thread of Control) 와의 관계

  * 정적 구성요소 : Device driver 를 유저 프로그램이 사용할 때 호출하는 API들이 정적 구성요소 이다. 
  * 동적 구성요소 : 별도의 Process 나 Thread 인가? 아니다. 오버헤드가 큼. 그렇다면 Device driver 는 디바이스와 interaction 하면서 active 해야 하므로... Interrupt service routine 이 중요한 동적 구성요소 이다. 

**=> 결국, device driver 들은 라이브러리 인데, 어떤 기능들은 API에 의해 유저로부터 호출, 어떤 기능들은 인터럽트로 인해 호출된다.** 



