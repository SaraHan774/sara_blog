---
layout: post
title: "[OS] File Systems"
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

**Lecture 23 :** 

# 2. Big Picture on File System 

커널 해커의 관점에서 보자 ... 

![{{site.baseurl}}/assets/img/oslec23/image-20200623112208071]({{site.baseurl}}/assets/img/oslec23/image-20200623112208071.png)

File system 은 운영체제의 subsystem 중 하나이다. 

**제공하는 기능** 

1. Logical Resource Name 을 Physical address 로 변환시키는 역할 
2. 원하는 (사용자의) 데이터를 Storage 에서 읽어오거나 쓰게 하는 기능 



File 을 접근할 때 delay 가 최소화 되기 위해 Buffer Cache 를 유지하기도 함. 

Fragmentation 문제를 해소하는 것 또한 File system 의 역할 





**유저의 관점** 

* Unix 는 File 을 매우 단순화 시켰음 -> File 은 Linear sequence of bytes. 따라서 파일에 접근하기 위해서는 파일의 text name + byte offset 을 알아야 함. 

* File pointer (fp) 로 접근할 수 있음. 

**주요 File Access 패턴** 

* Sequential access : File Pointer 를 한 칸씩 움직이면서 파일의 내용을 읽는 방법 
* Random Access : File Pointer 를 임의로 이동하면서 파일의 내용을 읽는 방법 



**커널의 관점** 

커널은 File name 이 아니라 Index Number (i-number) 과 오프셋을 이용해서 파일에 접근함. 

byte sequence 가 아니라 block 으로 바라봄. 



**디바이스 드라이버의 관점** 

* Volume (Logical Drive) : 단일 File System 으로 관리되는 저장 장치상의 영역 

* 개별적인 File 로 보지 않고, 볼륨안에 존재하는 여러 파일들, entire file system 이라는 관점에서 보게 됨. 
* Sector 혹은 block 들의 linear sequence 이다. 
* 디바이스 드라이버의 하단부가 3차원적인 디스크 어드레스를 알아냄 : 실린더 넘버 + 헤드 넘버 + 섹터 넘버 
* 디바이스 드라이버는 물리 블럭 넘버를 위와같은 3차원적 주소로 변환한다. 
* 디바이스 드라이버는 디바이스에 따른 정보들을 이미 다 갖고 있어야 함. (실린더는 몇개고 등 ... )



=> 위와 같이 유저가 파일에 접근하면 하단에서 View 의 전환이 일어남



![{{site.baseurl}}/assets/img/oslec23/image-20200623113042394]({{site.baseurl}}/assets/img/oslec23/image-20200623113042394.png)

* 유저 관점 : file 단위의 sequence of bytes 
* 커널 : sequence of block 
* 블럭 디바이스 : 섹터들의 넘버로 변환됨



=> 이런 뷰를 유저에게 제공하려면 커널은 어떤 구조를 가져야 하는가? 



![{{site.baseurl}}/assets/img/oslec23/image-20200623113142660]({{site.baseurl}}/assets/img/oslec23/image-20200623113142660.png)

>  위와 같은 레이어 시스템을 갖고 있다. 

**운영체제의 가장 위네는 VFS 라는 것이 있다.** 왜? 

* 여러가지 이유로 인해서 어플리케이션 마다 약간 특성이 다른 여러 File System 이 Unix File system 안에 공존하게 된다. 이를 Uniform 하게 유저에게 present 하기 위해서 이다. 



**Buffer Cache** 

* File 의 데이터 블럭을 메인 메모리에 저장해 두는 공간. 디스크로부터 한 번 읽어온 파일의 내용에 다시 접근할 때, 빠르게 읽어올 수 있게 한다. 
* 달나라로 다시 가지 않기 위한 방법. 
* Buffer cache 가 금방 차게 되면 LRU 같은 교체 정책을 시행함. 



**IO Scheduler** 

* 마지막 시간에 공부한다. 



가장 밑에는 하드웨어 스토리지 볼륨들이 존재함. 



# 3. File Structures 

![{{site.baseurl}}/assets/img/oslec23/image-20200623113615452]({{site.baseurl}}/assets/img/oslec23/image-20200623113615452.png)



파일 시스템 스트럭처를 디자인 하려면 가장 먼저 생각할 것들? 

* 유저들이 무엇을, 어떤 패턴으로 사용할 것인가 
* 어떻게 저장될 것인가 등



파일 시스템을 접근할 때 주로 어떤 형태의 접근 패턴을 보이는가? 

* **빈번히 사용되는 sequential access** : 처음부터 쫙 읽게 될 것. 굉장히 방대한 프로그램을 컴파일 하는 경우도 순차적 접근. 
* **순차 접근 뿐만 아니라 Random access** 도 필요함 : 파일이나 데이터를 찾을 때는 random access 를 하게 될 것. 
  * 예를들어, Demand Paging 을 하는 경우. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623113803403]({{site.baseurl}}/assets/img/oslec23/image-20200623113803403.png)

* 또 다른 접근 패턴은 keyed access : 예를들어 학번, 주소를 주고 접근하는 경우. 
  * 운영체제에서는 사실 이 접근 패턴에 신경쓰지는 않음. 어플리케이션 레벨 접근. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623113919127]({{site.baseurl}}/assets/img/oslec23/image-20200623113919127.png)

**파일 사이즈과 관련된 측면.** 

* 대부분의 파일들은 사이즈가 작다. 
* 적은 수의 큰 파일들이 디스크 공간을 차지하게 됨. 
* 파일 시스템을 설계할때는 작은 파일들을 많이 유지해야 하므로 이를 효율적으로 처리해야 함. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623114050070]({{site.baseurl}}/assets/img/oslec23/image-20200623114050070.png)

* 요즘 예라고 보기는 어렵고, 과거 설계하던 사람들이 어떻게 고민했는지 보자. 
  * Linked File 
  * indexed file 

![{{site.baseurl}}/assets/img/oslec23/image-20200623114124552]({{site.baseurl}}/assets/img/oslec23/image-20200623114124552.png)

**링크드 파일** 

* 파일이 하나 생기면 디스크 스토리지에 블럭을 할당해야 함. 
* 가장 원시적은 방법은 파일 전체에 필요한 블럭을 연속적으로 디스크 블럭에 할당하는 것. 초창기 memory management 와 비슷함. 과거에 배운 dynamic storage allocation 과 비슷한 모습. **=> Fragmentation 문제가 발생함.** 
  * Fragmentation 은 왜 생기는가? 다른 사이즈들을 연속적으로 할당하기 때문. 페이징의 경우 메모리를 페이지 단위로 나눠서 넣음. 다행스럽게도 디스크는 블럭이라는 단위로 나누어져 있음. 

![{{site.baseurl}}/assets/img/oslec23/image-20200623114607318]({{site.baseurl}}/assets/img/oslec23/image-20200623114607318.png)

Linked File 구조에서 File 에 Access 하기 위한 방법 

* 해당 File Descriptor 에 기록된 File 의 첫번째 블록의 주소를 통해 File 에 접근. 
* 블럭 단위로 할당하지만 pointer 를 달아줘서 파일 액세스를 하는 것. 
* 블럭 3를 읽기 위해서는 
  1. File C 의 Descriptor 를 읽어온다. (File C 의 첫번째 블록의 주소를 알아냄)
  2. File C 의 첫 번째 블록을 읽어온다. (File C 의 두번째 블록의 주소를 알아냄)
  3. File C 의 두 번째 블록을 읽어온다. (File C 의 세번째 블록의 주소를 알아냄)
  4. File C의 세 번째 블록을 읽어온다. 



어떤 점이 좋을 것 같나? 

* sequential access 는 문제가 되지 않지만 random access 에는 굉장히 많은 seek 가 필요해짐. 
* 70년대에 사용되었던 운영체제에서 사용한 기법 



![{{site.baseurl}}/assets/img/oslec23/image-20200623115043796]({{site.baseurl}}/assets/img/oslec23/image-20200623115043796.png)

**Indexed File** 

* 파일 디스크립터 에다가 파일이 갖고 있는 블럭들의 포인터를 연결해둠. 다시 말해서 파일 디스크립터 안에 index array 가 있어야 함. 그래서 indexed file 이라 부름. 
* 가장 좋은 점 : 순차 접근, 랜덤 접근 둘다 편함. 
* 가장 심각한 단점 : 한 File 의 크기가 인덱스의 개수만틈 제한된다. 



=> 이런 문제에도 불구하고 Unix 운영체제의 기본 파일 구조. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623115255063]({{site.baseurl}}/assets/img/oslec23/image-20200623115255063.png)

각 파일의 FD 가 해당 블럭들의 주소를 가리킨다. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623115320106]({{site.baseurl}}/assets/img/oslec23/image-20200623115320106.png)

> 버클리의 BSD 운영체제의 파일 스트럭처를 보자. (좀 오래되긴 했지만)

![{{site.baseurl}}/assets/img/oslec23/image-20200623115346717]({{site.baseurl}}/assets/img/oslec23/image-20200623115346717.png)

인덱스가 14개 저장되어 있다. (14개의 블럭만 넣을 수 있는가? 그건 아님)

데이터 블럭을 가리키는 것도 있고, 다른 배열을 가리키기도 함. 기하급수적으로 content 의 개수를, 트리 형태로 늘릴 수 있음. 

14번째 인덱스가 또 배열을, 해당 배열도 또 다른 배열을 가리킬 수 있음. 파일 사이즈가 큰 것도 수용할 수 있음. 

**장점 : 대부분의 파일들은 굉장히 크기가 작아, 10개 - 10KB 이내의 사이즈를 작음. 대부분의 파일들에 대해서 다 수용할 수 있어. (operation cost 가 작아)**

**하지만 파일 사이즈가 점점 커지도록 설계할 수 있으면 (double indirect block 을 사용함에 따라서) 데이터접근에 더 많은 seek 가 필요하게 됨. (대형 파일들은 seek 에 대한 비용을 지불해야.)**



![{{site.baseurl}}/assets/img/oslec23/image-20200623115642175]({{site.baseurl}}/assets/img/oslec23/image-20200623115642175.png)

> 장점과 단점 



![{{site.baseurl}}/assets/img/oslec23/image-20200623115654989]({{site.baseurl}}/assets/img/oslec23/image-20200623115654989.png)

파일 시스템은 디바이스 드라이버 사이에 버퍼 캐시를 갖고 있음. 

버퍼 캐시가 존재함으로 인해서 read write 횟수만큼 디바이스 드라이버가 작동하는 것이 아니라, cache 를 먼저 접근함



![{{site.baseurl}}/assets/img/oslec23/image-20200623115733050]({{site.baseurl}}/assets/img/oslec23/image-20200623115733050.png)

**비트맵 오퍼레이션** 

File 에 연속적인 블럭을 할당할 필요는 없다. 하지만 어떤 파일의 컨텐츠가 디스크 스토리지에 흩어져 있다면 좋지 않음. 왜냐하면 sequential file access 에 굉장히 많은 seek 가 필요하기 때문. 사람들이 바라는 것은 가급적 근처에 있는 (플레이트는 달라도 같은 실린더 상의) 블럭에 할당하는 것임. 

이런 3차원 structure 를 반영해서 linear sector array 를 만들게 됨. 

이렇게 하면 운영체제의 file system 이 contiguous 한 block 의 위치를 알아야 함. 이 때 많이 사용되는 것이 Bitmap 이다. 

블럭들에게 전부 한 비트를 map 시켜둠. 블럭이 free 하면 1, 사용중이면 0을 할당함. 

bitmap 이 디스크 스토리지의 allocation 에 대한 지표임. 

* **파일 저장할 때 6개의 블럭을 할당해야 하면 bitmap 에서 연속적인 6개의 1을 찾아서 할당하면 됨.** 
* 대부분의 마이크로 프로세서가 logical bit string 을 찾아주는 기능을 제공해 주어서 빠르게 할 수 있음. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623120119055]({{site.baseurl}}/assets/img/oslec23/image-20200623120119055.png)



# 4. Disk Space Management 

3차원적인 섹터들을 전부 리니어하게 만들어 놨을 때 File 이 생성 확장될 때 디스크 섹터들을 어떻게 파일에 할당해 줄 것인지 관리하는게 Disk space management 이다. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623120311450]({{site.baseurl}}/assets/img/oslec23/image-20200623120311450.png)

> 파일 시스템이 제공해 주어야 하는 기능을 보면 DSM 이 왜 필요한지 알 수 있다. 

Disk space 를 가능한 오버헤드가 적게, 빠르게 활용할 수 있도록 하는 것이 목적이다. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623120431929]({{site.baseurl}}/assets/img/oslec23/image-20200623120431929.png)

디스크상의 블럭들을 잘 유지해서 할당한다. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623120446705]({{site.baseurl}}/assets/img/oslec23/image-20200623120446705.png)

메타데이터 : 유저들에게는 보이지 않지만 커널이 파일별로 관리하는 공간. 

인메모리 캐시를 항상 유지하고 있어야 하고, system 이 shot down 되었을 때 consistency 를 잘 유지할 수 있어야 함. 

![{{site.baseurl}}/assets/img/oslec23/image-20200623120517289]({{site.baseurl}}/assets/img/oslec23/image-20200623120517289.png)

> 정책들. 

1. Contiguous 
2. Block based 
3. Extent based : 블럭들을 그루핑해서 좀 더 semantics 를 부여함으로써 최적화를 달성하는 정책  => 왜 필요한가? 
   * 블럭 단위로 액세스 하면 sequential access 에 너무 많은 seek 가 발생함. 
   * 디렉토리의 블럭 안에 들어있는 content 들은 File 의 인덱스 넘버와 File 의 이름. 
   * `ls` 를 하기 위해서는 많은 seek 가 일어난다. 자기 밑의 i-node 들을 읽어와서 디스플레이 하는 것이기 때문. 
   * 어떤 subdirectory 밑의 i-node 들이 흩어져 있으면 seek 가 많이 발생하므로, 가급적 동일한 실린더에 놓도록 한다. 
   * 블럭 단위의 allocation 보다 큰 단위의 allocation 이 필요함을 알 수 있음. 



![{{site.baseurl}}/assets/img/oslec23/image-20200623120845640]({{site.baseurl}}/assets/img/oslec23/image-20200623120845640.png)

> Fragmentation 문제 때문에 아무도 쓰지 않는 방식 



![{{site.baseurl}}/assets/img/oslec23/image-20200623120909910]({{site.baseurl}}/assets/img/oslec23/image-20200623120909910.png)

> 아무도 안씀. 





# 5. Evolution of UNIX File Systems 

![{{site.baseurl}}/assets/img/oslec23/image-20200623121008121]({{site.baseurl}}/assets/img/oslec23/image-20200623121008121.png)

1~5 순서대로 조금씩 개선됨. 

NFS 분산 처리 접목. 

너무나 많은 파일 시스템 생기자 VFS 의 등장. 

90년대에 들어서 disk storage 의 크기가 커지고 sudden shutdown 에 대처하기 위한 LFS 가 등장. 

=> 다음 시간에 더 자세히 살펴볼 것.