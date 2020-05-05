---
layout: post
title: "[OS] Linker and Loader"
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

## Lecture 13 

### 13-1 Linker and Loader 

링커와 로더는 컴파일러 관련된 이슈이다. 어떤 규약에 따라 executable 파일을 주고 받고 많은 일들을 한다. 

구체적인 얘기를 하기 위해 GNU의 링커를 갖고 설명하겠다. 

1. Program Development Process and Tools 
2. Linking Process 
3. Conclusion
4. Project Description

### Program Development Process and Tools 

![image-20200505125017273]({{site.baseurl}}/assets/img/image-20200505125017273.png)

C 프로그램의 특징 : 각각의 소스 파일이 하나의 파일로 변할 수 있다. 

* `.o` 파일을 묶는 것을 Linker, linkage editor 라고 한다. 묶이면 하나의 object 파일이 생기고 이것을 executable 파일이라 한다. 
* 이 파일이 로더에 의해 운영체제에 깔리고 실행할수 있게 된다. 

![image-20200505125131622]({{site.baseurl}}/assets/img/image-20200505125131622.png)

![image-20200505125139459]({{site.baseurl}}/assets/img/image-20200505125139459.png)

* 소스파일: 사람이 읽을 수 있는 심볼들로 작성한다. 문법 있음. 
* 소스파일이 컴파일러에 의해 object 파일로 변한다. 이 파일 안에는 instruction 과 data가 있음. 프로그램에서 사용하는 symbols 들에 대한 정보 모아둔 symbol table 도 있음. 링커가 프로그램 링킹을 하기 위해 사용하는 정보들 저장하는 relocation table 도 있음. 

![image-20200505125346591]({{site.baseurl}}/assets/img/image-20200505125346591.png)

* 소프트웨어 개발 툴들을 보자 
  * Compiler : src -> o 
  * Linker : o -> executable 
  * Loader (Tool 은 아니고 OS 의 일부분) : fork -> exec, 오브젝트 파일 실행할 수 있게 하는 역할. 

### Linking Process 

Compile 과 linking 을 보자. 

![image-20200505125459677]({{site.baseurl}}/assets/img/image-20200505125459677.png)

오브젝트 파일 안에는 여러개의 데이터 묶음 단위들이 있고 이것을 section 이라 한다. instruction, data, symbol table 각각 다른 섹션을 구성한다. 

이 섹션들은 독자적인 메모리 영역을 갖게 되고, 이것을 segment 라고 한다. 섹션 안의 심볼들은 절대 주소를 갖는게 아니라 섹션 내에서 자신의 주소, offset 을 주소로 갖는다. 

오브젝트 파일들을 구성하는 섹션들은 다음과 같다. 

![image-20200505125645459]({{site.baseurl}}/assets/img/image-20200505125645459.png)

* Text : 코드 섹션, 인스트럭션들. 
* Data : 초기화된 전역변수들.
* BSS : 초기화되지 않은 전역변수들이 있다. (Block started by symbol)
  * 초기화 여부로 왜 구분했을까? 
* Other sections : 심볼테이블, 리로케이션 테이블 ... 

> int c = 0; //0 이 차지하는 공간이 디스크상에 잡히게 된다. 

>  int valArray[1000000] //사이즈에 대한 정보만 넣는 것이 효율적이다. 

![image-20200505130106173]({{site.baseurl}}/assets/img/image-20200505130106173.png)

컴파일러가 텍스트 섹션, bss 섹션, data 섹션등으로 나눠서 저장한다. 섹션 안에서는 offset 을 주소로 받음. 

![image-20200505130202841]({{site.baseurl}}/assets/img/image-20200505130202841.png)

링커는 각 object file 안의 섹션들을 머징한다. 하나의 덩어리로 만듦. 

링커가 여러개의 섹션들을 레이아웃 할 때 어떤 형태로 하는가? 링커 스크립트라는 별도의 파일로 어떻게 저장하라 지정해 줄 수 있음. 

![image-20200505130315614]({{site.baseurl}}/assets/img/image-20200505130315614.png)

메모리 앞부분 1MB가 ROM 이라서 프로그래머는 1MB 다음부터 써야하고.. 이런 제약 조건이 embeded 쪽에서는 있다. 

![image-20200505130400816]({{site.baseurl}}/assets/img/image-20200505130400816.png)

레이아웃이 끝나면 exe 파일이 만들어 졌다는 거고, 이걸 OS 가 깔아주면 된다. 이미 몇번지에 뭐가 있는지 정보 있으니까 그대로 다 읽어주고, 맨 마지막에 프로그램 카운터를 그 exe 파일의 엔터리 포인트로 설정하면 끝난다. 



### 13-2 Linking Process 

하지만 여태까지는 굉장히 high level 관점이다. 

소스코드로 프로그래밍 할때는 모든 이름들이 다 심볼릭하다. 하지만 결국 수행할때는 010101.. 하면서 instruction fetch, decode, execute 를 반복한다. 프로세서는 피연산자, 연산자, 브랜치의 타겟 주소 등만 봄. 결국 심볼이 주소로 변환되어야 한다는 뜻. 심볼이 주소로 변환되려면 심볼이 어느 위치에 있는지 알아야한다. 

C는 소스파일, Obj 파일 다 쪼개져 있기 때문에 컴파일러가 하나의 o 파일을 만지고 있으면 다른 o 파일의 심볼을 알 길이 없다. cross reference 할 수 없다는 문제. 

![image-20200505130722835]({{site.baseurl}}/assets/img/image-20200505130722835.png)

컴파일러는 심볼이 나타나면 주소를 알아야 하고 ... 

파일 내에서 정의되고 사용되는 심볼들은 괜찮은데 **cross reference 문제 - 이걸 해결하는게 linker 의 의무** 

하지만 linker 도 이를 single path 로 해결할수는 없음. cross reference 에 관한 정보가 여러 o파일들에 흩어져 있기 때문이다. 여러 흩어진 o 파일들을 다니면서 주소들을 모아야함. 

![image-20200505130826769]({{site.baseurl}}/assets/img/image-20200505130826769.png)

**심볼 테이블 : 주소 정보를 일목요연하게 정리, 찾기 쉽게 게 한다.** 

my_func 라는 함수가 my_var 이라는 심볼이 심볼의 위치로 바뀌는 계산 작업이 필요. 이걸 symbol table 로 한다. 

symbol 의 이름과 주소가 테이블에 기재되어야 함. 

![image-20200505130943827]({{site.baseurl}}/assets/img/image-20200505130943827.png)

심볼 테이블의 엔트리 : 심볼의 이름, 섹션, 오프셋. 검색할때는 이름으로 검색한다. 

![image-20200505131024852]({{site.baseurl}}/assets/img/image-20200505131024852.png)

심볼 테이블을 보면 주소를 알수 있는것도 있고, 없는것도 있다. 알수있는건 같은 o 파일에 있다는 것. 주소가 섹션주소 + offset 이라 했는데 이것을 PC relative 주소 혹은 relative address 로 바꾼다. 

src 파일을 보면 이함수가 call instruction 으로부터 얼마나 떨어져 있는지 알 수 있다. 이런 주소를 pc relative address 라고 한다. 현재 주소로부터 앞, 뒤 점프하는 것. 

왜 이런 복잡한 주소가 필요한가? 전체 Section 의 위치에 관계 없이 PC 로부터의 상대적인 주소는 바뀌지 않기 때문. **pc relative 주소는 relocatable address 이다.** 전체 섹션이 어디로 갈지 모르지만, 이 주소는 더이상 fetch 할 필요가 없다. 항상 나와 일정한 거리로 떨어져 있는것만 생각하면 되므로. 

![image-20200505131352119]({{site.baseurl}}/assets/img/image-20200505131352119.png)

나머지들 : **다른 모듈에 있는 주소들이 어려워짐. cross reference.** 

**cross reference 는 글로벌 function/variable 인데, 다른 파일에 있는 경우 주소를 모르니까 확인을 해야 한다.** 

이런 cross reference 를 만나게 되면, 링커가 "난 몰라" 체크하고 internal reference 만 처리한다. 0 을 채워넣음. 이 0이 몰라서 체크한 0이라고 relocation table 에 기재한다. => relocation 테이블에는 cross reference 가 나타난 위치를 기재. 

* 비유 : 시험에 모르는 문제가 나오면 체크해두고 쉬운 문제들 다 풀고 돌아오는 것. 

![image-20200505131539265]({{site.baseurl}}/assets/img/image-20200505131539265.png)

Relocation table 에는 크로스 레퍼런스가 나타난 위치를 기록한다. 다음 path 에 이들을 해결한다. 

링커가 각 섹션들에 위치를 부여하면, 섹션 안에서 정의된 심볼들의 실제 주소를 얻어낼 수 있다. 

(위의 PPT) relocation table 에 들어간 엔터리들의 예시. extern 은 다른 모듈에 define 된 심볼이라는 뜻. my_var 과 func 는 cross reference 가 된다. 

(1) **타겟 주소를 몰라서 0으로 넣음. mov, call 인스트럭션 주소를 relocation 테이블에 넣는다.** 

**=> first path 에 하는 일!** 

링커 스크립트에 따라서 동일한 타입의 섹션들 모아서 하나 만들면 각 스몰 섹션들의 위치가 결정된다. **이것에 따라서 그 안의 심볼들의 주소도 정의되고 이것들은 모두 심볼 테이블에 들어간다.** 

(2) **relocation 테이블을 뒤져서 0이 들어간 것들을 뒤져서 심볼 이름을 찾아낸다.** 

**=> second path! 이러면 모든 주소들에 대한 정보가 심볼테이블에 들어가게 된다.** 

![image-20200505131802067]({{site.baseurl}}/assets/img/image-20200505131802067.png)

![image-20200505131912700]({{site.baseurl}}/assets/img/image-20200505131912700.png)

extern 에 대한 cross reference 가 필요한 상황 

![image-20200505131921974]({{site.baseurl}}/assets/img/image-20200505131921974.png)
> Compiler 가 수행하는 일들. 각 섹션들을 채워서 링커에게 준다. 링커는 relocation table 을 통해 0으로 기록된 곳들을 고쳐나간다. 

relocation table 안에는 func, my_var 연산에 대한 주소들이 저장이 된다. local function 에 관한 정보는 PC relative 주소로 알 수 있다. 

![image-20200505135640528]({{site.baseurl}}/assets/img/image-20200505135640528.png)
> Linker 가 수행하는 일들. 섹션들의 주소를 계산해서 이로부터 offset 을 더해 메모리 주소를 얻어낸다. 
링커가 섹션들을 레이아웃 한다 - 산술로 스몰 섹션들의 시작 위치를 정하고 이에 따른 오프셋 계산 가능. 

두번째 path 에서 relocation 테이블을 조사해서 실제 주소로 이를 변환한다. 

![image-20200505135734676]({{site.baseurl}}/assets/img/image-20200505135734676.png)

-> cross reference 를 resolve 하는 예시. 

0으로 표시된 곳을 찾아서 섹션 별 주소를 채워넣는다. 

![image-20200505135836394]({{site.baseurl}}/assets/img/image-20200505135836394.png)

물리적으로 동일한 타입의 섹션들을 모아서 하나의 exe 파일로 만드는 것이 링커의 마지막 스텝

운영체제의 메모리 레이아웃을 공부하기 전 사전준비를 한 것. 

### Conclusion 

code segment 

data segment 

stack segment 

heap segment 

- 위의 세그먼트들이 각각 어디 있는지 알 수 있다. 

코드, 데이터 세그먼트는 각 o 파일에 흩어진것을 링커가 묶어서 exe 로 만들고, exe 안에는 섹션이란 이름으로 흩어져 있는데 loading 할 때 이것들을 넣어주는 것. 

왜 BSS 는 없지? BSS 는 data segment 로 들어간다. int array 100만개 - 메모리 상에서는 공간을 차지해야 하므로 더이상 초기화 여부를 구분할 필요는 없다. 

로딩 단계에서 BSS 를 활용 : BSS 로 잡혀진 영역은 초기화가 되어있지 않으므로 0 fill 을 다 한 다음에 부여한다. 

stack segment 는 ? exe 파일에 없다. 로더가 파일을 로딩하는 시점에 만들어준다. 

코드와 데이터 세그먼트는 프로그램이 로딩될 때 한 번 allocation 이 된다. 프로그램이 terminate 될 때 deallocate 된다. 이것들을 static storage allocation 이라 한다. heap 과 stack 은 실행할때 필요할 만큼 할당. 이것은 dynamic storage allocation 이라 한다. 

---

### Summary 
* 프로그램 작성, 컴파일, 실행하는 단계
    * 여러개의 소스파일들이 각각 컴파일러에 의해 오브젝트 파일로 변환된다. 
        * 오브젝트 파일 안에는 instruction, data, symbol table, relocation table 이 있다. 
        * 컴파일러는 이러한 세그먼트들을 generate 하는 역할을 한다. 
    * 여러 오브젝트 파일들은 링커에 의해 하나의 오브젝트 파일로 합쳐진다. 합쳐진 파일을 executable 파일이라 부른다. 
        * 링커는 오브젝트 파일 안의 instruction 과 data 를 로드하기 위한 메모리 주소를 구체화한다. 
    * 로더에 의해 exe 파일이 메모리에 로드되고 실행된다. 
        * 로더는 링커가 명시한 주소들을 추출하여 instruction 과 data 를 로드한다. 
        * OS 가 로더 함수를 구현한다. (fork -> exec) 

* section (segment) 
    * 오브젝트 파일을 구성하는 단위이다. 메모리 관점에서 segment 라 부른다. 
    * 섹션 안에서의 오프셋을 주소로 갖는다. 
    * Text, data, BSS, symbol table, relocation table ... 등 모두 다른 섹션이다. 
    * 메모리에 로드 될때는 text, data 섹션만 로드 된다. 이를 static memory allocation 이라 함. 

* Compiling 과 Linking 의 문제 
    1. Symbolic 한 이름들을 컴퓨터는 모른다.
        * 해결책 : 심볼 테이블 사용, Symbolic name -> Memory address 로 변환. 
    2. 컴파일러는 로드 되어야 하는 섹션들의 주소를 모른다. 
        * Symbol table 만으로는 해결이 되지 않음. 
        * 해결책 : Relocation Table 사용, 다른 섹션의 심볼을 레퍼런스 하고 있는 instruction 들을 모두 기록해둔다. 이 instruction 들을 링킹 과정에서 relocation table 을 사용하여 실제 매모리 주소로 고친다.

* Relocation Table 
    * 같은 섹션 안의 심볼들은 PC relative addressing 으로 레퍼런스 된다. 
    * 이런 relocation 은 컴파일러에 의해 가능하다. ex) `call local_func` -> `call PC+0x1B` 
    * 다른 섹션 안의 심볼들을 참조하는 것을 Cross Reference 라고 한다. 
    * 이들은 PC relative addressing 으로는 불가능하다. 컴파일러가 unresolved symbol 들을 0으로 표시해두고, 해당 인스트럭션의 주소를 relocation table 에 기록한다. 
    * 컴파일러는 링커에게 relocation table 을 주고 해당 주소의 심볼들을 파악하게 한다. 
