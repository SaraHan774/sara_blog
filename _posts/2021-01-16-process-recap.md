---
layout: post
title: "운영체제 면접 질문 대비"
author: "Sara Han"
categories: OperatingSystem
comments: true
---

```
서울대학교 홍성수 교수님 운영체제 수업 복습
```

### 프로세스

- 프로세스는 CPU 스케줄링, CPU 실행의 단위. 자원 할당의 주체, 수행의 주체이다.
- 운영체제라는 거대한 소프트웨어의 복잡성을 해소하기 위해서 Decomposition 을 통해 작은 단위로 쪼갤 수 있다. 수행의 단위까지 소프트웨어를 쪼갰을 때 그것이 바로 프로세스가 된다.
- 프로세스를 한 마디로 정의하면 Program in execution 이다.
- An execution stream in the context of a particular process state.

#### Program vs Process

- Program 은 저장 매체에 저장된 수동적인 코드 시퀀스이다.
- 이 프로그램을 PC에 인스톨 해서 수행을 시작하면 Process 가 된다. 프로세스는 execution stream, thread of control. 능동적인 존재.
  - CD ROM 에서 executable file 을 읽어온 후 Main memory 에 로드를 하고, CPU를 할당하면 수행을 하게 됨.
- 즉, 프로그램은 저장 매체만 필요하지만, 프로세스는 CPU, IO디바이스, 메인 메모리가 필요한 존재이다.

#### Process state or context

- 프로그램이 수행되는 데 영향을 줄 수 있는 정보들, 수행의 결과로 영향을 받을 수 있는 정보들을 process state 이라고 한다.

- 세 가지 종류의 **context** 들에 대한 collection
  - **Memory context**
    - code segment, data segment, stack segment, heap
  - **Hardware context**
    - CPU registers, I/O registers
  - **System context**
    - Process table, open file table, page table

#### Process 를 위해 필요한 것들

1. Memory - code, data, stack 이 저장되어야 한다.
   저장되는 것은 컴파일러나 로더와 관련된 사항이다. - data 에 저장되는 것들 : 프로그램의 전역 변수들이 저장됨 - stack 에 저장되는 것들 : function call, local 변수들이 저장됨
2. CPU 가 필요하다.
   - register value
3. 운영체제가 여러 프로세스를 관리하므로, 각 프로세스별로 운영체제가 저장하는 Per-Process Kernel 정보들.
   - ex. 프로세스가 열은 파일 정보들

> 프로세스가 수행되려면 이 세가지 정보들을 유지하고 있어야 한다. 이 정보들이 바로 프로세스의 state 가 된다.

#### Execution stream

- 프로세스가 지금까지 수행한 모든 명령어들의 순서

#### Why is the process useful ?

- Design-time entity vs. run-time entity
  - system design is an activity of
    - accepting the system requirements and
    - generating a collection of tasks
  - Task is a design time entity
  - Process is a run-time entity
    - **Target of CPU scheduling and resource allocation**
  - Implementations
    - A mapping from design-time entities onto run-time entities
