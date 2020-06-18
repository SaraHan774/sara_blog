---
layout: post
title: "[OS] Files and Directories"
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

# Lecture 22 : Files and Directories 

![{{site.baseurl}}/assets/img/oslec22/image-20200618113752456]({{site.baseurl}}/assets/img/oslec22/image-20200618113752456.png)



## Files 

![{{site.baseurl}}/assets/img/oslec22/image-20200618113909261]({{site.baseurl}}/assets/img/oslec22/image-20200618113909261.png)

> *어떤 과목을 공부할 때 중요한 것은 Terminology 에 익숙해 지는 것이다. 나중에 profession 을 바꿔서 힘든 이유는 용어에 익숙하지 않기 때문이다. 용어에 익숙해져야 할 뿐 아니라, 주어진 기술을 정확한 용어로 표현하는 것도 중요하다. 그런 의미에서 File 이라는 용어의 정의를 눈여겨 보자.* 

* **File : Named Collection of bytes stored on disk** 

  * 이름이 있어야 하고 
  * 디스크에 저장이 되어있어야 한다. (요즘은 SSD 많이 쓰긴 하지만)

* **File system 이 제공해 주어야 하는 기능** 

  (1) String, Name 을 부여해야 한다. 

  (2) Disk storage 를 잘 관리해야 한다. 

  > 운영체제 입장에서 보면 디스크에 저장된 일련의 블럭들이 File 이다. (Block Sequence) 왜? 디스크는 Block Device, Block 단위로 operation 이 일어나므로. 유저 입장에서 보면 Byte Sequence 이다. 

  (3) File System 에서 유저가 보는 Byte Sequence 로 Block 을 매핑하는 역할도 해야 함을 알 수 있다. (Block sequence view -> Byte sequence view pack / unpack 해야 함)



> *Process 의 정의는 Program in execution : 짧고 명확하게 정의할수록 전문가이다.* 



![{{site.baseurl}}/assets/img/oslec22/image-20200618114415789]({{site.baseurl}}/assets/img/oslec22/image-20200618114415789.png)

> File 에 가해지는 연산들 



![{{site.baseurl}}/assets/img/oslec22/image-20200618114504358]({{site.baseurl}}/assets/img/oslec22/image-20200618114504358.png)

### File System 역할 : Naming 

Disk 위의 파일을 언급할 수 있어야 하므로 text name 을 사용하도록 OS 가 허용한다. 

하드웨어 IO장치에 대해서도 이런 naming 을 적용시킨다. 

Naming 을 하기 위해서 운영체제가 어떤 일을? 

* 내부적으로 숫자에 기반한 File identifier 를 사용함. 
* File descriptor 와 같은 정보들이 연관되어 있다. 
  * Data contents 들에 부속된 메타 데이터가 file descriptor 이다. 
  * 이 descriptor 를 가리키는 핸들은 integer ID 로 되어 있음. 

✔ Text 로 된 이름을 File Descriptor ID 로 바꾸는 역할을 해주어야 한다. 



File 이 너무 많으면 피라미드 원리에 따라서 파일들을 그루핑한다. 

​	=> Directory (Unix 에서 나온 말)



![{{site.baseurl}}/assets/img/oslec22/image-20200618115019917]({{site.baseurl}}/assets/img/oslec22/image-20200618115019917.png)

### File Descriptors 

* 파일의 메타데이터를 저장한 것. 
  * File 의 데이터 content 가 있는 한 영원히 메타데이터가 필요하다. 
  * 메타데이터는 File 과 마찬가지로 storage 에 저장해야 함. 
* File 의 크기, 최근 액세스 시간, 파일의 소유자, 접근 권한 등을 넣을 수 있다. 
  * 파일의 속성을 보고 싶으면 리눅스에서 `ls` 커맨드를 사용하면 된다. 
  * `ls -l` 은 verbose option 으로 좀 더 자세히 표시. 
  * `ls -li` 은 파일의 descriptor name 까지 보여줌. (File descriptor index)
    * Unix 에서는 이 파일 디스크립터를 `Inode` 라고 부름. 그리고 파일 디스크립터 아이디를 `Inumber` 라고 함. 

![{{site.baseurl}}/assets/img/oslec22/image-20200618115418467]({{site.baseurl}}/assets/img/oslec22/image-20200618115418467.png)

디스크에 배열을 만들어서 File descriptor 들을 할당했었음. 

* 단점 : File system 이 bad block 과 같은 문제들에 취약할 수 있음. (Floppy disk 시절 이지만 ... )



**File Descriptor 들을 디스크의 특정 영역에 모아두면 성능 저하가 발생하는 원인?**

* 디스크 상에서 File Descriptor 와 File contents 사이의 물리적인 거리가 멀기 때문에 디스크의 Seek Time 이 증가함. (Arm movement 는 mechanical operation...)

File system 의 데이터를 어떻게 설계하는지가 성능에 큰 영향을 미쳐 

* 요즘은 Copy 를 유지하면서 여러 곳에 흩어져 저장시켜 둠. 



File 을 접근하려면 반드시 File Descriptor 을 접근해야 하는데, 매 번 FD 를 읽어오는 것은 바람직하지 않다. in-memory cache 를 유지, copy 를 유지함. (성능은 향상됨)

* Trade-off : 디스크와 캐시 사이의 값을 sync 시키는 것에 문제가 발생할 수 도 있음. 

> *DBMS 분야는 redundancy 와의 전쟁이다. 읽기만 하는게 아니라 modify 를 하기 때문에 File Disk 와 Cache 에 있는 Descriptor 값이 달라질 수 있음.* 

운영체제 입장에서는 Disk 블럭들을 스캔하면서 Bottom up 으로 File 을 재구성 해야 함. 그리고 File System 을 설계할 때 이렇게 재구성 가능하도록 설계해야 함. 

하지만 요즘은 Disk 의 용량이 TB 수준이기 때문에 회복 시간이 오래 걸린다. 



![{{site.baseurl}}/assets/img/oslec22/image-20200618120324081]({{site.baseurl}}/assets/img/oslec22/image-20200618120324081.png)

> *File Descriptor : i-node / File Descriptor ID : i-number* 



## Directory 

Unix 에서 만든 neat 한 시스템 

![{{site.baseurl}}/assets/img/oslec22/image-20200618120510375]({{site.baseurl}}/assets/img/oslec22/image-20200618120510375.png)

디렉토리 : 파일 혹은 sub 디렉토리를 갖고 있는 엔티티. 내부적으로 파일과 똑같이 처리함. (데이터파일, 디렉토리 파일로 속성상의 구분은 있으나)

* 파일의 이름과 그 파일의 아이디 Pair 를 데이터 콘텐트로 받는다. 
  * 파일의 텍스트 string 을 파일 아이디로 바꿔주기 위한 링크를 디렉토리가 제공해줌. 
  * 디렉토리를 프로세싱 하는 함수들은 일반 파일들과 크게 다르지 않다. 



![{{site.baseurl}}/assets/img/oslec22/image-20200618120719474]({{site.baseurl}}/assets/img/oslec22/image-20200618120719474.png)

파일 시스템의 Top 을 root 디렉토리라 하는데, 모든 파일들은 Unique 한 file path 를 갖게 된다. (Tree 의 leaf 는 모두 unique 한 traverse path 를 갖게 된다.)

Full Name : Full path name 

![{{site.baseurl}}/assets/img/oslec22/image-20200618120816414]({{site.baseurl}}/assets/img/oslec22/image-20200618120816414.png)

>  *제일 앞에 있는 슬래시가 루트 디렉토리. a는 루트 밑의 디렉토리. b는 a 밑의 서브 디렉토리. c 는 b에 들어있는 파일이다.* 

* 2 -> 5 -> 7 -> 14 인덱스로 파일을 찾아가게 됨. 

이와 같은 path name 을 parsing 해서 결국 얻고자 하는 것은? 

* String name 을 통해서 14 라는 숫자를 얻고 싶은 것이다. 
* **이 넘버를 얻을 수 있는 매개체 역할을 하는 것이 tree structured directory 이다.** 



**현재 메인메모리에 cache 된 것이 root 밖에 없을 때 file c 를 접근하기 위해 disk access 를 몇 번 해야 할까?**

1) 5 를 Disk 에서 읽어온다 - 1번 

2) 5 가 가리키는 데이터 블럭을 읽어온다. - 2번 

3) 7을 읽어온다. - 3번 

4) 7이 가리키는 데이터 블럭을 읽어온다 - 4번 

5) 14를 읽어온다. - 5번 

6) 14 가 가리키는 파일을  읽어온다 - 6번 

=> 무려 여섯번의 디스크 액세스가 필요함. 

빈번하게 액세스 되는 것들이 버퍼 캐시에 있다면 더 빠를 것. 



![{{site.baseurl}}/assets/img/oslec22/image-20200618121535961]({{site.baseurl}}/assets/img/oslec22/image-20200618121535961.png)

> *very nice.* 





### 부수적인 것들 

![{{site.baseurl}}/assets/img/oslec22/image-20200618121610273]({{site.baseurl}}/assets/img/oslec22/image-20200618121610273.png)

파일의 이름만 주었을 때 뒤에 현재까지의 디렉토리의 path name 을 prepend 시킴. 



![{{site.baseurl}}/assets/img/oslec22/image-20200618121751714]({{site.baseurl}}/assets/img/oslec22/image-20200618121751714.png)

심볼릭 링크와 하드링크. 

**링크라는 것은?** 

* 트리 구조를 망가트리는 것.

**심볼릭 링크가 훨씬 더 유연하다.** 

i-number는 file system volume 단위로 부여한다. 따라서 i-number 는 file system volume 단위로만 unique 한 것임. volume 은 다른데 링크를 만들고 싶으면 sym link 를 만들어야 함. 