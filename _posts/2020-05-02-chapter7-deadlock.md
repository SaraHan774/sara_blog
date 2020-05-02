---
layout: post
title: "[OS] Deadlock"
author: "Sara Han"
categories : Tech
comments : false
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
