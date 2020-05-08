---
layout: post
title: "[OS] Process and Threads"
author: "Sara Han"
categories : OperatingSystem
comments : true
---

# Process and Threads (Chapter 3, 4)

**2020-02-17 월요일**

```
복잡해지는 소프트웨어 분석을 위해 필요한 무기 : 
1. Abstraction 
2. Decomposition 
```

**프로세스란?**

**프로세스의 정의 : Program in execution.** 

프로세스와 프로그램의 차이 

- 프로그램은 passive entity 이다. 디스크에 저장되어 있는 인스트럭션 파일 그 자체를 프로그램이라고 부른다.
- 프로세스는 active entity 이다. executable file 이 메모리에 로드된 것.
    - 프로세스는 스택, 데이터, 힙을 갖는다.

**프로세스는 왜 유용한가?** 

프로세스는 Design time entity 이자 runtime entity 이다.

소프트웨어 개발에는 설계와 구현의 단계가 있다. 

- 요구사항 → 설계 → A set of tasks 까지는 설계의 단계
- a set of tasks → 구현 → a set of programs 는 구현의 단계.

프로그램이 운영체제에 의해 Design time entity 에서 프로세스라는 runtime entity 로 매핑된다. 

설계, 구현의 단위로 사용될 수 도 있고, 실행의 단위로도 사용될 수 있다는 점에서 유용하다. 

```
운영체제를 공부할때는 추상화에 가려진 개념들을 걷어내야한다. 
이를 위해서는 구현에 대해 생각하는 것이 좋다. 
```

**프로세스의 state 혹은 context** 

memory context : 코드영역, 데이터영역, 스택영역, 힙 영역 

hardware context : CPU 레지스터, IO 레지스터 

system context : 프로세스 테이블, 열린 파일 테이블, 페이지 테이블 

⇒ 이렇게 세 가지의 state 가 있다. 

**운영체제는 프로세스에 대한 정보를 유지한다.** 

Process Control Block 에 자원 할당에 필요한 정보, 프로세스 아이디 등을 유지한다. 

PCB 여러개를 하나의 테이블에 관리하며, 이 테이블을 Process Table 이라 한다. 

리눅스 시스템 에서는 PCB 를 Doubly Linked List 로 구현하며, 현재 Running 중인 프로세스는 current 라는 포인터로 가리킨다. 

**프로세스 State Transition**

프로세스의 Lifecycle 

New → Ready → Running → Terminated 

(Waiting ... )

- **Ready 단계** : Active 하지만 CPU가 할당되지 않은 상태. 여러 프로세스가 Queue 로 관리된다. PCB 의 Linked List 를 이용해 큐를 구현한다. 이 큐를 Ready Queue 라고 부른다.
- **Running 단계** : CPU가 할당되면 러닝 상태가 된다. Single Process 인 경우 Running 상태인 것은 0개 혹은 1개 뿐이다. Running 상태에서 wait() 등을 호출하면 Waiting 상태가 된다.
- **Waiting 단계** : 특정한 이벤트가 일어나기를 기다리는 상태이다. 여러개의 프로세스가 있을 수 있다. 각 프로세스들이 기다리는 상태가 된 이유는 각자 다른데, 같은 이유별로 Queue 를 만들어 대기열을 유지한다. - 각 IO 디바이스별로 큐를 구성한다. 원하는 이벤트가 발생하면 Ready 상태가 된다.
- **Running → Ready** 로 가는 경우는 인터럽트가 발생했을 때의 경우가 있다.
- Running 이 끝나면 Terminated 상태가 된다.
- 시스템별로 더 구체화된 Lifecycle 이 있을 수 있다. 예를들어 리눅스의 경우 Zombie 상태의 프로세스가 존재한다.

### Threads

프로세스 : is a program that performs a **single thread of execution.** 

Single thread of control 은 프로세스가 한 번에 하나의 task 만을 수행하도록 한다. 예를들어 타이핑을 하면서 동시에 스펠 체크를 하는 경우 이 둘은 하나의 스레드에서 실행될 수 없다. 

대부분의 현대 OS는 멀티스레딩을 지원한다. 이로써 한 번에 하나 이상의 task 를 하는 것이 가능해졌다. 

이 기능은 특히 멀티코어 시스템에서 유용하다. 여러개의 thread 들을 병렬 처리할 수 있기 때문이다. 

스레드를 지원하는 시스템에서는 PCB에서 스레드에 대한 정보도 포함한다. 

**리눅스 시스템 에서의 Process Representation**

```
long state; // 프로세스의 스테이트 
struct sched_entity se; //스케줄링 정보 
struct task_struct *parent; //이 프로세스의 부모 
struct list_head children; //프로세스의 자식들 
struct files_struct *files; //열린 파일들의 리스트 
struct mm_struct *mm; //이 프로세스의 메모리 공간 
```

리눅스 커널에서 모든 active process 들은 Doubly Linked List 를 이용해 유지한다. (task_struct 의 Linked List). 커널은 current 라는 포인터를 유지하고 이는 현재 Running 상태인 task_struct 를 가리킨다. 


---

**2020-02-18 화요일** 

**Process Scheduling**

프로세스 스케줄링이란 행위의 목적은 무엇인가? 

각 프로세스들이 공평하게 CPU를 점유할 수 있도록 하는 것.

→ fairness 를 일단 1/n하는 것으로 정의하자. 

 CPU 프로텍션을 해주어야 한다. 

→ Protection. 내 프로그램에 문제 없이 분배되어야 한다. (공유되어야)

- 공평 이라는 것은 상대적인 개념이다. Application Specific 한 개념이다.

CPU를 다른 프로세스에게 줄 때 어떤 프로세스에게 넘길 것인가?

Dispatcher 라는 매커니즘을 통해서 넘겨준다. (넘겨주는 매커니즘은 동일함)

예를들어, 운정의 방법은 누구나 똑같지만, 운전을 해서 어디로 갈 지는 누구나 다르듯이. 

- **Separation of mechanism and policy  라는 설계 철학을 갖고있음.**

    OS 스케줄러는 두 단계로 구성 

    1. 교체 가능한 policy 와 (fairness, performance 에 영향을 줌)
    2. Dispatcher 라는 하나의 매커니즘 

**Dispatcher 매커니즘** 

현재와 다른 새로운 프로세스가 스케줄러에 의해 골라짐. 

OS의 가장 깊은 곳에서 무한 루프를 돌린다. 

인터럽트 오면 중단시키고 프로세스를 (process 의 state 를) 안전한 곳으로 대피 

중간 계산의 결괏값들을 대피시켜둠. 

OS가 지정한 프로세스를 새로 다시 실행시킴. 

```
loop forever{
	run the process for a while 
	stop & save its state 
	load state of another process 
}
```

→ 스케줄링 정책과는 무관하게 기계적으로 돌아가는 것이다. 

OS가 능동적인 존재인 것 처럼 보이나, 사실은 매우 수동적인 존재. 반면 디스패처는 매우 능동적. 

- 문제 ?

**Dispatcher 가 도는 CPU와 사용자의 CPU 이렇게 두 개 있어야 하는 것 아닌가?** 

**해결책 : Interleaving. 유저 모드를 중단시키고 커널 모드의 디스패처 실행. User process 와 OS 간에 원활하게 CPU 컨트롤을 넘겨주고 받아야 한다.** 


---

**2020-02-19 수요일**

- 컨트롤을 넘기는 매커니즘 : Hardware 매커니즘을 통해서 넘긴다.

인터럽트를 통해서 mode change : usermode → kernel mode (Context Switching)

⇒ Execution Control 이 넘어가는 것. 

- 유저모드에서 함수가 돌아간다. 유저모드에서 서브루틴 콜이 호출되는 것은 모드 변경이 필요 없음 ⇒ 커널 모드로 바뀌면 커널모드 함수가 돌아간다.
- 이렇게 인터럽트에 의해 모드를 변경할 수 있는 것은 하드웨어의 지원 때문이다.

**커널에 대한 오해**

- **active entity 라는 오해. 사실은 수동적이다. 마치 library code 같은 것. 운영체제는 커널 모드에서 수행되는 함수들의 콜렉션. 라이브러리이다. (System call functions)**
- 시스템 콜에 외에 인터럽트 서비스 루틴이 있다. (불려지는 모드는 커널 모드이다)

**커널 모드와 유저모드**

커널 모드 : 보안 등급이 격상된 수행모드. privileged instruction. 제약 없이 수행한다. 

유저 모드 : 제약된, 주어진 메모리만 가용. 

모드의 변경 : Microprocessor 안에는 상태를 나타내는 레지스터가 있따. 레지스터에 mode bit 가 바뀌면서 모드가 변경된다. 

디스패처 : 커널 함수중의 하나. 불려지려면 인터럽트 필요. 

**CPU를 다른 프로세스에게 넘겨주는 것**

1. non-preemptive 스케줄링. 소프트웨어 인터럽트인 Trap 을 사용한다. 

예) IO 기다릴 때 다른 프로세스에게 CPU를 넘겨준다. 

2. preemptive 스케줄링 : 운영체제가 강제로 뺐는 것. 

예) Time Sharing 스케줄링을 사용, 100ms 마다 바꾸기로 되어있을 때 운영체제가 강제로 CPU를 빼앗는다. → 어떻게 구현 ? 타이머 서킷을 사용해 일정 간격마다 인터럽트 발생. 하드웨어 인터럽트에 의해 촉발되는 스케줄링. 

- 인터럽트 두 종류 : 하드웨어 인터럽트, 소프트웨어 인터럽트(Trap)

**프로세스의 모드 변경**

malloc() 하면 커널 함수 (시스템 콜 호출, Trap 을 통해서 mode change 발생해야.) 실행. 

file read() → 인터럽트 발생시켜 → 커널에 "read() 호출할게!" 신호 전달 → kernel mode system call 발생 → return from interrupt 라는 인스트럭션으로 바로 유저모드 변경. (보안 등급이 높은 커널에서 유저 모드로 가는 것이므로 인스트럭션 만드로 변경이 가능하다)

- 리눅스의 모든 프로세스는 PID가 있다. 커널 함수 수행 중 getPid() 하면 무엇이 나오는가?

⇒ **여전히 수행의 주체는 system call 를 호출한 프로세스이다. 모드 변경 ≠ 프로세스 변경**

- 흔히 어떤 프로그램 수행중이면 이 상태는 stack 을 갖고 tracing 한다.

**프로세스 두 가지 요소**

1. "state" ⇒ "context" : 데이터, 스택, 코드 ... 
2. Thread of control (Execution stream) : **Stack (유저 모드의 메모리 공간)**

    ⇒ 어떻게 구현되는가? 

    디버깅 : 브레이크 포인트 걸고, 스택을 본다. 지금까지 호출된 함수들 들어가있음. 프로세스의 실행 이력을 보는 것. 

    함수는 instruction sequence 의 abstraction. 

    **stack 의 값들을 해당 프로세스의 runtime context 라고 한다.** 

유저모드 → 시스템콜 → 커널 함수 호출 

Stack 은 ? 유저 메모리 아님. 커널 영역의 메모리에 있음. 

**다른 말로, runtime context 가 변화한다 == Stack 이 변한다.** 

📌주의 : 프로세스는 여전히 유저 모드의 것. 

### ⭐ Context Switching

- 디스패처
1. 현재 수행중인 프로세스 안전하게 저장. 
2. **스케줄러가 골라준 다음 프로세스의 state 를 가져온다. A → B 프로세스 로의 Context 전환. 이것을 Context Switching 이라고 한다.** 

cf. 운영체제 코딩 방식 특이한 점? 

CPU를 자유자재로 넘겨주어야 함. Context Switching 할 때 많이 본다. 

- 프로세스는 state 가 있다. 이 때 State 라는 말의 뜻에 주의하자. 프로세스가 기억하는 정보도 State 라 하고, 프로세스 Lifecycle 즉 프로세스가 현재 어떤 상태에 있는가라는 것도 State 라고 부른다.

**무엇을 대피시켜야 하는가?** 

1. HW Context - CPU 레지스터 : 반드시 모두 대피시켜야 한다. 메인 메모리로 보낸다. (예를들어 현재 연산중인 것들의 값들)
2. Kernel Context - 커널의 자료 구조 등 : 링크드리스트, 배열을 두어 프로세스마다 따로 커널의 자료구조를 관리하므로 대피하지 않아도 된다. Kerenl Context 는 그냥 내버려두면 OK. 
3. Memory Context - 필요한 경우 부분적으로 (디스크로)대피시킨다. 
    1. 전혀 대피하지 않는 경우 : 배치 모니터 시절에는 대피가 필요 없었으나, 지금은 고려하지 X 
    2. 몽땅 다 대피시키는 경우(느리다) : UniProgramming 할 때 해야한다. CPU 빼앗기면 Disk 로, Disk 에서 메모리로. ⇒ Roll in, Roll out, Swapping 이라 한다. 
        - 예) 멀티 프로그래밍 정도가 크면 Disk 로 내릴수도 있다. Swapping, Swap out.

**CPU레지스터는 어떻게 대피 시키는가?**

- stack 에 값을 push 하면 예외없이 그 값을 안전한 곳에 대피한다.
- pop은 회복한다라는 뜻

과정

1. 0x100 이라는 주소에서 ADD 연산을 하는 중에 인터럽트가 걸리면 우선은 유저의 연산을 끝마친다. 레지스터 값들의 일관성을 위해서. 
2. 인터럽트를 확인한다. (IRQ Number 확인) 
3. 확인된 인터럽트의 주소를 확인하고, 이에 따른 인터럽트 서비스 루틴을 수행한다. 
4. 수행이 끝나면 다음 주소 (예를들어 0x104) 로 복귀한다. **Process Status Word의 모드가 1→ 0으로** 변한다. 

**스택에 쌓이는 것** 

1. PSW 값 → PSW 값 Pop 해서 복귀. 
2. 복귀할 주소값 → 주소 Pop 해서 복귀

**스택에 넣는 일은 누가 하는가?**

하드웨어가 스택에 넣는 것이다. 이 기능을 HW차원에서 지원 해 주어야 Dispatacher 만들 수 있다. 이것을 촉발시키는 것은 인터럽트 서비스 루틴의 첫 번째 인스트럭션이다. 

**디스패처 과정** 

1. 인터럽트 발생 
2. PSW 저장 
3. 현재 수행중인 인스트럭션 주소 저장 (마이크로프로세서가 프로그램 카운터에서 레지스터 값을 Pop해서 넣는다)
4. 인터럽트 서비스 루틴으로 Jump 
5. 루틴 초반부에서 현재 CPU레지스터 값들을 스택으로 대피시킴

**이 게임의 법칙이 적용되지 않는 경우?**

- 프로세스는 적어도 한 번 Context Switching 을 당해야 스택이 있다.
- 처음 프로세스 만들어질 때 OS가 Fake stack 을 만들어 주어야 한다.

**📌 주의 : 스택은 메인 메모리에 있는 것. 단지 active 한 스택은 한 번에 하나 뿐.** 

**Context Switching 은 비싸다.** 

직접 비용 : 프로세스의 state 를 로드할 때 걸리는 싸이클의 수. 

간접 비용 : 프로세스를 바꾸면서 캐시에서 이전의 PCB가 밀려나면, COLD Cache 문제가 발생한다. 


---

**2020-02-20 목요일** 

**Abstraction and Decomposition** 

- OS 를 잘 이해하기 위해서는 구현을 생각해야 한다.
- 운영체제는 하나의 복잡한 프로그램. 프로그램 = 자료구조 + 알고리즘. 커널 내부에 어떤 자료구조들이 있으며, 이 DS를 조작하기 위해서 어떤 알고리즘을 사용하는가 생각해야 한다.

- 도구가 없다면 창조적일 수 없다.
    - 도구 = Abstraction & Decomposition.

**Abstraction**

프로그램을 설계하다 보면 자연스럽게 Layered Architecture 가 된다. 

하드웨어 → 운영체제 → 미들웨어 → 어플리케이션 

⇒ 각 레이어의 사이에 API가 존재한다. 예) 커널 안의 함수를 호출하는 System call. 

아래 레이어의 복잡함을 가리고 위에서 사용할 수 있는 interface 를 제공한다. 

**Decomposition** 

**📌API를 이해하려면 Synthesis, Implementation 으로 점점 구체화 시켜서 내려가야 한다.** 

```
학부생은 지식의 소비자, 대학원생은 지식의 생산자이다. 
창조적인 능력은 구현을 얼마나 잘 하느냐에 달려있다.
```

📌주의 : Layering Principle - 위는 아래 계층의 기능에 종속적이지만, 그 반대는 안된다. 

**추상적인 것을 반드시 구체화 해야 한다.** 

**Abstraction ⇒ Decomposition! ⇒ Creativity**

**Process Creation & Termination** 

프로세스를 만든다는 것 ⇒ 운영체제가 자료구조를 초기화 - 메모리 할당 하는 것. 

1. File system 에는 exe 파일이 있어야 한다. 
2. 운영체제가 이 파일의 코드를 읽어들인다. 
3. 변수들에 선언된 정보들을 기반으로 데이터 영역을 잡아준다. 빈 스택과 힙 또한 생성한다. 

    **📌Code 영역과 Data 영역을 잡아주는 것 ⇒ "Program 을 Load 한다"**

4. 필요한 PCB를 생성, 초기화한다. 
5. PCB를 Ready Queue 에 넣는다. 

- 유닉스는 가장 처음, 0번 프로세스를 만들 때만 이 과정을 거치고 나머지 프로세스는 클로닝을 통해서 생성한다.
- **클로닝 = fork() 호출**
- Parent Process → fork() → child process ...

1. 운영체제가 현재 fork 를 호출한 프로세스를 중단시킨다. 
2. 해당 프로세스의 스냅샷을 찍는다 
3. Context 를 복사하여 Child를 생성한다. PID만 다르고 나머지 정보는 동일하다. 
4. Reday Queue 에 Child 를 넣고  end fork. 
5. Parent 로 return 한다. 

⇒ 이 방법이 마음에 들지 않는 이유? 

**non-functional 한 측면의 이유** : 매 번 copy 를 하는 것은 비효율적이다. 

**해결책**

Copy on write 매커니즘을 통해 name pipe 를 사용해 IPC를 하는 경우처럼, 실질적으로 부모의 코드가 복사되어 자식에 있어야 하는 경우만 복사가 일어난다. 

**function 한 측면의 이유** : 부모에서 child 가 계속해서 파생되는 식이므로, 궁극적으로 하나의 프로세스만 돌아갈 수 있다. 시스템에서 최초로 만든 프로세스 외에는 다른 프로세스를 실행시킬 수 없어짐. 

**해결책**

fork 를 한 후에 exec 을 하여 child 의 메모리 영역에 실행 파일을 로드해 아예 새로운 프로그램을 출발시킨다. 

**parent[ fork → [child : exec → exit ] → wait (exit 코드 회수) ]**

- exec : 부모와 자식이 다른 프로그램을 수행하게 된다.
- exit : 프로그램 수행이 끝나면 프로세스가 종료된다. 내가 exit 을 호출하지 않으면 운영체제가 자동으로, (compiler가 exit 를 삽입함)
    - exit을 마치고 wait 을 통해 결과가 회수되기를 기다리는 프로세스는 좀비 프로세스이다. 자식의 자원은 모두 회수되고 없어진 상태이다.
- 회수되지 못하면 고아 프로세스가 되는데, init (process 1) 이 주기적으로 wait()을 호출하도록 되어있어 결국 init 의 자식 프로세스가 된다.

**왜 굳이 이런 방식을 선택하게 되었나?** 

프로세스간의 IPC 를 원활하게 하기 위한 수단... named pipe. 


---

**2020-02-21 금요일** 

**Server Architecture** 

서버

- **Iterative Server 방식**

서버 안에 일종의 무한루프 있어 메시지큐에서 deque 해서 요청 처리해서 크라이언트에게 반환. 구현과 유지가 쉽다는 장점이 있으나 병렬성이 없어 responsiveness 가 떨어짐. 

- **Concurrent Server 방식**

서버가 일 처리할 때 워커 스레드를 동원해서 한다. 요즘은 하드웨어가 멀티코어화 되어있어 병렬성 있는 게 좋은데,  concurrent server 는 이를 만족한다. 하지만 처리할 요청의 수 만큼 프로세스를 생성해야 한다는 단점이 있다. 

**멀티스레딩의 목적** 

Concurrency 는 높이면서 Execution Unit 을 생성하거나 수행시키는데 드는 부담을 줄인다. 

멀티스레딩 응용 

- 인터넷 서버 설계
- 과학 관련된 연산. 병렬화 용이하게 쓰임. matrix 열들을 각각 읽어서 처리할 수행 entity 필요.

Thread 는 스택으로 구현한다. 

하나의 프로세스 안에 여러개의 스택이 있는 것이다. 

각 스레드마다 Thread Control Block 이 있다. 

**용어**

Process 

Thread 

Task 

이전에 말했듯이 process 는 design time entity 이자 runtime entity 이다. Task 는 보통 Design time entity process 를 일컫는다. 

A, B, C, D 라는 태스크들이 있다 (Design time) → ProcessA가 처리한다. (Runtime)

ProcessA 안에는 threadA, threadB ... 등이 있다. 

카네기멜론 대학의 Mach 운영체제는 Process = Task + Thread 라고 한다. 이 때 Task 는 프로세스에 부여된 자원들을 일컫는다. 

**멀티스레딩 - 구현** 

크게 세가지 방식으로 구현할 수 있다. 

1. 커널이 전혀 모르게 → 유저레벨 스레드 구현
    - OS 는 프로세스 하나만 지원, 프로세스 안에서 스레드를 만드는 것.
        - **Stack 을 구현해야 한다. (Thread)**
        - **각 스레드마다 TCB를 만들어야 한다. (Thread Control Block : 당연히 유저 레벨에 있어야)**
        - **스레드간에 스케줄링을 위해, 유저 레벨의 스케줄러가 있어야 한다.**
            - 커널 안에 디스패처, 스케줄러가 함수로 구현되어 있듯이 이것도 함수로 만들고 묶어서 유저 레벨 스레드 라이브러리를 만든다.
            - 이 라이브러리에는 스레드를 생성하고 소멸시키는 코드가 있고, 스레드들의 Context 들을 저장하는 코드가 있다. 스레드간에 데이터를 주고 받기 위해 Communication 을 지원해주는 함수가 있다.

        유저 레벨 스레드들이 서로 어떻게 동작하는가? 

        - 가장 어려운 것 : 한 스레드에서 다른 스레드로 스케줄링
2. 커널이 100프로 알고 지원 → 커널 레벨 스레드 구현 
3. 유저 레벨, 커널 레벨 혼합 

스레드 스케줄링 

non preemptive : 쉽게 구현할 수 있다. 라이브러리에서 제공해준 CPU yield 와 같은 함수를 호출하면 된다. 

preemptive : 반드시 인터럽트가 필요하므로 좀 어렵다. 인터럽트가 걸리면 핸들러가 돌아가고, service routine 이 돌아간다. 그 인터럽트를 처리할 프로세스를 깨워주는 역할을 이 루틴에서 한다. 인터럽트가 오면 그 프로세스로 컨트롤을 넘겨준다. 프로세스가 Execution entity 일 경우엔 간단했는데 지금은 thread 가 execution entity 이므로 thread 가 인터럽트에 대해 알아야 함. 

→ 한계가 있음. 

서로 코루틴처럼 내가 수행하다 쟤에게 주고 , 이런 형태의 스케줄링만 가능하다. 

thread → read system call 호출 → sysread 커널 함수 실행 → Disk IO 를 시작 → 끝날때까지 기다리게 만들어야. 그럴때는 이 프로세스 전체를 blocking 해버림. 

근데 다른 스레드들이 수행을 진전시키지 못하는 문제가 발생함. 블러킹 아노말리. 

**유저 레벨 스레드의 문제**

📌커널이 멀티스레딩의 존재를 전혀 모르기 때문에, **한 스레드가 블러킹 시스템콜을 하더라도 프로세스가 블러킹 시스템콜을 했다고 생각해서 전체 프로세스를 블러킹 해버림**. 

📌**운영체제가 스레드에게 직접 인터럽트를 전달할 수 없기 때문에 Preemptive 스케줄링을 할 수 없다.** 

⇒ reactive system 에 사용하면 당연히 안될 것이다. 

**장점은?**

운영체제를 고치지 않고도 멀티스레딩을 할 수 있다. 

**요점**

이 반쪽짜리 멀티스레딩은 리액티브 시스템이 아닌 곳에서 유용하다. 과학 계산 용도. 스레드들한테 연산 시키고 결과만 조인하면 됨. 

**커널레벨 멀티스레딩** 

태스크가 생성되고 소멸되는 것을 커널이 다 안다. 스레드 생성, 종료가 모두 시스템콜로 구현이 된다. 스레드 컨트롤 블락 또한 커널이 조작한다. 커널이 직접 스레드들을 다 스케줄링 한다. 

커널레벨 스레드 구현을 하면 인터럽트 포워딩도 가능해지고, 블라킹 아노말리같은 문제도 해결될 수 있다. 

윈도우즈는 90년대 이후에 발전했기 때문에 커널 레벨 스레드를 지원하고, 리눅스도 마찬가지. 

시스템콜을 확장해서 스레드를 관리하는 함수를 제공해야 하고, 동기화도 제공해야 한다. 

**장점과 단점**

유저레벨 스레드 구현의 모든 단점을 해결하지만, 이제는 **extra overhead 가 든다. 시스템 콜 수행 → 핸들러 수행 → 디스패처 → 커널 함수 수행 되어야함.** 하지만 반응성은 매우 향상시킬 수 있다. 

커널 레벨 스레드를 사용하면 과학 연산의 경우 오버헤드의 영향이 큼. 

**커널레벨과 유저레벨을 결합한 것** 

상당히 많은 부분을 유저 레벨 스레드가 처리한다. 

- 단지 인터럽트가 왔을 때 유저 레벨 스레드에게 포워딩하는 기법이 추가 됨.
- 또한 어떤 프로세스가 블러킹 시스템 콜을 했을 때 그 스레드만 중단시키고 새로운 커널 스택을 할당, 유저 프로세스에 이것을 붙이고 다시 return 을 함. 이렇게 하면 유저 레벨의 스케줄러가 다른 스레드를 골라서 수행을 할 것. 얘기 다시 커널로 돌아왔을 때 보존되어 있는 커널 레벨 스택이 있으니까 시스템콜 처리도 가능하다.


---

**2020-02-24**

**멀티스레딩 API** 

스레드 프로그래밍 모델을 어떻게 사용할 것인가? 

우리가 통상적으로 아는 프로그램은 Main() 이라는 진입점이 entry point 인데, 만약 스레드가 여러개 생기면 어떻게 되는가? 각 스레드가 수행하는 코드들은 무엇인가? 

**멀티 스레딩을 제공하는 API 중 많이 사용되는 POSIX pthread API 가 있다.** 

1. pthread_create() : **매개변수로 스레드가 수행해야 할 코드를 함수 포인터로 전달해준다.** 
2. pthread_exit() : 해당 스레드가 종료된다. 
3. pthread_join() : 스레드를 시작시킨 곳에서 해당 스레드가 종료되기를 기다린다. 매개변수에는 기다려야 할 스레드가 들어간다. 
4. pthread_yield() : CPU 를 양보하는 함수이다. 

프로세스에는 1개의 main thread 가 존재한다. **(process = state(context) + thread of control)**

![{{site.baseurl}}assets/img/process_and_threads/Untitled.png]({{site.baseurl}}/assets/img/process_and_threads/Untitled.png)

state transition 은 유저코드 상에 나타나지 않는다. function call 이 stop 되는 것으로반 유저 레벨에서 비춰진다. 

코드 예제 

- 위키피디아 참고

POSIX Threads 는 주로 pthread 라고 일컫는다. pthread 란 어떠한 실행 모델로서 언어와 독립적으로 존재하는 것일 뿐만 아니라 병렬 실행 모델이라 할 수 있다. 일정 시간 안에 겹치는 일들을 프로그램이 제어할 수 있도록 해준다. 각 일들의 플로우는 thread 라고 한다. 이 스레드들의 생성과 제어는 POSIX Threads API로의 시스템 콜을 함으로써 가능하다. 

pthread_ XXX 접두사가 붙은 함수들이 있고 크게 네 개의 그룹으로 나뉜다. 

1. Thread management : 스레드를 새성, Join, 하는 등 
2. Mutexes 
3. Condition Variables 
4. Synchronization between threads using read/write locks and barriers 

POSIX semaphore API는 POSIX Threads 와 같이 작동하지만 threads 표준의 일부는 아니다. 세마포어 절차들은 sem_ 이라는 접두사가 붙는다. 

```
void * perform_work(void * arguments){
	//some work 
}

int main(void){
	pthread_t threads[NUM_THREADS]; 
	...

	for(int i = 0; i < NUM_THREADS; i++){
		...
		result_code = pthread_create(&threads[i], NULL, perform_work, &thread_args[i]); 
		//스레드를 생성한다. 매개변수를 통해 스레드의 우선순위도 설정할 수 있다. 
	}

	...

	for(int i = 0; i < NUM_THREADS; i++){
		result_code = pthread_join(threads[i], NULL); 
		//0 ~ 4 번까지 종료되기를 기다린다. 
	}
...
}
```

**자원의 종류** 

preemptive resource : 양보할 수 있는 자원, 양보하거나 포기한다. 예) CPU, Main memory 

점유한 프로세스의 상태를 대피시키는 절차가 필요하다.  

non-preemptive resource : 양보할 수 없는 자원. 중간에 끊으면 프로세스가 한 일들이 의미없어진다. 예) 프린터. 프린터는 중간에 출력하다가 말면 안되므로. 

하지만 이는 작위적인 개념이다. 컵을 들고 이 컵이 breakable 할까, non-breakable 할까 하는 것. 만약 정말로 땅에 떨어트려서 깨트린다면 이것은 breakable 한 자원이 될 것이다. 위의 개념도 마찬가지로 작위적임. 

CPU를 여러 프로세스가 잡으려 할 때 발생하는 문제 

- 누구에게 줄 것인가?
- 잡았다면, 얼마나 오래 사용하게 할 것인가?

⇒ 의사결정이 CPU 성능에 큰 영향을 미친다. 

**CPU Burst**

배치 모니터 시절 수행의 주체는 Job, 지금은 Process. 

Job 과 Process 의 차이점 : Job 은 수행 완료가 될 때 까지 계속해서 수행한다. Process 는 반면 여러개가 동시에 메모리에 존재하여 일이 처리될 수 있다. 꼭 하나의 프로세스가 끝나기를 기다리지 않아도 됨. 

Process 가 수행의 주체일 때 스케줄링 Unit = CPU Burst 

![{{site.baseurl}}assets/img/process_and_threads/Untitled%201.png]({{site.baseurl}}/assets/img/process_and_threads/Untitled 1.png)

 **연속적으로 CPU 연산이 일어나는 구간을 CPU burst 라 하며, 이것이 스케줄링의 단위가 된다.** 

→ 크기는 보통 3ms 정도 된다. 사이즈에 따라 스케줄링이 달라져야 한다. 

- CPU Burst 가 큰 것 : 연산이 많은 것. 과학 기술 연산들.
- 작은것 : IO 인터랙티브 한 것들.

**스케줄링은 언제 일어나는가?** 

Process State Transition 을 다시 보면 

Ready → Running 으로 갈때는 Dispatcher 가 담당하므로 이때는 스케줄링이 필요하지 않다. 

Running → Ready 상태로 갈 때는 하드웨어 인터럽트가 필요하므로 스케줄링이 필요하다. 이 때 CPU를 강제로 빼앗는 것이므로 Non-preemptive 스케줄링이 일어난다고 할 수 있다. 

Running → wating 으로 가는 것은 자기 스스로 CPU 를 양보하는 것이므로 non-preemptive 스케줄링이 일어난 것이다. 

Running → Terminated 로 가는 것도 마찬가지로 non-preemptive 하다. 

Waiting → ready 상태로 가는 것은 preemptive 스케줄링의 한 부분이다. **비동기 이벤트는 preemptive 스케줄링을 촉발한다.** 

무슨 말이지 ? 

**스케줄링 정책의 목표** 

성취하고자 하는 바는? 

1. resource utilization 극대화. 컴퓨터의 uptime 중 (실행되는 중) 얼마나 의미있는 역할을 했는가? CPU, IO ... 
2. minimize overhead : 운영체제가 스케줄링을 하려면 의사결정을 해야한다. 즉, OS의 코드가 돌아가야 한다. 이 때 코드가 O(n^2) 이면 비효율적. 코드의 양이 적어야 한다. 
3. 너무 빈번하면 Context Switching 이 많아서 줄여야 한다. 
4. Fairness. 골고루 나눠 가질 수 있게 해야한다. 

    범용 시스템에서의 fariness 는 1/n의 share 를 갖는 것을 fair 하다고 보는 반면 우선순위 기반 시스템에서는 우선순위에 따라 분배가 이루어져야 한다. 상황에 따라 이 둘을 섞어서 사용하기도 한다. 

    - **Aging : 태스크가 CPU를 기다리는 시간에 비례하여 우선순위를 절차적으로 높이는 기법.**

**Schedule Policy 가 성능에 미치는 영향 (단순한 증명)**

![{{site.baseurl}}assets/img/process_and_threads/Untitled%202.png]({{site.baseurl}}/assets/img/process_and_threads/Untitled%202.png)

프로세스 1 = P1 , 프로세스 1의 Computation Time = C1, 기다리는 시간 W(P1), wating time

 **스케줄링 정책의 평가, 측정 기준 ?**

Throughput : 단위 시간당 종료시키는 프로세스의 개수. Throughput 에 민감한 앱은? CPU 인텐시브한 것이다. 백그라운드의 코덱 연산 등 ... 

Response Time : 터치폰 에서는 반응 속도가 중요하다. 

cf. 애플의 아이폰 ⇒ In house kernel 사용. 커널에 맞는 스케줄링 정책을 만들기 쉬웠다. 반응성이 뛰어났음. /// Android Jelly Bean ⇒ Linux 커널 사용. 리눅스 커널은 클라우드 컴퓨터, 서버 등 사용되는 범위가 넓기때문에 범용 스케줄러를 사용한다. 따라서 반응성이 최우선순위가 아님. 젤리빈은  터치 이벤트가 일어나는 경로를 (Vector 값들) 미리 예측해서 반응성을 개선시키는 방법을 취함. 

- **Throughput 과 response time 은 서로 충돌한다.**

**어떤 스케줄링 정책이 있는가?**

- **FIFO 정책 : 먼저 온 것을 먼저 처리한다. 다음 IO가 일어날 때 까지 FIFO. CPU Burst 단위로. Blocking 이 일어 날 때 까지 CPU를 갖고 있는다.**
    - 단점 : CPU Burse 안에 무한루프가 있는 경우. 극복 → OS가 타이머를 걸어놓고 100ms 가 지나면 강제로 빼앗도록 함.
- **Shortest Job First 정책 : CPU Burst 가 가장 작은 것을 먼저 수행시킨다. pair wise exchange 를 하여 가장 작은 것을 미리 수행함.**
    1. non-preemptive SJF 
    2. preemptive SJF (Shortest time to completion first)
    - 문제점 : 운영체제가 태스크의 남은 CPU burst 사이즈를 예측할 수 없다. 미래를 알 수는 없으므로

    📌나머지 스케줄링 정책들은 SJF 를 달성하기 위해 CPU Burst Size 를 어떻게 예측할지 에 관한 것이라 해도 과언이 아니다. 

- **Round Robin 정책**

    FIFO 로 하되, Timeout 값을 두어서 CPU Burst 가 Timeout 보다 길면 다른 프로세스로 대체될 것이고, 만약 더 짧으면 불필요한 Switching 이 많이 일어날수도.... ? 다음 애가 작기를 기대하며 불러들여 ... 

    다른 프로세스로 대체되면 기존의 프로세스는 ready queue 에 들어간다. 작은 것을 먼저 끝내기 위함. 

    **문제점 : Timeout 값을 얼마로 설정해야 하는가? (Time slice, Time Quantum)**

    ⇒ 적절한 값이어야 Round Robin 이 효율적이다. 

    - 너무 딱 맞으면 인터럽트가 많아져서 overhead...