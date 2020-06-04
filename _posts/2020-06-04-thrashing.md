---
layout: post
title: "[OS] Thrashing and Working Set"
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

# Lecture 19 



## 19-1 Thrashing and working set 

복습 : 로칼리티 때문에 과거의 참조 패턴을 통해 미래를 예측할 수 있다. 이를 기반으로 개발된 것이 LRU 이다. 하지만 LRU 는 구현하는 데 각 페이지 프레임마다 counter 가 필요하여 이 대신 reference bit 를 사용한 clock algorithm 이 구현된다. 이 비트가 없으면 FIFO 를 사용하는데 FIFO 의 단점을 극복하기 위해 frame buffer 를 사용한다. 



### Thrashing 

* 하기는 열심히 하는데, 유용한 결과는 내지 못하는 것. 
* CPU 스케줄러와 메모리 매니저 사이에 서로 interaction 할 때 thrashing 이 발생한다. 
* CPU 관점에서는 CPU utilization 이 중요한 척도이다. 
  * 현재 시스템에 degree of multiprogramming 이 높다고 생각해보자. 이런 경우 page fault 가 증가할 가능성이 높아진다. page fault handling 을 기다리는 프로세스들은 sleep - disk IO 를 기다리게 된다. 이런 프로세스들이 늘어나면 CPU utilization 이 낮아져 새로운 프로세스들을 더 많이 시작하게 된다. 
  * 이런 상황이 thrashing 이다. 
* 기대했던 것은 demand paging 을 사용해서 기억장치의 성능을 더 높이는 것인데 thrashing 이 발생하면 기대와 달리 성능이 더 낮아진다. 



#### Local Allocation 을 통해 Thrashing 을 해결할 수 있는가? 

해결할 수 없다. 특정 process 가 과도하게 memory 를 사용하여 다른 process 가 영향을 받는 현상은 막을 수 있다. 하지만 시스템의 메모리가 부족한 상황인 경우, 어떤 프로세스가 충분한 메모리를 확보하고 있더라도 메모리가 부족한 다른 프로세스에 의해 페이지 폴트 처리가 지연되게 됨으로써 성능이 급격히 나빠지는 현상이 발생한다. 



**Thrashing 이 발생하는 본질적인 이유는 무엇인가?** 

현재 active 한 프로세스들이 요구하는 메모리보다 physical memory 가 작기 때문이다. 이런 상황에서 thrashing 은 피할 수 없는 문제이다. 이 원인을 제거하려면 degree of multi-programming 을 줄이는 것이다. 프로세스를 꺼버리든 해서. 



**Smartphone 처럼 Swap Device 가 없는 경우 많은 process 로 인해 memory 가 부족하면 어떻게 처리하는가?** 

이 문제는 swap device 가 존재할 때 발생하는 문제이다. 하지만 swap device 가 없더라도 multi-programming 을 지원하는 기기 - 스마트폰 - 이 요새 많아진다. 

Android 의 Low memory killer 와 같이 동작 중인 프로세스를 강제로 종료시켜 완전히 메모리에서 제거할 수 밖에 없음. 

특히 TV에서는 프로세스가 차지하는 메모리가 어마어마하게 크다. HD resolution 의 영상 처리를 해야하기 때문이다. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604142502235]({{site.baseurl}}/assets/img/oslec19/image-20200604142502235.png)

* Thrashing : memory 가 overcommit 되는 경우 발생하는 문제. 
* p : page 가 memory 에 로드되어 있을 확률 
* 1 - p : page fault 가 발생할 확률 



![{{site.baseurl}}/assets/img/oslec19/image-20200604142527987]({{site.baseurl}}/assets/img/oslec19/image-20200604142527987.png)

* 시스템에서 왜 thrashing 이 발생하는가? 
  * 시스템은 시스템이 다룰 수 있을 만큼의 work 를 갖고 있는건지 모르기 때문이다. 
  * LRU 메커니즘은 마지막으로 접근한 것을 기준으로 페이지들에 순서를 매기지만 절대 thrown out 되어서는 안될 페이지라 표시하는 절대적인 숫자를 부여하지는 않는다. 
  * 비유 : 강의가 너무 많으면 하나를 drop 한다. 

![{{site.baseurl}}/assets/img/oslec19/image-20200604142857811]({{site.baseurl}}/assets/img/oslec19/image-20200604142857811.png)

* OS 는 thrashing 에 대비해서 무엇을 할 수 있는가? 
  * 만약 하나의 프로세스가 메모리에 비해 너무 크다면 OS 는 할 수 있는게 아무것도 없다. 이 프로세스는 단지 thrash 할 것이다. 
  * 만약 몇몇 프로세스들이 문제를 일으킨다면 
    * 각 프로세스들이 필요한 메모리를 알아낸다. 
    * 메모리 요구가 충족될 수 있도록 프로세스들이 그룹으로 돌아가게 스케줄링 우선순위를 바꾼다. 
    * 로드를 shed ? (보관)한다. 

![{{site.baseurl}}/assets/img/oslec19/image-20200604142905911]({{site.baseurl}}/assets/img/oslec19/image-20200604142905911.png)

* Thrashing 의 문제는 timeshared machine 보다 work station 에서 덜 중요할 수 도 있다. 
  * 한 명의 유저만이 있다면 그냥 응답이 좋지 않은 Job 을 죽이면 된다. 
  * 여러 명의 유저가 있다면 유저와 Job 사이에서 중재하는 OS 의 역할이 필요하다. 

![{{site.baseurl}}/assets/img/oslec19/image-20200604142914341]({{site.baseurl}}/assets/img/oslec19/image-20200604142914341.png)

* Working Set : 어느 시점에 특정 프로세스가 접근하는 페이지들의 집합. 
* Working set 을 정확히 안다면 swap 시에 어떤 프로세스의 working set 을 모두 pre fetching 하는 것이 아이디어. 
* 학기 초에 배운 스케줄러 : 다음에 돌아야 할 프로세스를 선정하는 애. (short term scheduler)
* long term scheduler : degree of multi programming 을 낮추는 애. 켜진지 오래 되었는데 하는 일이 없으면 그냥 내보내는 것. 불러 들여 올 때 해당 프로세스의 working set 을 전부 다 깔아줘야 해서 오래 전에 사용하던 프로그램을 킬때는 속도가 좀 느려진다. (after launch syndrome) 
* 과연 우리가 working set 을 알 수 있는가? 
  * 과거에 빈번하게 사용된 프로세스를 알면 이들을 working set 에 포함시킨다. 
  * 어느정도 stable 한 locality 가 메모리에 들어와 있는 것.  

* 어떻게 하면 적합한 working set 을 찾아내는가? 
  * 이 모델 하에서는 더 이상 방법이 존재하지 않다. 
  * 구현을 가능하게 하는 방법은 찾을 수 없음. 
* working set window size - 단위를 무엇으로 하는게 좋을까? 
  * 시간으로 하면 안됨. multi tasking kernel 이므로 하나의 프로세스가 항상 도는 것은 아니므로 . 
  * 메모리 reference 의 패턴을 갖고 page reference 몇 회를 윈도우 사이즈로 한다 - 라고 할 수 도. (정답은 아님. 내가 액세스 하지 않았던 카운트까지 담을 수 도 있으므로, 불필요하게 working set 에 들어올 수 도 있음.)

![{{site.baseurl}}/assets/img/oslec19/image-20200604144356986]({{site.baseurl}}/assets/img/oslec19/image-20200604144356986.png)

* 구현하기 어렵지만 여기서 멈추면 안된다. 어떻게 극복할 수 있는가? 



![{{site.baseurl}}/assets/img/oslec19/image-20200604144432368]({{site.baseurl}}/assets/img/oslec19/image-20200604144432368.png)

* Reference Bit 을 사용 : 10000 회의 reference 동안 페이지들이 몇 회 액세스 되었는가 카운트. 하지만 오버헤드가 크다. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604144529166]({{site.baseurl}}/assets/img/oslec19/image-20200604144529166.png)

* working set 의 approximation 인 다른 기법들을 사람들이 생각함. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604144723799]({{site.baseurl}}/assets/img/oslec19/image-20200604144723799.png)

**만약 현재 메모리에 존재하는 active 한 프로세스들이 적절히 잘 돌고 있다면 이는 무엇을 뜻하는가?** 

"현재 메인 메모리에 있는 페이지들이 그 프로세스의 working set 일 가능성이 높다." 



**운영체제가 어떻게 현재 특정 프로세스가 잘 돌고 있는지를 판단하는가?** 

page fault 가 얼마나 발생하는지를 확인하여 판단할 수 있다. 



**Page Fault 발생 횟수에 따른 운영체제의 memory 할당 정책** 

* Threshold 보다 많이 발생하는 경우 : 해당 프로세스에게 더 많은 메모리를 할당. 
* Threshold 보다 적게 발생하는 경우 : 해당 프로세스에게 할당된 메모리를 회수.



**Resident Set : working set 의 approximation.** 

* 어떤 프로세스가 어느 시점에 메인 메모리에 갖고 있는 페이지들. 
* page fault rate 에 대한 upper bound 와 lower bound 를 운영체제가 잘 유지해야 한다. 
* 프로세스들이 idle 해졌을 때 long term scheduler 에 의해서 swap 되는데, 이를 다시 불러올 때 pre paging 할 때 resident set 을 사용할 수 있다. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604145351979]({{site.baseurl}}/assets/img/oslec19/image-20200604145351979.png)



---



## 19-2 Trends in Memory Management 

* 기타 등등 ... 
* 최근 운영체제 트렌드와 연관된 부분 



![{{site.baseurl}}/assets/img/oslec19/image-20200604150349619]({{site.baseurl}}/assets/img/oslec19/image-20200604150349619.png)

1. 물리 메모리의 사이즈가 점점 커진다. => 페이지 프레임이 넉넉하다. 
   1. 페이지 교체 알고리즘이 옛날보다는 덜 중요하게 되었다. 
   2. 하드웨어 서포트도 더 줄어듦. 
   3. 소프트웨어로 에뮬레이션 하는 것이 특징이다. 
   4. 페이지 사이즈도 커지게 됨. 
      1. TLB 캐시 라인이 커버할 수 있는 데이터 영역이 커져 hit ratio 가 늘어나 
      2. 페이지 테이블 사이즈도 줄어들고. 



하지만 한 방향으로 일반화 하기에는 어렵다. 스마트폰이나 임베디드 에서는 메모리가 부족하다. 특히 스마트 TV는 메모리에 대한 수요가 매우 크다. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604150612353]({{site.baseurl}}/assets/img/oslec19/image-20200604150612353.png)

2. 가상 메모리의 사이즈가 매우 커졌다. 
   1. 가장 큰 페이지 테이블 사이즈가 64000 TB 
   2. 전통적인 방법으로는 해결하기 어렵다. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604150707834]({{site.baseurl}}/assets/img/oslec19/image-20200604150707834.png)

3. 다른 운영체제의 요소들과 Memory management 이 결합해서 새로운 서비스를 제공할 수 도 있다. 



Large page table 문제. 

1. Page Table 을 유지하기 위해 필요한 공간이 너무 큼. 
2. Page Table 이 Physical memory 의 연속적인 공간에 존재해야 함. 

아무리 커도 조각을 만들어서 piece wise swapping 을 하면 OS가 매니징 할 수 있음. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604150922854]({{site.baseurl}}/assets/img/oslec19/image-20200604150922854.png)



1. 계층적 페이지 테이블 : 페이지 테이블이 scatter 될 수 있도록 조각화 하는 것. 
2. hashed 
3. inverted : 페이지 테이블 사이즈를 매우 줄이는 방법. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604151000161]({{site.baseurl}}/assets/img/oslec19/image-20200604151000161.png)

* address 의 포맷을 조금 건드려야 함. 
* 22bit 를 14 / 8 로 쪼개서 인덱스로 사용



![{{site.baseurl}}/assets/img/oslec19/image-20200604151100979]({{site.baseurl}}/assets/img/oslec19/image-20200604151100979.png)

14bit 를 갖고 페이지 테이블을 뒤진다. first level 혹은 outer page table 이라고 부른다. 

이 엔트리를 따라서 포인터를 가면 실제 페이지 테이블이 존재한다. 

페이지 테이블은 조각이다. 조각이 2의 14승개 만큼 물리 메모리에 흩어져 있는 것, 이 주소를 별도의 인덱스로 가리키는 것이다. 

page of page table 에 가면 비로소 page table entry 가 있다. 

page table entry 에 가면 page frame number 를 얻을 수 있다. 이것과 offset 을 결합하면 물리 주소를 얻을 수 있음. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604151243292]({{site.baseurl}}/assets/img/oslec19/image-20200604151243292.png)

페이지 테이블 엔트리는 4byte 를 차지한다. 

한 엔트리에 256개가 들어갈 수 있음. 



first level page table 은 연속적이어야 하지만 사이즈가 매우 작아서 OS 가 충분히 핸들할 수 있다. 

나머지는 다 page 단위로 나누어져서 swap in swap out 할 수 있다. 



* 문제점 : 메모리 레퍼런스 한 번을 위해서 first -> second 와 같은 메모리 레퍼런스가 늘어난다. 
* 내가 원하는 페이지가 swap device 에 있으면 읽어오는 오버헤드 발생할수도 



![{{site.baseurl}}/assets/img/oslec19/image-20200604151539485]({{site.baseurl}}/assets/img/oslec19/image-20200604151539485.png)



![{{site.baseurl}}/assets/img/oslec19/image-20200604151553117]({{site.baseurl}}/assets/img/oslec19/image-20200604151553117.png)

페이지 넘버를 hash table 을 통해서 얻어오는 기법이다. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604151615985]({{site.baseurl}}/assets/img/oslec19/image-20200604151615985.png)

실제로 페이지 테이블의 사이즈를 획기적으로 줄이는 기법. 

주소 공간 만큼 페이지 테이블을 갖는 것이 아니라 물리 멤리 만큼 테이블을 갖자는 아이디어 



![{{site.baseurl}}/assets/img/oslec19/image-20200604151720351]({{site.baseurl}}/assets/img/oslec19/image-20200604151720351.png)

페이지 테이블의 사이즈가 실제로 시스템에 존재하는 페이지 프레임의 개수이다. 

CPU 는 페이지 넘버와 오프셋을 주소로 낸다. 이 때 페이지 넘버와 PID 를 키로 해서 해싱을 한다. 해싱을 해서 어느 엔트리로 가게 되면 거기에 물리 프레임 넘버가 들어가 있다. 

각 엔트리들은 물리 프레임 넘버를 갖고 있음. 

성공적으로 동작 한다면 페이지 테이블 사이즈를 줄일 수 있을 것이다. 

TLB처럼 보이지 않나? TLB 에는PID, 페이지 넘버 등 정보 있었다. 병렬 서치를 해서 PID 와 page number 일치하는 애를 찾아왔다. => 거대한 TLB라 생각하면 되겠다. 

Hash function 의 성능에 의존적이다. collision 이 발생해서 linked list 길어질수도. 



![{{site.baseurl}}/assets/img/oslec19/image-20200604152001873]({{site.baseurl}}/assets/img/oslec19/image-20200604152001873.png)

