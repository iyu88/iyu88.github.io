---
layout: post
title: "[Javascript] Tagged Templates"
date: 2022-10-16
author: Hyunbin Lee
categories: Javascript
tags: Javascript String TaggedTemplates
cover: "/assets/instacode.png"
published: true
---

# Tagged Templates

> ### 💡 Javascript 의 Template Literals 를 발전시켜서 함수로 파싱하여 사용할 수 있습니다.

<br />
<br />

자바스크립트에서는 ES6 부터 백틱 ( \` \` ) 을 사용하여 문자열을 표현할 수 있습니다. 플레이스 홀더 ( '${ }' ) 를 사용하면 추가적으로 문자열 안에 변수를 담을 수도 있습니다.

<pre>
<code class="hljs javascript">const today = new Date();
console.log(`Today is ${today}`); // Today is Sun Oct 16 2022 23:01:55 GMT+0900 (한국 표준시)</code>
</pre>

→ today 에 저장된 값이 문자열에 담깁니다.

<br />
<br />

이를 발전시킨 Tagged Templates 는 백틱 안에 들어오는 값들을 함수의 인자처럼 사용할 수 있습니다.

<pre>
<code class="hljs">const today = new Date();
const weather = "sunny";

function taggedTemplate(strings, ...values) {
console.log(strings); // ['Today is ', ', weather is ', '', raw: Array(3)]
console.log(values); // [Sun Oct 16 2022 23:02:09 GMT+0900 (한국 표준시), 'sunny']
return strings[0] + values[0] + strings[1] + values[1];
}

const template = taggedTemplate`Today is ${today}, weather is ${weather}`;
console.log(template);
// Today is Sun Oct 16 2022 23:02:09 GMT+0900 (한국 표준시), weather is sunny</code>
</pre>

→ 백틱을 사용해서 마치 함수를 호출할 때 소괄호 안에 인자를 전달하는 것처럼 값을 전달하고 있는 것을 알 수 있습니다.

→ 첫 번째 인자로 전달되는 값 `strings` 는 플레이스 홀더에 해당하지 않는 문자열들이 배열로 들어가 있는 것을 확인할 수 있습니다. 마지막 값은 항상 빈 문자열 (’’) 입니다.

→ 두 번째 인자로 전달되는 값 `values` 는 플레이스 홀더에 해당하는 값을 구조 분해 할당을 통해서 가져옵니다.

→ 반환값이 없을 수도 있습니다.

<br />
<br />

위의 예시에서 strings 을 출력하면 ‘raw’ 값이 포함되어 있는 것을 알 수 있습니다.

<pre>
<code class="hljs">0: "Today is "
1: ", weather is "
2: ""
length: 3
raw: (3) ['Today is ', ', weather is ', '']</code>
</pre>

→ raw 에는 각각의 strings 요소가 들어있고 일치하는 것을 확인할 수 있습니다.

<br />
<br />

해당 값은 다음과 같이 문자열 안에 escape sequence 가 포함되어 있을 때 달라집니다.

<pre>
<code class="hljs">const template = taggedTemplate`Today is ${today}\n weather is ${weather}`;
console.log(template);
// Today is Sun Oct 16 2022 23:02:09 GMT+0900 (한국 표준시)
// weather is sunny

/***************************/

// raw 값
0: "Today is "
1: " \n weather is "
2: ""
length: 3
raw: (3) ['Today is ', ' \\n weather is ', '']</code>
</pre>

→ template literal 은 입력 받은 문자열에 대해서 escape sequence 를 처리합니다.

→ 단, raw 프로퍼티에 escape sequence 를 처리하지 않은 값이 들어 있는 것을 확인할 수 있습니다.

<br />
<br />

<pre>
<code class="hljs">const today = new Date();
const weather = "sunny";

function taggedTemplate(strings, ...values) {
console.log(strings.raw[1]); // \n weather is
}

const template = taggedTemplate`Today is ${today}\n weather is ${weather}`;
const rawStr = String.raw`Today is ${today}\n weather is ${weather}`;
console.log(rawStr);
// Today is Sun Oct 16 2022 23:03:12 GMT+0900 (한국 표준시)\n weather is sunny</code>
</pre>

→ tagged template 내부의 strings 에서 `.raw` 메서드를 사용하여 원시 문자열에 접근할 수 있습니다.

→ 또는 `String.raw` 라는 template literal 의 태그 함수를 사용하여 원시 문자열에 접근할 수 있습니다.

<br />
<br />

이러한 방식을 활용하여 HTML 태그의 style 속성을 Javascript 로 설정할 수 있습니다. 이를 'CSS-in-JS' 라고 부릅니다.

<br />
<br />

[출처]

- [Template literals - JavaScript \| MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Template_literals#tagged_templates)

- [String.raw() - JavaScript \| MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String/raw)

- [웹 컴포넌트 스타일링 관리 CSS-in-JS vs CSS-in-CSS](https://www.samsungsds.com/kr/insights/web_component.html)
