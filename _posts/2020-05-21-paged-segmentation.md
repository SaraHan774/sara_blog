---
layout: post
title: "[OS] Paged Segmentation & more"
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

# Lecture 16 : Paged Segmentation 

**1. Segmentation**

문제 : 초창기에는 한 Job 이 하나의 세그먼트를 받아서 multi programming 을 했다. 하지만 섹션마다 access pattern 이 다르다. 코드 섹션의 경우 sharing 이 되어야 하기도. 그래서 한 프로세스에 여러개의 세그먼트를 부여하게 됨. 이게 바로 segmentation. 

프로세스별로 하나의 세그먼트만 있을때는 하나의 register pair 가 필요했는데, 이게 많아져서 segment table 이 등장함. table 을 통해서 물리 주소에 접근할 수 있고, 이는 MMU 의 STBR 에 의해 접근된다. 


**2. Pagination**

여전히 존재하는 문제는 Fragmentation 이다. 왜 생기는 문제인가? 동적으로 메모리를 할당 받을 때 크기가 다 다르고, 이 블럭들이 contiguous 해야 한다는 제약 때문에 발생하는 문제. 그래서 할당 받는 메모리를 Unit 사이즈로 정하게 됨. 이를 Paging 이라 한다. Paging 을 하려면 Page 테이블을 유지해야 하고, 블럭 디바이스의 블럭 사이즈로 Page 사이즈를 맞춘다. Paging 을 사용할때 테이블을 위한 공간이 많이 필요하다는 문제가 존재한다. 그리고 page table 을 접근하여 주소를 구해야 하므로 접근 시간이 좀 느려지는 문제가 있다. 

메모리 접근에 시간을 낮추려면 어떻게 해야하는가? 



* Memory Hierarchy : CPU 안에 register, Main Memory, Disk ... 등이 있다. 
  * Register 입장에서 Memory access 는 굉장히 느리다. 이를 좀 더 빠르게 하기 위해 CPU와 Memory 사이에 Cache 를 도입한다. 
  * Page table entry 들을 캐싱하는데, 이를 위한 것이 TLB (Translation lookaside Buffer) 이라고 한다. 
  * 속도 저하 문제는 비교적 쉽게 해결 가능. 


---

### 16-1 Paged Segmentation 

Segmentation 의 취지와 달리 Paging 을 하면 하나의 프로세스가 하나의 세그먼트만 갖게 됨. 



**Paged Segmentation : Segmentation 과 Paging 각각의 장점을 한꺼번에 얻기 위해 두 기법을 동시에 사용한다. 이를 위해서 먼저 Segmentation 을 수행하고, 각 Segment 별로 Paging 을 수행.** 



* **Table 이 어떻게 구성되어야 하는가?** 
  * Segment table 에 들어가는 정보 : Segment Number, Base Address, Bound 값 
  * Page Table 에 들어가는 정보 : Page Number, Page Frame Number 
  * Paged Segmentation 을 하면 Segment Table 에서 Base address 값 대신 Page Table 에 대한 정보를 얻는다. Page Table 의 Base address 가 저장된다. 기존 Bound 값 대신 해당 segment 의 page 개수를 저장한다. 



![image-20200514203252678]({{site.baseurl}}/assets/img/oslec16/image-20200514203252678.png)

> Paging + Segmentation 



* 이 방법의 가장 큰 단점은 ? 
  * 1 memory reference 당 전체 3번의 접근 과정이 필요 - 성능이 더 저하됨. 



![image-20200514203350578]({{site.baseurl}}/assets/img/oslec16/image-20200514203350578.png)

> 어떻게 Address Translation 이 일어나는지 알아보자. 

* 과거의 IBM 370 머신 
* 상위 4비트를 세그먼트 넘버, 8비트는 페이지 넘버, 그 다음 12비트는 페이지 오프셋으로. 
  * 페이지당 사이즈가 4KB 이고, 세그먼트당 최대 2^8 개의 페이지를 가질 수 있다. 
  * 프로세스 하나당 부여할 수 있는 세그먼트의 개수는 2^4 개. 
    * 왜 16개 씩이나 필요하지? 운영체제에게 그만큼 선택권을 준 것. 운영체제가 어떤 메모리 공간을 공유해야 하면 별도의 메모리 세그먼트를 만들어서 공유하는 경우 등.  
* 세그먼트 테이블에는 16개의 엔트리가 있고, 각각의 엔트리에는 자신의 페이지 테이블 베이스 어드레스가 있다. 



![image-20200514203706900]({{site.baseurl}}/assets/img/oslec16/image-20200514203706900.png)

> 위와 같은 세그먼트 테이블과 그에 해당하는 메모리 주소들이 있을 때 아래의 논리 주소에서 물리 주소를 얻는 과정에 대해 학습한다. 



![image-20200514203746968]({{site.baseurl}}/assets/img/oslec16/image-20200514203746968.png)

> 0x002070 을 예로 들면 다음과 같다. 

<img src="{{site.baseurl}}/assets/img/oslec16/image-20200514203844170.png" alt="image-20200514203844170" style="zoom:50%;" />

> 주소를 다음과 같이 나눠서 해석할 수 있다. 

물리적 프레임은 03 에 매핑된다. 이를 이어서 쓰면 0x003070 과 같은 물리 주소로 변환할 수 있음. 



**MMU 는 운영체제 입장에서 Transparent 하지 않다.** 

* 프로세스의 문맥 전환이나 새로운 프로세스의 생성 등이 일어날 때, 운영체제가 Base/ Bound Register 및 Segment / Page Table 들을 관리해야 하므로 운영체제에게 Transparent 하지는 않음. 
* 새로운 프로세스가 forking 될 때 이런 테이블들도 다 만들어 주어야 함. 



![image-20200514204355944]({{site.baseurl}}/assets/img/oslec16/image-20200514204355944.png)

**장점과 단점** 

* 만약 세그먼트가 사용되지 않으면 이에 상응하는 페이지 테이블도 있을 필요가 없다. 
* 두 레벨에서 공유할 수 있다. 
  * Single Page 
  * Single Segment (Whole Page Table) 
  * 프로세스들 간의 세그먼트 공유가 용이하다. 
* 페이지들은 external fragmentation 문제를 해결하고, reshuffling 없이 세그먼트가 커질 수 있도록 한다. 
* 만약 페이지 사이즈가 대부분의 세그먼트와 비교했을 때 아주 작을 경우 internal fragmentation 은 큰 문제가 되지 않는다. 
* User 는 페이지 테이블에 대한 접근 권한이 없다. 



Pure paging 을 할 때 보다 페이지 테이블 스페이스를 줄일 수 있다. pure paging 만을 하게 되면 어느 페이지가 사용 되고, 안되고를 알기가 없다. 가상 주소 공간에 hole 들이 생기게 됨. 하지만 P + S 를 하면 페이지 개수를 카운트해서 그 양 만큼만 테이블을 만들기 때문. 하지만 메모리 레퍼런스 단계가 증가한다는 단점. 



![image-20200514205642291]({{site.baseurl}}/assets/img/oslec16/image-20200514205642291.png)




---

### 16-2 VAX VMS case 

![image-20200514205652044]({{site.baseurl}}/assets/img/oslec16/image-20200514205652044.png)

이제 가장 많이 사용되는 운영체제는 Linux 이다. (PC 를 넘어서 모바일 기기가 많아지므로) 

리눅스에서는 P+S 를 사용하지 않고 Pure Paging 만 사용함 ... 왜? 

1. **Memory Access Control :** 과거의 세그먼트 단위의 액세스 컨트롤을 페이지 테이블로 내림. Read, Read Write 를 페이지 단위로 관리해서 페이지 테이블에 기록함. 

2. **Sharing :** 공유 문제도 페이지 단위로 나눠서 해결함. fork exec 설명할때 부모 프로세스의 데이터 코드 세그먼트를 다 copy 하는데 이것을 page 단위로 하게 됨. 하지만 완벽하게 페이징을 하지 않는 것은 아님. OS 가 SW적으로 어느 페이지가 어떤 세그먼트인지 그 region 을 다 관리함. 



=> 하드웨어 MMU가 다 하는게 아니고 운영체제가 관리하는 것. Segmentation 에 필요한 많은 일을 운영체제가 하는 대신 Pure paging 을 사용함. 



그렇다면 리눅스 만드는 사람들은 왜 복잡하게 페이징을 하게 되었을까? 

* **운영체제 코드를 대략 크게 두 개로 나눈다 => machine independent / dependent 한 것 두개.** 
  
  * 운영체제를 만드는 사람들은 Machine independent 한 코드의 비중이 커지게 만들고 싶어함. 하드웨어가 바뀌어도 사용할 수 있으므로. 
    
  * 하지만 **아래의 세 가지 차이 때문에 machine dependent 한 코드들이 생기게 됨**. Context Switching 코드,  Task Creation 과정이 특히나 micro architecture 에 의존적이다. 
  
    * CPU가 제공하는 register 들을 바로 접근해야 하므로. 
    * 이들 중에는 C로 작성할 수 있는 부분도 있지만 **Context Switching 의 경우 어셈블리 언어로 작성해야함.** 
    * IO device driver 도 머신 의존적이고, 
    * 또 Memory management 부분도 의존적. 
      * MMU 지원의 여부도 있고, 
      * 시스템의 메모리 맵도 고려해야 함. 
      * 심지어 인텔 프로세서는 paged segmentation 을 지원할 수 있는 하드웨어가 있음. 
      * **근데 ARM 은 이런 segmentation 을 지원하지 않아**. ***그래서 리눅스를 개발하는 사람들은 아예 segmetation 기능이 없다고 가정하고 paging 만 함.*** 
    
    
    
  * **하드웨어적 차이를 논할때 크게 세 가지를 얘기한다.** 
  
    * Micro architecture 의 차이 (CPU architecture)
    * 전체 시스템의 차이 (버스 등)
    * IO device 
      
  
* 따라서 Pure paging 만 지원하고 segmentation 은 운영체제에서 소프트웨어적으로 해결. 이는 portability 를 위해서이다. 



두 가지 이슈를 고민해야 한다. 

1. 성능 저하의 문제 
2. large page table 의 문제 (Large Table Space) 
   * 페이지 테이블이 요즘에는 점점 커질 수 밖에 없다. 64 bit 머신이 많아지므로. 
     

Page frame 들의 속성 

* mapped page : 전통적인 페이지 address translation mechanism 을 거쳐서 주소를 얻어오는 페이지.
* unmapped page : translation 없이 그냥 물리 주소로 액세스 하는 페이지. 
  * 영역중의 일부는 이렇게 direct access 를 하는데 사용함. 
* cached page : L1, L2 캐시를 사용하는데 데이터일수도 있고 인스트럭션일수도 있는데, 일반 페이지들은 일반적으로 cached page 이다. 한 번 특정 워드가 액세스 되었으면 캐시에 빈 공간이 있는한 캐싱을 시켜서 hit 되도록 한다. 
* uncached page : 절대 캐싱을 하면 안되는 페이지. 메모리에 페이지 프레임이 있는데 이를 읽고 쓰는 프로세스가 하나뿐이면 캐싱을 하든 안하든 상관이 없다. 하지만 어떤 페이지가 sharing 되는 경우 절대 캐싱을 하면 안됨. 
  * DMA 의 대상 Page 처럼 여러 프로세스에 의해 동시에 읽고 쓰기가 일어날 수 있는 경우 사용. 
  * DMA operation : CPU 버스에 master 가 여러개 있어서 하나가 읽을 때 다른 하나도 쓸 수 있는 것. 



그렇다면 페이지 테이블은 어디 들어가야 하나? 

* 당연히 unmapped space 안에 들어가 있어야 한다. 
* mapped 된 주소를 번역해 주는 것인데 만약 테이블이 mapped 이면 계속 recursion 이 일어날 것이므로. 



페이지 테이블은 물리 주소상에서 흩어져 있어야 할까 contiguous 하게 있어야 할까? 

* 배열은 array index 로 접근, 연속되어 있으므로. 
* 테이블도 마찬가지로 연속되어 있어야 한다. 
* 단, 성능의 문제로 시스템에 unmapped row memory 를 운영체제의 필요에 의해 조금만 유지하도록 한다. 



Large Page Table 의 문제 해결 조건 

1. 페이지 페이블의 크기를 줄임 
2. 연속적인 메모리 공간 요구량을 줄임 



1980년대 이 문제를 해결하려 했던게 VAX computer architecture. VMS 라는 운영체제 있었음. 

* 큰 페이지 테이블 문제를 해결하는 것.
* Virtual Address Extension 
* 전체 가상 공간을 4개의 세그먼트로 나눔 : system, p0, p1, unused 



가상 공간에 운영체제를 위치시키는 방법이 두 가지 있음 

* 가상 공간의 일부를 OS에 할당하고 유저들은 가상 공간의 뒷부분을 사용하는 방법. 
  * VAX는 이 방법을 사용해서 유저 공간을 p0, p1 으로 나눔 
* 유저가 전체의 가상 공간을 다 사용하고, 운영체제 코드는 별도의 가상 공간을 갖는 방법. 



![image-20200521160751376]({{site.baseurl}}/assets/img/oslec16/image-20200521160751376.png)

가상 주소를 번역하는 페이지 테이블은 System 영역안에 존재한다. 이 영역은 mapped page space 이다. 운영체제이지만 유저의 가상 주소 공간에 위치하므로, mapped 임. 

유저의 페이지 테이블이 여기 있다는 것은 뭘 의미하는가? 

* 페이지 테이블의 엔트리 주소를 알아서 가보면, 이 영역의 주소 또한 virtual address 이다. 



이 페이지 테이블은 실제 물리 메모리 상에서 contiguous 하지 않다. 

* Large table space 의 문제 중 페이지 테이블이 물리 주소 상에서 연속적으로 존재해야 한다는 문제를 해결함. 
* 가상 공간 상에서만 연속적인 것. 
* 페이지 테이블이 아무리 커져도 swap 될 수 도 있는 장점이 있음. 



하지만 기껏 페이지 테이블 주소 얻어왔는데, 또 translation 을 해야 한다는 단점 존재. 

* 결국 물리 메모리에 있는 페이지 테이블은 운영체제의 주소를 번역해주는 페이지 테이블이다. 



유저의 페이지 테이블 -> 가상 공간의 페이지 테이블 -> 물리 주소 찾아감. 



![image-20200521161302905]({{site.baseurl}}/assets/img/oslec16/image-20200521161302905.png)

![image-20200521161314607]({{site.baseurl}}/assets/img/oslec16/image-20200521161314607.png)




---

### 16-3 Translation Look Aside Buffer 



Large Page Table 문제를 설명했고, 성능 저하의 문제를 설명하겠다. 



자주 look up 되는 페이지들의 주소를 캐싱 해두면 좋겠다 해서 나온게 TLB 이다. 



TLB 가 잘 작동하려면 hit ratio 가 중요하다. 

1. 크게 만들면 된다. 

* 하지만 비용이 들어... -> 적절한 trade off point 를 찾아야. 

* 그런데, 우리가 생각했던 것 보다 큰 게 필요하지 않다. 64개 ~ 128개만 있어도 98프로 정도 hit ratio 가 나옴. 왜 그럴까? 

  * 프로그램의 수행에는 locality 라는 속성이 있기 때문이다. 
  * 1. Spatial locality 
    2. Temporal Locality : 매우 짧은 시간안에 다시 수행된다. loop 때문에. 프로그램은 90프로의 시간을 loop 안에서 보내더라. (크누스의 실험) 

  * 어떤 페이지가 액세스 되었고, 루프를 갖고 있다면 계속 액세스 되니까 이런것들을 캐싱을 해둔다. 

2. 페이지의 크기를 크게 만든다. 



![image-20200521162000258]({{site.baseurl}}/assets/img/oslec16/image-20200521162000258.png)

논리 주소가 발사되면, 이 주소를 갖고 MMU안의 TLB를 먼저 되진다. TLB hit 가 나면 프레임을 찾고, 아니면 페이지 테이블을 본다. 



![image-20200521162052238]({{site.baseurl}}/assets/img/oslec16/image-20200521162052238.png)

TLB의 구성 

기본적으로 캐시이다. 



캐시를 구현하는 세 가지 방법 

1. directed mapped cache 
2. fully associative cache 
3. set associative cache 



TLB도 위의 방법을 통해 구현한다. 



![image-20200521162153411]({{site.baseurl}}/assets/img/oslec16/image-20200521162153411.png)

1. 처음부터 끝까지 다 뒤진다. 
2. 페이지 테이블 엔트리를 특정 영역에만 mapping 시킨다. Direct mapped cache. 캐시 라인이 4개라고 하면 페이지 넘버의 하위 두 비트가 00, 01, 10, 11 인지 확인, 어디에 매핑 되어있는지 확인하는 방법. 



<img src="{{site.baseurl}}/assets/img/oslec16/image-20200521162328208.png" alt="image-20200521162328208" style="zoom:33%;" />

하위 비트, 중간비트, 앞부분 비트를 따서 찾을수도 있는데, 왜 앞부분의 비트를 따는건 왜 안좋은 방법인가? 똑같은, 근처에 있는 페이지들이 전부 하나의 캐시 라인에 매핑이 될 것이므로. space locality 때문에 하나에 많이 들어가면 충돌이 날 가능성이 높아짐. 



3. 위의 캐시 라인에 전부 comparator 를 설치한다. 페이지가 들어오면 쭉 병렬적으로 비교해서 바로 찾아주는 방법. (Fully associative cache. 매우 빠르다.)



![image-20200521162535074]({{site.baseurl}}/assets/img/oslec16/image-20200521162535074.png)

Direct mapped cache 는 싸지만, 단점이 있다. 

* 공교롭게 00번 페이지 테이블에 많이 매핑이 되면 계속 이 페이지 라인이 교체되면서 hit ratio 가 떨어질 것이다. 
  * 이럴 때는 이 cache line 을 두 개의 세트로 나눈다. 그래서 끝자리가 0인 애는 위의 두개로 보내고 끝자리 1인것은 아래 두 개로 보내. 한 비트만 매핑 시키는 방법. 
  * 2 way set associative cache



<img src="{{site.baseurl}}/assets/img/oslec16/image-20200521162704732.png" alt="image-20200521162704732" style="zoom:33%;" />



![image-20200521162737640]({{site.baseurl}}/assets/img/oslec16/image-20200521162737640.png)





![image-20200521162752470]({{site.baseurl}}/assets/img/oslec16/image-20200521162752470.png)

TLB 의 캐시 라인 안에는 어떤 정보들이 있어야 할까? 

얻고자 하는 정보는 **Physical frame number.** 

그런데 cache 를 하게 되면 태그가 있어야 비교할 수 있음. 이 태그는 페이지 넘버. 그리고 밑에 여러가지 flag bit 들이 쫙 온다. 

어떤 Flag number 가 있는가? 

* uncached page 인지 에 대한 정보 
* dirty bit 인지. (swap device 에서 읽혀 온 후 modify 되었는지에 대한 정보. 나중에 swap out 할때 clean 하면 그냥 버려버림. swap device 에 있는 거랑 같으므로. 변경되면 그때 disk operation 을 함)

* 나중에 가상 주소 관리를 할 때 MMU를 매커니즘으로 활용해서 여러 정책을 세우는데 이 때 필요한 정보들을 저장함. 



TLB는 하드웨어인가 소프트웨어인가? 

* 하드웨어이다. 



TLB는 운영체제에게 투명한가 아닌가? 

* 운영체제가 무시하면 안됨
* 어떤 프로세스가 TLB에 자기의 페이지에 관한 정보를 다 채워넣음. 그 후에 Context switching 이 일어났는데, TLB에서 hit 가 나면 자기의 frame 정보가 아님. 따라서 문맥교환이 일어나면 운영체제는 TLB를 flush 시켜야 함. invalidate 시킴. 
* 문맥교환이 일어나면 empty TLB 때문에 메모리 레퍼런스가 늦어지는 문제 발생. 



근데 어떤 마이크로 프로세서에서는 TLB를 운영체제에 투명하게 만들었음. 

* MIPS 라는 것. 
* TLB의 캐시 라인에 PID 를 넣음. 프로세스 아이디. 그래서 TLB 룩업을 할 때 페이지 넘버만 태그로 사용하는 것이 아니라 지금 수행하는 PID까지 확인을 해서 valid 한 entry 인지 검증함. 
* 하드웨어 기능이 추가가 되면 운영체제가 어떻게 대처해야 할지 고려해야 함. 
* PID는 다른 말로 ASID라고도 부른다. Address Space ID. TLB의 캐시 라인 엔트리들은 주소 공간 별로 다른 아이디를 가져야 하므로 그렇다. 



