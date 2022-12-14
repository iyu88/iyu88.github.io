---
layout: post
title: "[Javascript] Tagged Templates"
date: 2022-10-16 00:00:00
author: Hyunbin Lee
categories: Javascript
tags: Javascript String TaggedTemplates
cover: "/assets/instacode.png"
---

# Tagged Templates

> ### π‘Β Javascript μ Template Literals λ₯Ό λ°μ μμΌμ ν¨μλ‘ νμ±νμ¬ μ¬μ©ν  μ μμ΅λλ€.

<br />
<br />

μλ°μ€ν¬λ¦½νΈμμλ ES6 λΆν° λ°±ν± ( \` \` ) μ μ¬μ©νμ¬ λ¬Έμμ΄μ ννν  μ μμ΅λλ€. νλ μ΄μ€ νλ ( '${ }' ) λ₯Ό μ¬μ©νλ©΄ μΆκ°μ μΌλ‘ λ¬Έμμ΄ μμ λ³μλ₯Ό λ΄μ μλ μμ΅λλ€.

<pre>
<code class="hljs javascript">const today = new Date();
console.log(`Today is ${today}`); // Today is Sun Oct 16 2022 23:01:55 GMT+0900 (νκ΅­ νμ€μ)</code>
</pre>

β today μ μ μ₯λ κ°μ΄ λ¬Έμμ΄μ λ΄κΉλλ€.

<br />
<br />

μ΄λ₯Ό λ°μ μν¨ Tagged Templates λ λ°±ν± μμ λ€μ΄μ€λ κ°λ€μ ν¨μμ μΈμμ²λΌ μ¬μ©ν  μ μμ΅λλ€.

<pre>
<code class="hljs javascript">const today = new Date();
const weather = "sunny";

function taggedTemplate(strings, ...values) {
console.log(strings); // ['Today is ', ', weather is ', '', raw: Array(3)]
console.log(values); // [Sun Oct 16 2022 23:02:09 GMT+0900 (νκ΅­ νμ€μ), 'sunny']
return strings[0] + values[0] + strings[1] + values[1];
}

const template = taggedTemplate`Today is ${today}, weather is ${weather}`;
console.log(template);
// Today is Sun Oct 16 2022 23:02:09 GMT+0900 (νκ΅­ νμ€μ), weather is sunny</code>
</pre>

β λ°±ν±μ μ¬μ©ν΄μ λ§μΉ ν¨μλ₯Ό νΈμΆν  λ μκ΄νΈ μμ μΈμλ₯Ό μ λ¬νλ κ²μ²λΌ κ°μ μ λ¬νκ³  μλ κ²μ μ μ μμ΅λλ€.

β μ²« λ²μ§Έ μΈμλ‘ μ λ¬λλ κ° `strings` λ νλ μ΄μ€ νλμ ν΄λΉνμ§ μλ λ¬Έμμ΄λ€μ΄ λ°°μ΄λ‘ λ€μ΄κ° μλ κ²μ νμΈν  μ μμ΅λλ€. λ§μ§λ§ κ°μ ν­μ λΉ λ¬Έμμ΄ (ββ) μλλ€.

β λ λ²μ§Έ μΈμλ‘ μ λ¬λλ κ° `values` λ νλ μ΄μ€ νλμ ν΄λΉνλ κ°μ κ΅¬μ‘° λΆν΄ ν λΉμ ν΅ν΄μ κ°μ Έμ΅λλ€.

β λ°νκ°μ΄ μμ μλ μμ΅λλ€.

<br />
<br />

μμ μμμμ strings μ μΆλ ₯νλ©΄ βrawβ κ°μ΄ ν¬ν¨λμ΄ μλ κ²μ μ μ μμ΅λλ€.

<pre>
<code class="hljs javascript">0: "Today is "
1: ", weather is "
2: ""
length: 3
raw: (3) ['Today is ', ', weather is ', '']</code>
</pre>

β raw μλ κ°κ°μ strings μμκ° λ€μ΄μκ³  μΌμΉνλ κ²μ νμΈν  μ μμ΅λλ€.

<br />
<br />

ν΄λΉ κ°μ λ€μκ³Ό κ°μ΄ λ¬Έμμ΄ μμ escape sequence κ° ν¬ν¨λμ΄ μμ λ λ¬λΌμ§λλ€.

<pre>
<code class="hljs javascript">const template = taggedTemplate`Today is ${today}\n weather is ${weather}`;
console.log(template);
// Today is Sun Oct 16 2022 23:02:09 GMT+0900 (νκ΅­ νμ€μ)
// weather is sunny

/***************************/

// raw κ°
0: "Today is "
1: " \n weather is "
2: ""
length: 3
raw: (3) ['Today is ', ' \\n weather is ', '']</code>
</pre>

β template literal μ μλ ₯ λ°μ λ¬Έμμ΄μ λν΄μ escape sequence λ₯Ό μ²λ¦¬ν©λλ€.

β λ¨, raw νλ‘νΌν°μ escape sequence λ₯Ό μ²λ¦¬νμ§ μμ κ°μ΄ λ€μ΄ μλ κ²μ νμΈν  μ μμ΅λλ€.

<br />
<br />

<pre>
<code class="hljs javascript">const today = new Date();
const weather = "sunny";

function taggedTemplate(strings, ...values) {
console.log(strings.raw[1]); // \n weather is
}

const template = taggedTemplate`Today is ${today}\n weather is ${weather}`;
const rawStr = String.raw`Today is ${today}\n weather is ${weather}`;
console.log(rawStr);
// Today is Sun Oct 16 2022 23:03:12 GMT+0900 (νκ΅­ νμ€μ)\n weather is sunny</code>
</pre>

β tagged template λ΄λΆμ strings μμ `.raw` λ©μλλ₯Ό μ¬μ©νμ¬ μμ λ¬Έμμ΄μ μ κ·Όν  μ μμ΅λλ€.

β λλ `String.raw` λΌλ template literal μ νκ·Έ ν¨μλ₯Ό μ¬μ©νμ¬ μμ λ¬Έμμ΄μ μ κ·Όν  μ μμ΅λλ€.

<br />
<br />

μ΄λ¬ν λ°©μμ νμ©νμ¬ HTML νκ·Έμ style μμ±μ Javascript λ‘ μ€μ ν  μ μμ΅λλ€. μ΄λ₯Ό 'CSS-in-JS' λΌκ³  λΆλ¦λλ€.

<br />
<br />

[μΆμ²]

- [Template literals - JavaScript \| MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Template_literals#tagged_templates)

- [String.raw() - JavaScript \| MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/String/raw)

- [μΉ μ»΄ν¬λνΈ μ€νμΌλ§ κ΄λ¦¬ CSS-in-JS vs CSS-in-CSS](https://www.samsungsds.com/kr/insights/web_component.html)
