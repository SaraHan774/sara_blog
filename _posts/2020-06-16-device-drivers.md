---
layout: post
title: "[OS] Device Drivers"
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


# Lecture 21 : Device Drivers 



## 21-1 Device Drivers (1)

![{{site.baseurl}}/assets/img/oslec21/image-20200616112442833]({{site.baseurl}}/assets/img/oslec21/image-20200616112442833.png)

* 디바이스 드라이버 : 컴퓨터 프로그램으로, 다른 프로그램 특히 OS 가 하드웨어 디바이스와 interact 할 수 있도록 하는 것이다. 



**하드웨어가 나오면 하드웨어를 `Bring Up` 한다** 

디바이스에 운영체제를 탑재할 수 있도록 빠르게 드라이버를 개발해야 한다. 

삼성의 입장에서 봤을 때 디바이스 드라이버가 가장 중요함. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616112810881]({{site.baseurl}}/assets/img/oslec21/image-20200616112810881.png)

* 주로 UNIX 계열의 디바이스 드라이버를 소개하겠다. 
* Unix 의 특징 : 단순하게 개발하려 했고, 이식성을 중요시 여겼다. Unix Device Driver Architecture 또한 정형화하여 이쁘게(?) 잘 만들었음. 
* 드라이버는 유저들이 Device 를 접근하기 위해 필요한 함수들을 제공해 주는데, 디바이스가 다양하지만 정형화된 함수 제공 
  * open : 디바이스 초기화, 필요한 공간 할당. 
  * close : 공간 해제 
  * read, write 
  * **ioctl (I/O Control) : 매우 강력함.** 

* I/O 를 할 때는 polling IO 도 하고, interrupt I/O 도 하기 때문에 이를 위한 메커니즘도 필요하다. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616113212814]({{site.baseurl}}/assets/img/oslec21/image-20200616113212814.png)

![{{site.baseurl}}/assets/img/oslec21/image-20200616113541025]({{site.baseurl}}/assets/img/oslec21/image-20200616113541025.png)

**Unix 가 바라보는 디바이스의 종류** 

* Character 
  * 캐릭터 단위, 주로 sequential access 하는 것이 특징
* Block 
  * 주로 Random access 를 하는 것이 특징 
* Network 



**리눅스에서는 디바이스를 잘 정리된, 정형화된 symbolic 한 이름으로 명명함.** 

`/dev/ ...` 밑에 있는 파일들



**캐릭터, 블럭 간의 차이는?** 

블럭 디바이스는 블럭 버퍼 캐시가 있다는게 차이점. 

캐릭터 디바이스 -> 사람과 인터랙션 하므로 느리다.

블럭 디바이스 -> 큰 볼륨을 높은 속도로 전달해야 한다. CPU에 비해 상대적으로 속도가 느린 블럭 디바이스의 성능을 보완하기 위해 캐시를 많이 사용한다. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616113553160]({{site.baseurl}}/assets/img/oslec21/image-20200616113553160.png)

* Device File 을 봐야한다. 



**File System 의 역할** 

1. 디스크에 있는 데이터를 읽거나 쓸 수 있는 기능을 제공 

2. 시스템에 존재하는 자원(하드 디스크 드라이브에 있는 파일들, 임의의 Device 들)들에게 잘 정리된 Name Space 를 제공 ; 디바이스들에게 Slash 로 표현되는 path name style 의 **symbolic name 을 제공**. 



**Unix 이전의 운영체제 에서는?**

프린터가 있다고 하면, 해당 프린터에 임의의 넘버를 부여했음. sys admin 이 이를 공지하고, 프린터를 사용하는 사람들은 해당 넘버를 입력해야 했음. 넘버가 바뀌면 소프트웨어가 꼬여버림... 



**Unix 는 어던 매직을 썼기에 심볼릭 네임을 디바이스에 부여했는가?** 

디바이스 파일이라는 특별한 파일을 생성해서 파일과 디바이스를 연결했음. 

디바이스 파일은 일반 데이터 파일과 차이 없어 보임. 

root 밑에 dev (device) 폴더 밑에 파일로 존재함. 

`tty` : type writer 

`lp` : line printer 

`psmouse` : mouse 



**하나의 디바이스에 이름을 여러개 붙일 수 도 있다.** 

어떤 시리얼 포트에 마우스가 들어가 있으면`/dev/psmouse` 

마우스가 아니라 시리얼 포트에 초점을 두어야 한다면 동일한 디바이스를 `/dev/psaux` 

동일한 디바이스에게 서로 다른 디바이스 드라이버를 제공할 수 있음을 의미한다. 



 ![{{site.baseurl}}/assets/img/oslec21/image-20200616114151838]({{site.baseurl}}/assets/img/oslec21/image-20200616114151838.png)



**Device File 안에는 무엇이 있는가?** 

Device File 에도 약간의 contents 가 있을 것이다, 보통의 파일과 같이. 

**보통 파일과는 다른 메타데이터** 

* Type : 어떤 타입의 디바이스 인가 - 블럭, 캐릭터, 네트워크 
* Major Number : 어떤 타입의 디바이스인가 나타내는 것. 메이저 넘버당 하나의 디바이스 드라이버가 있는 것. 
* Minor Number : 어느 유저가 어떤 터미널을 이용하는지 알아야 하는데, (터미널의 인스턴스를 구분해야) 이를 해주는 것. 디바이스 드라이버의 루틴을 호출할 때 너의 대상이 되는 디바이스 드라이버를 찾으라고 매개변수로 넘겨줌. 



**예시**

/dev/fd0 - 플로피 디스크 - 블럭디바이스 - 2 - 0 



**File = Named collection of Data** 

 * Data 는 두 종류 
    * Real Data 
    * Meta Data (파일의 특성을 표현하는 부가적인 데이터 )



**파일의 메타 데이터는?** 

파일 생성 날짜, 업데이트 날짜, 파일의 주인, 접근 권한 ... 등. 디바이스 파일은 독특한 메타데이터를 갖고 있다. 



**💡 Unix 운영체제는 심볼릭한 이름으로 디바이스 드라이버를 유저에게 표현해준다.** 

 - 👍 실제로 시스템 안의 디바이스들이 어떻게 설정되었는지, 운영체제가 어떻게 처리했는지 신경 쓸 필요 없어 편리하다. 
 - 👎 편한 만큼 부담 : OS 입장에서는 (과거와 마찬가지로) 내부적으로 Numerical identifier 를 이용해서 디바이스를 다 관리해야 한다. **심볼릭한 네임스페이스를 디바이스 네임스페이스로 Mapping 하는 기능이 필요하다. 이것이 파일 시스템의 상단부에 존재함.** 





## 21-2 Device Drivers (2)



**캐릭터 디바이스 드라이버** 

![{{site.baseurl}}/assets/img/oslec21/image-20200616115051399]({{site.baseurl}}/assets/img/oslec21/image-20200616115051399.png)



**Unix 계열에서는 디바이스 드라이버들이 잘 정리되어 있다.** 

배열 형태로 정리됨. 



캐릭터 디바이스 테이블 이라는 배열이 있음. 

시스템에 존재하는 디바이스 타입 만큼 엔트리를 갖고 있다. 



**캐릭터 디바이스 테이블의 인덱스는?** 

Major Number 이다. 운영체제가 메이저 넘버를 알아야만 해당 드라이버를 얻어올 수 있다. 이를 **지원하는 system call은 `open()` 이다.** 매개변수로 file path name 을 주는데 이게 device name임. 운영체제는 dev 디렉토리를 뒤져서 해당 major number 를 뒤져서 사용. 

해당 엔트리에 가면 실제 드라이버가 아니라 포인터들이 있음. 포인터를 따라가면 디바이스 드라이버를 지원하는 함수들의 포인터를 얻어올 수 있고, 디바이스의 속성들도 얻어올 수 있음. 

얻어온 드라이버의 매개변수로 minor number 가 넘어온다(?)



![{{site.baseurl}}/assets/img/oslec21/image-20200616115451174]({{site.baseurl}}/assets/img/oslec21/image-20200616115451174.png)





**블락 디바이스 드라이버** 

![{{site.baseurl}}/assets/img/oslec21/image-20200616115540146]({{site.baseurl}}/assets/img/oslec21/image-20200616115540146.png)



=> 블락 디바이스 스위치 테이블에 드라이버들의 배열을 갖고 있음. 



**블락 디바이스를 캐릭터 디바이스처럼 접근하고 싶은 경우?**

이를 처리할 수 있도록 지원해준다. 캐릭터를 블락화 시켜서 block transfer 한다. OS의 하부 부분은 공통적으로 사용하지만, 상부에서 char, block 둘 다 사용할 수 있게 해줌. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616115711341]({{site.baseurl}}/assets/img/oslec21/image-20200616115711341.png)



**캐릭터 디바이스 드라이버와 비교했을 때 추가된 것은?** 

Block Buffer Cache (운영체제의 기능이지만 디바이스 드라이버에 묻혀있음)가 추가된다. 디바이스 드라이버가 이 버퍼 캐시를 어떻게 매니지 하는지 이해해야 한다. 

캐릭터와 달리 **블락 디바이스는 바로 디바이스로 달려가지 않고, 캐시를 먼저 거친다.** 



**과정 설명** 

block_read() => 버퍼 캐시를 뒤진다. => 존재하면 가져온다. 

=> 없으면 block IO request 가 형성됨 => request queue 로 명령이 들어감 => 디바이스 드라이버가 제공하는 scheduler 에 의해서 dequeue 되고 요청이 처리됨. 



**블럭 디바이스는 성능이 매우 낮아**

스케줄링 알고리즘이 중요함. 이를 strategy routine 이라 부른다. 



**캐릭터 디바이스를 단순하게 설명했으나, 실제로는 더 복잡하다.** 

한 자 씩 입력한다고 해서 그게 바로 어플리케이션으로 전달되지는 않는다. return (end of line) 을 쳐야 간다. 이런 기능들로 인해 내부적으로는 상당히 복잡함. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616120350524]({{site.baseurl}}/assets/img/oslec21/image-20200616120350524.png)



**지금까지 코드 스트럭처의 측면에서 Device Driver 를 이해했다.** 

단순화 시켜서 이해하면, 사용자가 필요로 하는 read write 과 같은 함수를 제공해주는 일종의 라이브러리라고 생각할수도 있다. 이 라이브러리 함수들을 바인딩 시켜야 호출할 수 있는데, 바인딩 시키는 방법이 디바이스 스위치 테이블을 위한 function pointer 들이고, 이를 찾기 위해 major number 와 minor number 를 사용한다. 



**나머지 부분은 Device Driver 의 Behavior 에 관한 것들**

Behavior 중 가장 큰 것은 인터럽트 핸들링이다. 



**리눅스 운영체제에서 어떻게 디바이스에서 발생한 인터럽트를 처리하는가?** 

**two - level interrupt handling 이라는 방식이 자주 사용되어 왔다.** 

우리가 공부한 바에 의하면 인터럽트가 발생하면 현재 수행중인 프로세스가 중단되고, 인터럽트 서비스 루틴이 호출된다. 

이 루틴이 호출되는 동안 또 다른 인터럽트 인스턴스가 도착하면 어떻게 해야 하는가? 

동일한 코드가 동일한 데이터를 접근하면서 인터럽트 핸들링 하면 consistency 의 문제가 발생한다. 이 문제가 생겼을 때 **인터럽트를 disable 하는 방법이있음.** 인터럽트가 발생해도 핸들러를 호출할 수 없게 함. 



> 인터럽트가 여러개 동시에 발생하면 처리할 수 있는 것 빼고 나머지는 Disable 시킨다. 이 시간이 짧을수록 반응이 좋아지는데 이를 위해 Two - Level Handling 을 한다. 



**인터럽트를 disable 시키는 시간은 짧은 게 좋다.** 

어떻게 하면 짧게 만들 수 있는가? 그 때 사용하는게 바로 투 레벨 인터럽트 핸들링이다. 인터럽트가 발생하면 핸들러가 떠서 자기가 수행해야 하는 최소한만 하고, 나머지 중요한 로직은 별도의 context 에서 따로 돌아감 (커널 혹은 유저 둘다 가능). 



**유닉스에서는 bottom half 라는 특이한 방식으로 수행시킴. 일종의 커널 레벨 스레드에서 별도로 로직 수행하는 것.** Bottom half 는 시간이 났을 때 따로 스케줄링 한다. 



> Two Level Handling 의 독특한 처리 방식 중 하나가 Bottom Half 라는 것이다. 유닉스에서는 별도의 커널스레드를 만들어 핸들러가 처리해야하는 중요한 로직들을 처리한다.



![{{site.baseurl}}/assets/img/oslec21/image-20200616121122583]({{site.baseurl}}/assets/img/oslec21/image-20200616121122583.png)

> 리눅스의 Bottom half 가 수행되는 모습 



핸들러가 돌고 있는 동안 핸들러 2가 돌면, 핸들러2 는 bottom half 를 등록하고 나옴. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616121218481]({{site.baseurl}}/assets/img/oslec21/image-20200616121218481.png)

> 함수들이 각각 어떤 역할을 하는지 알아보자. 



Open : Minor Number 를 찾아온다. 디바이스 초기화. 데이터 구조 초기화. -> Device Driver 를 register 하는 역할. 디바이스를 사용하기 위해 오픈한 프로세스들의 개수들을 카운터로 유지함. 프로세스가 close 하면 카운터를 감소시킴. 카운터가 0이되면 해제함. 

Close : 인터럽트 핸들러를 특정 IR큐와 연결 시켜서 인터럽트 벡터 테이블에 register 함. 



![{{site.baseurl}}/assets/img/oslec21/image-20200616121823779]({{site.baseurl}}/assets/img/oslec21/image-20200616121823779.png)

> 중요한 read write 함수 



**Polling 과 Interrupt 의 차이**

**Polling** : 디바이스 드라이버가 데이터 레지스터에 읽어간 값이 있는지 check 해 보는 것. 

어떤 데이터 레지스터에 읽어간 데이터가 있는지 없는지 어떻게 알 수 있을까? 

* Flag 를 두어서 안다. 데이터 레지스터로 데이터를 읽어오면 input data 가 available 하다는 flag 를 켜주고, 이를 보고 device driver 가 읽어간다. 

**Interrupt** : 주도권을 디바이스가 쥐는 것. read 의 경우 내가 실제로 storage 나 device media 에서 데이터를 읽어오면 데이터 레지스터에 쓰고 CPU에 인터럽트를 걸어줘서 이 값을 CPU가 읽어감. 



**Blocking IO 와 Non Blocking IO** 

Blocking : Polling 인 경우, flag 값을 ready 인지 아닌지 계속해서 check 하는 것이다. 

Non Blocking : ready 인지 체크 한 후에 ready 가 아니면 바로 read operation 을 terminate 하는 것. 



**왜 이렇게 두가지로 나누는게 의미 있는가?** 

Blocking 을 한다는 것은 어떻게 보면 device driver 가 데이터를 읽겠다는 의사 결정을 하는 것. Non Blocking 은 스스로 의사 결정을 하지 않고, 없다 라는 status 를 application code 로 넘겨준다. application 이 기다리겠다 하면 또 read 를 하는 방식. 

예 ) Real Time System 에서는 Non Blocking IO 를 하는 것이 좋을 것. 