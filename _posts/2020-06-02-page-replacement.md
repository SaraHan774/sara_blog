---
layout: post
title: "[OS] Page Replacement Policy"
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


# Lecture 18 

### 18-1 Key Issues in Paging 

![{{site.baseurl}}/assets/img/oslec18/image-20200602115714427]({{site.baseurl}}/assets/img/oslec18/image-20200602115714427.png)

![{{site.baseurl}}/assets/img/oslec18/image-20200602120120440]({{site.baseurl}}/assets/img/oslec18/image-20200602120120440.png)

* Demand Paging : 페이지가 요구될 때 RAM 에 불러들이는 것. 
  * **Pure demand paging** : RAM 에 0개의 페이지로 시작, 페이지 fault 나면 또 읽어들이고 ... 처음에는 많은 fault 를 야기시키다가 충분히 들어오면 원활하게 돌아간다. 
  * 문제점 : 시간이 오래 걸림. => 개선하고자 함. 
* 이상적인 페이지 선택 정책 : 페이지가 언제 들어올지 미리 알고 불러오는 것. 
  * 효과적인 **Pre-paging** : Spatial locality 에 기반하여 page fault 가 발생한 근처에 있는 page 들을 main memory 로 로드. 
  * **몇 페이지를 읽어 들일건지 결정해야 하는 문제.** 

![{{site.baseurl}}/assets/img/oslec18/image-20200602120435421]({{site.baseurl}}/assets/img/oslec18/image-20200602120435421.png)

* Pre-paging 과 비슷한 **request paging** : 논리적으로 연관성이 있는 page 들을 미리 main memory 로 로드하는 기법. 
  * 예) 어떤 프로그램이 A B C D 네 모듈로 이루어져 있을 때 A -> D 자주 호출하고 B -> C 자주 호출하는 것을 프로그래머가 알면 둘을 같이 올려 준다. 
  * 다른 말로 overlay 라고도 함. 올라가 있는 것을 지우고 다른 것을 올리는 방식으로 작동하기 때문에. 
  * 프로그램의 속성을 프로그래머가 판단해야 한다는 불편함이 있다. 이를 OS 가 알아서 해주면 좋을 것. 

* Pre-paging 보다 더 나은 방법은? 
  * **Working Set 이라는 개념** : 일종의 pre paging 과 request paging 을 결합, OS 가 자동으로 처리해주는 page selection policy





---



### 18-2 Page Replacement Policy

![{{site.baseurl}}/assets/img/oslec18/image-20200602120744893]({{site.baseurl}}/assets/img/oslec18/image-20200602120744893.png)

* 강제로 Free frame 을 만드는 방법 => 쫓아낸다. HOW? 

  * **Random** : 랜덤으로 내보낸다. (어차피 인생은 RANDOM 이니까 (?))
    * 잘 동작하는 경우 : 메모리 replacement 를 하드웨어 적으로 구현할 때. Cache line replacement 에는 랜덤 정책 사용. 
  * **⭐FIFO** : 들어온지 가장 오래 된 것을 내보낸다.  
    * Fair 한가? NO. Daemon 과 같이 부팅 때에 불려왔지만 계속 사용되는 페이지들이 쫓겨나가면 성능에 나쁜 영향을 미친다. 
    * 많은 경우 좋은 정책이 되지만 페이지 교체 정책으로는 적절하지 않을 수 도. 
    * BUT, 의외로 많이 사용됨. => 뒤에서 설명. (FIFO 의 단점을 극복해주는 별도의 메커니즘이 필요하지만)
  * **OPT (Optimal)** : 무엇을 내보내는 것이 최적인가? 가장 먼 미래동안 전혀 사용되지 않을 페이지를 골라서 내보내는 것. 
    * 신(God)만이 할 수 있다 (?)
    * page replacement algorithm 짜고 나서 성능 비교를 위해 쓰인다. 
    * reference string (sequence of referenced page) 벤치 마크 프로그램 통해서 로그를 얻어온다. 이 로그를 미리 알고 있으면 더 좋은 알고리즘을 개발할 수 있다.

  ![{{site.baseurl}}/assets/img/oslec18/image-20200602121555530]({{site.baseurl}}/assets/img/oslec18/image-20200602121555530.png)

  * **⭐ LRU (Least Recently Used)** : 최근에 사용되지 않은 페이지를 골라서 내보내겠다. 
    * 왜 의미있게 동작하는가? : Locality 때문이다. 지금 사용되는 페이지들은 가까운 미래에도 사용될 것이고, 최근에 사용되지 않은 페이지들은 미래에도 사용되지 않을 것이라 가정한다. 
    * 하지만 프로그램의 execution pattern 이 sequential 하다면 잘 맞지 않는다. 
    * 루프가 존재하는한 locality 있기 때문에 잘 동작함.  



* 가장 많이 사용되는 것은 FIFO 와 LRU 
  * FIFO : linked list 로 구현 
  * **LRU : time stamp 를 찍는다.** 페이지 프레임 하나마다 카운터 레지스터를 둔다. 어떤 페이지가 참조되면 타임 스탬프를 읽어서 정책을 사용. 하지만 이렇게 구현하기에는 HW 오버헤드가 커서 **실제로 이렇게 구현하지는X** 
    * **Reference Bit (Use Bit)** : Main memory 에 로드된 page 가 access 되면 set 되는 bit flag 
    * 사용되면 1, 안되면 0 
    * LRU 를 100프로 타임스탬프로 제대로 구현을 못하고, 그 근처까지 구현하는 것. LRU 를 approximation 한다. 



* Memory Management 에 중요한 HW 서포트 
  * Resident Bit 
  * Reference Bit (Use Bit) : 1이면 RAM 에 있지 않고, Swap space 에 있다. 



![{{site.baseurl}}/assets/img/oslec18/image-20200602122410711]({{site.baseurl}}/assets/img/oslec18/image-20200602122410711.png)

* Reference String : 어떠한 페이지가 참조하는 메모리의 sequence 



![{{site.baseurl}}/assets/img/oslec18/image-20200602122840930]({{site.baseurl}}/assets/img/oslec18/image-20200602122840930.png)

* 비레이디의 아노말리 : 돈을 더 내서 physical memory 를 더 붙였는데, page fault 가 더 많이 발생하는 것. 기대하는 것과 다른다. 
  * page frame 3개 보다 4개로 돌렸을때 page fault 가 더 많이 나는 경우 생겨.
  * FIFO 가 주로 이런 문제를 보인다. 
* Stack algorithm 은 이런 아노말리를 보이지 않는다. 
  * **Stack algorithm** : N 개의 Frame 으로 구성된 Memory 에 로드된 Page 들이 항상 **N + 1 개의 Frame 으로 구성된 Memory 에 로드된 Page 들의 부분집합**이 되는 Algorithm. Belady's Anomaly 가 절대 발생하지 않는다. 
* LRU 가 스택 알고리즘이다. 가장 오랫동안 사용되지 않은 것들만 왔다갔다 하지 그 외의 것들은 항상 substring 으로 존재. 하지만 FIFO 는 들어오는 것과 나오는 것이 분리되어 있다. 



![{{site.baseurl}}/assets/img/oslec18/image-20200602123311938]({{site.baseurl}}/assets/img/oslec18/image-20200602123311938.png)

* 그래서 사람들은 LRU 에 집중을 했으나, time stamp 때문에 HW 오버헤드가 있다. 
* 각 페이지 프레임에 붙어야 하는 카운터의 size 문제 -> overflow 가 나면 오래된 페이지도 작은 카운터 값을 갖게 되는 문제 있을 수도. 
* One bit register 를 갖고 time 을 어떻게 표현하지? 
  * 시간을 일정한 interval 로 나누고, 그 interval 에서만 `사용된 적 있니 없니`를 비교한다. 
  * 매 interval 마다 bit 를 reset 한다. 교체를 할때는 그 interval 에 1인 놈을 내쫓는다. 
* 이런 알고리즘을 조금 더 체계화 시킨 것이 Clock algorithm 이다. 



![{{site.baseurl}}/assets/img/oslec18/image-20200602123659070]({{site.baseurl}}/assets/img/oslec18/image-20200602123659070.png)

* Clock Hand 가 필요 : 가리키고 있는 임의의 페이지. 
* 페이지 교체 시점에 clock 을 움직이면서 어느 페이지를 교체할지 선택한다. 



**Clock Algorithm 에서 교체할 Page 를 선택하는 방법** 

* Clock Hand 가 가리키는 page 의 reference bit 이 1인 경우, 0으로 바꾸고 (Skip 한다) clock hand 가 다음 page 를 가리키도록 한다. 이 과정을 반복하다가 reference bit 이 0인 page 를 만나면 해당 page 를 교체한다. 
* 전부 1인 경우 : 모든 페이지들이 다 reference 되어 있다. 메인 메모리가 작아서 모든 페이지들이 빈번히 access 되는 경우. 그렇기 때문에 clock hand 는 더 빨리 돈다. **결국 FIFO 로 degenerate 된다.** 
* OS 가 과부하가 걸렸는지 판단하는 기준으로 clock hand 가 얼마나 빨리 도는지를 보기도. 



![{{site.baseurl}}/assets/img/oslec18/image-20200602124430969]({{site.baseurl}}/assets/img/oslec18/image-20200602124430969.png)

* Dirty Bit 을 보기도 한다. 쫓아낼 때 clean page 를 쫓아내고 싶다. swap device 를 쓸 필요가 없으므로. reference 되지 않고, clean 한 것 을 1순위로 내보내는 것. 
* 이런 휴리스틱은 항상 코너 케이스를 발생시킨다. clean page 가 코드 페이지라서 빈번하게 사용된다면 문제가 되는 경우 등. 



![{{site.baseurl}}/assets/img/oslec18/image-20200602124620717]({{site.baseurl}}/assets/img/oslec18/image-20200602124620717.png)

* Unix 계열의 OS 는 메모리 상태를 분석할 수 있는 도구들을 많이 제공해준다. 
  * Clock hand 가 얼마나 자주 돌고 있는지
  * vmstat 이라는 명령어를 사용하면 성능 관련 통계를 볼 수 있음. 



---



### 18-3 Page Replacement Policy (2)

![{{site.baseurl}}/assets/img/oslec18/image-20200602131040265]({{site.baseurl}}/assets/img/oslec18/image-20200602131040265.png)

FIFO 는 VMS 에서 사용하던 메모리 매니지먼트 방식. 

* FIFO : page frame 들을 linked list 로 구성한다. used page frame 들을 연결해둠. VMS 에서는 free list 를 별도로 둔다. free page frame 들을 들어온 순서대로 linked list 로 연결한다. used 는 활발하게 접근되는 페이지 들이고, free 는 잘 사용되지 않는 것들. 새로운 페이지를 하나 읽어 들여야 하면, used 를 건드리지 않고 free 에서 하나 따와서 used 에 연결함. free 는 계속해서 줄어들게 됨. 그러면 강제로 used frame 에서 일정 량의 페이지를 free 로 반환하도록 함. 
  * 기본 FIFO 와 뭐가 다른가? : **Frame Buffering** 
    * 우연히 들어온지 오래 되지 않은 페이지가 free 로 가면, second chance 를 주어서, free list 에서 used 로 다시 갈 수 있도록 한다. 이런 기법을 frame buffering 이라고 한다. 

* 근원이 뭔가? 
  * 이런 frame pool 을 두지 않는다면, 빈번하게 사용 되었지만 들어온지 오래 된 페이지에게 second chance 를 줄 수 없다.

* **Reference bit 하드웨어 서포트가 없는 경우 FIFO 와 Frame Buffering 을 같이 사용한다.** 
* 추가적인 Optimization 이 가능하다. 
  * free page frame 들의 리스트에는 dirty page 가 섞여있다. swap device 가 idle 할 때 dirty page 들을 찾아서 이 페이지들을 빈 시간에 swap device 로 보낼 수 있다. 그러면 페이지들이 clean 하게 됨. 



![{{site.baseurl}}/assets/img/oslec18/image-20200602131756828]({{site.baseurl}}/assets/img/oslec18/image-20200602131756828.png)

* Page Placement Style : Allocation style. 
  * 메모리를 요구하는 주체를 프로세스 단위가 아니라 유저 단위로 생각할 수 도 있다. 
  * 시스템 단위로 allocation 할 수 도 있을 것. 



1. Global : system 단위로. 모두가 선하게 동작한다면 메모리를 효율적으로 사용할 수 있으나 그렇지 않으면 문제가 된다. 
2. Per-Process : process 단위로. 하나의 메모리 hog 가 존재해도 다른 프로세스는 보호해 줄 수 있으나, 프로세스간 분배가 불공평 할 수 도 있다. 어떤 것이 너무 많이 받았으면 동적으로 페이지 프레임을 줄여야 하고, 적게 받은 프로세스에는 늘려줘야 함. **어떻게 Adaptive, dynamic 하게 allocation 을 할 수 있을까*?**
3. Per-Job : User 단위로.



* *per-process : 운영체제가 프로세스간 메모리 분배를 characterize 할 수 있어야 한다. 뭐를 보면 알 수 있는가? **단위 시간 동안 page fault 발생 횟수를 확인.** 
  * Page fault 많이 나면 메모리 부족하다고 판단. 



---

다음 시간에는 thrashing 과 working set 에 대해서 공부한다. 
