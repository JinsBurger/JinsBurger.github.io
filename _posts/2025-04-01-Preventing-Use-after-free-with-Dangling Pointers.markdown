---
layout: post
title:  "[Paper Review, Kor] Preventing Use-after-free with Dangling Pointers Nullification"
date:   2025-04-01 09:00:00 +0900
categories:
---

# [Paper Review] Preventing Use-after-free with Dangling Pointers Nullification

# Paper Information
Conference: NDSS
Year: 2015
Category: Pointer nullification
Authors: Lee et al.
URL: [https://www.ndss-symposium.org/ndss2015/ndss-2015-programme/preventing-use-after-free-dangling-pointers-nullification/](https://www.ndss-symposium.org/ndss2015/ndss-2015-programme/preventing-use-after-free-dangling-pointers-nullification/)
Presentation PDF: [Preventing-Use-after-free-with-Dangling Pointers.pdf](/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/Preventing-Use-after-free-with-Dangling Pointers.pdf)


# Introduction  

UAF는  해결되지 않았고,   Chromium browser for two years (2011–2013) 의 CVE를 보면  다른 취약점들과 비교했을 때, UAF 버그 케이스가 많았으며, 그 중 88%는 high 혹은 critical 버그다.

free와 use가 분리되어있고 복잡하기 때문에 UAF를 Static analysis로 잡는 것은 Challenging하다.

기존 대부분의 연구들은 Dynamic 하게 포인터들을 추적했으며, dangling pointers를 잡고 dangling pointer를 통한 메모리 접근을 막았다. 
하지만, 모든 포인터에 대해 정밀하게 추적하는 것은 False positive 가능성과 심각한 성능 저하를 야기한다.  그러므로 해당 방법론은 규모가 큰 소프트웨어에서는 적용할 수 없었다.

기존 Memory error detectors 의 연구들은 software testing 과정에서 메모리의 allocated/freed 상태를 유지(저장)함으로써 freed memory에 접근하는 것(즉, UAF)을 막을 수 있었다. 하지만, 이것들은 Realworld 상에서 공격자가 Heap 할당 프로세스를 부분적으로 조작할 수 있는 Exploit을 막지는 못한다. 또한, Exploit의 목표는 대부분 Control Flow 를 바꾸기 위함인데, 이를 위해 CFI가 제안이 되었어도, 대부분은 Coarse-grained(프로그램은 크게 쪼개는 것) CFI이기 때문에 모두 Exploit을 막지 못한다.

종합적으로 이전 보호 기술들은 높은 FP를 보여줬고, 우회가 가능하며 아직까진 규모가 큰 프로그램에서 잘 작동되는 UAF 보호 기술들이 없다는 것이다.

DANGNULL은  임시 메모리 접근을 막는다. Free 된 영역을 가르키는 포인터가 NULL로 초기화 되지 않아 UAF가 발생하는 것에 초점을 맞췄으며, DANGNULL은 자동으로 object들의 관계를 Trace하고 free가 된 곳을 가르키는 pointer를 NULL로 초기화 시켜 dangling 포인터를 막는다. DANGNULL은 runtime에서 object range analysis 통해 트래킹 하고 free 되자마자 추적한 것을 기반으로 nullify 를 진행한다.

DANGNULL의 장점

\- Free 되자마자 Nullfication을 즉시 진행하기 때문에, Dangling pointer 존재하기 힘듦.
\- runtime object range analysis 통해, 효율적으로 pointer 추적가능.
    \- Full pointer semantics 대신에, Abstracted pointer semantics을 쓰므로 큰 프로그램에서도 돌릴 수 있음.
        \- Full은 포인터에 대한 모든 연산 등을 다 추적, Abstract 는 훨씬 적은 정보만 저장.
\- Memory access에 대한 직접적인 체크가 없으므로, bottleneck 덜하다.  
 
실제 크로미움 브라우저의 UAF 7개 익스플로잇 막음. 자바스크림트에서는 4.8, 렌더링에서는 53.1 오버헤드 증가. 

30,000개 테스트에서 False Positive 를 보이지 않았음.

---

# Background

\- From dangling pointers to use-after-free.
    \- dangling pointer로만은 취약점 유발 X
    \- 그 포인터에 접근해야 생김
\- Challenges in identifying dangling pointers.
    \- 할당,해제, 확장 등은 분산되어있어 잡는게 복잡함
    \- 서로 다른 스레드에서 작동할 수 있음.
\- Exploiting dangling pointers.
    \- Free된 영역에 가치있는 데이터를 배치시키고, 이를 조작해야함.
        \- 즉, Free와 Use 사이에서 데이터 배치가 이루어져야함.
    \- 얼마나 가치있는 데이터이냐에 따라 exploitability 가 결정됨.

---

# Design

False Positive 없이 Detect하는게 목표. 그래서 근본적인 원인인 Dangling pointer에 집중했음.

근본적인 원인: 처음으로 assigned된 pointer가 다른 곳으로 복사되거나 포인터 연산을 해서 대입 됨. 해당 target 포인터가 free 됐음에도 이동한 곳은 free 안됨

Design

**1:** 자동으로 pointer와 memory objects 과의 관계 추적
**2:** free 된 memory objects를 가르키는 pointer를 nullify.

<img  width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_3.07.58.png"/>

컴파일 단에서 DANGNULL를 같이 실행해줘야함.

\- Identify pointer assignments operation
    \- tracing routine 호출하는 코드 넣음.
    \- Runtime 단에서 instruction trace에 도움 줌.
        \- Tracing 하는 포인터에 대한 Shadow object 관리
    \- Thread-safe RBTree로 Trace 관리함.
\- Free 되는 오트벡트를 가르키는 ShadowObject에 저장된 포인터들을 다 nullify

### Static Instrumentation

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_3.19.52.png"/>

LLVM IR [24] level and is designed to achieve one primary goal: to monitor pointer assignments to maintain the point-to relations.

또한, 퍼포먼스 오버헤드를 줄이기 위해, Stack에 있는 dangling pointer는 Stack 특성상 short lifetime을 갖기 때문에 heap 에만 담기는 포인터들을 추적하기로 함.

이 조건들이 만족하는 경우, assign instruction 다음에 `trace` 함수를 주입한다.(Runtime에서 자세히 다룸)

만약, Pointer 타입을 Non-Pointer 타입으로 Cast 하는 경우, 제대로 추적이 안된다. 또한, 컴파일 단에서 DANGNULL를 실행시키지 않으면 추적이 안된다.

### Runtime Library

`trace`  를 통해 얻은 Object Relationship을 이용해서 Runtime Library는 자동으로 모든 dangling pointer의 nullify를 진행.

**Shadow Object Tree**

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_4.11.19.png"/>

`shadowObjTree`  는 hierarchical 구조로 relationship 구성.

RBTree를 고른 이유:

- Insertion, remove 보다 검색 Operation 많아도 괜찮다.
- 힙 포인터에 대한, Object를 찾을 때, `Object's base <= given p < Object's end` 를 검색해야 하므로,  검색 횟수가 많음.

RBTree 안에 SubTree:

- IN: 다른 node에 의해  자신이 가리켜지는 것
- OUT: 자신이 다른 노드를 가르키는 것.

**Runtime Operations and Nullification**

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_5.00.56.png"/>

Runtime Library는 할당/해제를 각각 `allocObj`와 `freeObj`로 교체. Static Analysis 과정에서 추가된  `trace` 도 있음.

<img width="100%"  src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_5.02.37.png"/>

`allocObj` 는 할당 후에, shadowObject Tree에 넣음. 

`trace` 는 ( `a := ptr` 에서, `a` = lhs, `ptr` = rhs)의 lhs, rhs가 실제로 heap-allocated 된 영역이 맞는 지 확인한 후, 기존 Shadow(전의 Relationshop)를 지우고 새롭게 만든 후, 이어 준다.

`free` 는 포인터의 shadowObject 를 가져온 후, 이를 가르키고 있는 Inbound 포인터들을 nullify 시켜주고 Outbound 포인터들로 하여금 해당 포인터들의 Inbound를 다 삭제 한다. 즉, shadowObject 간에 연결 고리를 끊어준다.

이렇게 함으로써 deference도 안전하게 보장헤줄 수 있음. (부적절한 deference는 SEGFAULT를 일으키기 떄문.)

---

# Implementation

정적 분석은 LLVM으로 했음.

ELF의 preinit_array에 initialization function을 둠으로써 코드 맨처음에 실행되도록.

여러 Standard functions (Malloc, free, new, delete 등)를 위해 구현 했음.

Thread issue 방지 위해, shadowObject을 위한 Mutex lock.

기존 메모리 레이아웃은 안 건들기 떄문에 이에 관환 side-effect 존재 X

---

# Evaluation

evaluation environment

- CPU : quadcore 3.40 GHz CPU (Intel Xeon E3-1245)
- memory : 16GB RAM
- OS : Ubuntu 13.10 system (Linux Kernel 3.11.0)

**A. Attack Mitigation**

Chromium browser 에서 테스트

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_9.31.33.png"/>

No-Nullify 가 적용 됐을 때, SEGFAULT가 발생했지만, 6개가 유의미한 주소였기에 공격 가능성이 있었다.

(*stopped by assertion* 은 CHROME 자체의 assertion으로 중지 된 것.)

mark (✓) 는 DANGNULL 했을 때,  CHROME 에서 safe-deference 하기 전에  포인터 NULL 체크에 걸린 것이다. 즉, Nullify 함으로써 마치 패치 된 것처럼 행동하는 것이다.

즉,  DANGNULL은 UAF 막는게 아니라, 기존 Null pointer 체크 구문을 utilization 에도 기여를 하며 patch 된 것 처럼 행동하게 한 것 이다 (expected behavior).

**B. Instrumentation**

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_9.40.24.png"/>

SPEC CPU 2006 Benchmark를 보면, 프로그램에 삽입된 instructions들은 전체 프로그램의  instructions 과 어떻게 구현되었는 지에 대해서도 영향을 받는다. mcf와  lbm을 비교하면 mcf가 instructions 개수는 작지만, 삽인된 instructions 들은  10배 넘는다.

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_9.46.56.png"/>

전체 프로그램의 1% 정도  instructions가 삽입 됐으며 파일 크기는 0.5% 늘어났다.

**C. Runtime Overheads**

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_9.48.55.png"/>

평균 적으로는 80% 늘어났으며, `povray` 같은 경우는 포인터의 개수가 많아 실행시간 270% 증가 및 메모리 오버헤드도 213% 늘어났다. 포인터 개수가 작은 `h264ref` 은 1% 실행시간 증가 및 472% 메모리 오버헤드 증가했다.

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_9.52.13.png"/>

7개의 브라우저 테스트 했을 때, Javascript 엔진에서는 평균 4.8% averaged overhead 증가했다. 대부분 JIT으로 컴파일 되기 때문에, 해당 부분은 DANGNULL의 범위를 벗어나서 별로 안늘어났다.  렌더링은 53.1% 정도 렌더링 computations가 증가했다.

Alexa top 100 websites 페이지의 오버헤드도 측정해봤을 때, 평균 7% 로딩 시간이 걸렸다.  2326ms 의 317.6ms 표준 편차로 걸렸으며 일반 크롬은 2165 ms 의 377.9m 표준 편차가 걸린다.

<img width="100%" src="/assets/2025-04-01-Preventing-Use-after-free-with-Dangling Pointers/pic_10.06.53.png"/>

위 인기 사이트를 봤을 때, 특히 유튜브의 경우 많은 이미지가 렌더링이 되기 때문에 32.8% 의 로딩 시간이 걸렸다.

**C. Compatibility**

Program의 semantics를 바꿈으로써 생길 수 있는 False Positive를 측정하기 위해, Chrome으로 stress tests를 했다. 여러 사이트들, unit tests를 했지만 문제 없었다.

(정말 정말 rare한 케이스가 있지만, Critical한 문제가 아니고 30000개 테스트한 결과 문제가 없음을 확인)

---

# Limitation

1. Performance 부분에 있어서 개선이 필요하다.
2. 포인터가 실제 사용되지 않는 benign dangling pointer에 대해서도 NULL화를 수행하여 이론적으로는 False Positive 가능성이 존재함.  
    \- 포인터 타입만 추적해서, 포인터 타입이 아닌 변수를 통한 propagation은 못잡음.

---

# Conclusion

임시 메모리 사용 위반(UAF, DFB 등)을 런타임에서 잡는다. Chromium에서도 효과적임을 보였다.

UAF도 막을 수 있음을 보였다.