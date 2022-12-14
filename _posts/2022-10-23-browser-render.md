---
layout: post
title: "[Web] Browser Render Test - Reflow, Repaint, dev Tools"
date: 2022-10-23 00:00:00
author: Hyunbin Lee
categories: Web
tags: Web Browser Render Reflow Repaint devTools 
cover: "/assets/instacode.png"
---

# Browser Render Test - Reflow, Repaint, dev Tools

> ### π‘ λΈλΌμ°μ μ Reflow μ Repaint λ₯Ό κ°λ°μ λκ΅¬λ‘ νμΈν΄λ΄λλ€.

## 1. λΈλΌμ°μ  λ λλ§ νλ‘μΈμ€ 

μΌλ°μ μΌλ‘ μΉμ κ°λ°ν  λ μΈ κ°μ§ μΈμ΄ HTML, CSS, Javascript λ₯Ό μ¬μ©ν©λλ€. μμ±λ κ°κ°μ νμΌλ€μ λΈλΌμ°μ λ₯Ό ν΅ν΄μ μ¬μ©μλ€μκ² λ³΄μ¬μ§λλ€. μμ€ μ½λλ€μ΄ μ κ°μΆ°μ§ ννμ μΉ νμ΄μ§λ‘ λ§λ€μ΄μ§λ κ³Όμ μ λΈλΌμ°μ  λ λλ§ νλ‘μΈμ€λ₯Ό ν΅ν΄ μ€λͺν  μ μμ΅λλ€. κΈ°λ³Έμ μΈ νλ¦μ λ€μμ κ·Έλ¦Όκ³Ό κ°μ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197388724-e57fd66f-c537-4a42-9639-d8ab0c430157.jpeg">
*Rendering Process λ₯Ό λμνν μ΄λ―Έμ§*
<br />

1. λ¨Όμ  HTML νμΌμ `DOM Tree` λ‘ λ³ννλ κ³Όμ μ κ±°μΉ©λλ€. HTML νμΌ λ΄λΆμ μμ±λ μ¬λ¬ κ°μ§ νκ·Έλ€μ΄ Tokenizing (μμ λ¨μλ‘ λλκΈ°), Lexing (μλ―Έλ₯Ό κ°μ§ ννλ‘ λ§λ€κΈ°) κ³Ό Parsing (μ¬λ°λ₯Έ λ°μ΄ν°μΈμ§ νμΈνκΈ°) μ κ³Όμ μ κ±°μ³μ Tree ννμ κ΅¬μ‘°λ‘ λ§λ€μ΄μ§λλ€.

2. CSS νμΌλ λμΌνκ² Parser λ₯Ό κ±°μ³μ `CSSOM Tree` λ‘ λ§λ€μ΄μ§λλ€.

3. μμ λ§λ€μ΄μ§ λ κ°μ νΈλ¦¬μΈ DOM Tree μ CSSOM Tree λ₯Ό ν©μ³μ `Render Tree` λ₯Ό λ§λ­λλ€. CSSOM μμ μμ±λ μ€νμΌμ΄ μ μ©λμ΄μΌ ν  λΆλΆμ DOM Tree μμ μ°Ύμμ μ μ©νλ λ°©μμΌλ‘ μμ±λ©λλ€. μ΄ λ, Render Tree μ νΉμ§μ μ€μ λ‘ λΈλΌμ°μ μμ λνλλ ννμ Render Tree μ ννκ° λ€λ₯Ό μ μλ€λ μ μλλ€. μλ₯Ό λ€μ΄μ νλ©΄μ μ€μ λ‘ νμλμ§ μλ visibility: "hidden" κ³Ό κ°μ μμ±μ΄ μ μ©λ νκ·Έμ κ²½μ°μλ Render Tree μ μ‘΄μ¬νμ§λ§ λΈλΌμ°μ λ₯Ό λ λλ§ν  λλ μ μΈλ©λλ€.

4. Render Tree λ₯Ό κΈ°λ°μΌλ‘ νμ¬ λΈλΌμ°μ μμ κ° νκ·Έκ° νμλμ΄μΌ νλ μμΉμ ν¬κΈ°λ₯Ό κ³μ°νλ κ³Όμ μΈ `Layout` μ κ±°μΉ©λλ€. μ£Όλ‘ μμΉμ κ΄λ ¨λ μμ±μ κ³μ°ν©λλ€. 

5. νκ·Έλ€μ κ°μ λ€λ₯Έ κ³μΈ΅μΌλ‘ λΆλ₯λ©λλ€. νκ·Έκ° μν  Layer λ₯Ό λλλ κ³Όμ μ `Layer Tree` λΌκ³  λΆλ¦λλ€. 

6. μ€μ  λ λλ§λ  μμλ€μ λΈλΌμ°μ μ κ·Έλ¦¬λ κ³Όμ μ `Paint` (draw) λΌκ³  λΆλ¦λλ€.

7. λ§μ§λ§μΌλ‘ μμκ° λ λλ§λ κ³μΈ΅λ€μ΄ νλλ‘ ν©μ³μ§ κ²μ `Composite Layer` λΌκ³  ν©λλ€. μ¬λ¬ Layer λ₯Ό νλμ λΉνΈλ§΅μΌλ‘ λ§λ€μ΄μ μ΅μ’μ μΈ νμ΄μ§λ₯Ό λ§λ­λλ€. 

<hr />

## 2. Reflow μ Repaint 

Javascript λ μΉ νμ΄μ§μ UIλ₯Ό λμ μΌλ‘ λ°κΏ μ μμ΅λλ€. μ΄λ κ² λ°λ λΆλΆμ λΈλΌμ°μ μ λ°μνκΈ° μν΄μ λ λλ§ νλ‘μΈμ€λ₯Ό λ€μ μ§νν΄μΌ ν©λλ€. μ΄μ²λΌ λ°λ λΆλΆμ΄ λ λλ§λ  λ `Reflow`, `Repaint` κ° λ°μνλ€κ³  ν©λλ€. μ΄λ μμ μ€λͺν λ λλ§ κ³Όμ μμ κ°κ° `Layout` κ³Ό `Paint` κ° λ€μ λ°μνλ κ²μ λ§ν©λλ€.

<br />
<img src="https://www.freecodecamp.org/news/content/images/size/w1600/2022/02/Twitter-post---55.png">
*Reflow μ Repaint κ° λ°μνλ μ‘°κ±΄μ λν΄ κ°λ¨νκ² μ λ¦¬ν μ¬μ§*
<br />

<hr />

## 3. Chrome dev Tools μ΄ν΄λ³΄κΈ° 

μ½λλ‘ Reflow μ Repaint λ₯Ό νμΈνκΈ° μ μ μ κΉ Chrome κ°λ°μ λκ΅¬μ λν΄μ μ΄ν΄λ³΄κ² μ΅λλ€. 

<br />

### 1) Performance 

κ°λ°μ λκ΅¬μ Performance ν­μμ μΉ νμ΄μ§λ₯Ό μΌμ μκ° λΉννκ³  λ°μν μ΄λ²€νΈλ₯Ό λͺ¨λν°λ§ν  μ μμ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197391162-910e3dd3-d64d-413e-a9be-6cd8aa1c8788.png">
*Performance ν­μ κ΅¬μ±*
<br />

1. μΉ νμ΄μ§ λΉνλ₯Ό μμν©λλ€. λΉνλ₯Ό μ€λ¨νκ³  μΆμ λ λ©μΆ μ μμ΅λλ€. 
2. μΉ νμ΄μ§λ₯Ό μλ‘κ³ μΉ¨νμ¬ μ²μ λ‘λ©λ  λλΆν° λΉνλ₯Ό μμν©λλ€. 
3. λΉνλ κΈ°λ‘μ μ­μ ν  μ μμ΅λλ€. 
4. λΉνλ μκ°λμ κ° νλ μλ§λ€ μ€λμ·μ νμΈν  μ μμ΅λλ€. 
5. κ°κ°μ ν­μ ν΅ν΄μ μνλ μ±λ₯ μ λ³΄λ₯Ό λͺ¨λν°λ§ν  μ μμ΅λλ€. 
6. μ°Ύκ³  μΆμ Activity λ₯Ό κ²μνμ¬ νν°λ§ν  μ μμ΅λλ€. 
7. λΉνλ μκ°λμ λ°μν Activity λͺ©λ‘μ νμΈν  μ μμ΅λλ€. 

<br />

### 2) Layers 

Performance ν­μμ Paint Profiler λ₯Ό ν΅ν΄μ νμΈν  μ μλ μ λ³΄λ₯Ό Layers ν­μμ λ μμΈνκ² νμΈν  μ μμ΅λλ€. λ€μμ μ½λλ‘ μ΄ν΄λ³Ό μμ  μ½λλ₯Ό μ¬μ©νμ¬ Layers ν­μ λν΄μ μ΄ν΄λ³΄κ² μ΅λλ€. ν­μ λ€μ΄κ°λ©΄ λ€μκ³Ό κ°μ νλ©΄μ΄ λμ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197393780-9e50f7db-03af-45fd-b978-14a5022dc4c3.png">
*Layers ν­μ κ΅¬μ±*
<br />

1. νμ¬ μλμ°μ Layers λ₯Ό λνλλλ€. 
2. Layers λ₯Ό μ΄λνκ±°λ νμ νλ©΄μ μ΄ν΄λ³Ό μ μλ μ»¨νΈλ‘€λ¬μλλ€. 
3. Layers κ΅¬μ±μ νμΈν  μ μμ΅λλ€. 
4. Details ν­μμ Layer μ μΈλΆμ λ³΄μ Profiler μμ κ΅¬μ²΄μ μΈ λ λλ§ κ³Όμ μ νμΈν  μ μμ΅λλ€. 
5. νμ¬ νμ΄μ§μμ λ λλ§λ μμλ€μ λ‘κ·Έλ₯Ό νμΈν  μ μμ΅λλ€. 
6. μ¬λΌμ΄λλ‘ λ²μλ₯Ό μ§μ νμ¬ 3λ²κ³Ό 5λ² μμ­μ νν°λ§ν  μ μμ΅λλ€. 

<br />

### 3) Rendering 

λ§μ§λ§ κ°λ°μ λκ΅¬μΈ Rendering μ λν΄μ μ΄ν΄λ³΄κ² μ΅λλ€. ν΄λΉ ν­μμ μ΅μμ νμ±ννλ©΄ μΉ νμ΄μ§μμ μ€μκ°μΌλ‘ λ³νλ μμλ₯Ό λμ± νΈνκ² μ°Ύμ μ μμ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197394371-ef3393b8-40e7-493a-b528-7152c2222a05.png">
*Rendering ν­μ μλ¨μ μλ κ²½μ°κ° μμ΅λλ€. λ€μ κ²½λ‘μμ μ°Ύμ μ μμ΅λλ€.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197394796-309bc21d-092e-40e9-81c6-ce453922a774.png">
*μ²μ λ€μ΄κ°λ©΄ λͺ¨λ  μ²΄ν¬λ°μ€κ° ν΄μ λμ΄ μμ΅λλ€. μμμ μΈ κ°μ§ λκ΅¬λ§ νμ±ννμ΅λλ€.*
<br />

- Paint flashing : Repaint κ° μΌμ΄λλ λΆλΆμ μ΄λ‘μμΌλ‘ νμν©λλ€. 
- Layout Shift Regions : μ νλ λΆλΆμ νλμμΌλ‘ νμν©λλ€. 
- Layer borders : λ μ΄μ΄ νλλ¦¬λ₯Ό μ£Όν©μμ΄λ μ¬λ¦¬λΈμμΌλ‘ νμν©λλ€. 

κΈ°λ₯μ νμ±νν λ€ μΉ νμ΄μ§λ₯Ό λ³΄λ©΄ μ€μ ν μ΅μμ λ°λΌμ μμμ΄ λνλλ κ²μ νμΈν  μ μμ΅λλ€. 

![Rendering ν­ μ€μ νμ λ](https://user-images.githubusercontent.com/31645195/197394449-d78e7760-bdd3-4635-920e-244dd7a22703.gif)

<hr />

## 4. μ΅μ ν λ°©λ² νμΈν΄λ³΄κΈ° 

μμ μκ°ν κ°λ°μ λκ΅¬ μ€μμ Performance λ₯Ό μ£Όλ‘ νμ©νλ©°, λ€ κ°μ§ μλ‘ λ€λ₯Έ λ°©μμΌλ‘ μ½λ©ν λ°©μμ λΉκ΅νλ©΄μ Reflow μ Repaint λ₯Ό νμΈν΄λ³΄κ² μ΅λλ€. 

*μμλ HTML Element λ₯Ό μλ―Έν©λλ€. 
*μμ±μ CSS Property λ₯Ό μλ―Έν©λλ€. 

<br />

### 1) μ¬λ¬ μμμ μμ±μ μλ°μ΄νΈ 

λ¨Όμ  μμμ `width` λ₯Ό λ³κ²½νλ κ²½μ°μ λν΄μ νμ€νΈ ν΄λ³΄μμ΅λλ€. 

<br />

#### 1-1) μ¬λ¬ μμμ μμ±μ λ°λ‘ μλ°μ΄νΈ 

<pre>
<code class="hljs javascript">const btn = document.querySelector('button');
const first = document.querySelector("#first")
const second = document.querySelector("#second")
const third = document.querySelector("#third")
const forth = document.querySelector("#forth")

btn.addEventListener("click", () => {
  const step = 10;
  const firstWidth = first.getBoundingClientRect().width;
  first.style.width = firstWidth + step + "px";

  const secondWidth = second.getBoundingClientRect().width;
  second.style.width = secondWidth + step + "px";

  const thirdWidth = third.getBoundingClientRect().width;
  third.style.width = thirdWidth + step + "px";

  const forthWidth = forth.getBoundingClientRect().width;
  forth.style.width = forthWidth + step + "px";
});</code>
</pre>

<br />

#### 1-2) μ¬λ¬ μμμ μμ±μ νκΊΌλ²μ μλ°μ΄νΈ 

<pre>
<code class="hljs javascript">btn.addEventListener("click", () => {
  const step = 10;
  const firstWidth = first.getBoundingClientRect().width;
  const secondWidth = second.getBoundingClientRect().width;
  const thirdWidth = third.getBoundingClientRect().width;
  const forthWidth = forth.getBoundingClientRect().width;

  first.style.width = firstWidth + step + "px";
  second.style.width = secondWidth + step + "px";
  third.style.width = thirdWidth + step + "px";
  forth.style.width = forthWidth + step + "px";
});</code>
</pre>

<br />

λ μ½λμ μ°¨μ΄μ μ `Element.style.Property` λ₯Ό μ€μ νλ μ½λκ° μ°μμ μΌλ‘ μ‘΄μ¬νλμ§ μ¬λΆμλλ€. μ΄λ¬ν μμ€ μ½λμ μ°¨μ΄κ° μ΄λ ν μ μμ λ€λ₯Έμ§ Performance ν­μμ νμΈν΄λ³΄κ² μ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197396610-7bde0249-f447-48e8-995d-7e64a737e1db.png">
*1-1) μ κ²°κ³Όμλλ€. Layout μ΄λ²€νΈκ° μ΄ 5λ² λ°μν κ²μ μ μ μμ΅λλ€.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197396664-fa5d85b1-bd09-42f3-afb2-13a74bd90741.png">
*1-2) μ κ²°κ³Όμλλ€. Layout μ΄λ²€νΈκ° μ΄ 2λ² λ°μν κ²μ μ μ μμ΅λλ€.*
<br />

μμ λ μ¬μ§μμ νμΈν  μ μλ―μ΄ `1-2) μ¬λ¬ μμμ μμ±μ νκΊΌλ²μ μλ°μ΄νΈ` μ κ²½μ°κ° Layout μ΄λ²€νΈλ₯Ό 3λ² λ λ°μμν€λ κ²μ μ μ μμ΅λλ€. μ΄λ `Element.style.Property` μ²λΌ μμ±μ μλ°μ΄νΈνλ μ½λκ° λͺ¨μ¬ μκΈ° λλ¬Έμ Layout μ μ¬κ³μ°ν  λ λ³κ²½λ κ°μ΄ ν λ²μ μ μ©λκΈ° λλ¬Έμλλ€. λ°λΌμ Layout μ λ³κ²½μν€λ κ°μ μ½λ μμμ λͺ¨μμ μμ±ν΄μΌ Reflow λ₯Ό μ κ² λ°μμν¨λ€λ κ²μ μ μ μμ΅λλ€. 

<hr />

### 2) ν μμμ μμ±μ μλ°μ΄νΈ  

λ€μμΌλ‘λ μμ΄λκ° `first` μΈ μμμ μμ±λ§ μλ°μ΄νΈνλ κ²½μ°μ λν΄μ νμ€νΈ ν΄λ³΄μμ΅λλ€.  

<br />

#### 2-1) ν μμμ μμ±μ λ°λ‘ μλ°μ΄νΈ

<pre>
<code class="hljs javascript">const btn = document.querySelector('button');
const first = document.querySelector("#first")

btn.addEventListener("click", () => {
  const step = 10;
  const firstFontSize = "4em";
  first.style.fontSize = firstFontSize;

  const firstWidth = first.getBoundingClientRect().width;
  first.style.width = firstWidth + step + "px";

  const firstBackground = "burlywood";
  first.style.backgroundColor = firstBackground;
});</code>
</pre>

<br />

#### 2-2) ν μμμ μμ±μ νκΊΌλ²μ μλ°μ΄νΈ

<pre>
<code class="hljs javascript">btn.addEventListener("click", () => {
  const step = 10;
  const firstFontSize = "4em";
  const firstWidth = first.getBoundingClientRect().width;
  const firstBackground = "burlywood";

  first.style.fontSize = firstFontSize;
  first.style.width = firstWidth + step + "px";
  first.style.backgroundColor = firstBackground;
});</code>
</pre>

μμ `1) μ¬λ¬ μμμ μμ±μ μλ°μ΄νΈ` μ μμμμ μ€νν΄λ΄€λ μ μ νλμ μμμμ λμΌνκ² μ§νν΄λ³΄μμ΅λλ€. λ€λ₯Έ μ μ width μμ±λΏλ§ μλλΌ `fontSize` μ `backgroundColor` μμ±κΉμ§ λ³κ²½νλ€λ μ μλλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197397451-2b37fdb9-2a77-4fec-a7bb-f37f17e106d6.png">
*2-1) μ κ²°κ³Όμλλ€. Layout μ΄λ²€νΈκ° μ΄ 3λ² λ°μν κ²μ μ μ μμ΅λλ€.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197397454-01140744-8da7-4b78-9eea-da4019224e88.png">
*2-2) μ κ²°κ³Όμλλ€. Layout μ΄λ²€νΈκ° μ΄ 2λ² λ°μν κ²μ μ μ μμ΅λλ€.*
<br />

width μ λλΆμ΄ fontSize λ₯Ό λ³κ²½νλ€λ μ  λλ¬Έμ `2-1) ν μμμ μμ±μ λ°λ‘ μλ°μ΄νΈ` μμ Reflow κ° 1λ² λ μΌμ΄λλ€λ μ μ μ μ μμ΅λλ€.

<hr />

### 3) λΈμΆ μ μ΄λ₯Ό ν΅ν μμ± μ€μ  

μμλ₯Ό νλ©΄μ νμλμ§ μλ μνλ‘ λ§λ  λ€μ μνλ μμ±μ μ€μ νκ³ , λ€μ μ΄μ μ²λΌ νλ©΄μ λνλλλ‘ μμ±μ λ³΅μν©λλ€. μ΄λ₯Ό ν΅ν΄μ νλ©΄μ λ λλ§λμ§ μλ μνμμ μ€νμΌμ λ³κ²½ν  μ μκΈ° λλ¬Έμ Paint ν  λ λλ μ°μ°μ μ€μΌ μ μμ΅λλ€. μ΄λ₯Ό μ€νμ ν΅ν΄μ νμΈν΄λ³΄κ² μ΅λλ€. 

<br />

#### 3-1) Opacity λ₯Ό ν΅ν λΈμΆ μ μ΄ 

<pre>
<code class="hljs javascript">btn.addEventListener("click", () => {
  const step = 10;
  const firstFontSize = "4em";
  const firstWidth = first.getBoundingClientRect().width;
  const firstBackground = "burlywood";

  first.style.opacity = 0;
  first.style.fontSize = firstFontSize;
  first.style.width = firstWidth + step + "px";
  first.style.backgroundColor = firstBackground;
  first.style.opacity = 1;
});</code>
</pre>

<br />

#### 3-2) Display λ₯Ό ν΅ν λΈμΆ μ μ΄ 

<pre>
<code class="hljs javascript">first.style.display = "none";
first.style.fontSize = firstFontSize;
first.style.width = firstWidth + step + "px";
first.style.backgroundColor = firstBackground;
first.style.display = "block";</code>
</pre>

<br />

#### 3-3) Visibility λ₯Ό ν΅ν λΈμΆ μ μ΄ 

<pre>
<code class="hljs javascript">first.style.visibility = "hidden";
first.style.fontSize = firstFontSize;
first.style.width = firstWidth + step + "px";
first.style.backgroundColor = firstBackground;
first.style.visibility = "visible";</code>
</pre>

μΈ μ½λλ κ°κ° opacity, display, visibility μμ±μ μ‘°μ νμ¬ μμλ₯Ό νλ©΄μμ μ§μλλ€. `2) ν μμμ μμ±μ μλ°μ΄νΈ` μμ μ§νν κ²κ³Ό λμΌνκ² 3κ°μ μμ±μ μμ ν λ€μ λ€μ νλ©΄μ λ λλ§ν κ²μ Performance ν­μμ λΉνν κ²°κ³Όλ λ€μκ³Ό κ°μ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197398277-7b7619c0-fb4f-4034-821f-17ca21638238.png">
*3-1) μ κ²°κ³Όμλλ€. Layout μ 0.3ms, Paint λ 0.4ms, Composite Layers λ 1.2ms κ° κ±Έλ Έμ΅λλ€.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197398275-e2721df1-d890-4386-8caf-e3e007503d02.png">
*3-2) μ κ²°κ³Όμλλ€. Layout μ 0.3ms, Paint λ 0.5ms, Composite Layers λ 0.9ms κ° κ±Έλ Έμ΅λλ€.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197399118-2553adbb-5604-4b45-a6c2-4e378aee2dba.png">
*3-3) μ κ²°κ³Όμλλ€. Layout μ 0.6ms, Paint λ 0.9ms, Composite Layers λ 1.4ms κ° κ±Έλ Έμ΅λλ€.*
<br />

μμ λ κ²½μ°μμ opacity μμ±μ μ¬μ©ν κ²½μ°κ° 0.3msλ§νΌ λ Composite Layers μ΄λ²€νΈμ μκ°μ ν μ ν κ²μ νμΈν  μ μμ΅λλ€. λν Layout μ κ²½μ° μ λμ μΈ μκ°μ λμΌν κ²μΌλ‘ λ³΄μ΄λ λΉμ¨λ‘ λ³΄μμ λ, opacity λ 4.7%, display λ 6.1% κ±Έλ¦° κ²μ νμΈν  μ μμ΅λλ€. μ΄λ `2. Reflow μ Repaint` μμ μ²¨λΆν μ΄λ―Έμ§μμ νμΈν  μ μλ―μ΄ opacity μμ±μ Composite Layers μ μν₯μ μ£Όκ³  display μμ±μ Layout μ μν₯μ μ£ΌκΈ° λλ¬Έμ΄λΌκ³  ν΄μν  μ μμ΅λλ€. ννΈ, λ§μ§λ§ μμ μμ Paint κ° 0.9ms κ° κ±Έλ Έμ΅λλ€. μ΄λ visibility μμ±μ΄ Paint μ μν₯μ μ£ΌκΈ° λλ¬Έμ΄λΌκ³  ν  μ μμ΅λλ€. μ°Έκ³ λ‘ κ° νμ€νΈ κ²½μ°μμ Composite Layers μ΄λ²€νΈκ° λ°μν νμλ 92ν, 52ν, 53νμμ΅λλ€. 

<hr />

### 4) λΈλ λ³΅μ λ₯Ό ν΅ν μμ± μ€μ  

λ§μ§λ§μΌλ‘ λΈλλ₯Ό λ³΅μ νκ³ , λ³΅μ ν λΈλμ μμ±μ μμ νλ λ°©μμΌλ‘λ λ λλ§ν  μ μμ΅λλ€. 

<br />

<pre>
<code class="hljs javascript">const btn = document.querySelector('button');
const first = document.querySelector("#first")

btn.addEventListener("click", () => {
  const cloned = first.cloneNode(true);
  const step = 10;
  const firstFontSize = "4em";
  const firstWidth = first.getBoundingClientRect().width;
  const firstBackground = "burlywood";

  cloned.style.fontSize = firstFontSize;
  cloned.style.width = firstWidth + step + "px";
  cloned.style.backgroundColor = firstBackground;

  first.parentNode.replaceChild(cloned, first);
});</code>
</pre>

μ€νμΌμ νκΊΌλ²μ μλ°μ΄νΈ ν `2-2) ν μμμ μμ±μ νκΊΌλ²μ μλ°μ΄νΈ` μ κ²°κ³Όμ λΉκ΅ν΄λ³΄κ² μ΅λλ€. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197400100-3396f28f-28eb-4b7a-b695-b121ce92209f.png">
*4) μ κ²°κ³Όμλλ€. Layout μ΄ 0.3ms, Paint κ° 0.4ms κ° κ±Έλ Έμ΅λλ€.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197400088-020b6924-08cf-41b1-86cd-b1ad5359fa93.png">
*2-2) μ κ²°κ³Όμλλ€. Layout μ΄ 0.3ms, Paint κ° 0.4ms κ° κ±Έλ Έμ΅λλ€.*
<br />

λ κ°μ§ μ΅μ ν λ°©μ λͺ¨λ Reflow μ Repaint κ° λμΌν μκ°μ΄ κ±Έλ¦° κ²μ νμΈν  μ μμ΅λλ€.

<hr />

## 5. κ²°λ‘ 

- Reflow μ Repaint λ μΉ νμ΄μ§ λ³κ²½ μμμ λ°λΌμ λ°μν©λλ€. 
- ν΄λΉ μ΄λ²€νΈλ€μ λΈλΌμ°μ μ κ°λ°μ λκ΅¬λ₯Ό ν΅ν΄μ νμΈν  μ μμ΅λλ€. 
- λ λλ§ μ΅μ νλ₯Ό μν΄μ
  + 1) μ€νμΌ μμ±μ μλ°μ΄νΈνλ μ½λλ ν κ³³μ λͺ¨μμ μμ±ν©λλ€. 
  + 2) λΈμΆ μ μ΄λ₯Ό ν΅ν΄μ μ΅μ νν  μ μμ΅λλ€. opacity μμ±μ΄ κ°μ₯ μ ν¨ν©λλ€. 
  + 3) λΈλλ₯Ό λ³΅μ¬νλ λ°©μμΌλ‘λ μ΅μ νν  μ μμ΅λλ€. 

<hr />

## 6. μκ°ν΄λ³Ό μ 

- λ¨μν κ΅¬μ‘°λ‘ νμ€νΈν΄μ λμ λλ μ°¨μ΄λ₯Ό νμΈνκΈ° νλ€μμ΅λλ€. λ λ³΅μ‘ν κ΅¬μ‘°μμ μ€νν΄λ³Ό νμκ° μμ΅λλ€. 

<hr />

[μΆμ² & μ°Έκ³ ]


- [https://www.webperf.tips/tip/measuring-paint-time/](https://www.webperf.tips/tip/measuring-paint-time/)

- [Detecting when the Browser Paints Frames in JavaScript](https://www.webperf.tips/tip/measuring-paint-time/)

- [Understanding Repaint and Reflow in JavaScript](https://medium.com/swlh/what-the-heck-is-repaint-and-reflow-in-the-browser-b2d0fb980c08)

- [DOM Performance (Reflow & Repaint) (Summary)](https://gist.github.com/faressoft/36cdd64faae21ed22948b458e6bf04d5)

- [Render-tree Construction, Layout, and Paint](https://web.dev/critical-rendering-path-render-tree-construction/)

- [[Browser] Reflowμ Repaint](https://beomy.github.io/tech/browser/reflow-repaint/)

- [λΈλΌμ°μ  λ λλ§μ λν μ΄ν΄μ μ΅μ ν](https://velog.io/@yrnana/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EB%A0%8C%EB%8D%94%EB%A7%81%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%ED%95%B4%EC%99%80-%EC%B5%9C%EC%A0%81%ED%99%94)

- [λΈλΌμ°μ μ Reflow μ Repaint](https://oyg0420.tistory.com/entry/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%9D%98-Reflow-%EC%99%80-Repaint)

- [Analyze runtime performance - Chrome Developers](https://developer.chrome.com/docs/devtools/evaluate-performance/)

- [Profiling site speed with the Chrome DevTools Performance tab / DebugBear](https://www.debugbear.com/blog/devtools-performance)

- [Debug like a Pro in Chrome Dev Toolsβ-βHow to use Rendering tab](https://tharakamd-12.medium.com/debug-like-a-pro-in-chrome-dev-tools-how-to-use-rendering-tab-ba25f5bb6264)

- [What is the Chrome Dev Tools "Hit Test" timeline entry?](https://stackoverflow.com/questions/33174286/what-is-the-chrome-dev-tools-hit-test-timeline-entry)

- [Render-tree Construction, Layout, and Paint](https://web.dev/critical-rendering-path-render-tree-construction/)

- [Web Animation Performance Fundamentals - How to Make Your Pages Look Smooth](https://www.freecodecamp.org/news/web-animation-performance-fundamentals/)

- [Dev-Docs/μΉ λΈλΌμ°μ μ μλ μλ¦¬.md at master Β· im-d-team/Dev-Docs](https://github.com/im-d-team/Dev-Docs/blob/master/Browser/%EC%9B%B9%20%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%9D%98%20%EC%9E%91%EB%8F%99%20%EC%9B%90%EB%A6%AC.md)

<br />

[μ½λ λ§ν¬]
- [https://github.com/iyu88/Browser-Render-Test](https://github.com/iyu88/Browser-Render-Test)

<br />
