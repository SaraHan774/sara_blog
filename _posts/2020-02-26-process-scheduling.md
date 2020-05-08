---
layout: post
title: "[OS] Process Scheduling"
author: "Sara Han"
categories : OperatingSystem
comments : true
---

# Process Scheduling


```
[DISCLAIMER]
서울대학교 홍성수 교수님의 운영체제 강좌 필기입니다. 
본 PPT 캡처가 문제될 시 즉시 삭제하겠습니다. 
```

* [SNU etl 에서 수강 가능합니다](http://etl.snu.ac.kr/)



**2020-02-26**

![{{site.baseurl}}/assets/img/process_scheduling/Untitled.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled.png)

복습 

자원에는 preemptive resource 와 non-preemptive resource 가 있다. 

스케줄링의 단위는 CPU Busrt 이다. 

스케줄링 정책 ⇒ SJF 가 가장 Optimal 하다. 하지만 CPU Burst 를 찾아야 하는 어려운, 불가능한? 문제가 있다. 

따라서 차선책으로 FIFO 를 고안. 먼저 온 것을 먼저 스케줄링하는 알고리즘. 문제는 SJF 를 잘 처리하지 못한다는 단점이 있다. 

좀 더 Preemptive 하게 만든 것이 Round Robin 이다. CPU Burst 가 아니라 정해진 인터벌만큼 CPU를 점유하도록 하고, 그 시간이 끝나면 큐의 맨 뒤로 돌아가게 한다. 

**Round Robin 을 통해 FIFO 읜 한계를 극복하는것인가? NO.** 

Round Robin 일때 각 프로세스의 평균 Response time = 1000ms

FIFO 는 100, 200, ... 1000 이렇게 들어왔을 때 합의 평균이 RR 보다 훨씬 작아. 

workload 의 패턴에 따라 다른 것이지, 항상 FIFO 가 더 나쁘다고 할 수 없다.

사고의 도구로 Slide switch 를 이용하자. 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%201.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%201.png)

Time Slice 의 크기를 조절해서 알고리즘을 특징을 동적으로 조절하는 알고리즘이 필요하다. 

static 한 알고리즘의 한계를 극복한다 ⇒ Adaptive Scheduling 이 중요. 

**Time Slice 를 workload 에 따라 동적으로 바꾼다!**

- FIFO 는 CPU Burst 사이즈가 균재할 때 좋음.
- Round Robin 은 불균등할 때 좋음.

**Adaptive Scheduling** 

CPU Bound Process : CPU 연산이 많은 프로세스. 어떤 성능적 요구를 갖는가? 

- **CPU Utilization 이 중요한 척도이다. Throughput 중요. Long CPU Burst 를 갖는 프로세스 이므로.**

IO Bound Process : IO 작업이 많아서 대부분 wait 하면서 보내는 프로세스. 

- **IO의 응답성을 높이기 위한 멀티스레딩이 중요하다. Interactivity, Response Time 이 중요한 척도이다. short CPU busrt 를 갖기 때문에.**

**IO Bound 인지 CPU bound 인지에 따라 스케줄링 방식이 다르다.** 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%202.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%202.png)

proc 1 → IO bound process 를 대표 

proc 2 → CPU bound process 를 대표 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%203.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%203.png)

⇒ 시나리오 1. 타임 슬라이스가 100ms 일 때 프로세스 1은 짧게 1ms 돌아가고, 1ms 후에 100ms 동안 프로세스 2가 돌아간다. 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%204.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%204.png)

⇒ 시나리오2. 타임 슬라이스가 1ms 일 때. 프로세스 1이 먼저 1ms 돌고, 10ms 동안 프로세스2 가 돌아가다가 인터럽트를 받아 다시 프로세스1이 돌아간다. 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%205.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%205.png)

시나리오 1 과 2 모두 CPU Utilization 은 100프로 이다. CPU가 쉬는 일이 없음. 하지만 IO device utilization 은 각각 10프로, 90프로 이다. 시나리오1 의 경우 100ms 동안 프로세스2 를 기다려야 하므로, 10/100 + 1 의 비율로 IO의 실행 시간을 갖게 된다. 

📌 **결론 : CPU와 IO 를 모두 고려하면 Time Slice 가 작은 것이 좋다. 하지만 프로세스 2는 패널티가 있다.  적지 않은 인터럽트 서비스의 오버헤드가 든다.** 

Adaptive 스케줄링이 가능하려면 운영체제는 무엇을 해야 하는가? 

- 어떤 프로세스가 IO bound 인지, CPU bound 인지 파악해야 한다. (characterize 해야 함)
- **파악하는 방법 : Time Slice 를 (기본적으로 짧게)주고, 주어진 시간 안에 IO를 해서 CPU를 release 하면 IO bound 라 보고, 계속 돌려고 하는 애는 CPU bound 인 것을 알 수 있다.**
- 두번째로 더 큰 Time Slice 를 할당 하였는데도 더 돌려고 하면 "극심한 CPU bound" 프로세스 임을 알 수 있다.

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%206.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%206.png)

위의 방법을 활용하여 고안된 것이 Multi level feedback queue 이다. 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%207.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%207.png)

자신의 Time Slice 보다 빨리 끝나면 머물고, 아니면 강등시킨다. 강등되면 우선순위가 낮아지는 대신 Time Slice 가 길어진다. 이에 따라 CPU bound process 는 적은 인터럽트를 받으며 실행될 수 있다. 아래로 내려갈수록 Time Slice 가 exponential 하게 커진다. 1, 2, 2의 제곱 ... 과 같이 커진다.

⇒ Exponential Queue Scheduling 이라고도 함. 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%208.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%208.png)

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%209.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%209.png)

→ batch OS → Time sharing OS 

Time sharing OS 중 가장 보편화 된 것이 UNIX OS. 

**이 중 실험적인 CTSS 라는 OS 에서 이 MLFQ 를 사용하기 시작함.** 

리눅스 OS 는 1991년 경 Desktop OS 로 출발, 그 후에 Server OS 로 확장. 그 후로 Linux 가 임베디드에, 지금은 스마트폰에 들어가기 시작. 자동차에까지 사용되기 시작함. 

자동차 쪽에 Infortament system 에 리눅스가 중요한 역할. (도요타 쪽에서 많이 사용하려 노력중)

Desktop 의 어플리케이션과 자동차의 것은 특성이 달라, 하나의 스케줄러가 다 서비스 하기에는 어려움이 있다. 

게다가 요즘 스마트폰에는 코어가 여러개 있어. 스케줄러가 Single 이 아닌 Multi process scheduling 을 해야한다는 것. 

리눅스 OS 가 사용되는 영역이 넓어짐에 따라서 변화하고 있다. 

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%2010.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%2010.png)

지금까지 배운 스케줄링 정책은 범용 시스템에 사용되는 것이다. 하지만 이렇게 Fair 한 방식의 스케줄링이 통하지 않는 시스템이 있다. ⇒ Real time system. 

Real Time System : Mission critical 하다. 작업 수행이 요청되었을 때 이를 주어진 시간 안에 성공적으로 마쳐야 하는 시스템이다. 

예) 비행기 자세, 항법 제어시스템. fairness 보다 critical process 를 먼저 수행해야 한다. 

- critical process ⇒ 프로세스 선호도, 중요도에 따라 "우선순위 스케줄링"을 해야 함.

Priority based scheduling (문제 : Starvation)

수행할 준비를 마치고, CPU를 할당받기를 기다리는 프로세스가 무기한 연기되는 현상인 starvation 문제가 있다. 

PBS 는 거꾸로 General purpose system 에는 적합하지 않다. 

운영체제 발전 단계 

General → Priority → **Fair-Share (다른 이름으로 Band width scheduling, proportional share scheduling 이라고도 한다.)**

**Fair-Share 스케줄링** 

서로 많이 자원을 얻어가고자 하는 것. Workload 에 따라서 스케줄링 하면 변동이 심해져 안정적이 지 못하다. 

예) 요즘의 멀티 미디아 앱. 동영상의 경우 33ms 에 하나의 프레임을 렌더링 해서 뿌려주어야 안정적으로 스트리밍을 할 수 있다. CPU 의 Band Width 가 30%는 보장되어야 한다. 

- Performance Isolation : CPU를 많이 차지할 것 같은 어플리케이션이 들어왔을 때 걔의 영향을 격리시키는 것. 이런 경우 Bandwidth Scheduling 이 필요하다.
- **Linux 의 CFS (Compleletly Fair Scheduling) 이 밴드위스 스케줄링의 일종이다.**

![{{site.baseurl}}/assets/img/process_scheduling/Untitled%2011.png]({{site.baseurl}}/assets/img/process_scheduling/Untitled%2011.png)

SJF (preem) ⇒ Optimal → but, remaining time 예측 불가, FIFO 사용해야 → but non preem 하므로 짧은 Job이 와도 스위칭을 못해 → Round Robin, 기본적으로 FIFO, 짧은 간격마다 CPU Burst 확인. 

→ but, RR 도 Optimal 하지 않은 workload 발생, MLFQ 도입. 

Mission critical 한 시스템을 위해 우선순위 스케줄링 사용. 

최근 멀티미디어 사용을 위해 BWS 관심 커져, CFS 알고리즘 제시. 

Rotating Staircase Deadline scheduler. 

계단 모양. 멀티레벨 큐가 쌓여있는 모양. Rotaitng 이라는 것은 기본적으로 Round Robin 이라는 뜻. 차이점은 Deadline. 

Round Robin 은 내가 CPU를 빼앗기면 큐의 가장 뒤로 가게 됨. 하지만 이거는 뒤로 가는 것이 아니라 자기의 deadline 값에 따라서 큐의 중간에 갈 수도, 앞으로 갈 수 도 있음. 

deadline 이라는 것은? 

내가 BW 15프로 밖에 못받아서 (원래는 30프로 받아야 하는데) 큐에 들어간 경우 큐의 중간에 위치하도록 하는 방식. 

리눅스의 스케줄러 메인테이너가 받아들이지 않음. 기존의 Fair 스케줄링 수정해서 CFS 만들고 리눅스의 기본 스케줄러로 들어가게 함.