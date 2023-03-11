---
layout: post
title: "[JavaScript] V8 Garbage Collection"
date: 2023-03-11 00:00:00
author: Hyunbin Lee
categories: JavaScript
tags: JavaScript GarbageCollection V8
cover: "/assets/instacode.png"
---

# V8 Garbage Collection

> ### 💡 V8 엔진에서 JavaScript Garbage Collection 동작 방식을 정리합니다. 

## 1. Garbage Collection 

`Garbage Collection`을 단어 그대로 해석하면 '쓰레기 수집'입니다. 

프로그래밍에서 더 이상 사용하지 않는 메모리(Garbage)를 모아서 처리하는(Collection) 소프트웨어의 내부 동작을 일컫습니다. Garbage Collection처럼 메모리를 비워주는 단계를 거치지 않는다면 컴퓨터에서 응용 프로그램이 실행되는 데에 큰 제약이 있을 것입니다. 왜냐하면 하드웨어로 존재하는 메모리는 물리적인 용량이 제한되어 있고 소프트웨어가 별도로 메모리를 관리하지 않는다면 프로그램에서 더 이상 사용하지 않는 데이터가 그대로 메모리에 방치되어 새로운 정보를 할당할 주소가 부족해질 수 있기 때문입니다. 

JavaScript도 사용하지 않는 메모리를 해제하기 위해서 Garbage Collection을 수행합니다. JavaScript가 주로 실행되는 환경인 브라우저나 Node.js에서 `V8 엔진`의 Garbage Collection에 대해서 살펴보겠습니다.

<br />

---

## 2. V8 엔진

V8 엔진은 Chromium 기반 브라우저나 Node.js와 같은 런타임에서 사용되는 오픈 소스 JavaScript 엔진입니다. 대표적인 특징으로는 `JIT 컴파일 방식`을 사용합니다. 일반적인 JavaScript의 인터프리터 방식처럼 코드를 실행할 때 기계어 코드를 생성할 뿐만 아니라 이를 캐싱하여 재사용함으로써 반복 작업을 줄입니다. 

간단하게 V8 엔진 구조를 살펴보면 다음과 같습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224472605-23908a6f-abea-4416-a93a-0a1d591c8c3a.png">
*[이미지 출처] https://deepu.tech/memory-management-in-v8/*
<br />

이미지에서 크게 두 영역으로 구분되어 있는 것을 확인할 수 있습니다. 

- Stack : Primitive Value (원시값)나 Heap에 저장되어 있는 Object (객체)의 주소를 저장합니다. 
- Heap Memory : Object의 실제 값이 저장되는 위치입니다. 

실제로 Garbage Collection에 대해 알아보기 위해서 주의 깊게 살펴볼 부분이 바로 Heap입니다. 이제부터 Heap 영역에서 일어나는 Garbage Collection 과정에 대해서 알아보겠습니다. 

이후부터는 Garbage Collection을 `GC`로 줄여서 부르겠습니다. 

<br />

---

## 3. Minor GC

앞서 언급했던 Heap 영역은 적용되는 GC 방식에 따라 크게 New Space와 Old Space로 구분됩니다. 

먼저 New Space는 새롭게 생성된 객체가 가장 먼저 저장되는 공간입니다. New Space는 또 다시 내부적으로 2개의 Semi Space로 구성되어 있으며 각각 From과 To라는 이름으로 구분합니다. 간단하게 그림으로 살펴보면 다음과 같습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479011-6145e318-fc0c-4ee3-9c8a-5c516201bd0a.jpeg">
*From과 To로 구분되는 New Space*
<br />

Minor GC의 과정을 다음과 같은 순서로 진행됩니다. 

<b> 1. 모든 객체의 출발점이 되는 root 객체로부터 시작하여 참조 관계를 찾아냅니다. </b>

*참조란?
> 가비지 콜렉션 알고리즘의 핵심 개념은 참조입니다. A라는 메모리를 통해 (명시적이든 암시적이든) B라는 메모리에 접근할 수 있다면 "B는 A에 참조된다" 라고 합니다. - MDN 가비지 콜렉션 -

즉, 객체를 하나씩 탐색하면서 다른 객체에서 사용하고 있는지 확인합니다. 여기서 확인된 객체는 Garbage로 분류되지 않고 살아남습니다.

<b> 2. 확인된 참조 관계를 기반으로 Garbage는 삭제되고 그렇지 않은 객체만 To 영역으로 옮깁니다. </b> 

<b> 3. 이후 두 Semi Space는 서로의 역할을 변경합니다. (From ↔️ To) </b> 

<b> 4. 새로 들어온 객체가 From 영역에 저장됩니다. </b> 

---

### * 예시

A 라는 객체가 메모리에 저장되어야 한다면 다음과 같이 New Space ➡️ Semi Space ➡️ From 영역에 저장됩니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479109-87c9c1dd-9f86-4c83-b12d-5cc202042f40.jpeg">
*From에 추가된 객체 A*
<br />

개발자가 추가적인 객체를 메모리에 더 할당했다고 가정해보겠습니다. 

(예시에서는 From Space가 최대 5개의 객체만 저장할 수 있다고 가정합니다.)

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479155-608545c4-b2c9-40fa-b626-cb532c52a3af.jpeg">
*A, B, C, D, E 객체로 From이 가득 찬 상태*
<br />

만약 이 상태에서 추가로 F 객체가 들어오게 된다면 어떻게 될까요? 바로 이 시점에 위에서 언급한 순서대로 Micro GC가 동작하게 됩니다. 만일 B, C 객체가 root에서부터 참조 관계를 확인했을 때 Garbage로 분류되었다면, 해당 객체들을 제외한 나머지 객체들이 To 영역으로 이동합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479201-0b49bcd6-933d-43c6-86dc-c98880b44057.jpeg">
*더 이상 참조되지 않는 객체는 From에 남고, 나머지는 To에 저장*
<br />

이후 두 영역이 Swap되고 (From ↔️ To) 새로운 F 객체는 From 영역에 추가됩니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479249-056a6118-9401-4a60-9fdb-56601c986160.jpeg">
*B, C객체는 삭제되고 From영역에 F 객체를 추가*
<br />

조금 더 진행해보기 위해서 G, I 객체를 추가해보겠습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479284-f17b3597-004c-4945-86b1-09347702eb6f.jpeg">
*G 객체를 From에 추가한 뒤, I 객체를 저장하기 위해서 Minor GC가 한 번 더 필요해졌습니다.*
<br />

또 다시 From 영역이 가득찼기 때문에 동일한 방식으로 Micro GC가 동작합니다. 

만약 여기서 직전에 살아남았던 A, D, E 객체가 한 번 더 살아남게 되면 어떻게 될까요?

이렇게 Minor GC에서 2번 살아남은 객체는 반대쪽 Semi Space가 아니라 맨 처음에 설명했던 Old Space로 이동하게 됩니다.

<br />
<img src="https://user-images.githubusercontent.com/31645195/224479327-9b1b7ee9-d435-4d59-8637-6ebd038cb96a.jpeg">
*I, F, G 객체는 반대쪽 Semi Space로 이동하게 되고 A, D, E 객체는 더 이상 New Space에 저장하지 않습니다.*
<br />

---

<br />

이렇게 New Space에 적용되는 Minor GC를 다른 이름으로 `Scavenger`라고 부릅니다. 

특징을 정리해보면,

- 처음 생성된 객체가 저장됩니다.
- 정기적으로 짧고 빠르게 수행됩니다. (대체로 1ms보다 적은 시간이 소요됩니다.)
- 객체의 수에 의해 Minor GC의 총 처리 시간이 결정됩니다. 
- New Space 내부의 모든 객체가 살아있을 경우 실행시간이 상당히 증가할 수 있습니다. 

다음으로 Old Space의 Major GC에 대해서 알아보겠습니다.

<br />

---

## 4. Major GC

사람이 단기 기억에 있는 내용을 반복해서 사용하면 장기 기억으로 넘어가는 것처럼 객체도 New Space에서 Old Space로 넘어옵니다. 이러한 Old Space 역시 Old Pointer Space와 Old Data Space라는 두 가지 영역으로 나뉩니다. 각각 다른 객체를 참조하는 객체를 저장하는 영역과 원시 타입만 속성으로 가지는 객체가 저장되는 영역입니다. 

New Space처럼 Old Space도 내부에 살아있는 객체들의 총량이 한계에 다다랐을 때 Major GC가 동작하는데 `Mark and Sweep and Compact` 알고리즘을 기반으로 작동합니다. 

#### 1. Mark

root 객체부터 출발하여 참조 관계를 확인합니다. 이러한 Mark 단계에서는 `Tri-color` 알고리즘을 사용합니다.  

<b> *Tri-color: 흰색, 회색, 검정색 세 가지 색상을 사용하여 탐색하는 객체의 상태를 구분합니다. </b>

root부터 객체의 참조 관계를 살필 때, `DFS(Depth First Search, 깊이 우선 탐색)` 방식으로 탐색하며 객체의 상태를 각각의 색상으로 표시합니다. 

- 흰색: 탐색 전
- 회색: 탐색되었으나 아직 참조 관계가 확인되지 않음 
- 검정색: 탐색과 참조 관계 확인이 모두 완료됨

#### 2. Sweep 

앞서 표시한 색깔을 기반으로 참조 관계에 있는 객체들만 메모리에 유지합니다. 다시 말해, Mark 단계에서 검정색으로 표시된 객체만 저장하고 흰색으로 표시된 객체는 메모리에서 삭제합니다. 

#### 3. Compact 

메모리가 낭비되지 않게 하기 위해서 메모리 주소를 모아주는 역할을 합니다. OS를 학습하다 보면 메모리와 관련된 문제점 중 하나로 메모리 단편화 현상을 배울 수 있습니다.

<b> 메모리 단편화: 사용 중인 메모리가 해제된 이후, 임의의 프로세스가 비어있는 메모리 공간보다 필요로 하는 양이 많아 메모리를 순서대로, 효율적으로 할당할 수 없어 발생하는 메모리 낭비 문제를 말합니다. </b> 

GC에서도 메모리를 해제하는 작업이 빈번하게 발생하기 때문에 별도로 Compact 단계를 두어 객체들이 사용하고 있는 메모리를 연속된 주소에 배치하게 됩니다. 

---

### * 예시

Minor GC에서 들었던 예시에서 살아남은 A, D, E 객체 뿐만 아니라 F, G 객체까지 Old Pointer Space에 저장되어 있는 상태를 가정하고 이어서 설명해보겠습니다. 다음과 같이 I 객체가 Old Pointer Space에 저장되어야 하는 상황입니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224487504-30874d2c-8ae4-4635-8067-d225aca9c85d.jpeg">
*A, D, E, F, G 객체가 저장되어 있는 상황에서 I 객체도 Old Space로 넘어오게 되었습니다.*
<br />

Old Pointer Space에 추가적인 공간이 필요하기 때문에 Major GC가 시작됩니다. 아래와 같은 참조 관계가 있다고 가정하고 root부터 DFS 방식으로 Tri-color 알고리즘을 수행합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224487542-abd64452-346d-4747-a98c-d2150d5d2310.jpeg">
*초기 객체들은 흰색으로 마킹되어있습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/224487573-73d9cbd9-070d-4b2c-b59c-fb933becde83.jpeg">
*가장 좌측부터 탐색합니다. 참조 관계가 파악되면 검정색, 그렇지 않으면 회색으로 표시합니다.*
<br />

root가 D 객체를 참조하지 않기 때문에 D, F 객체는 흰색으로 남게 됩니다. 이를 Sweep하여 메모리를 확보합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224487608-74bc8a78-e5f1-4bac-8c80-ae48492680ce.jpeg">
*탐색 결과 D, F 객체는 메모리에서 삭제되어야 합니다.*
<br />

만약 다음과 같이 D, F 객체가 각각 11~20, 31~40까지의 메모리 주소를 차지하고 있었을 경우 해당 주소 공간에 공백이 생깁니다. 따라서 Compact 단계에서 이를 연속된 주소 공간에 배치하여 단편화 현상을 방지합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224487624-6a542d87-2623-46d7-90f4-5694063e66f0.jpeg">
*가령, 11~20까지의 주소에 E 객체를 저장하고 31~40까지의 주소에 G 객체를 저장합니다.*
<br />

최종적인 Old Space 모습은 다음과 같이 변경됩니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/224487648-721709e2-9acb-4faf-b29f-01cf8d1fac92.jpeg">
*살아남은 A, E, G 객체와 새로운 I 객체가 저장됩니다.*
<br />

---

<br />

Old Space에서의 Major GC 특징을 정리해보면 다음과 같습니다.

- New Space보다 저장할 수 있는 객체의 양이 더 많습니다. 
- root 객체부터 전체를 순회하기 때문에 느립니다. 
- 따라서 Major GC는 최대한 적게 실행하는 것이 좋습니다. 

<br />

---

## 5. GC와 Idle 상태

여기까지 GC 동작 방식에 대해서 살펴보았습니다. 다행스럽게도 초반에 언급한 V8이 동적으로 메모리를 관리해주기 때문에 GC와 관련하여 개발자가 별도의 처리를 하지 않아도 됩니다. GC는 꽤 복잡한 일을 내부적으로 여러 단계에 걸쳐 처리해야 하기 때문에 투자되는 자원도 무시할 수 없을 것 같습니다. 그렇다면 응용 프로그램이 실행되고 있는 상황에서 이러한 GC는 언제 수행되는 것일까요? 이를 위해 브라우저의 Idle 상태에 대해서 살펴볼 필요가 있습니다. 

일반적으로 브라우저는 60FPS의 사용자 경험을 제공하기 위해 16.6ms동안 브라우저에서 필요한 작업을 수행하고 업데이트를 진행합니다. 이 때, 모든 작업이 16.6ms 안에 끝나게 되어 다음 프레임까지 할 일이 없는 상태를 `Idle 상태`라고 합니다. Idle 상태에서는 다음 프레임까지 처리해야 하는 작업 목록에 Idle Task를 스케줄링하게 되는데 GC가 이러한 Idle Task에 해당됩니다. Idle Task는 Real-Time 프로세스처럼 Deadline (`16.6 - 이전 작업 시간`ms)이 명확하게 존재한다는 점이 특징적인데, 정확한 종료 시간을 맞출 수 있는 가장 적절한 GC 방식을 선택하여 실행하게 됩니다. 

앞에서 살펴본 내용을 바탕으로 GC를 4가지로 나눌 수 있습니다. 각각의 GC는 수행될 때 다음 번을 위해 예측 시간을 계산합니다. 


#### 1. Scavenges
현재 New Space의 상태를 파악하여 Young Generation이 가득찰 것인지를 예측하고 GC를 수행합니다. 

#### 2. Concurrent Mark

또한 Heap 용량의 한계에 다다르면 위에서 언급한 Mark를 진행합니다. root부터 모든 객체의 참조 관계를 한 번에 파악하게 될 경우 비지니스 로직이 실행되는 메인 스레드가 멈출 수 있기 때문에 Mark 작업을 여러 번에 걸쳐 점진적으로 실행합니다. 이 경우 각각의 Mark 작업이 5ms 이하로 처리될 수 있습니다. 

[Getting garbage collection for free](https://v8.dev/blog/free-garbage-collection)

#### 3. Full-GC

전체 Idle 시간이 넉넉하고 Old Generation이 가득일 때 Full-GC (Minor GC + Major GC)를 수행합니다. 이 경우에는 살아있는 객체 수 * 개별 Marking 속도로 시간을 예측합니다. 

#### 4. Full-GC + Compaction 

마지막으로 Full GC에 Compaction을 더한 방식은 브라우저가 상당 시간동안 Idle 상태일 때에만 수행합니다. 

<br />

---

## 6. 맺으며 

GC는 앱의 규모가 커질수록 자원이 많이 필요해지기 때문에 이를 위해 V8 엔진 내부적으로 많은 최적화 기법이 적용되고 고도화되어 왔습니다. Space를 New와 Old로 나누어서 관리하는 것도 Generational Collection (세대별 수집) 방식 중 하나이고, 멀티 스레드를 이용하는 Parallel Mark Evacuate, Concurrent Mark, 병렬 Scavenge 등이 여기에 해당합니다. 

과거에는 싱글 코어 하드웨어라는 점을 고려했을 때 알맞은 알고리즘을 사용했다면, 하드웨어 성능이 좋아짐에 따라 GC 방식을 현대화하고 있는 것입니다. 여기에 기인하여 끝으로 두 가지 생각이 듭니다. 예측 불가능한 시점에 돌아갈 수 있는 GC를 염두에 두며 부드러운 앱을 제공해야 한다는 점과 외부 상황에 맞는 적절한 성능의 소프트웨어를 만들어 내야 한다는 점 말이죠.

---

[참고]

- [자바스크립트의 메모리 관리 - JavaScript / MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Memory_Management)

- [JIT 컴파일](https://ko.wikipedia.org/wiki/JIT_컴파일)

- [🚀 Visualizing memory management in V8 Engine (JavaScript, NodeJS, Deno, WebAssembly)](https://deepu.tech/memory-management-in-v8/)

- [가비지 컬렉션](https://ko.javascript.info/garbage-collection)

- [자바스크립트 v8 엔진의 가비지 컬렉션 동작 방식 / 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2022/220519-garbage-collection/)

- [Getting garbage collection for free · V8](https://v8.dev/blog/free-garbage-collection)

- [Optimizing V8 memory consumption · V8](https://v8.dev/blog/optimizing-v8-memory)

- [Concurrent marking in V8 · V8](https://v8.dev/blog/concurrent-marking)
