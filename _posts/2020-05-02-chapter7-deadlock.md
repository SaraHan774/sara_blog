---
layout: post
title: "[OS] Deadlock"
author: "Sara Han"
categories : OperatingSystem
comments : true
---

# Deadlocks (Chapter 7)

멀티프로그래밍 환경에서는 여러개의 프로세스들이 한정된 자원을 갖고 경쟁한다. 프로세스가 어떠한 자원을 요청했을때 자원이 없는 경우 해당 프로세스는 wating state 가 되어 자원이 사용 가능할 때 까지 기다린다. 가끔 기다리는 프로세스는 다른 프로세스가 자원을 해제하기를 계속해서 기다리는 상황에 놓여 wating state 에 머물게 된다. 이러한 상황을 **Deadlock** 이라고 한다. 

* 비유 : 두 개의 기차가 서로를 향해 달려올 때 두 개 모두 동시에 멈추고 하나가 철로에서 사라져야만 다른 하나의 기차가 철로를 지나갈 수 있다. 

최근의 트렌드에 따르면 데드락 상황은 더욱 더 흔해질 수 밖에 없다. 

* 프로세스의 개수가 많아진다. 
* 멀티스레디드 프로그램이 많아진다. 
* 시스템 안에 자원의 개수가 많아진다. 
* 배치 시스템 대신 long-lived file 과 database server 가 강조된다. 



### 7.1 System Model 

> System 을 자원 + 프로세스로 보고 이런 모델 하에서 데드락을 정의한다 . 

System = finite number of resources + competing processes

* resources 는 여러개의 동일한 인스턴스들일 수 있다. CPU 사이클, 파일, IO devices (Printers, DVD drives) 들은 resource type 의 일종이다. 만약 시스템에 CPU가 두 개 있다면 *"resource type CPU has two instances*" 라고 말한다. 
* 프로세스가 instance of a resource type 을 요청하면 해당 type 의 **어떠한** 인스턴스의 할당이 이루어져야 한다. 
  * 한 방에 두개의 프린터가 있고, 어떤 프린터로 출력물이 나오든 상관이 없는 경우 이 프린터들은 identical resource type 으로 정의되어야 한다. 
  * 하나의 프린터는 10층에 있고 하나는 1층에 있는 경우 이 둘은 같은 resource type 으로 취급되어서는 안된다. 별도의 resource class 들이 각각의 프린터들에 대해 정의되어야 한다. 
* System resource 의 또 다른 예로 mutex lock, semaphore 등이 있다. 
  * 한편 lock 은 특정한 자료 구조를 보호하는 것과 연관된다. 예를들어 하나의 락은 queue 를 보호하는데 사용되고, 다른 락은 linked list 를 보호하는 데 사용되는 등. ***이러한 이유로 각각의 lock 은 자신만의 resource class 를 할당받으며 따라서 resource class 의 definition 은 문제가 되지 않는다. (?)***
* 프로세스는 자원을 사용하기 전에 반드시 request 해야 하며 사용 후 반드시 release 해야한다. 

* a process may utilize a resource in only the following sequence:
  1. Request : 프로세스가 자원을 요청한다. 만약 자원을 사용하기 위해서 기다려야 한다면 자원을 얻기 전까지 기다린다.  
  2. Use : 자원에 어떠한 일을 수행한다. (operate on the resource)
  3. Release : 프로세스는 자원을 해제한다. 

* System Call 예시 

  * Device : `request()` , `release()` 
  * File : `open()` , `close()` 
  * Memory : `allocate()`, `free()` 

  * Semaphore : `wait()`, `signal()` 
  * Mutex Lock : `acquire()`, `release()` 

* System table 에서는 각각의 자원이 free 인지 allocated 된건지 기록한다. 

  * 만약 allocate 되었다면 어떤 process 가 사용하고 있는지도 기록한다. 
  * 다른 프로세스가 해당 자원을 요청하면 그 프로세스는 wating queue 에 들어간다. 
  * a set of process 들이 데드락 상태에 빠지는 경우 : set 안의 어떠한 프로세스에 의해서만 일어날 수 있는 어떠한 이벤트를 모든 프로세스들이 기다리는 경우. 

* Resources 
  * physical resources : Printers, tape drives, memory space, CPU cycles. 
  * logical resources : semaphores, mutex locks, files. 
  * 이러한 자원 말고도 IPC facility 들 과 같이 다른 이벤트들도 데드락에 빠질 수 있음. 

* Deadlock 은 같은 종류의 자원들 끼리에 대해 일어날 수 도 있고, 다른 종류의 자원들 사이에서도 일어날 수 있다. 
  * 같은 종류의 자원들 : 예를들어 세 개 의 프로세스들이 세 개의 CD RW Drive 를 잡고 있을 때 이 중 하나의 프로세스가 다른 Drive 를 요청하는 경우 
  * 다른 종류의 자원들 : 하나의 프린터와 하나의 DVD Drive 가 있는 시스템의 경우, P-i 가 DVD 를, P-j 가 프린터를 잡고 있을 때 각자가 서로의 자원을 요청하는 경우 

### 7.2 Deadlock Characterization 

데드락이 발생하면 프로세스는 절대로 실행을 끝낼 수 없고, 시스템 자원들은 묶여있게 되어 다른 job 들이 시작할 수 없다. 데드락의 여러 문제들을 살펴보기 전에 무엇이 데드락을 구성하는지에 대해서 더 자세히 살펴보자.

#### Deadlock with Mutex locks 

multithreaded Pthread 프로그램에서 mutex lock 을 사용해서 deadlock 이 어떻게 발생할 수 있는지 살펴보자. `pthread_mutex_init()` 함수는 unlocked mutex 를 초기화한다. mutex lock 들은 `pthread_mutex_lock()` 와 `pthread_mutex_unlock()` 을 통해서 aquire, release 된다. 쓰레드가 locked mutex 를 얻고자 하면 `pthread_mutex_lock()` 함수는 mutex 의 소유자가 `pthread_mutex_unlock()` 을 호출하기 전까지 해당 쓰레드를 블럭한다. 

```c
/* create and initialize the mutex locks */ 
pthread_mutex_t first_mutex; 
pthread_mutex_t second_mutex; 

pthread_mutex_init(&first_mutex, NULL); 
pthread_mutex_init(&second_mutex, NULL); 
```

다음으로 두 개의 쓰레드들이 생성되고, 둘 다 위의 두개의 mutex 락에 접근할 수 있다. thread_one, thread_two 는 `do_work_one()` 과 `do_work_two()` 함수 안에서 돌아간다. 

```c
/* thread one runs in this function */ 

void * do_work_one(void *param){
    
    pthread_mutex_lock(&first_mutex);
    pthread_mutex_lock(&second_mutex);
    // do some work 
    pthread_mutex_unlock(&second_mutex); 
    pthread_mutex_unlock(&first_mutex); 
    
    pthread_exit(0); 
}

/* thread two runs in this function */

void * do_work_two(void * param){
    
    pthread_mutex_lock(&second_mutex); 
    pthread_mutex_lock(&first_mutex); 
    // do some work 
    pthread_mutex_unlock(&first_mutex);
    pthread_mutex_unlock(&second_mutex); 
    
    pthread_exit(0); 
}
```

위의 예제에서 thread_one 이 first - second 의 순서로 mutex 락을 획득하려 하고, thread_two 가 second - first 의 순서로 mutex 락을 획득하려 한다. 만약 thread_one 이 first 를 취할 때 thread_two 가 second 를 취한다면 deadlock 이 발생한다. 

어떤 순서로 thread 가 실행될지는 CPU 스케줄러에 의해 달라진다. 

### 7.2.1 Necessary Conditions 

다음과 같은 네 가지 상황에서 데드락이 발생할 수 있다. 

1. Mutual exclusion : 적어도 하나의 자원이 nonsharable mode 에 잡혀있어야 한다. 다시 말해 한번에 오직 하나의 프로세스만이 해당 자원을 사용할 수 있다. 만약 다른 프로세스가 그 자원을 요청한다면 요청하는 프로세스는 반드시 자원이 해제되기 전까지 기다려야 한다. 
2. Hold and Wait : 프로세스는 반드시 적어도 하나의 자원을 hold 하고 있어야 하고 추가로 다른 프로세스가 갖고 있는 자원을 획득하기를 기다리는 상황이어야 한다. 
3. No preemption : 자원들은 빼앗길 수 없다.  다시 말해, 자원은 그것을 잡고 있는 프로세스가 해당 자원을 다 사용하고 나서 자발적으로 해제하는 방식으로만 release 되어야 한다. 
4. Circular wait : 기다리는 프로세스의 set {P0, P1, P2 ... } 이 존재해야 한다. P0 -> P1 을 기다리고, P1->P2 를 기다리고 ... Pn -> P0 를 기다리는 상황이 있어야 한다. 

> 위의 네 가지 상황이 모두 만족되어야 데드락이 발생한다. 

### 7.2.2 Resource-Allocation Graph 

데드락은 system resource allocation graph 를 통해 더 구체적으로 묘사할 수 있다. 

* P = {p1, p2, p3 ... } : active processes in the system 
* R = {r1, r2, r3 ... } : all resource types in the system 
* p -> r : 프로세스가 자원을 요청한다. (request edge)
* r -> p : 자원이 프로세스에 할당되었다. (assignment edge)

그래프에 사이클이 존재하지 않는다면 시스템에 데드락이 없다는 뜻. 만약 사이클이 있다면 deadlock may exist. 

만약 각 자원의 인스턴스가 한개씩밖에 없으면 사이클이 있다는 것은 데드락 발생의 필요충분 조건이다. 

만약 자원의 인스턴스가 여러개 있다면 사이클이 있다고 해서 반드시 데드락이 발생하는 것이 아님. 



### 7.3 Methods for Handling Deadlocks 

데드락은 아래와 같은 세 가지 방법으로 다룰 수 있다. 

* Protocol 사용 : 시스템이 절대로 데드락에 빠지지 않도록 프로토콜을 사용한다. 
* Detect & Recover : 시스템이 데드락에 빠지게 놔두고, 데드락을 감지했을 때 이를 recover 한다. 
* **Ignore : 데드락 문제를 아예 무시하고 마치 데드락이 시스템에 절대 일어나지 않다는 듯이 간주한다.** 

(놀랍게도) 세 번째 방법이 거의 모든 운영체제에서 채택하고 있는 방식이다. 프로그래머가 프로그램을 짤 때 데드락이 일어나지 않도록 해야한다. 

* Deadlock Prevention : 데드락이 시스템에서 절대 일어나지 않도록 하기 위해 시스템은 deadlock prevention 혹은 deadlock avoidance scheme 을 사용한다. 위의 4가지 데드락 발생 조건에서 적어도 하나는 만족할 수 없도록 방법을 만들어둔다. **이 방법들은 자원에 대한 요청을 하는 방식을 제약한다.** 
* Deadlock avoidance : 운영체제에 프로세스 하나의 lifecycle 동안 어떠한 자원을 요청할지에 대한 정보를 제공해야 한다. 이 정보를 통해서 해당 프로세스가 언제 기다려야 하는지 예측할 수 있다. 

만약 시스템이 deadlock-prevention 혹은 deadlock-avoidance 알고리즘을 사용하지 않는다면 데드락 상황은 반드시 발생할 수 밖에 없다. 이런 환경에서는 시스템은 시스템의 상태를 검진하는 알고리즘을 제공하여 데드락이 발생했음을 감지하고, 또 이를 회복할 수 있는 알고리즘을 제공해야 한다. 

이런 알고리즘들(detect & recover)이 제공되지 않는다면 시스템에서 데드락이 일어났는지 발견하기 어렵다. 이런 경우 데드락이 발생한다면 시스템의 성능이 저하될 것이다. 최악의 경우 수동으로 시스템을 리부트 하는 방법밖에 남지 않는다. 

데드락을 감지하고 회복하는 알고리즘을 제공하지 않는 방법은 그럴싸해 보이지는 않지만 현대 거의 모든 운영체제들이 채택하고 있는 방식이다. 데드락의 가능성을 아예 무시하는 것이 다른 방법들보다 훨씬 cost 가 적기 때문이다. 만약 하나의 시스템에서 데드락이 1년에 한 번 발생한다면 이를 무시하는 것이 더 경제적이다. 뿐만 아니라 다른 조건들로부터 recover 하기 위한 방법들을 데드락을 해결하는 데 가져다 쓸 수 있다. 예를 들어 가장 높은 우선순위를 갖고 돌아가는 real-time process 가 있다고 하자. 이 프로세스가 운영체제에 control 을 계속해서 돌려주지 않는다면 시스템은 frozen state 가 된다. 이 때 recover 하는 방식을 데드락 회복 방식으로도 사용할 수 있다. 

---

```
[DISCLAIMER]
서울대학교 홍성수 교수님의 운영체제 강좌 필기입니다. 
본 PPT 캡처가 문제될 시 즉시 삭제하겠습니다. 
```

* [SNU etl 에서 수강 가능합니다](http://etl.snu.ac.kr/)

## Lecture 12 Deadlock 

### 12-2 Deadlock Prevention and Detection 

#### Bankers Algorithm

![image-20200505150526233]({{site.baseurl}}/assets/img/image-20200505150526233.png)

프로세스가 자원을 점유하고 있는 상태를 두 상태로 나눈다. 

* Safe state : 프로세스가 원하는 요구를 다 들어줄 수 있는 상태. 자원 부족이 해결 가능. 절대 deadlock 이 발생하지 않는 상태. 

* Unsafe state : safe state 와 달리 deadlock 이 발생할 수 있는 상태. 

어떤 의미에서는 좀 비관적, 보수적인 알고리즘이다. Unsafe 하다고 반드시 데드락이 발생하는 것은 아니지만, unsafe 하다고 단정짓기 때문. 

* 비유 : IMF 얘기. 제일 은행에서 대출 가능한 예금 1000억이 있다고 해보자. 대우가 600억, 한보가 300억을 대출해 갔으면 제일은행의 잔고는 100억. 별 문제 없어보임. 문제는 대우, 한보가 회계 감사 결과 외국 은행에서 더 차입을 받았다는 것이 알려져. 대출 갚으려면 200억을 제일은행에서 빼와야 하는 상황 발생. 이 때 제일은행은 충분한 예금이 없다. 즉, 돈을 갚기 위해 다시 돈을 빌려야 하는 상황. => deadlock ... 돈이 없어서가 아니라 유동성의 위기. 
* exit plan 이 잘 되어있는지 확인하고 빌려줬어야 하는데 마구잡이로 빌려줘서 문제 된 것. 뱅커스 알고리즘은 이런 상황이 발생하지 않도록 한다. 

![image-20200505151106911]({{site.baseurl}}/assets/img/image-20200505151106911.png)

12개의 마그네틱 드라이브 자원을 세 개의 프로세스가 사용한다. p1, p2, p0 가 최대 필요한 자원의 수와, 현재 사용하고 있는 자원의 수를 표시한다. 

은행장은 이 상태가 안전한 상태인지 체크해야 한다. 

자기가 요청할 수 있는 최대의 자원을 요청했을 때 deadlock 이 발생하는지 확인하면 된다. 

가장 먼저 확인해야 할 것이 현재 남아있는 **가용 자원이다.** => 3개 남아있음. 

p0 는 terminate 시킬 수 없고, p1을 보니 2개만 줘도 종료 가능. p1을 종료시킨다. p1이 종료하면 회수된 자원 5개. 

p0 에게 5개를 주면 수행을 종료, 10개를 반환할 수 있다. 이를 갖고 p2의 수행을 종료시킬 수 있다. **따라서 모든 p 들을 종료시킬 수 있으므로 safe state 라 한다.** 

하지만 safe 하다고 항상 safe 한 것은 아니다. p2가 1개를 더 달라는 요청을 하는 경우, 이 요구를 들어줄 수 있는가? 3이 되면 가용 자원이 2개가 된다. (처음에 3개 있었는데 여기서 -1) p1을 종료시키고 이를 통해 총 4개의 가용 자원이 생기지만 이것을 갖고는 p0를 종료시킬 수 없어 deadlock 이 발생한다. 

![image-20200505151629806]({{site.baseurl}}/assets/img/image-20200505151629806.png)

자원 몇개 필요한지 가늠해야 한다. safe state 가 아니라면 자원을 요청한 프로세스를 waiting state 로 만든다. 

![image-20200505151727276]({{site.baseurl}}/assets/img/image-20200505151727276.png)

* 시스템에 존재하는 프로세스(n)와 자원(m)을 표현해야 한다. 

![image-20200505151847800]({{site.baseurl}}/assets/img/image-20200505151847800.png)

* 작다 라는 것의 정의 

![image-20200505151909086]({{site.baseurl}}/assets/img/image-20200505151909086.png)

* safey 확인 

work array : 어떤 자원의 중간 계산 결과를 저장하고 있는 배열. 

finish array : 어떤 프로세스가 현재 종료되었는지를 나타내는 boolean array. 

**step 0 : finish = false (어떤 프로세스도 끝나지 않았다)

자원들을 할당해주면서 프로세스를 종료시키고, 자원을 회수한다. 회수한 후 다시 자원이 필요한 곳에 할당하며 모든 프로세스를 종료시키는 작업을 반복한다. 

![image-20200505152607954]({{site.baseurl}}/assets/img/image-20200505152607954.png)

Request 배열에 자원의 요청을 기록한다. 필요한 자원이 가용 자원보다 적으면 safety check 을 돌려서 safe 하면 요청을 들어주고, 아니면 요청을 들어주지 않는다. 

![image-20200505152716977]({{site.baseurl}}/assets/img/image-20200505152716977.png)

> 자원 타입이 여러개 있다는 점이 아까와는 다르다.

**뱅커스 알고리즘은 deadlock avoidance 의 한 방법이다.**  

### 12-3 Deadlock Detection and Recovery

데드락을 피할 수 없을 때는? 

* 피하기 위해서 미리 계산하고, 
* 보수적인 플랜을 사용해야 하므로... 언제고 피할 수는 없다. 

![image-20200505152941061]({{site.baseurl}}/assets/img/image-20200505152941061.png)

> 데드락 Avoidance 의 한계 

디텍션은 Resource allocation graph 를 그려서 사이클이 존재하는 것을 확인하여 디텍트 할 수 있는데, 이는 자원의 인스턴스가 여러개인 경우 통하지 X 

![image-20200505153034087]({{site.baseurl}}/assets/img/image-20200505153034087.png)

자원을 주면서 끝낼 수 있는 것 끝내고, 데드락 때문에 종료되지 못하는 것들을 확인한다. 

데드락을 디텍트 하면 이를 회복해야 한다. 

데드락이 발생하면 이를 가장 잘 핸들할 수 있는 사람은 OS 가 아니라 프로그래머, 사용자이기 때문에 여태 주목을 받지 못했다. PC에서는 데드락이 그렇게 큰 문제라고는 얘기할 수 없다. 하지만 서버 컴퓨터 에서는 문제가 될 수 있을 것. 하지만 이렇게 정교한 방법으로 데드락을 detect 하는 것이 아니고, 대개 프로세스 response time 을 모니터링 하는 타임아웃 루프를 둔다. 워치독 타이머. 어떠한 프로세스가 일정 시간이 지나도 종료되지 않으면 강제적으로 종료시킴. 

![image-20200505153239196]({{site.baseurl}}/assets/img/image-20200505153239196.png)

하지만 경우에 따라서는 강제 종료가 expensive 할 수 있다. 이 때 CPR 이라는 방법을 많이 쓴다. 

CPR : Check Point and Resume. 응용 프로그램이 동작 중에 일정 시간 간격으로 모든 프로그램 상태를 디스크에 저장하고 문제가 발생했을 때 저장된 프로그램의 상태를 사용해 이의 수행을 재개하는 기법. 

매일 백업을 했을 때 28일 째 정오에 죽었으면 28일째 새벽의 체크 포인트를 갖고 수행하면 그 이전의 것은 백업할 수 있음. 

데드락을 detect 하면 그냥 죽이거나, 함부로 죽일 수 없으면 CPR을 사용한다. 그러려면 반드시 rollback 이 필요하다. 

![image-20200505153501172]({{site.baseurl}}/assets/img/image-20200505153501172.png)

데드락이라 하는것은 흔히 발생하는 것이지만 환경에 따라 중요도는 다르다. 

mission critical 한 시스템에서는 데드락이 매우 중요한 문제. 

커널 안에도 굉장히 lock 을 많이 쓰는데, 커널 안의 데드락이 발생하면 시스템에 큰 문제가 발생한다. 커널이 발생하지 않으면 워치독 차이머가 떠서 커널을 죽여버림. 

안드로이드 운영체제에 간혹 데드락이 발생하여 커널이 리부트 하는 경우가 있다. 