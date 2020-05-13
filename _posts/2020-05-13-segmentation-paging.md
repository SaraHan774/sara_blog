---
layout: post
title: "[OS] Segmentation & Paging"
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


# Lecture 15 : Segmentation & Paging

### 15-1 Segmentation 

4 단계에 걸친 스토리. 

1. Segmentation 
2. Paging 
3. Paged Segmentation 
4. Enhancing Mechanisms 
5. Case Studies 
6. Conclusion 

![image-20200513110035583]({{site.baseurl}}/assets/img/oslec15/image-20200513110035583.png)

**멀티 프로그램드 배치 모니터 시스템** 

* 동시에 active 한 Job 들이 메인 메모리에 존재할 수 있다. 
* 메모리의 한쪽 끝에는 Batch Monitor 가 있고, 나머지 영역들은 User process 를 위해서 사용되었다. 
* Dynamic Storage Allocation 이 떠오른다. Fragmentation 의 문제가 존재함. 
* 유저 프로세스가 하나의 메모리 세그먼트를 할당받아 발생하는 문제가 존재. 
  * 프로세스의 메모리에는 Data Section, Text Section, + Stack + Heap 이 있는데, 이 전체가 하나의 메모리 세그먼트에 매핑되는 문제가 있음. 
  * 하나의 메모리 세그먼트에 매핑하면 발생하는 문제는 ? => 링커가 프로세스의 exe 파일을 나눈 이유가 있다. 이들의 속성이 다르기 때문이다. Read-only, Read & Write 인지에 따라 메모리 섹션들을 달리 봐야 할 필요가 있다. Text Section 은 코드 섹션이기 때문에 수행중에 수정할 필요가 없다. 오히려 text section 을 수정하는 프로세스가 있다면 이는 에러이다. Data section 은 read write 가 둘 다 가능해야. Stack 과 Heap 도 마찬가지로 Read & write 가 되어야 하는 부분. 

**한 User Process 의 Memory Section 들이 하나의 Memory Segment 를 할당받을 때 발생하는 문제점** 

1. 두 User Process 가 동일한 코드 (Text Section) 을 공유하기 힘들다. 
2. 각 Memory Section 들에게 다른 Read Write 권한을 설정하기 힘들다. 



=> 속성이 다른 부분은 다른 세그먼트를 할당해주자. 

![image-20200513110636482]({{site.baseurl}}/assets/img/oslec15/image-20200513110636482.png)

Single Process 였을때 메모리 transition 은 base register 와 bound register 를 이용해서 했다. 메모리 세그먼트마다 별도의 base register 와 bound register 를 유지해야 했음. 

각 세그먼트마다 별도의 레지스터를 할당하려면 하드웨어적인 구현은 어떻게 해야하는가? 

* 프로세스당 몇 개의 세그먼트를 할당할지 정해두고, 각 세그먼트마다 다른 레지스터를 할당. 
* 대표적인 예가 과거의 인텔 프로세서. 세그먼트 레지스터 어드레스를 별도로 갖고 있음. 
* 좀 더 일반적인 형태로는 별도의 레지스터를 할당하지 않고, 메인 메모리 에다가 세그먼트의 base + bound 레지스터 페어를 테이블 형태로 기록하는 방식이 있다.

![image-20200513110934666]({{site.baseurl}}/assets/img/oslec15/image-20200513110934666.png)

=> Single segment 를 해결하기 위한 Multiple segment 는 좀 더 복잡한 연산들은 요구하게 되었음. 

![image-20200513111108220]({{site.baseurl}}/assets/img/oslec15/image-20200513111108220.png)



**Address Translation** 

**CPU 가 생성하는 주소 => MMU 가 변환시키는 주소 => 궁극적으로 Physical Memory 의 주소**

​					(Logical Address)								(Physical Address)

경우에 따라서는 Logical Address 를 Virtual Address 라고도 부른다. 

* MMU 의 기능이 중요하게 되었다. Table lookup 을 하는 장치. 
* 논리 주소는 프로그래머가 보는 주소이다. 요즘은 프로그래머들이 low level 프로그래밍을 하지 않기 때문에 컴파일러와 링커가 보는 주소를 우리가 보게 된다. 
* 0부터 시작하는 주소가 논리 주소. 이것이 CPU에서 나온다. 
* 과거의 인텔에서도 CPU와 MMU칩이 별도로 나왔었다. 실리콘 집적도가 낮았기 때문. 지금은 전체가 CPU칩 안에 들어가 있음. 



CPU 주소가 나오게 되면 MMU가 동작하도록 만들어 주어야 한다. 

MMU가 하는 첫 번째 일은 Table lookup. 세그먼트에서 나온 주소가 있으면 세그먼트 테이블을 뒤져서 해당 엔트리를 뒤져서 base address 를 가져와야 함. 

Instruction fetch 주소가 날라갔다, 이 명령은 text segment다 라는 것을 알아야함. 어떻게 아는가? 

* logical address space 를 어떻게 분할하는지 생각해보자. 
* 32bit 머신이 있다고 했을 떄 모든 주소가 0~ 2^32-1 만큼의 주소 공간을 갖는다. 링킹이 되는 모든 프로그램이 별도로 하나씩 갖고 있음. 
* 앞부분은 코드, 그 다음 부분은 데이터, 스택, unused -- 와 같이 영역을 분할한다. MMU 가 어떻게 섹션을 구분하는가? 코드 섹션은 상위 두 비트가 00, 그 다음은 01, 10, 11 과 같이 시작할 것이다. 
* 30bit 는 오프섹으로 사용될 것이고 상위 두 비트는 섹션의 아이디와 같이 사용된다. 



**세그먼트 테이블 안에는 무엇이 들어있는가?** 

* 메모리 세그먼트 Sharing 을 지원하기 위해서 어떤 정보가 필요할까? 

  * 세그먼트가 어디서부터 시작하는지에 대한 base address, 세그먼트 크기 bound, RW 에 대한 정보가 필요함. 

* Base Address 값은 논리 주소? 물리 주소?

  * 물리주소이다. 
  * 논리 주소라면 또 매핑하기 위해서 또 다른 테이블을 뒤져야 하니까 ...

* 세그먼트 테이블의 엔트리를 어떻게 인덱싱 하는가 까지는 알았는데, MMU 입장에서 하나의 정보가 없다. 무엇이 빠져있는가? 

  * MMU의 입장에서는 테이블을 뒤지려면 인덱스만 알아서는 안되고, 이 테이블의 시작주소를 알아야 한다. 
  * **MMU에는 special purpose register 가 있는데 이를 Segment Table Base Address Register 라고 부른다.** (STBR)
  * MMU는 SW가 아니라 HW이다. 
  * 그렇다면 MMU의 존재가 OS에게 transparent 한가? 당연히 transparent 하지 않다. 모든 프로세스는 자기만의 세그먼트 테이블을 갖고 있어야 하고, OS는 context switching 을 할 때 지금 들어오는 프로세스의 segment table 의 시작 주소를 MMU의 STBR 에다가 reload 해줘야 한다. CPU register 값들만 바꿔주는게 아니라 테이블도 바꿔줘야 함. 
  * 그래서 Context Switching 에 비용이 많이 든다는 것이다. 테이블까지 다 바꿔주어야 하기 때문에. (Paging 을 하게 되면 테이블이 커진다 ... ) 메모리에서만 가져오는게 아니라 디스크에서 가져와야 하는 경우도 생김. 

  

![image-20200513112515563]({{site.baseurl}}/assets/img/oslec15/image-20200513112515563.png)


  

Table driven segmentation 을 하게 되면 상위 비트를 통해서 세그먼트 넘버를 얻고, 그것을 인덱스로 하여 MMU가 갖고 있는 segment table base address register 를 통해서 세그먼트를 찾고, 인덱스를 통해 해당 엔트리를 찾아서 base register 값을 얻어온다. bound 와 비교해서 valid 한 주소인지를 확인한다. 



![image-20200513112942158]({{site.baseurl}}/assets/img/oslec15/image-20200513112942158.png)

세그먼테이션에는 여러가지 장점이 있다. 하지만 문제점도 있는데 ... 

* 메모리를 접근하기 위해서 2 번의 메모리 접근이 필요한 문제. 

  * MMU -> Table lookup -> Physical 주소 찾기  

* 당연히 세그먼트가 메인 메모리에 많이 존재하다 보니 fragmentation 문제가 발생한다. 

* Stack 이 더 자라면 자기의 공간을 넘어가면 Stack overflow 가 발생하다. OS 가 필요한 만큼 Stack segment 를 키워줄 수 도 있는데 ... 위 쪽이 비어있는 공간이면 가능하지만, 다른 segment 가 차지하는 공간이면 어떻게 하는가? 

  * 해당 segment 가 당장 사용되지 않는다면 Disk 로 swap 을 한다. 
  * 오히려 메모리를 더 필요로 하는 스택 세그먼트를 disk 로 보내고, 나중에 충분한 공간이 생겼을때 메모리로 불러들이는 방법도 있다. 

  

![image-20200513113337046]({{site.baseurl}}/assets/img/oslec15/image-20200513113337046.png)






![image-20200513113350119]({{site.baseurl}}/assets/img/oslec15/image-20200513113350119.png)

=> Segmentation 도 fragmentation 문제로부터 자유롭지 않다. 



### 15-2 Paging 

그래서 사람들이 고민을 많이 한 결과 나온 기법이 Paging 이다. 



Fragmentation 이 왜 생긴다고 했나? 

* 프로세스들이 동적으로 요구하는 메모리 크기가 그때그때 다르기 때문이다. (Memory allocation request 의 size 가 다르다)
  * Memory allocation request 가 들어오면 물리 주소 상에서 공간을 contiguous 하게 할당해야 한다는 문제도 있음. 
  * Memory allocation 할때 Unit size 로 Allocation 하면 fragmentation 은 없어진다. 
  * 요구하는 공간이 Unit 사이즈보다 커지면, 존재하는 Unit 사이즈 만큼 모아서 할당하면 된다. 



![image-20200513113835815]({{site.baseurl}}/assets/img/oslec15/image-20200513113835815.png)

* Segmentation 이 갖는 fragmentation 문제를 극복하기 위해 Paging 을 도입했다. 
  * **Paging : 메모리 할당을 페이지라 하는 Unit 사이즈로 하는 것이다.** 



* 여러 프로세스가 동시에 존재하면 메모리 공간이 모자라 swapping 을 해야한다. Segmentation 만 보면 segment 가 단위가 되는데 Paging 을 도입하면 swap 을 page 단위로 할 수 있다. 



용어 

* Page : Logical Address space 를 일정한 Unit size 로 나눠서 해당 블럭을 Page 라 부른다. 
* Frame : Physical address space 또한 일정한 크기로 나눈다. Page 와 크기가 동일한데 이를 Frame 이라고 부른다. 

<img src="{{site.baseurl}}/assets/img/oslec15/image-20200513114158397.png" alt="image-20200513114158397" style="zoom:50%;" />

> Paging : 메모리 Allocation, management 기법이다. 



각 프로세스들이 갖고 있는 메모리 공간들을 어떻게 할당하는지 공부해야 한다. 



![image-20200513114243112]({{site.baseurl}}/assets/img/oslec15/image-20200513114243112.png)

"일정한 크기"로 나눈다 

* 과거에는 0.5K 정도로 나누었는데, 요즘에는 점점 Page 크기가 커지고 있다. 
* 과거에 나눌때는 주로 swapping 을 염두에 두고 페이지 사이즈를 결정함. 
  * disk 같은 볼륨 디바이스를 block 디바이스라 부른다. 1byte 를 디스크에 적어도 그 바이트를 보함한 블럭 전체가 디스크로 가게 됨. 이 사이즈와 비슷하게 페이지 크기를 설정함. swapping 할 때 1page 로 스와핑 하려고. 
* 요즘에는 점점 물리 메모리 사이즈가 커지니까 1MB, 2MB 정도 단위까지 페이지 사이즈가 커짐. 



Address Translation 을 해야한다 

* Segmentation Table 에서 하듯이 Page Table 이라는 유닛을 이요해서 MMU가 Page Table 을 이용해서 Address translation 을 한다. 
* Paging 을 사용하는 경우 MMU가 해야하는 일은? 
  * Page Number 를 얻어오는 것이 첫번째 해야 할 일이다. 
  * 얻어오는 방법은 Segmentation 과 같다. 각 주소의 상위 비트를 페이지 너버로 사용하고 하위 비트를 페이지 오프셋으로 사용함. 
* 위의 예제에서 2^12 (4KB) 정도가 페이지 사이즈인것. 



![image-20200513114647654]({{site.baseurl}}/assets/img/oslec15/image-20200513114647654.png)

페이지 넘버를 이용해서 페이지 frame 넘버를 얻어오게 된다. 

이와 같은 Translation 을 하려면 MMU 에 Page Table Base Register (PTBR) 이 존재해야 한다. 

* PTBR : Page Table 의 시작 주소를 가리키는 MMU 의 register 

* Context Switching 할 때 위의 정보들도 같이 바꿔주어야 함. 



Segmentation 과 거의 같은 것이 아닌가? 

* 큰 차이 존재 : Segmentation 에서 세그먼트의 크기는 가변적이다. 이걸 표현하는게 bound register 이고. 페이징에서는 페이지의 크기가 다 일정하다. 이로 인해 fragmentation 을 없앨 수 있다. 



Internal Fragmentation 을 초래하는 Paging 

* Logical address space 의 마지막 페이지가 마지막 페이지의 사이즈와 딱 맞지 않을 수도. 이 문제를 internal fragmentation 이라 하는데, 페이지 사이즈가 작으므로 그렇게 심각한 문제가 아니다. 



![image-20200513114952706]({{site.baseurl}}/assets/img/oslec15/image-20200513114952706.png)

장점 

1. Fragmentation 해결 
2. Swapping 용이해짐 

단점 

1. Address reference 하나 날라갈 때마다 2번의 과정을 거쳐야 함 
2. 심각한 문제 : Segmentation 과 달리 페이지 테이블의 크기가 너무나 방대해진다는 문제. 



32bit 머신 - 한 워드가 32bit 라는 것. 

페이지 테이블의 크기도 한 워드에 align 하는게 좋다. 

MMU에 있는 Page Table Base Register 가 테이블의 시작 주소를 갖고 있다. 

(아까를 예로 들면) 2^20 만큼의 엔트리가 들어갈 수 있음. 가장 중요한 정보는 Physical address. 페이지가 저장된 physical address 를 다 저장해 버리면 추가로 뭔가 들어갈 수 없다. 하지만 다 저장할 필요는 없다. 12bit 는 페이지 오프셋으로 사용되기 때문에 첫 시작은 해당 비트들이 다 0이다. 이것들은 저장할 필요가 없으므로, 20bit 의 정보만 저장한다. 이 20bit 가 Frame Number 이다. 하위 12bit 가 남는데 이 공간은 그 페이지를 표현하는 여러가지 속성들을 넣는다. 이 페이지가 mapped인지 cached 인지  ... 기타 등등 여러가지 



<img src="{{site.baseurl}}/assets/img/oslec15/image-20200513115355707.png" alt="image-20200513115355707" style="zoom:50%;" />



![image-20200513115415523]({{site.baseurl}}/assets/img/oslec15/image-20200513115415523.png)

테이블 사이즈 커지는 문제 (예시)

<img src="{{site.baseurl}}/assets/img/oslec15/image-20200513115523960.png" alt="image-20200513115523960" style="zoom:50%;" />

페이지 사이즈가 1KB 이므로 10bit 를 오프셋으로 사용한다는 뜻. 따라서 테이블의 사이즈는 16MB

=> 굉장히 큰 크기의 페이지 테이블을 유지해야 하므로 Context Switching 의 오버헤드가 큰 것이다. 

당연히 16MB 의 페이지 테이블을 다 메모리에 넣을 수 없으니까 Disk Swap 을 해야 하는 상황이 발생하는 것. 

특히나 32bit 머신이 보편적이었을때는 메모리 사이즈가 작아서 더 오버헤드가 컸음. 



### 15-3 Paged Segmentation 

다음 단계는 paging 과 segmentation 을 결합시키는 paged segmentation이다. 

왜 필요한가? 

* Segmentation 은 어떤 문제를 해결했는가?
* Paging 만 한다는 것은 무엇을 뜻하는가? 
  * 프로세스당 한 개의 세그먼트만 제공한다는 것. 이건 원래 Segmentation 에서 개선하려고 했던 문제인데. 프로세스에게 multiple segment 를 할당하는 목적은 달성하지 못한다. 



