---
layout: post
title: "[OS] Evolution of UNIX File System"
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

# Lecture 24 : Evolution of UNIX File System 

![{{site.baseurl}}/assets/img/oslec24/image-20200624203119712]({{site.baseurl}}/assets/img/oslec24/image-20200624203119712.png)

> Unix 에는 변종이 굉장히 많다. 

* FFS 에서는 s5fs 의 성능 저하 문제들을 많이 해결했음. UFS는 Unix File System 을 의미함. 
* 삼바 같은 것이 대표적인 NFS이다. 
* VFS : User interface 를 단일화 하기 위한 가상 레이어. 
* LFS : recovery 의 오버헤드를 줄이기 위한 방식 



![{{site.baseurl}}/assets/img/oslec24/image-20200624203432434]({{site.baseurl}}/assets/img/oslec24/image-20200624203432434.png)

> System Five File System 

* 하나의 디스크 드라이브에 파일 시스템이 존재함. File System 은 Volume 당 하나, 두 개의 Volume 에 걸쳐서 존재할 수 없다. 
* 블럭 사이즈는 512 바이트 (그 당시의 섹터 사이즈) 
* 논리 블럭 넘버가 생김. 
  * Disk 를 sector 들의 linear array 로 볼 수 있다. 

![{{site.baseurl}}/assets/img/oslec24/image-20200624203647581]({{site.baseurl}}/assets/img/oslec24/image-20200624203647581.png)

> 디스크상의 레이아웃 

* ROM 에 있는 부트 프로그램은 이미 설정되어 있는 부트 디바이스의 부트 블럭을 가져오는 역할을 함. 부트 블럭안의 부트로더가 운영체제의 이미지를 찾고 일을 하게 됨. **부트 로더가 저장되어 있는 공간을 Boot Area 라 함.**

![{{site.baseurl}}/assets/img/oslec24/image-20200624204257832]({{site.baseurl}}/assets/img/oslec24/image-20200624204257832.png)

* Super Block : 볼륨의 파일 시스템에 대한 모든 메타데이터가 들어가있음. (전체 블럭의 개수, Free Node 의 개수 등 ... )
* inode List (Array) : 어떤 파일이 새로 만들어지면 inode 배열에서 하나 가져와서 메타데이터를 기록하게 됨. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624204344194]({{site.baseurl}}/assets/img/oslec24/image-20200624204344194.png)

**굉장히 단순함** 

* 성능상의 문제 : File open 후 read 해서 읽어오려면 반드시 파일의 i-node 를 가져와야 함. 이를 위해 **seek 를 하게 되고**, i-node 상의 block index 를 따라서 data block 을 가져오게 됨. seek 를 해야 해서 느려짐. 또, 블럭을 할당할 때는 data block 영역에서 가져오게 되는데, allocation free 를 하게 되면 **free block 들이 data block 에 산재되어 있게 됨**. Sequential read 할 때 많은 seek 가 발생함. 
* 안정성의 문제 : 이 외에도 super block 이 깨지게 되면 (bad block) file system 전체가 망가지게 됨. 

* 컴퓨터 시스템에서 빈번하게 일어나는 연산들에 대해 최적화가 되어있지 않음. 예를 들어 `ls -al` 하면 i-node 들 쭉 읽어와야 하는데 이 연산이 매우 느림. 

![{{site.baseurl}}/assets/img/oslec24/image-20200624204404378]({{site.baseurl}}/assets/img/oslec24/image-20200624204404378.png)

> 80년대만 해도 고칠게 많아서 논문 쓰기 쉬웠다는 ... : (



![{{site.baseurl}}/assets/img/oslec24/image-20200624204429576]({{site.baseurl}}/assets/img/oslec24/image-20200624204429576.png)

> 버클리에서 만든 Fast File System 

* 안정성과 성능 문제를 해결하기 위해 실린더 그룹이라는 개념을 도입. 논리적으로 연관된, 같은 디렉토리의 파일들을 동일한 실린더 그룹에다가 할당함. 
* Super Block 의 안정성 문제를 위해 이 블럭을 복사해서 실린더 블럭마다 수퍼 블럭을 놓음. 아주 조직적으로 배치해서 한 실린더 혹은 플레이트가 날라가도 정보를 어딘가에서 회복할 수 있게끔 함. 

![{{site.baseurl}}/assets/img/oslec24/image-20200624204635012]({{site.baseurl}}/assets/img/oslec24/image-20200624204635012.png)

> 새로운 개념들 도입 

* Fragment 는 기본적으로 sector 임. 어떤 경우는 sector 를 여러 개 할당해야 유리한 경우가 있는데, 이 때 Fragment 를 여러 개 묶어서 블럭의 크기를 더 크게 잡음. 

![{{site.baseurl}}/assets/img/oslec24/image-20200624204728509]({{site.baseurl}}/assets/img/oslec24/image-20200624204728509.png)

* 실린더 그룹을 적절히 활용해서 연관된 블럭들을 같은 그룹에 할당 



![{{site.baseurl}}/assets/img/oslec24/image-20200624204755126]({{site.baseurl}}/assets/img/oslec24/image-20200624204755126.png)

> 한 실린더 그룹에 모든 걸 담으면 좋지 않다. 

* 디렉토리에 있는 큰 파일들은 하나의 실린더 그룹에 담을 수 없어, 인접한 실린더 그룹에 데이터를 넘김. 더 커지면 더 멀리 보냄. 옆의 실린더 기준은 48KB임. 이를 넘어가면 indirect access 를 해야 함. 
* 적절히 펴 놓음으로서 블록들이 랜덤하게 흩어지는 것을 제어할 수 있다. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624204925615]({{site.baseurl}}/assets/img/oslec24/image-20200624204925615.png)

> seek 뿐만 아니라 rotational delay 까지 최적화 하려 함. 

Rotational Delay 가 2이면 두 섹터를 보낼 만큼의 시간이 보장되어야 한다는 뜻. 

디스크에 데이터를 저장할 때, 0번 블럭을 저장하고 바로 1번을 저장하는게 아니라, 2개의 간격으로 배치하게 됨. 한 바퀴를 기다리지 않아도 된다는 장점이 있음. 

**=> FFS는 깨알같은 최적화가 많다.** 성능이 괜찮아서 많이 쓰임 



![{{site.baseurl}}/assets/img/oslec24/image-20200624205228596]({{site.baseurl}}/assets/img/oslec24/image-20200624205228596.png)

* 과거의 시스템 5에는 파일 이름에 14 캐릭터밖에 사용 못하고, 한 파일 시스템이 64KB만큼의 파일만 갖고 있을 수 밖에 없는 제약들이 있었음. 이를 개선함. 
* 심볼릭 링크의 개념도 이 때 처음 도입됨. 
* 하드 링크는 동일한 파일 시스템 안에서 가능, 심볼릭은 이 바운더리를 넘을 수 있는 것. 
* 상업 시스템에서 여러 유저가 이용할 수 있도록 disk quota 개념을 추가. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624205428516]({{site.baseurl}}/assets/img/oslec24/image-20200624205428516.png)

> Disk space 관리도 잘 되고 read write 도 빨라짐. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624205448867]({{site.baseurl}}/assets/img/oslec24/image-20200624205448867.png)

> 하지만 FFS 도 어쩔 수 없이 한계를 보이게 됨. 

* Crash recovery 의 시간이 점점 더 오래 걸리게 되고
* Sequential read 의 문제도 그대로 있었음 
* `fsck` : File system check 가 오래 걸림 



![{{site.baseurl}}/assets/img/oslec24/image-20200624205613398]({{site.baseurl}}/assets/img/oslec24/image-20200624205613398.png)

> 그래서 자연스럽게 나온게 Log structured File system 

**Crash Recovery 왜 해야하는지** 

* File system 의 접근 성능을 높이기 위해서 i-node cache 를 갖고 있어야 하고, i-node 가 수정될 때마다 write back 을 하는데, 미처 write back 하지 않은 상태에서 shutdown 이 일어나면, i-node 가 생각하는 file structure 와 disk 의 file structure 가 일치하지 않을 수 있음. 
* Disk 의 file 을 조작하는 연산이 사실은 single transaction 이어야 하는데 여러개의 sub-operation 으로 이루어져 있음. (transaction : 하거나 아니면, 아예 roll back. all or nothing) 
  * 예를 들어 delete file 을 한다고 하면 디렉토리에서 엔트리를 제거하고, 해당 i-node 와 data block 들을 free 해야 한다. 이런 순서가 지켜지지 못하고 꼬인다면 문제가 될 것. 기존의 Unix 시스템에서는 이런 연산들을 serialize 시킴. (**synchronous write**; 문제 - 운영체제가 디스크의 연산이 끝까지 일어나는지 계속 지켜봐야 함. 달나라까지 가는 것을 기다리는 셈)



![{{site.baseurl}}/assets/img/oslec24/image-20200624210149429]({{site.baseurl}}/assets/img/oslec24/image-20200624210149429.png)

> fsck 가 어떻게 작동하는지 먼저 알아보자. 

리눅스의 lost + found 디렉토리에는 정체 불명의 파일 조각들이 들어가 있다. 

1. 시스템에 존재하는 아이노드들을 다 얻어온다. 
2. 아이노드들의 인덱스를 이용해 블럭들을 재구성 한다. 
3. 디렉토리 계층구조를 얻으면 구조별로 파일 콘텐츠를 확인. 
4. 파일들의 아이노드의 인덱스에 매핑된 데이터들이 잘 있는지 확인함. 
5. 이 과정에서 확인되지 않는 것들을 lost + found 로 보냄 



![{{site.baseurl}}/assets/img/oslec24/image-20200624210333879]({{site.baseurl}}/assets/img/oslec24/image-20200624210333879.png)

> LFS 는 과연 어떻게 구현하는 것인가? 

* LFS는 기존의 파일 시스템과 달리 write, re-write 하면 새로 추가될 정보만 file 의 뒤에다가 append 하게 됨. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624210436711]({{site.baseurl}}/assets/img/oslec24/image-20200624210436711.png)

> 디자인 고려 사항 

* 무엇을 로그로 남길 것인가 
* redo, undo 로그도 필요할 것 
* GC 필요. 앞의 invalidated 데이터들을 모아서 정리해줘야 함. 
* Log 가 많아지면 sequential read performance 가 매우 떨어짐. 그래서 어떤 시기마다 로그들을 전부 읽어서 처리하여 정규 파일의 구조로 만드는 작업을 해야 함. 



✔ 시스템이 reboot 될 때 log 만 보고 파일 시스템을 재구성 하면 됨. 

뿐만 아니라, 플래시에서는 erase before write 제약 때문에 LFS를 사용할 수 밖에 없다. 





# 24-2 Disk Scheduling 

![{{site.baseurl}}/assets/img/oslec24/image-20200624210744032]({{site.baseurl}}/assets/img/oslec24/image-20200624210744032.png)

> File system 구조를 다시 상기해 보자. 

디스크 스케줄링... 어떻게 일어나는 것인가? 



![{{site.baseurl}}/assets/img/oslec24/image-20200624211050601]({{site.baseurl}}/assets/img/oslec24/image-20200624211050601.png)

![{{site.baseurl}}/assets/img/oslec24/image-20200624211013995]({{site.baseurl}}/assets/img/oslec24/image-20200624211013995.png)

> 디스크 스케줄링이란 ? 

**디스크 스케줄링** : 디스크로 IO 요청이 들어온다. 예를 들어 몇번 플레이트, 트랙의 몇번째 섹터를 읽으라는 명령이 들어온다. seek 만 생각한다 하면 (실린더 넘버만 생각) 1, 5, 2, 4, 3 순서대로 들어온 요청을 어떻게 처리해야 할까? 

헤드가 1번 트랙에 있다면 순서대로 seek 를 반복하면서 처리를 해주게 된다. 엘리베이터가 층을 누른 순서대로 움직이는 ... 비효율적인 방법이다. 

위와 같은 요청을 효율적으로 처리하려면 reordering 이 필요하다. reordering 을 하려면 요청을 queue 에 넣어야 하고 이를 request queue 라 한다. 이 큐는 디바이스 드라이버와 디바이스 사이에 존재하게 된다. 최종적으로 sector read 명령을 날리는 것은 디바이스 드라이버 이므로. 

**큐의 순서를 조작하는 것을 IO 스케줄러라 한다.** 



![{{site.baseurl}}/assets/img/oslec24/image-20200624211524763]({{site.baseurl}}/assets/img/oslec24/image-20200624211524763.png)

들어온 순서대로 하면 총 10번의 seek 를 하게 되는데, 1~5 순서대로 하면 seek 가 4번으로 확 줄어든다. 

![{{site.baseurl}}/assets/img/oslec24/image-20200624211559889]({{site.baseurl}}/assets/img/oslec24/image-20200624211559889.png)



![{{site.baseurl}}/assets/img/oslec24/image-20200624211607388]({{site.baseurl}}/assets/img/oslec24/image-20200624211607388.png)

> 오더링을 위한 정책들 

* FIFO : 성능 안좋음 
* SSTF (shortest seek time first) : 헤드가 5번 실린더에 있으면, 먼 실리더는 하지 않고 근처에 들어온 것들 먼저 처리하는 것. 만약 우리 엘리베이터를 그렇게 가동하면 starvation 이 발생할 것이다 ... 
* 엘리베이터, 일종의 스캔 알고리즘처럼 해야. 한방향으로 쭉 가다가 걸리는 손님들을 태운다. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624211755659]({{site.baseurl}}/assets/img/oslec24/image-20200624211755659.png)

> 엘리베이터 방식대로 해야 함. 

* 스캔 : 쭉 가다가 걸리는 애들 처리해 주는 것. 문제는 양 쪽 끝은 기회가 한 번 밖에 주어지지 않아 unfair 하다는 점. 
* Circular Scan : 한 방향으로만 계속 도는 더 fair 한 방식. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624211854766]({{site.baseurl}}/assets/img/oslec24/image-20200624211854766.png)

> 스케줄링 정책 예시 

스캔, 서큘러 스캔이 이상적인 디스크 스케줄링 알고리즘 이라 할 수 있다. 



**요즘에는 약간 다른 이슈들이 있다.** 

플래시에는 seek 가 없어서 이런 식의 스케줄링이 통하지 않아... 요즘은 플래시 메모리 컨트롤러가 굉장히 복잡해. 디스크보다 훨씬 더 복잡한 스케줄링 알고리즘 필요. 



![{{site.baseurl}}/assets/img/oslec24/image-20200624212118250]({{site.baseurl}}/assets/img/oslec24/image-20200624212118250.png)

> 한 학기의 마지막 수업 요약. 