---
layout: post
title: "[CSS] @supports로 fallback style 적용하기"
date: 2023-07-16 00:00:00
author: Hyunbin Lee
categories: CSS
tags: CSS @supports
cover: "/assets/instacode.png"
---

# @supports로 fallback style 적용하기 

> ### 💡 CSS 쿼리를 사용한 하위 브라우저에서의 스타일 대응 방법을 소개합니다.

## 1. 들어가며

웹 개발 환경이 빠르게 변한다는 것은 비단 유행하는 프레임워크가 시시각각 변한다는 것을 의미할 뿐만 아니라 새롭게 사용할 수 있는 기능들이 많아진다는 것을 의미하기도 하는 것 같습니다. 라이브러리나 프레임워크의 버전이 올라가면 이전에 쓰던 기능의 사용법이 바뀌기도 하고 신기능이 추가되기도 합니다. 이러한 버전에 따른 차이점은 바닐라 HTML, CSS, JavaScript도 예외가 아닙니다. 

언어에서 새로운 기능을 지원하는 경우에는 기존에 있던 불편함을 간단하게 해소해주는 경우가 있기 때문에 빨리 사용해보고 싶은 마음도 듭니다. 하지만 특정 브라우저에서 새로운 스펙을 지원하지 않는 경우에는 설레는 마음으로 코드를 작성해도 실제로는 적용되지 않는 경우도 있습니다. 따라서 웹 개발자들은 브라우저라는 실행 환경 또한 고려해야 합니다. 

이 글에서는 CSS 파일에서 사용한 기능에 대해 브라우저 지원 여부에 따라 fallback style을 적용할 수 있게 해주는 CSS 쿼리에 대해서 다루어 봅니다. 


## 2. 브라우저가 CSS 최신 스펙을 지원하지 않는다면

CSS를 작성할 때 특정 선택자나 속성을 브라우저에서 지원하는지 확인하기 위해 [Can I use...](https://caniuse.com/)와 같은 사이트를 이용하곤 합니다. 해당 사이트에서는 CSS 키워드를 검색하면 다음 사진처럼 각 브라우저 버전별 지원 여부를 보여줍니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/7c79c8ed-4f0c-4f9c-b8cc-ef5415bf8d88" alt="CANIUSE 사이트에서 flex를 검색한 결과">
*해당 사이트에서 flex라는 키워드를 검색한 결과입니다.*
<br />

[What's new in CSS and UI: I/O 2023 Edition](https://developer.chrome.com/blog/whats-new-css-ui-2023/#has)에서 소개된 최신 CSS 스펙 중 하나인 `:has`에 대해서 살펴보겠습니다. :has 선택자는 has로 전달한 선택자를 포함하는 요소를 선택합니다. 가령, 아래와 같은 예제에서 `.parent:has(.children)`처럼 작성하면 `children 클래스를 가지고 있는 parent 클래스`라는 의미가 됩니다. 

<pre>
<code class="hljs html">// index.html
&lt;div class="parent"&gt;
  &lt;div class="children" /&gt;
&lt;/div&gt;</code>
</pre>

<pre>
<code class="hljs js">// index.css
.parent {
  width: 200px;
  height: 200px;
  background-color: green;
}

.children {
  width: 100px;
  height: 100px;
  background-color: red;
}

.parent:has(.children) {
  display: flex;
  justify-content: center;
  align-items: center;
}</code>
</pre>

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/5502c1ef-1549-4acb-9bbb-568726e01895" alt="index.html과 index.css를 렌더링해본 결과 초록색 사각형 중앙에 빨간색 사각형이 위치해 있다">
*parent 클래스에 display:flex가 정상적으로 적용되었습니다.*
<br />

하지만 앞서 설명했던 것처럼 `:has` 선택자 같은 경우에는 비교적 최신 스펙이기 때문에 다른 브라우저에서 지원하지 않을 가능성이 높습니다. Can I use에서 검색해보면 다음과 같이 Chrome 105 이상, Safari 15.4 이상 브라우저 버전에서는 지원하지만 Firefox 같은 브라우저에서는 아직 지원하지 않는 것을 알 수 있습니다. 위에서 정상적으로 스타일이 적용된 이유는 코드를 실행한 브라우저가 Chrome 114 버전이기 때문입니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/f9274054-f476-4ddf-9f83-3ebe02627779" alt="CANIUSE 사이트에서 :has를 검색한 결과">
*Firefox 전 버전에서 사용이 불가능합니다.*
<br />

따라서 이러한 경우에는 해당 속성을 지원하지 않는 브라우저에서 개발자의 의도대로 스타일이 적용되지 않을 수 있습니다. 실제로 Firefox 브라우저에서 동일한 코드를 실행해보면 다음과 같이 렌더링됩니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/4608a560-2431-418c-a389-63ebfa568342" alt="index.html과 index.css를 렌더링해본 결과 초록색 사각형의 좌상단에 빨간색 사각형이 위치해 있다">
*Firefox에서는 .parent:has(.children) 내부에 정의한 속성이 적용되지 않습니다.*
<br />


## 3. @supports로 스타일 보완하기

위와 같은 상황을 해결할 수 있는 여러 방법이 있겠지만 여기서는 `@supports` 쿼리를 활용한 해결법을 소개합니다.

[MDN @supports](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports)에서 말하는 @supports의 역할은 다음과 같습니다.

> The @supports CSS at-rule lets you specify CSS declarations that depend on a browser's support for CSS features.

즉, 개발자가 사용하고자 하는 CSS 기능에 대해 브라우저 지원 여부를 판단하여 추가적인 속성을 정의할 수 있도록 해줍니다. @supports를 사용하기 위해서 supports condition을 정의해야 하고, 가장 손쉽게 사용할 수 있는 방법은 `selector()` 함수를 사용하는 것입니다. 예를 들어, selector() 함수 내부에 A라는 선택자를 전달하면 `A 선택자를 지원하는 경우` 라는 의미가 됩니다. 

위에서 살펴보았던 예시에서 @supports 쿼리를 적용하여 :has 선택자를 지원하지 않을 경우 parent 클래스의 background-color를 변경하고 flex 속성도 함께 적용해보겠습니다.

<pre>
<code class="hljs js">// index.css
.parent {
  width: 200px;
  height: 200px;
  background-color: green;
}

.children {
  width: 100px;
  height: 100px;
  background-color: red;
}

.parent:has(.children) {
  display: flex;
  justify-content: center;
  align-items: center;
}

@supports not selector(:has) {
  .parent {
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: blue;
  }
}
</code>
</pre>

not 키워드를 selector() 앞에 사용하면 부정의 의미로 한정할 수 있습니다. 위와 같이 @supports 내부에 작성한 fallback style이 정상적으로 적용되었는지 Firefox 브라우저에서 확인해본 결과는 다음과 같습니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/3cf7764d-96b2-4092-93e8-1b1d19cba95a" alt="index.html과 index.css를 렌더링해본 결과 파란색 사각형 중앙에 빨간색 사각형이 위치해 있다">
*의도한대로 스타일이 정상 적용된 것을 확인할 수 있습니다.*
<br />


## 4. 맺으며

@supports를 활용하여 특정 브라우저 버전에서 CSS 기능을 지원하지 않을 때 대응하는 방법에 대해서 알아보았습니다. 위에서 정말 간단한 예제를 살펴보았지만 실제 개발 상황에서는 다른 쿼리들과 혼합하여 복잡한 상황에 충분히 대응할 수 있을 것 같습니다. 혹은 `@supports selector(:has)`처럼 작성하면 오히려 :has와 같은 최신 스펙을 지원하는 브라우저에서만 사용하고 싶은 스타일을 적용할 수 있습니다. 단, @supports와 supports condition을 정의하는 데에 사용되는 기능의 지원 범위를 염두에 두고 적절하게 사용하는 것이 중요할 것 같습니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/83168fa2-6e2c-4e6f-9c39-3231288637ed" alt="CANIUSE 사이트에서 @supports를 검색한 결과">
<br />


[참고]

- [Can I use...](https://caniuse.com/)

- [What's new in CSS and UI: I/O 2023 Edition](https://developer.chrome.com/blog/whats-new-css-ui-2023/#has)

- [MDN - :has()](https://developer.mozilla.org/en-US/docs/Web/CSS/:has)

- [MDN - @supports](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports)
