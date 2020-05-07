---
layout: post
title: "[OS] Dynamic Storage Allocation"
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


## Dynamic Storage Allocation 

스마트폰의 lock-in 문제, Dynamic Storage allocation 이 잘 못되어서 메모리가 고갈되어 스마트폰의 성능이 나빠지는 문제. 

### Agenda 

1. Background 
2. Heap 
3. Garbage Collection
4. Conclusion

### 14-1 Why Dynamic Allocation ?

배경을 설명하고, DSA 의 대표적인 자료구조인 Heap, Heap 이 초래하는 Fragmentation 의 문제, 이를 극복하는 방법에 대해 설명한다. 

![image-20200507192759738]({{site.baseurl}}/assets/img/image-20200507192759738.png)

* Dynamic 이라는 것은 어떤 의미가 있는지? 
  * 반대되는 말은 Static ~ 이다. Static 이라는 것은 무슨 뜻인가? 컴퓨터 사이언스에서는 execution 전의 것을 static 이라고 한다. 같은 말로 pre-runtime, offline 이 있다. 
  * Dynamic 은 runtime 에 이루어지는 여러 행위들을 의미한다. 

* Dynamic Storage Allocation 은? 
  * Compiler 는 static 한 것의 일종. 
  * Static Storage Allocation 은 프로그램 수행 전 메모리가 이미 할당되는 것. 

* 한 프로세스가 갖게 되는 메모리 세그먼트에는 코드 세그먼트, 데이터 세그먼트, 스택 세그먼트, 힙 세그먼트가 있다. 
  * Stack 세그먼트는 동적으로 할당된다. (Dynamic Allocation)
  * 코드 세그먼트와 데이터 세그먼트의 위치는 로딩을 해서 메모리에 위치가 잡혀야 주소가 결정되는것이 아니라, 링커가 이미 주소를 다 할당한다. 실제로 메모리에 올라가 있지는 않지만 주소를 할당받는다는 것은 storage allocation 이 다 된것이다. 

* **Static allocation 특징 : 변수의 lifecycle 이 프로그램의 시작과 종료와 일치한다.** 
  * 글로벌 변수에 배열을 많이 잡는 행위가 대표적인 static allocation 이다. 우리가 공부할 것은 dynamic allocation. 
  * 즉, stack 과 heap 에 잡히는 메모리 공간에 대해서 본다는 것. 

* 왜 이런 Dynamic Allocation 이 필요한가?

![image-20200507193413500]({{site.baseurl}}/assets/img/image-20200507193413500.png)

프로그램 안에서 함수를 호출했을때 함수가 필요로 하는 local 변수들을 담아주는 공간이 필요하기 때문에 stack이 필요하다. 

어떠한 함수가 호출이 되면 매개변수, 리턴값 등을 위해 Stack에 공간이 잡힌다. 이 공간을 그 함수의 activation record 라 한다. 

* **Activation Record (Stack Frame 또는 Activation Frames): Procedure 가 수행되기 위해 필요한 정보들을 저장하고 관리하기 위해 Stack 에 저장되는 데이터 구조.** 
  * 이런 Activation record 는 함수가 불려져서 수행이 끝날 때 까지만 의미있는 값을 유지한다. 함수 호출과 궤를 같이한다. 
  * 예를들어 main() -> A() -> B() 순서로 함수를 호출하고 다시 리턴되어서 main 에서 C() 를 호출했다고 했을 때 Stack 에 순서대로 쌓이고, 가장 최근의 레코드가 수행이 끝난 수 차례대로 사라진다. Stack 의 push, pop 연산과 일치한다. 

Dynamic => 예측 불가능성. on demand 로 필요한 만큼만 공간을 할당해 주어야 한다. 함수의 calling sequence 가 어떻게 될 지 모른다.

프로세스가 생성이 되면 그 프로세스 관리를 위해서 OS 가 유지해야 하는 고유 정보들을 PCB에 저장한다. 과거 OS 가 단순할때는 PCB를 배열로 잡아서 썼다. 그러면 프로세스를 사용자가 배열의 길이만큼만 생성할 수 있는 한계가 생김. 예측 불가능성을 줄일수는 있지만 그렇게 좋지는 않다. 더 adaptive 하게 대응하려면 PCB를 배열에 할당하는게 아니라 동적으로 링크드 리스트에 할당하는 방법이 있다. 배열보다는 복잡한 구조지만 adaptive, dynamic 하다. 

링크드리스트같은 on demand 구조로 PCB를 유지하면 어떤 장점이 있는가? 메모리가 감당할 수 있는한 언제든지 공간을 할당할 수 있음. 

![image-20200507194145135]({{site.baseurl}}/assets/img/image-20200507194145135.png)

1. Stack 사용 
2. Heap 사용 

두 가지 방법이 있다. 

Stack 의 단점 : Top 에서만 뺄 수 있다. activation record 를 관리할때는 좋을 수 도 있지만. 

Heap 의 단점 : Flexible 하지만 ... 

두가지 핵심 연산 1. allocate, new 연산자 2. deallocate, free, release ... 

![image-20200507194332072]({{site.baseurl}}/assets/img/image-20200507194332072.png)

### 14-2 Heap Management 

Heap 은 on demand 로 프로세스가 필요한 만큼 메모리를 할당할 수 있다는 점에서 좋은 자료구조이다. 

![image-20200507194417092]({{site.baseurl}}/assets/img/image-20200507194417092.png)

예를들어보자. 

if (C) then use 100 data items 

else use 1000,000 data items 

이런 경우 static 하게 배열을 사용한다면 공간이 낭비되지만 malloc 을 하면 더 효율적으로 메모리를 사용할 수 있다. 이 때 Heap 을 사용한다. 

Heap 을 쓸 때, C에서는 malloc() 함수가 있어서 매개변수에 필요한 공간을 할당하면 된다. C++, Java 에서는 new 키워드로 가능하다. 

Heap 은 그냥 big chunk of memory block. malloc, free 와 같은 연산을 지원한다. 

메모리를 할당하면 일정 공간들을 프로세스에 순차적으로 할당해준다. 어떠한 프로세스가 free 를 하게 되면 Heap 의 중간 중간에 빈 공간들이 생겨난다. 

처음에는 contiguous 한 메모리 블럭으로 시작했으나, malloc, free 를 반복하면 중간 중간에 빈 공간이 생기는 것. 이런 작은 조각들을 어떻게 관리하면 되는가? 

작은 조각들을 빨리 찾아서 필요로 하는 프로세스들에 전달해 줄 수 있어야 한다. 일부를 포인터로 사용해서 다음 available 조각들을 이어서 linked list 로 구성한다. 가장 첫 시작을 가리키는 포인터도 갖고 있어. Free List 를 유지하는 것. 

![image-20200507194857396]({{site.baseurl}}/assets/img/image-20200507194857396.png)

Free 함수의 역할 : Free List 로 반환되는 공간이 기존 가용한 공간과 병합 가능한 경우 하나의 큰 공간으로 병합하고 아닌 경우 새로운 포인터를 추가한다. 

* Merge 하기도 하고, 안되는 경우 새로운 Free Node 를 추가하는 연산을 함. 

하지만 malloc 과 free 를 많이 반복하게 되면 원래 큰 덩어리였던 메모리에 작은 구멍들이 생기게 된다. hole 들은 궁극적으로 계속해서 더 작아질 수 밖에 없다. 이와 같은 현상을 Fragmentation 이라 한다. 

![image-20200507200337272]({{site.baseurl}}/assets/img/image-20200507200337272.png)

Fragmentation 이 많이 일어나면 전체 available 한 공간이 있다 하더라도 프로세스의 요청을 들어줄 수 없는 경우 생겨. 

![image-20200507200437033]({{site.baseurl}}/assets/img/image-20200507200437033.png)

malloc 20K 를 한다 하면 연속된 공간이 없으므로 요청을 들어줄 수 없다. 

이런 경우 어떻게 해야 하는가? 가장 쉬운 방법은 그 프로세스를 sleep 시키는 것. 다른 프로세스가 갖고 있던 블럭들을 반환 받아서 merge 되기를 기다리며 ... 

중요한 앱을 하나 띄워야 하는데 이런 fragmentation 문제 때문에 잘 안돌아가면 유저 사용 경험에 좋지 않아. Smart device 의 user perception 에 악영향을 준다. 

조금 조금씩 프로세스들이 메모리를 점유하면서 내놓지 못하고 sleep 하게 되는... 이도 저도 못하는 상황까지 갈 수 도 있다. 

* Dynamic Storage allocation 에 사용되는 Heap 은 Fragmentation 이라는 치명적인 문제를 갖고 있다. 이 문제가 발생하지 않도록 최대한 노력해야 한다. 
* Fragmentation 을 최소화 할 수 있는 많은 기법들이 제안됨. 

### Buddy Allocator 

**리눅스에서는 Slab Allocator 라는 것도 같이 사용한다. 

Buddy allocator 또한 heap 이다. 최초 메모리 덩어리 사이즈가 2의 지수승 만큼 잡힘. 

예를들어 2^8 이라고 했을 때, malloc (33) 을 했다고 생각해보자. 2^5 < 33 < 2^6 이다. 버디 애로케이터는 이 요구를 들어줄 수 있는 만큼의 공간을 찾는다. 

2^8을 두 개로 쪼개면 2^7, 또 쪼개면 2^6 이므로, 64 만큼의 공간을 이 프로세스에 할당하게 된다. 

malloc 이 들어오면 free 공간을 친구를 맺을 수 있게 계속 쪼갠다. 요청을 만족시킬 수 있을 만큼의 공간이 나올때까지 계속해서 2등분 하는 것. 왜 이렇게 하는가? 

다름에 어떤 프로세스가 malloc 45 를 했다고 생각해보자. 얘는 다음 2^6 만큼의 공간을 할당받게 된다. 

![image-20200507201300270]({{site.baseurl}}/assets/img/image-20200507201300270.png)

이렇게 하면 좋은 점이 free() 했을 때 두 개의 빈 공간을 묶어 큰 공간을 만들 수 있다. 

**Buddy Allocator 의 Free : 인접한 Buddy 들이 가용 상태인 경우 이들을 병합함으로써 쉽게 큰 크기의 가용한 메모리 공간을 확보할 수 있다.** 

굉장히 큰 메모리 덩어리들로 복구해 나가는 것. 

![image-20200507201454844]({{site.baseurl}}/assets/img/image-20200507201454844.png)

Free 리스트를 관리 : 통상적으로는 linked list 로 관리한다. 혹은 bitmap 을 사용. 

주로 disk allocation 할 때 bitmap 을 사용한다. 

runtime 에 어떤 프로세스가 메모리 요청을 했을 때 어떤 정책에 의해서 메모리를 부여하는지도 연구가 많이 되었다. 

Free list 에서 malloc 을 할 때 좋은 정책을 사용하면 되지 않을까? 하는 생각들. 

Free list 에 각종 사이즈의 메모리 조각들이 있고, 어떤 프로세스가 malloc 을 했을 때 어떤 순서대로 이 조각들을 할당해 주면 좋을까? => Best fit, first fit, worst fit. 

Free list 를 쫓아가면서 여분이 가장 작은 것을 찾아서 allocation 해주는 것이 best fit 이다. runtime 에 linear 한 연산이 필요. 리스트 순회하면서 찾아야 하므로. 이를 오래하게 되면 hole 의 사이즈가 굉장히 작아질 것. 작은 hole 들이 많아진다. 

First fit 을 하면 평균적인 사이즈의 구멍들이 많아질 것이다. 두가지 별로 마음에 드는 방법은 아님. 

그래서 차라리 큰 구멍을 만들자 ... 해서 나온게 worst fit. 이쯤되면 아무 소용이 없다... 

![image-20200507201927929]({{site.baseurl}}/assets/img/image-20200507201927929.png)

그래서 memory fragmentation 의 문제는 스케줄링과 달리 이런 policy 로는 해결이 어려운 문제라는 것을 알수있음

![image-20200507201953413]({{site.baseurl}}/assets/img/image-20200507201953413.png)

- 주로 free list implementation 을 얘기하는 것. 

### 메모리 풀을 사용하는 방법

왜 fragmentation 이 발생하는지 생각해보자. 

=> 결국 hole 과 메모리 요청 사이즈가 딱 맞지 않아서 발생하는 문제이다. 

그렇다면 이를 피하려면 어떻게 해야 하는가? 딱 맞게 만들면 되지 않을까? 

=> 메모리를 요청할 때 Unit 사이즈로만 요청하도록 하는 것이다. 1KB 단위로만 요청하도록 하면 fragmentation 은 발생하지 않는다. 하지만 문제는 이보다 더 큰게 필요한 경우 또 문제가 생겨 ... 

문제의 원인에 기반해서 만든 해결책이 메모리 풀이다. 

Unit 사이즈로 메모리를 요청하도록 하는데 대신 다양한 사이즈를 허용한다. 주로 2의 지수승을 단위로 Pool 을 구성한다. 

![image-20200507202330174]({{site.baseurl}}/assets/img/image-20200507202330174.png)

동일한 pool 에 있는 노드들의 사이즈는 같다. 이러면 fragmentation 은 발생하지 않겠지만, 1K 짜리 풀이 다 비고 나머지는 차 있는 불균형의 문제가 있다. 하지만 그나마 현실적인 대안으로 평가됨. 

![image-20200507202419773]({{site.baseurl}}/assets/img/image-20200507202419773.png)

---

### Summary
* 프로세스를 수행하는 데 있어서 얼만큼의 메모리가 필요할지 모를수 도 있다. 따라서 static storage allocation 만으로는 한계가 있기 때문에, 실행 도중에, runtime 에 동적으로 메모리를 할당해주는 방법이 필요하다. 
* DSA 를 위해 지원하는 연산 : `allocate()`, `free()` 가 있다. 
* DSA 는 Heap 을 사용해서 구현하다. 하지만 Heap 에 계속해서 malloc 과 free 연산을 반복하면 memory fragmentation 문제가 발생한다. Heap 에 작은 구멍들이 생겨 프로세스가 필요한 메모리 공간을 할당받지 못하는 문제이다. 
* 문제 해결 방법 
    1. Buddy Allocator : 2의 지수승 단위로 메모리 공간을 할당해준다. 
    2. 메모리 풀을 이용 : 2의 지수승 Unit 만큼 공간을 할당하도록 한다. 1KB, 2KB, 4KB 과 같은 공간들에 대한 링크드 리스트를 만들어 두고 필요한 만큼 할당해 주는 방식. 단점은 만약 한 Unit 이 많이 요청되었을 때 리스트에 불균형이 일어날 수 도 있다는 점이다. 