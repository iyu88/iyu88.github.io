---
layout: post
title: "[JavaScript] 꼬리 물기 최적화"
date: 2023-05-21 00:00:00
author: Hyunbin Lee
categories: JavaScript
tags: JavaScript TailCallOptimization TailRecursion
cover: "/assets/instacode.png"
---

# 꼬리 물기 최적화

> ### 💡 ES6에서 추가되었던 꼬리 물기 최적화에 대해서 정리해봅니다.

## 1. 들어가며

이전에 작성했던 [ECMAScript 2023 살펴보기](https://iyu88.github.io/javascript/2023/04/23/ecma-script-2023.html) 글을 작성하고 나서 이전 ECMAScript 버전의 변경 사항도 살펴볼 겸 ES6부터 추가된 스펙에 대해서 훑어보았습니다. 대부분은 그동안 JavaScript를 사용하며 자주 접했던 내용들이라 큰 문제없이 이해할 수 있었습니다. 그 중에서 얼마 전 JavaScript 스터디를 진행할 때에도 잠깐 등장했던 개념에 눈길이 갔습니다. 바로 `꼬리 물기 최적화`입니다. 

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/f994e97b-a8b1-462f-bfc9-93f698faaf6b" alt="ECMAScript 2015 문서에서의 proper tail calls">
*ECMAScript 2015 (ES6) 에서 추가된 proper tail calls*
<br />

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/9eef94c7-52a7-4320-8e13-4be69e3a40d7" alt="모던 JavaScript 튜토리얼에 등장하는 tail call optimization">
*모던 JavaScript 튜토리얼에 등장하는 tail call optimization*
<br />

<br />

## 2. 꼬리 물기 최적화

JavaScript에서 함수를 호출하면 엔진 내부에 있는 Call Stack이라는 영역에 하나씩 함수가 쌓이게 되고, 이를 `Stack Frame`이라고 부릅니다. `꼬리 물기 최적화`(Tail Call Optimization, 이하 TCO)는 함수를 함수의 꼬리(return 등)에서 호출할 때 Stack Frame을 재활용할 수 있게 하는 최적화 방식을 말합니다. 이러한 동작이 큰 임팩트를 줄 수 있는 상황인 재귀 함수를 실행하는 경우를 예시로 들어보겠습니다. (TCO가 반드시 재귀적인 상황에서만 적용될 수 있는 것은 아닙니다)

<pre>
<code class="hljs js">const pow = (x, n) => {
  if (n === 1) {
    return x;
  } else {
    return x * pow(x, n-1);
  }
}

pow(2, 4);
</code>
</pre>

위와 같이 x를 n제곱하는 재귀 함수 `pow()`를 정의하고 x와 n을 각각 2와 4로 설정하여 호출하면 다음과 같은 흐름으로 진행됩니다. 

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/260f8e6a-7b54-47cb-9938-1375a6005cc0" alt="pow() 함수를 실행했을 때 Call Stack에 쌓이는 재귀 함수">
*재귀 함수이기 때문에 호출한 함수가 Call Stack에 쌓이게 됩니다.*
<br />

pow()의 내부가 재귀 함수로 동작하기 때문에 Call Stack 영역에 그림처럼 n이 1이 될 때까지 함수가 쌓이게 됩니다. 이후에는 쌓여 있는 함수가 값을 반환하기 시작하면서 점차 Call Stack을 비우게 됩니다.

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/46cbe899-ed5e-4076-a5fa-61534b0a210c" alt="n이 1일 때부터 값을 반환하면서 Call Stack이 점점 비워짐">
*Call Stack이 점점 비워집니다.*
<br />

Call Stack에 이전에 호출한 함수가 쌓여있는 이유는 n이 1이 아닐 때 `x * pow(x, n-1)` 를 반환하기 위해서 함수 내부에 <strong>메모리가 유지된 채로 pow(x, n-1)의 실행 결과를 기다리고 있어야 하기 때문입니다.</strong> 지금은 n이 4인 경우를 예시로 들었기 때문에 Call Stack에 최대 4개의 재귀 함수가 등록됩니다.

하지만 만약 n이 매우 커지면 어떻게 될까요? 분명 지금의 함수 구조에서는 Call Stack에 재귀 함수가 n개 쌓이게 될 것이고, 앞의 예시와 동일하게 하나씩 값을 반환하면서 Call Stack을 비우게 될 것입니다. Call Stack에 함수가 쌓인다는 것의 의미를 파악하기 위해 JavaScript의 실행 컨텍스트에 대해서 간단하게 다루어 보겠습니다.

<br />

## 3. 실행 컨텍스트 

실행 컨텍스트는 `실행할 코드에 제공할 환경 정보들을 모아 놓은 객체`입니다. 함수를 호출할 때마다 새로운 실행 컨텍스트가 생성되며 함수 실행 절차를 내부적으로 관리합니다. 즉, 함수 실행에 필요한 변수, 함수 식별자와 그것에 해당하는 실제 값, 외부 실행 환경에 대한 참조도 저장합니다. 이를 위해서 실행 컨텍스트가 생성될 때 내부 메모리에 다음 세 가지가 생깁니다.

1. Variable Environment : 식별자 정보나 외부 환경 정보입니다. (이후 설명할 Lexical Environment의 스냅샷으로 고정된 값입니다)
2. Lexical Environment : 실제로 변경 사항이 반영되는 영역입니다. `Environment Record`와 `Outer Environment Reference`로 이루어져 있습니다. 
3. this binding : 식별자가 바라보아야 할 객체입니다. 

<br />

가령, pow(2, 4)를 호출했을 때 생기는 실행 컨텍스트는 다음과 같습니다. 

<br />

1. Variable Environment : 초기 Lexical Environment의 스냅샷을 저장합니다. 
2. Lexical Environment
  - Environment Record : 매개변수로 전달받은 값이나 지역 변수에 대한 데이터를 저장합니다. 이 경우에는 x와 n에 대한 메모리를 확보합니다. 
  - Outer Environment Reference : 전역 환경에서 함수를 호출한 것이기 때문에 전역 실행 컨텍스트를 참조합니다.
3. this binding : 함수가 특정한 객체에 대해서 실행되고 있는 것이 아니기 때문에 전역 this를 참조합니다. 

여기서 주목해야 할 부분은 `Environment Record`입니다. 함수를 실행할 때마다 필요한 지역 변수를 메모리에 할당하게 되는데, 앞선 pow() 함수 예시처럼 Call Stack에 함수가 계속해서 쌓이면 메모리가 해제되지 않은 채로 남아있게 됩니다. 즉, Call Stack에 함수가 올라올 때마다 실행 컨텍스트가 생성되고 이는 곧 메모리 할당이 일어남을 의미합니다. 이러한 실행 컨텍스트가 Call Stack에 적재되고 적절한 시점에 해제되지 않을 경우 메모리를 효율적으로 사용할 수 없게 되며 최악의 경우에는 `Maximum call stack size exceeded` 에러를 마주할 수 있습니다. 

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/62fe40b7-379c-4355-8aa4-c586763e8fa3" alt="Call Stack에 쌓인 함수마다 실행 컨텍스트가 별도로 존재">
*함수마다 실행 컨텍스트가 생성됩니다.*
<br />

이러한 문제가 발생하지 않도록 하기 위해 언어 차원에서 도입된 TCO로 해결할 수 있습니다.

<br />

## 4. 다시 꼬리 물기 최적화

결국 기존 pow() 함수의 return 문에서 `pow(x, n-1)`의 실행 결과를 기다리지 않기 위해 함수를 다음과 같이 변환할 수 있습니다. 

<pre>
<code class="hljs js">const pow = (x, n, acc) => { // acc 추가
  if (n === 1) {
    return x * acc;
  } else {
    return pow(x, n-1, x * acc); // 변경
  }
}

pow(2, 4, 1);
</code>
</pre>

n이 1이 아닌 경우 `x * pow(x, n-1)` 부분에서 곱하기 연산을 제거해주기 위해서 `acc`라는 누계 값을 저장하는 인자를 추가했습니다. 이로 인해서 더 이상 pow()의 결과를 반환할 때 이전 실행 컨텍스트를 유지하지 않고 다음 함수를 호출합니다.

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/9c545c8a-54b9-483a-89fd-74be010994ed" alt="Call Stack에서 Stack Frame을 하나씩 유지">
*더 이상 이전 함수들이 Call Stack에서 기다리지 않습니다.*
<br />

여기에 추가적으로 TCO가 적용되면 기존 함수가 사용하고 있던 Stack Frame을 재활용합니다.

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/9b888446-1a9f-4f37-8876-eeacfa3bff18" alt="첫번째 Call Stack은 Stack Frame을 재활용한다는 설명, 네번째 Call Stack은 반환 주소를 변경한다는 설명">
*하나의 Stack Frame을 재활용합니다.*
<br />

가령, pow(2, 4, 1)을 호출했을 때 `x = 2, n = 4, acc = 1` 이라는 Stack Frame이 생성되었다면 다음에 호출되는 pow(2, 3, 2)를 호출했을 때는 이전에 있던 Stack Frame을 `x = 2, n = 3, acc = 2` 로 재활용할 수 있다는 것입니다. 또한 개선 후 코드처럼 메모리를 유지하지 않아도 되게끔 함수를 작성하면 언어가 알아서 return point를 변경해줍니다. 

<br />

물론 여기에 조건이 몇 가지 필요합니다. 

먼저 꼬리에서 함수를 호출해야 합니다. 함수의 꼬리 종류는 다음과 같습니다. 

1. return 문
2. 객체의 메서드 
3. call 이나 apply 로 바인딩된 함수

즉, pow() 함수가 `return pow(x, n-1, x * acc)`로 함수를 호출하는 것처럼 작성하면 됩니다. 


다음으로는 아래처럼 지연 평가로 인해 Stack Memory를 붙잡지 않는 연산자를 사용하면 됩니다. 

- 삼항 연산자 (`? :`)
- 논리 연산자 OR (`||`)
- 논리 연산자 AND (`&&`)
- 쉼표 연산자 (`,`)

pow() 함수에도 이를 반영하면 다음과 같이 간단하게 변경할 수 있습니다. 

<pre>
<code class="hljs js">// 삼항 연산자 적용 
const pow = (x, n, acc) => (
  n === 1 ? x * acc : pow(x, n-1, x * acc) 
)

pow(2, 4, 1);
</code>
</pre>

마지막 조건은 반드시 `strict mode`에서 실행되어야 한다는 점입니다. 만일 strict mode가 아닐 경우에는 Call Stack 정보에 접근할 수 있는 `function.prototype.arguments`나 `function.prototype.caller` 같은 속성을 사용할 수 있습니다. 하지만 Stack Frame을 재활용하게 되면서 속성 값들이 유지되지 않을 수 있기 때문에 해당 속성을 사용할 수 없는 strict mode에서만 TCO가 작동합니다. 

<br />

## 5. 맺으며 

일반적으로 반복문이 재귀 함수보다 빠른 이유는 반복문의 경우 Loop Block에 대한 Stack Cleaning을 하기 때문에 블록 내부에서 사용하는 메모리를 해제하는 반면, 재귀 함수에서는 최종적으로 반환될 때까지 메모리를 해제하지 않기 때문입니다. TCO가 적용된다면, 메모리를 적절하게 관리하면서 for나 while을 사용하지 않고도 높은 성능의 재귀 함수를 구현할 수 있습니다. 

한 가지 주의해야 할 점은 ES6에 추가되어 언어 스펙에는 추가되었지만 사실 이를 지원하고 있는 JavaScript 엔진은 사실상 WebKit이 유일하다는 점입니다. 

<br />
<img src="https://github.com/iyu88/iyu88.github.io/assets/31645195/5b80f9a2-20bb-45be-9fde-392be3199a10" alt="compat-table에서 TCO를 지원하는 엔진은 WebKit이 유일">
*Safari와 iOS에서만 지원하는 TCO 스펙*
<br />

최적화 영향을 받지 못하더라도 Call Stack을 잘 관리할 수 있는 방식으로 평소에도 신경 쓰면서 프로그램을 짜면 좋을 것 같습니다. 

---

[참고]

- [ECMAScript® 2015 Language Specification](https://262.ecma-international.org/6.0/#sec-tail-position-calls)

- [ES6 compat-table](https://kangax.github.io/compat-table/es6/#test-proper_tail_calls_(tail_call_optimisation))

- [실행 컨텍스트](https://junilhwang.github.io/TIL/Javascript/Domain/Execution-Context/)

- [꼬리 물기 최적화(Tail Call Optimization)란?](https://velog.io/@yesdoing/%EA%BC%AC%EB%A6%AC-%EB%AC%BC%EA%B8%B0-%EC%B5%9C%EC%A0%81%ED%99%94Tail-Call-Optimization%EB%9E%80-2yjnslo7sr)

- [Tail Call Optimizations - ES6](https://www.linkedin.com/pulse/tail-call-optimizations-es6-michael-clark/)

- [Tail call optimization in ECMAScript 6](https://2ality.com/2015/06/tail-call-optimization.html#the-conditional-operator)
