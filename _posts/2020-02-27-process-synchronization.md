---
layout: post
title: "[OS] Process Synchronization"
author: "Sara Han"
categories : OperatingSystem
comments : true
---

# Process Synchronization, semaphore, deadlock


```
[DISCLAIMER]
서울대학교 홍성수 교수님의 운영체제 강좌 필기입니다. 
본 PPT 캡처가 문제될 시 즉시 삭제하겠습니다. 
```

* [SNU etl 에서 수강 가능합니다](http://etl.snu.ac.kr/)



**2020-02-27**

CPU 코어의 수는 계속해서 증가하고 있다. 프로그램을 빨리 돌리는 것이 목적. 

**하지만 CPU의 성능이 늘어나는 것에 비례해서 성능이 증가하지 않는다.** 

**Process Synchronization 의 문제 때문이다.** 

1:1 로 증가는 못해도 linear 하게 증가했으면 좋겠는데 그러지도 않는다. 

- lock up

스마트폰이 입력에 반응하지 않고 freeze 되는 문제. 해결은 하드 리셋뿐. 껐다가 키기. 

왜? 인풋으로 key 를 누르면 → 인터럽트가 발생 → 하지만 처리되지 않아서 앱으로 전달이 안되는 것. sync가 꼬이면 deadlock 이 발생. 끔찍한 버그. 

- synchronization 어원

sync = together 

chro ~ = timely

시간 적절하게 함께 하는 것. 

- 왜 프로세스는 interaction 해야 하는가?
1. 프로세스는 Design time entity 라고 했다. 디자인 할 때 모듈러 하게 디자인 한다. 또한 runtime entity 일 때에도 **modular execution 이 필요하다**. 
2. **degree of concurrency** 를 높이기 위해서 필요하다. 
3. 초기 batch 모니터 시절 uniprogramming 에서 multiprogramming 으로 옮겨갔듯이 자원이 idle 할 때는 공유 하는 것이 바람직하다. **resource efficiency 를 위해서 필요하다.** 

- 겉으로 보기에 상관 없는 프로세스들도 상호작용 한다.

![{{site.baseurl}}/assets/img/process_synchronization/Untitled.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled.png)

같은 컴퓨터 안의 전혀 다른 듯한 두 프로세스도 자원 측면에서는 같은 자원을 공유하고 있을 수 있으므로 상호작용 해야 한다. 

다른 컴퓨터 이더라도 이더넷으로 연결 되어서 socket communication 을 한다고 하면 자원을 공유한다고 할 수 있다. 

**즉, 기능적으로 상호작용 없는, 논리적 연관이 없는 프로세스들도 서로 상호작용을 많이 함을 알 수 있다.** 

- 프로세스간 상호작용

자원을 공유한다. global information 도 sharing 의 대상이다. 

시스템 상태를 공유한다. Main memory 등. 

**⇒ 문제점 : 프로세스의 수행이 non-deterministic 해진다. 즉, process 의 수행 결과가 reproduce 되지 않아서 디버깅 할 때 어려움이 발생한다.** 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%201.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%201.png)

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%202.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%202.png)

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%203.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%203.png)

- 데이터 공유 문제 예시

**인터럽트 루틴과 태스크 코드가** 서로 소통하기 위해 한 개 이상의 변수들을 사용할 수 있다. 이런 경우 데이터 공유 문제가 발생하는데 이는 일종의 동기화 문제이다. 

이런 task 들은 non-reentrant code 라고 불리운다. 

**⇒ Interrupt service routine 과 process 가 함께 돌면서 올바르지 못한 결과를 내는 코드 = non-reentrant code !** 

- Reentrant Code (재진입 가능 코드)

여러 프로세스들에 의해 동시 호출되거나 이전 호출이 완료되기 전에 반복 호출되어도 올바르게 수행되는 코드. 

프로세스간의 동기화 & 인터럽트 서비스 루틴과 프로세스간의 동기화 

**원자로 온도 센서 값 모니터링 프로그램** 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%204.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%204.png)

→ 온도 값을 배열에 담고, 무한 루프를 돌면서 값을 읽어서 값이 다르면 알람을 울린다. 

📌 이 프로그램은 제대로 동작하지 않는다! 

**Reentrant Code : 여러 프로세스들에 의해 동시 호출되거나 이전 호출이 완료되기 전에 반복 호출되어도 올바르게 수행되는 코드.** 

- 어떻게 Reentrant 하게 만들까?

읽어오는 구간을 non-breakable 하게, atomic 하게 만들어야 한다. 

인터럽트를 disable, enable 하면 그 사이에 있는 코드는 동기화 문제가 발생하지 않는다. 

⇒ 문제점 : 너무 강력한 조치이다. 마치 하나의 신호등이 고장났다고 모든 신호등을 초록불로 바꾸는 듯한 그런 선택... 

- Atomic operation 을 위해 하드웨어적 서포트가 있다.

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%205.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%205.png)

⇒ 두 값을 비교하는 곳에서 인터럽트가 일어날 수 있다. 문제 해결되지 않음. 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%206.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%206.png)

⇒ 코드를 읽는 동안 인터럽트를 disable 시키면 이 구간은 atomic 하게 된다. 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%207.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%207.png)

atomic operation 이 필요하게 된다. 

- 동기화의 중요성 예시

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%208.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%208.png)

⇒ 프로세스 여러개가 하나의 버퍼를 사용하는 경우, flag 를 도입해서 (예를들어 BufferIsAvail 이라는 변수) flag 값이 true 인 경우에만 들어가고, 프로세스가 이 flag 를 다시 false 로 셋팅해 놓음으로써 그 섹션에서는 하나의 프로세스만이 수행될 수 있도록 한다. 

하지만 프로세스1이 아직 false 로 변수 값을 바꾸지 않은 순간 다른 프로세스가 flag 가 true 인 것을 보고 지입하면 아예 엉터리 계산 결과가 나와버린다. 

**⇒ 이처럼 시스템의 Correctness 에도 큰 영향을 미치게 된다.** 

- Race Condition

여러 프로세스들이 동기화 절차를 거치지 않고 동시에 자원을 사용하기 위해 경쟁하는 상태. 

- 프로세스와 프로세스간 atomic operation 은 어떻게 해야 하는가?

Single Process system 의 경우 인터럽트를 disable, enable 하는 게 만병 통치약. 언제나 atomic 하게 된다. 

**⇒ 너무 극단적인 방법이므로, Critical Section 만들어**

어떤 코드 섹션을 critical section 으로 만들어서, 내부에서 Mutual exclusion 일어나도록 해야 한다. 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%209.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%209.png)

⇒ 자원을 공유하는 구간을 atomic 하게 만든다. 

- Mutual Exclusion

하나의 사람, 혹은 하나의 프로세스만이 한 순간에 특정 일을 하는 것을 보장하는 메커니즘. 

- Critical Section

어떠한 코드의 부분 혹은 연산의 집합으로, 이 부분은 한 번에 하나의 프로세스만이 실행시킬 수 있다. 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2010.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2010.png)

- Mutual Exclusion 을 위한 필요 조건

오직 하나의 프로세스만이 Critical Section 안에 진입할 수 있다. 

만약 여러개의 프로세스가 동시에 요청하면 하나의 프로세스가 진입하도록 허용해야 한다. 

Critical Section 바깥의 프로세스에 의존적이면 안된다. 

### ⭐ Semahphore!

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2011.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2011.png)

- 세마포어

네덜란드의 교수인 다익스트라 님 께서 고안한 Mutual Exclusion 메커니즘. 

매우 정확히 이해하고 있어야 한다. 구현도 할 줄 알아야 함. 

- 예시

어떤 회사가 프린터를 사용하는데 네트워크로 연결을 해 두니 내용물이 가끔 겹쳐서 소용이 없게 되더라. 그래서 아예 프린터 방을 하나 만들고 수위아저씨를 고용해서 열쇠를 맡겼다. 들어가서 프린트 하려면 열쇠를 받고→ 문을 열고 → 프린트를 해야 함. 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2012.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2012.png)

자신의 순서가 아닌 사람은 벤치에 앉아서 기다려야 한다. 

**세마포어의 정의 : Synchronization 을 제공해주는 integer 변수로, 열쇠가 있음 - 없음 만 표현한다. (정확히 말하면 binary semaphore 의 정의)**

- 사용자 입장에서는 수위아저씨와 대화할 수 있는 API가 필요하다.
1. 열쇠 줘 2. 열쇠 받아 

**lock, unlock** 

**wait, signal ⇒ State transition 을 야기. 스케줄링의 관점! 따라서 Semaphore 는 state transition 을 야기한다. Control transfer 의 관점도 됨. 사용해야 할 다음 프로세스를 찔러서 깨워주는 것.** 

**P(S), V(S) ⇒  P operation, V operation 이라고 하는데 네덜란드어 앞 글자를 따서 만든거임.** 

- Mutual Exclusion 보다 더 많은 일을 하게 된다.

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2013.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2013.png)

- 세마포어로 어떻게 프로그래밍을 하는가?
1. 어떤 데이터가 공유되고 있는지 파악한다. 즉 세마포어가 몇 개 필요한지, 사용하는 자원을 종류만큼 준비해야 함. 
2. **자원의 인스턴스 개수만큼 세마포어를 초기화 해야한다. 초기화 하는 값은 binary semaphore 의 경우 0과 1로 초기화 하고, Counting Semaphore 의 경우 0, 1, 2 등과 같이 설정한다. 이때 초기값은 `열쇠가 있음` 상태여야 하므로 `있음` 상태를 나타낼 수 있는 정수 값으로 초기화 해야 한다.** 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2014.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2014.png)

- 세마포어가 스케줄링 목적으로 사용될 때

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2015.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2015.png)

⇒ Task 를 Sleep 상태로 해 두고 인터럽트 서비스 루틴이 V operation 으로 이를 깨우는 방식. 

초기값을 0으로 주어야 Task 를 Sleep 에 들어가게 할 수 있음. 

**문제점 : 코드의 시작과 끝이 위의 세마포어처럼 Pair 로 사용되는 것이 아님. (코드 위 아래로 존재하는게 아니라 따로 떨어져 있음) 따라서 디버깅 할 때 어느 부분이 빠졌는지 확인하기 매우 어려워짐.** 

- Producer, Consumer 의 예시

Producer 는 버퍼에 write 를 하려고 하고, Consumer 는 버퍼에서부터 읽으려 한다. 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2016.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2016.png)

- 공유되는 자원은
    - data
    - buffer

    **이렇게 두 가지 있으므로 buf_avail, data_avail 이란 세마포어를 도입한다.** 

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2017.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2017.png)

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2018.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2018.png)

- Dining Philosophers

![{{site.baseurl}}/assets/img/process_synchronization/Untitled%2019.png]({{site.baseurl}}/assets/img/process_synchronization/Untitled%2019.png)

→ 세 명의 철학자가 있다. 그들은 thinking → hungry → eating 과 같은 state transition 을 거치는데, eating 을 할 때 포크를 반드시 두 개를 사용해야 한다. 

deadlock 의 예시.