---
layout: post
title: "[OS] Demand Paging"
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


## Lecture 17 : Demand Paging 



### 17-1 Demand Paging 

#### Demand Paging 이란? 

메모리 관리 메커니즘 (MMU 메커니즘) 을 사용해서 여러 프로세스가 시스템의 메모리를 효율적으로 공유할 수 있도록 하는 기술. 



![{{site.baseurl}}/assets/img/oslec17/image-20200526124020739]({{site.baseurl}}/assets/img/oslec17/image-20200526124020739.png)

* MMU가 한 가장 큰 기여는? 

  * Logical memory 라는 것이 가능해졌다. 
  * Logical address space 를 도입하게 됨. 
  * 새로운 가상의 주소를 제공해 준다는 것은 논리적인 주소를 물리 주소로 변환시켜준다는 의미를 갖는다.

  

* 어떤 유저 프로그램이 컴퓨터에서 돌려면 그 프로그램이 필요로 하는 메모리 세그먼트들이 모두 물리 주소 상에 있어야 한다는 가정 하에 얘기해왔다. 꼭 필요한 것일까? 

  * 그렇지 않다. 왜냐하면 프로그램은 locality 를 가지고 있기 때문이다. 
  * Locality 를 가지면, 프로세스가 필요로 하는 모든 공간을 물리 주소 공간에 할당할 필요가 없다. 



* Principle of Locality (Locality of Reference) 란? 
  * 프로그램이 가장 최근에 접근했던 데이터를 다시 접근(Temporal Locality) 하거나, 최근에 참조했던 데이터 근처의 주소를 참조(Spatial Locality) 하는 경향이 있음. 
  * 근거 : 크누스의 실험. 프로그램의 임의의 순간에 세워보니, 10프로의 고정된 공간에 항상 있더라. 90프로의 시간동안. 



* 90/10 Rule 
  * 불과 10프로의 코드가 프로그램 총 수행시간의 90프로를 차지함. 
  * Loop 때문에 그렇다. 



* 현재 물리 주소 공간에 없는 세그먼트들은 swap device, swap space (disk) 에 있어야 한다. 
  * disk 와 swap 은 무슨 차이가 있는가? 혹은 file system. file 하고는 뭐가 다른거지 ? 
  * Disk 에는 디렉토리, 파일 등이 있음. Read, Write 시스템 콜에 의해서 파일의 블럭들에 접근하다. 하지만 파일의 실제 데이터에 접근하는 데 까지는 많은 운영체제 기능들이 수행되어야 한다. 파일의 이름을 물리 주소로 변환시켜야 하고, device driver 들도 실행시켜야 함. 그래서 시간이 오래걸림. 
  * 하지만 swap device 는 file 의 한 부분이 아니라 raw device 로 빠르게 접근할 수 있게 만든 영역임. (Raw disk block)



📌Swap Area (Swap Device) 란? : Physical Memory 에 저장되지 못한 Page 들을 저장하는 디스크의 공간으로 File 에 비해 운영체제가 직접 빠르게 접근할 수 있음. Swap Device 가 꽉 차는 경우 예외적으로 File 을 만들어서 Page 들을 저장할 수 도 있음. 



* physical memory 와 swap area 사이에는 많은 데이터 operation 이 발생한다. 

  * process 가 루프를 돌다가 나와서 jump 를 했는데, page 를 swap device 에서 읽어와야 하는 경우. 
  * user process 에서 swap device 로 write back 하는 경우도 있다. 새로운 Page 를 Physical memory 에 저장해야 하는데 빈 공간이 없고, physical memory 에서 쫓아내려는 Page 가 dirty page 인 경우. 

  

  => 이런 과정들이 빈번하게 일어나는 것은 좋지 않다. 

  

  어떻게 하면 실제로 두 메모리 사이의 block transfer operation 이 일어나지 않도록 하는가? 

  

#### Demand Paging Policy 의 목적. 

* **Physical Memory 와 Swap Device 사이의 데이터 전송이 최소한으로 발생하도록 함.** 



* Memory hierarchy : register - cache - memory - disk (가격은 싸지고 memory latency 가 높아지는 순서) 

  * register 는 cycle 단위, memory 는 ms 단위. disk 는 달나라를 갖다오는 정도. 

    

📌 Memory Access Latency 

* 레지스터 (1 사이클) < L1 캐시 (수 사이클) < L2 캐시 (수십 사이클) < Main Memory (수백 사이클) < Disk (수백만 사이클)



* Thrashing 
  * 사용자의 수가 증가하면 (x 축) latency (y 축)가 서서히 나빠지다가, 어느 순간을 넘어서면 급격히 나빠지는 것. Thrashing point 를 거치기 때문이다. 



📌 프로세스들이 사용하는 Page 들의 크기보다 사용가능한 Physical Memory 의 크기가 작을 때, 사용하려고 Swap - in 하는 Page 에 의해 앞으로 사용할 Page 가 Swap - out 되면서 반복적으로 Page Fault 가 일어나는 현상. 



* 예) 지금 들어있는 프로세스가 100개의 페이지가 필요한데 물리 주소 공간에는 99개의 페이지 프레임만이 존재한다. 100개가 필요하니까 어떤 것을 swap 으로 보내고, 100개를 줌. 근데 다른 곳에서 쫓아낸 페이지를 요청하면 또 swap 으로 가서 읽어와야 함. 계속해서 달나라를 가는 것 과 같다. 



![{{site.baseurl}}/assets/img/oslec17/image-20200526130733887]({{site.baseurl}}/assets/img/oslec17/image-20200526130733887.png)

* 지금까지는 논리 주소를 물리 주소로 translation 했다. 이제는 RAM 뿐만 아니라 Swap Device 로 가는 경우도 염두에 두어야 함. 



* Virtual Address space 가 의미하는 것은, 실제 존재하는 물리 메모리보다 더 큰 프로세스를 받아들일 수 있다는 것. 부족한 물리 메모리는 swap device 를 사용해서 충당한다. 



* Swap device 로의 접근 횟수를 최대한 줄이는 것이 우리가 가장 바라는 것. 
  * 가상 주소를 변환해서 RAM Address 로 최대한 가게 하려면 어떻게 해야 하지? 
  * (신과 같은 예지력이 있다면) 현재 접근될 페이지들이 모두 물리 메모리에 있게 하고, 당장 접근되지 않을 것들을 swap device 로 보내면 되지 않을까? 언제 보내냐? 이 swap device 가 idle 할때. 



📌 이상적인 Demand Paging 의 동작 : 앞으로 사용될 페이지와 사용되지 않을 페이지를 미리 알아서 Physical Memory 로 불러들이거나 Swap Device 로 내보내는 것. (Clairvoyant Replacement Algorithm)

=> 하지만 불가능하다. 적어도 이와 비슷하게 예측할 수 있는 방법에 대해서 알아볼 것. Demand paging 의 핵심. 



Page Table Entry 가 virtual memory management 를 지원하려면 타겟 주소가 RAM 이 아닐 때, 현재 메모리에 있다 없다 표시해주는 Residence bit 이 필요함. 

![{{site.baseurl}}/assets/img/oslec17/image-20200526131406334]({{site.baseurl}}/assets/img/oslec17/image-20200526131406334.png)

> 어떤 놈은 RAM 으로 가고 어떤 놈은 Swap Area 로 간다. 



![{{site.baseurl}}/assets/img/oslec17/image-20200526131415575]({{site.baseurl}}/assets/img/oslec17/image-20200526131415575.png)

> Demand Paging 의 근거는 Memory Access Latency 이다. 

* Disk 는 값이 싸지만 느리다. 
* Demand Paging 을 잘 하면 disk 를 빠른 메모리 장치처럼 사용할 수 있다. 
* Demand Paging 이 무너지면 발생하는 문제가 Thrashing 이다. 



![{{site.baseurl}}/assets/img/oslec17/image-20200526131601030]({{site.baseurl}}/assets/img/oslec17/image-20200526131601030.png)

> 페이징이 가능한 핵심 원리 - spatial locality 와 temporal locality. 





#### 17-2 Page Fault Handling 

디맨드 페이징을 할 때 다루어야 하는 중요한 이슈 중 하나가 Page Fault Handling 이다. 



![{{site.baseurl}}/assets/img/oslec17/image-20200526133701321]({{site.baseurl}}/assets/img/oslec17/image-20200526133701321.png)



* Page Fault 
  * interrupt 할 때 배웠다. 
  * SW 인터럽트를 Trap, 혹은 fault, exception 이라고 부른다. 
  * **Virtual Address 가 가리키는 Page 가 Physical Memory 에 없어서 프로세스가 더 이상 진행할 수 없는 상태를 운영체제에게 알려주는 인터럽트.** 



* Page Fault 가 뜨면 인터럽트 서비스 루틴이 뜨는데 이것을 Page Fault Handler 라 한다. 
  * Page Fault Handler 가 뜰때는 Fault 를 만든 프로세스의 수행을 중단시키고, 해당 프로세스의 context 를 모두 저장한다. 이 프로세스는 sleep 상태가 되고, disk operation 이 끝나기를 기다린다. 



* Page Fault Handler 는 무엇을 해야하는가? 
  1. DMA Controller 에게 해당 Page 의 주소를 전달 
  2. DMA Controller 는 해당 Page 를 Physical Memory 로 로드 
  3. DMA Controller 의 작업이 끝나면 인터럽트 발생
  4. 운영체제는 Page Fault 를 일으킨 프로세스의 수행을 재개



* Resident bit 를 페이지 테이블 엔트리에서 확인하고, 페이지의 존재 여부를 판단한다. 

* Page fault 는 이 bit 가 off 되어 있을 때 발생한다. 

![{{site.baseurl}}/assets/img/oslec17/image-20200526134356353]({{site.baseurl}}/assets/img/oslec17/image-20200526134356353.png)



📌 Page Fault Handling 의 어려운 이슈 

인터럽트는 항상 인터럽트 시그널이 걸리고 instruction 의 수행이 다 종료되고 난, 새로운 instruction 이 수행되기 직전에 인지한다. 만약 instruction 의 수행 중간에 인지하게 된다면 machine state 가 애매모호하게 된다. 그리고 이 instruction 을 다시 반복하기 어려울 수 도 있다. 

예를들어, MOVE(SP)+, -(R2) 

stack pointer 가 가리키고 있는 메모리 주소의 값을 R2 라는 레지스터가 가리키고 있는 메모리 위치로 copy 하라는 instruction. 그 직전에 R2 의 포인터 값을 하나 감소시켜야 한다. 이 auto decrement 가 수행되기 직전에 인터럽트가 있었는지, 직후에 있었는지 판단이 어려울 수 없다. 마이크로프로세서 입장에서 보면 인스트럭션이 수행의 최소 단위이기 때문이다. 

단, 예외는 page fault 이다. instruction 이 수행될 때 memory reference 가 최소 1번 (memory fetch) 혹은 2번 이상(memory access 하는 instruction 인 경우, operand 개수만큼) 일어난다. 따라서 매번 memory reference 마다 page fault 가 발생할 수 있는 것. 그래서 위의 예제에서 보면, stack pointer 가 갖고 있는 메모리를 참조할 때도, R2 를 참조할때도 page fault 가 발생할 수 있다. 이런 경우에는 instruction 을 restart 하기 어려울 수 있다. 



📌 RISC 는 restartable 하게 설계된다. 



💡 정리 : 과거에는 프로그램이 수행될 때, 프로그램과 관련된 메모리가 전부 물리 메모리 안에 잡혀있어야 했다. 하지만 이렇게 하면 비싸고 locality 때문에 효율적이지도 않다. RAM 의 사이즈는 조금 가져가는 대신 싼 Disk 를 swap device 로 사용하는 기법이 virtual memory management이다. 이 기법은 물리 주소를 가상 주소로 매핑시켜주는 MMU 라는 하드웨어 지원이 필요한 기법이다. 



![{{site.baseurl}}/assets/img/oslec17/image-20200526135427975]({{site.baseurl}}/assets/img/oslec17/image-20200526135427975.png)



swap device 를 사용할 수 없는 과거 상황으로 돌아가는 경우도 있다 -> smart phone, digital TV 

smart phone 안에는 disk 가 없음. 스마트폰 안에는 flash memory 를 사용함. 

* flash 안에다가 swap 을 하면 안되는 이유는? 
  * flash 는 erasable ROM 이다. 종이위에 글을 썼다 지웠다 하면 닳아버리는 장치. 어떤 경우에는 스마트폰의 수명만큼 ROM 이 견디지 못할수도 있음. 
  * swap 없이 매니징 하는게 과거처럼 다시 중요해졌다. 



* 어떻게 하면 될까? 
  * 그냥 프로그램을 죽인다. 



* 하드웨어 시스템의 변화에 따라 운영체제가 대응해야 할 문제들이 생겨난다. 



⭐ 우리가 원하는 것은 page fault rate 를 최소화 시키는 것이다. 



Demand Paging 을 여러가지 관점으로 나눠볼 수 있다. 

1. 어떤 페이지를 언제 메모리로 불러 들일 것인가? page selection policy. 예) 근처에 있는 것을 불러들인다. 
2. 페이지 교체 정책. victim 을 골라서 swap device 로 보내는데, 누구를 보낼건지. 
3. replace 할 때 어느 풀에서 고르느냐. 전체 혹은 일부 프로세스 그룹? 
4. page allocation style. 페이지들을 어떤 프로세스에게 할당할 때 어떻게 할당을 하겠느냐. 



=> 각각의 정책에 따라 성능에 다른 영향을 미치게 된다. 

