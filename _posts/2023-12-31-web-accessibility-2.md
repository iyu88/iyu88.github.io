---
layout: post
title: "[A11y] WAI 문서로 접근성 이해하기 (2/2)"
date: 2023-12-31 00:00:00
author: Hyunbin Lee
categories: A11y
tags: Accessibility A11y
cover: "/assets/instacode.png"
---

# WAI 문서로 접근성 이해하기 (2/2)

> ### 💡 W3C에서 출판한 기술 사양을 읽어보며 접근성을 높일 수 있는 개발 요소들을 알아봅니다.

## 1. 들어가며 

지난 번에 작성한 [WAI 문서로 접근성 이해하기 (1/2)](https://iyu88.github.io/a11y/2023/12/24/web-accessibility-1.html)에서는 웹 접근성이 무엇인지에 대해서 간략히 알아보고 시맨틱 태그를 사용할 때 접근성을 높이는 방법과 접근성을 고려하며 메뉴, 이미지 등의 요소를 개발하는 방법에 대해 알아보았습니다. 

이번에는 이어서 [Tutorials](https://www.w3.org/WAI/tutorials/)에 남은 부분인 Tables, Forms, Carousels 파트에 있는 내용과 예시를 살펴보겠습니다. 

<br />

## 2. 표 (Tables)

`<table>`는 그리드 형식으로 데이터를 표현하는 데에 사용합니다. 따라서 콘텐츠의 레이아웃을 표현하기 위해 사용하는 것은 적절하지 않습니다. 표를 구성할 때 머리글은 `<th>`, 데이터는 `<td> `를 사용합니다. 또한 복잡도가 높은 표에서는 scope, id, headers 속성을 함께 사용하기도 합니다. 이는 **스크린 리더가 table 태그를 탐색할 때, 맥락을 잃지 않도록 데이터 ➡️ 머리글 (td ➡️ th) 순서로 읽기 때문**입니다. 따라서 보조 기술의 지원을 받으려면 적절한 마크업을 통해 데이터를 올바른 머리글과 연관 지어야 합니다. 종종 표가 PDF 등의 다른 파일 포맷에서 표현될 때 시맨틱 태그들이 사라지기도 하는데, 접근성을 위해 이를 유지시키는 것이 좋습니다. 

### 2-1. 머리글이 한 개인 경우 

#### 2-1-1. 첫번째 행 또는 열이 머리글인 경우 

행 또는 열에 머리글이 하나만 있고 각 데이터가 분명하게 구분되는 경우 머리글에 th 태그를 사용합니다.

{% highlight html%}
<!-- 머리글인 첫번째 열을 모두 th 태그로 표현합니다. -->
<table>
  <tr>
    <th>Date</th>
    <th>Event</th>
    <th>Venue</th>
  </tr>
  <tr>
    <td>12 February</td>
    <td>Waltz with Strauss</td>
    <td>Main Hall</td>
  </tr>
  […]
</table>
{% endhighlight %}

{% highlight html%}
<!-- 머리글인 각 열의 첫번째 행을 th 태그로 표현합니다. -->
<table>
  <tr>
    <th>Date</th>
    <td>12 February</td>
    <td>24 March</td>
    <td>14 April</td>
  </tr>
  <tr>
    <th>Event</th>
    <td>Waltz with Strauss</td>
    <td>The Obelisks</td>
    <td>The What</td>
  </tr>
  <tr>
    <th>Venue</th>
    <td>Main Hall</td>
    <td>West Wing</td>
    <td>Main Hall</td>
  </tr>
</table>
{% endhighlight %}

<br />

#### 2-1-2. 모호한 데이터가 있는 경우 

데이터가 모호하여 머리글 정보 없이 임의의 칸에 있는 데이터를 다른 칸에 있는 데이터와 구분할 수 없는 경우 **scope 속성으로 머리글 방향을 명시합니다.** 다음 예시에서는 `scope='col'`을 명시하여 스크린 리더에 의해 'Phoenix'가 'Last Name'임을 알 수 있습니다. 

{% highlight html%}
<table>
  <caption>Teddy bear collectors:</caption>
  <tr>
    <th scope="col">Last Name</th>
    <th scope="col">First Name</th>
    <th scope="col">City</th>
  </tr>
  <tr>
    <td>Phoenix</td>
    <td>Imari</td>
    <td>Henry</td>
  </tr>
  <tr>
    <td>Zeki</td>
    <td>Rome</td>
    <td>Min</td>
  </tr>
  <td>
    <td>Apirka</td>
    <td>Kelly</td>
    <td>Brynn</td>
  </tr>
</table>
{% endhighlight %}

<br />

### 2-2. 머리글이 두 개인 경우 

머리글이 행과 열에 동시에 존재하면 데이터의 의미가 모호해지기 쉽습니다. 따라서 앞서 진행한 것처럼 th 태그와 scope 속성 처리를 모든 머리글에 적용해야 합니다. 아래 예시에서 td 태그에 있는 'Closed', 'Open'이라는 정보는 행과 열에 있는 머리글과 연관되어야 의미가 있습니다. 따라서 th 태그 사용과 동시에 첫번째 열 머리글에는 `scope='col'`, 첫번째 행 머리글에는 `scope='row'`를 적용합니다. 

{% highlight html%}
<table>
  <caption>Delivery slots:</caption>
  <tr>
    <td></td>
    <th scope="col">Monday</th>
    <th scope="col">Tuesday</th>
    <th scope="col">Wednesday</th>
    <th scope="col">Thursday</th>
    <th scope="col">Friday</th>
  </tr>
  <tr>
    <th scope="row">09:00 – 11:00</th>
    <td>Closed</td>
    <td>Open</td>
    <td>Open</td>
    <td>Closed</td>
    <td>Closed</td>
  </tr>
  <tr>
    <th scope="row">11:00 – 13:00</th>
    <td>Open</td>
    <td>Open</td>
    <td>Closed</td>
    <td>Closed</td>
    <td>Closed</td>
  </tr>
  […]
</table>
{% endhighlight %}

<br />

### 2-3. 머리글이 불규칙할 경우

머리글이 여러 칸을 확장할 수도 있고, 계층을 형성하는 경우도 있습니다. 이러한 상황에서 사용할 수 있는 table 마크업은 다음과 같습니다. 

1. `<colgroup>` : 열 그룹을 정의할 때 사용합니다. 해당 태그 내에서 명시한 span 속성값 만큼 `<col>`을 사용한 것과 동일한 역할을 합니다. 
2. `<thead>`, `<tbody>`, `<tfoot>` : 행 그룹을 정의할 때 사용합니다. thead, tfoot은 table 태그 내에서 한번만 사용할 수 있고 tbody는 여러 번 사용할 수 있습니다. 

#### 2-3-1. 머리글에 계층이 있는 경우 

아래 예시 코드 주석에 있는 각 숫자에 대응하는 부연설명을 하면 다음과 같습니다. 
1. 기본적으로 col 태그를 사용하고 여러 열을 차지할 경우 colgroup 태그를 사용합니다.
2. `scope='colgroup'`로 설정하면 모든 열 그룹과 연관됩니다. 즉, Mars는 Produced 값인 50,000과 Sold 값인 30,000과 연관되어 해당 데이터를 스크린 리더가 읽을 때 함께 읽힙니다. 
3. `scope='col'`로 설정하면 오직 대응되는 열만 연관됩니다. 즉, Produced는 50,000과 연관되어 해당 데이터를 스크린 리더가 읽을 때 함께 읽힙니다. 

{% highlight html%}
<table>
  <col>
  <colgroup span="2"></colgroup> <!-- 1 -->
  <colgroup span="2"></colgroup>
  <tr>
    <td rowspan="2"></td>
    <th colspan="2" scope="colgroup">Mars</th> <!-- 2 -->
    <th colspan="2" scope="colgroup">Venus</th>
  </tr>
  <tr>
    <th scope="col">Produced</th> <!-- 3 -->
    <th scope="col">Sold</th>
    <th scope="col">Produced</th>
    <th scope="col">Sold</th>
  </tr>
  <tr>
    <th scope="row">Teddy Bears</th>
    <td>50,000</td>
    <td>30,000</td>
    <td>100,000</td>
    <td>80,000</td>
  </tr>
  <tr>
    <th scope="row">Board Games</th>
    <td>10,000</td>
    <td>5,000</td>
    <td>12,000</td>
    <td>9,000</td>
  </tr>
</table>
{% endhighlight %}

<br />

#### 2-3-2. 머리글이 여러 개의 행과 열을 확장한 경우

아래 예시처럼 머리글이 행을 확장한 경우 데이터 칸의 개수와 합쳐진 행의 수가 동일해야 합니다. 행을 확장하는 머리글에는 `scope='rowgroup'`을 명시하고 그룹으로 묶인 행은 tbody 태그로 감쌉니다. (머리글이 표의 상단에 있을 경우 thead 태그, 하단에 있을 경우 tfoot 태그를 사용합니다.)

{% highlight html%}
<table>
  <caption>
    Poster availability
  </caption>
  <col>
  <col>
  <colgroup span="3"></colgroup>
  <thead>
    <tr>
      <th scope="col">Poster name</th>
      <th scope="col">Color</th>
      <th colspan="3" scope="colgroup">Sizes available</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" scope="rowgroup">Zodiac</th>
      <th scope="row">Full color</th>
      <td>A2</td>
      <td>A3</td>
      <td>A4</td>
    </tr>
    <tr>
      <th scope="row">Black and white</th>
      <td>A1</td>
      <td>A2</td>
      <td>A3</td>
    </tr>
    <tr>
      <th scope="row">Sepia</th>
      <td>A3</td>
      <td>A4</td>
      <td>A5</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <th rowspan="2" scope="rowgroup">Angels</th>
      <th scope="row">Black and white</th>
      <td>A1</td>
      <td>A3</td>
      <td>A4</td>
    </tr>
    <tr>
      <th scope="row">Sepia</th>
      <td>A2</td>
      <td>A3</td>
      <td>A5</td>
    </tr>
  </tbody>
</table>
{% endhighlight %}

<br />

### 2-4. 머리글 여러 계층으로 이루어진 경우

이미 위에서 머리글이 많고 복잡해질 경우 데이터와의 연관성을 분명하게 하기 위해 scope 속성을 사용했습니다. 여기서는 표의 구조가 한층 더 복잡해졌을 때 머리글의 id 값을 사용하는 방법을 다룹니다. 하지만 **가능하면 구조를 단순하게 만들기 위해 정보 구조를 재설계하는 것이 더 낫습니다.**

#### 2-4-1. 각 열마다 여러 개의 열 머리글(column headers)이 있는 표

<img src="https://github.com/iyu88/Algorithm/assets/31645195/6a678372-ad3e-4572-86ce-40d1af422b56" alt="각 열마다 여러 개의 열 머리글이 있는 표">
*출처: https://www.w3.org/WAI/tutorials/tables/multi-level/*
<br />

위의 사진을 보면 표를 반으로 나눴을 때 위쪽과 아랫쪽이 제공하고 있는 정보가 다릅니다. 각 데이터를 올바른 머리글과 연관짓기 위해 머리글(th)은 고유한 id 값을 가지고, 데이터(td)는 **headers 속성에 관련된 머리글의 id 값을 공백으로 구분하여 나열합니다.** 이 때, 비어있는 머리글은 `id='blank'`로 설정하여 스크린 리더가 해당 칸을 읽지 않도록 합니다.

{% highlight html%}
<td id="blank">&nbsp;</td>
<th id="co1" headers="blank">Example 1 Ltd</th>
<th id="co2" headers="blank">Example 2 Co</th>
[…]
<th id="c1" headers="blank">Contact</th>
[…]

<td headers="co1 c1">James Phillips</td>
<td headers="co2 c1">Marie Beauchamp</td>
[…]
{% endhighlight %}

<br />

#### 2-4-2. 각 데이터 칸에 관련된 세 개의 머리글이 있는 표

<img src="https://github.com/iyu88/Algorithm/assets/31645195/c0fd6588-a336-4014-a1ab-6bf44e157365" alt="각 데이터 칸에 관련된 세 개의 머리글이 있는 표">
*출처: https://www.w3.org/WAI/tutorials/tables/multi-level/*
<br />

머리글을 부제목으로 사용하는 경우 데이터를 명확하게 전달하기 위해 해당 머리글의 id 값도 headers 속성에 명시해야 합니다. 

{% highlight html%}
[…]
  <th id="par" colspan="5" scope="colgroup">Paris</th>
</tr>
<tr>
  <th id="pbed1">1 bedroom</th>
[…]
{% endhighlight %}

{% highlight html%}
[…]
<td headers="par pbed1 stud">11</td>
<td headers="par pbed1 apt"> 20</td>
[…]
{% endhighlight %}

<br />

### 2-5. Caption & Summary

#### 2-5-1. caption 태그로 표 구분하기

`<caption>`을 사용하여 표에 짧은 제목을 지을 수 있습니다. 대부분의 스크린 리더가 caption의 내용을 읽어주기 때문에 이를 table 태그의 첫번째 자식 요소로 사용하면 **사용자가 표 내용을 계속해서 탐색할지 결정하는 데에 도움을 줍니다.** 만약 'Tables Mode'를 사용하고 있다면, caption은 표를 찾는 데에 가장 중요한 요소가 됩니다.

{% highlight html%}
<table>
  <caption>Concerts</caption>
  <tr>
    <th>Date</th>
    <th>Event</th>
    <th>Venue</th>
  </tr>
  <tr>
    <td>12 Feb</td>
    <td>Waltz with Strauss</td>
    <td>Main Hall</td>
  </tr>
  […]
</table>
{% endhighlight %}

<br />

#### 2-5-2. caption 태그 내부에 요약을 포함하기

caption 태그에서 표가 다루고 있는 정보를 적고 자세한 설명은 요약하여 caption의 자식 요소로 배치합니다. 이 때, 요약 설명에는 caption에서 제공한 정보를 되풀이하면 안됩니다.

{% highlight html%}
<caption>Availability of holiday accommodation <br>
  <span>Column one has the location and size of accommodation, other columns show the type and number of properties available</span>
</caption>
{% endhighlight %}

<br />

#### 2-5-3. 요약을 제공하기 위해 aria-describedby 사용하기 

table 태그에 aria-describedby 속성을 사용하고 표에 대한 요약 설명이 있는 요소의 id 값을 명시하여 둘을 연관지을 수 있습니다. 해당 요소가 table 태그 주변에 있을 경우 스크린 리더를 사용하지 않는 사람들이 빠르게 발견할 수 있습니다. 

{% highlight html%}
<p id="tblDesc">Column one has the location and size of accommodation, other columns show the type and number of properties available.</p>
<table aria-describedby="tblDesc">
[…]
{% endhighlight %}

<br />

#### 2-5-4. 요약을 제공하기 위해 figure 태그 사용하기 

아래의 예시처럼 `<figure>`로 표를 감싸고 `<figcaption>`의 자식 요소로 표의 제목과 요약 설명을 위치시키는 방법입니다. table 태그에 aria-labelledby와 aria-describedby를 사용하여 각각 figcaption 태그, 요약 설명이 있는 요소와 연관 짓습니다. 

단, 이 방식은 스크린 리더를 사용할 때 다음과 같은 두 가지 단점이 있다는 것을 유념해야 합니다.
1. caption 태그를 사용하지 않아 스크린 리더가 'Tables Mode'에서 표를 찾을 수 없습니다. 
2. 표의 제목과 요약 설명이 스크린 리더에 의해 여러 번 읽힐 수 있습니다. 

{% highlight html%}
<figure>
  <figcaption>
    <strong id="paris-caption">Paris: Availability of holiday accommodation</strong><br>
    <span id="paris-summary">Column one has the location and size of accommodation, other columns show the type and number of properties available.</span>
  </figcaption>
  <table aria-labelledby="paris-caption" aria-describedby="paris-summary">
[…]
  </table>
</figure>
{% endhighlight %}

<br />

## 3. 폼 (Forms)

폼은 로그인, 결제 등 웹 페이지에서 사용자에게 상호작용을 제공합니다. 사용자는 간단하고 짧은 폼을 선호하기 때문에 되도록 꼭 필요한 것만 입력하도록 구성해야 합니다. 만약 불필요한 데이터를 요청한다면 사용자는 도중에 입력을 포기할 수 있습니다. 또한 각자의 속도로 폼을 입력할 수 있도록 시간 제한을 두지 않는 것이 좋습니다. 

### 3-1. 레이블 제어하기

텍스트 필드, 체크박스, 라디오 버튼 등 폼을 제어하는 모든 요소를 식별하기 위해 `<label>` 태그로 레이블을 제공합니다. 여기서는 레이블을 제공하는 방법에 대해 설명합니다. 

#### 3-1-1. 명시적으로 레이블 연관 짓기

가능하면 label 태그를 사용해서 폼 요소와 연관 짓습니다. 폼 요소의 id가 label의 for 속성값이 되어야 합니다.

{% highlight html%}
<label for="firstname">First name:</label>
<input type="text" name="firstname" id="firstname"><br>

<input type="checkbox" name="subscribe" id="subscribe">
<label for="subscribe">Subscribe to newsletter</label>
{% endhighlight %}

<br />

#### 3-1-2. 레이블 글씨 숨기기 

이미 렌더링된 요소로 폼 요소의 목적이 명확하게 전달된 경우 레이블을 시각적으로 숨겨도 좋습니다. 단, 스크린 리더나 다른 보조 기술의 지원을 받기 위해서 코드 상으로는 계속 존재해야 합니다. 

<br />

1️⃣ 레이블 요소 숨기기 

아래 예시에는 버튼에 이미 Search 라고 명시되어 있어 맥락을 충분히 파악할 수 있습니다. 따라서 중복을 피하기 위해 'visuallyhidden' 클래스를 추가하여 레이블을 렌더링하지 않습니다. 

{% highlight html%}
<!-- 
.visuallyhidden {
  border: 0;
  clip: rect(0 0 0 0);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
}
-->

<label for="search" class="visuallyhidden">Search: </label>
<input type="text" name="search" id="search">
<button type="submit">Search</button>
{% endhighlight %}

<br />

2️⃣ aria-label / aria-labelledby 속성 사용

aria-label과 aria-labelledby 속성은 스크린 리더나 보조 기술의 지원을 받을 수 있지만 사용자에게 렌더링되지 않습니다. 따라서 **폼 요소의 레이블로 삼을만한 콘텐츠가 주변에 있을 때만 사용합니다.** aria-labelledby를 사용할 때는 레이블 텍스트를 가지고 있는 요소의 id 속성값을 aria-labelledby로 설정합니다. 

{% highlight html%}
<!-- aria-label -->
<input type="text" name="search" aria-label="Search">
<button type="submit">Search</button>

<!-- aria-labelledby -->
<input type="text" name="search" aria-labelledby="searchbutton">
<button id="searchbutton" type="submit">Search</button>
{% endhighlight %}

<br />

3️⃣ title 속성 사용

title 속성을 제공하면 마우스로 호버했을 때 title 내용이 툴팁으로 나타납니다. 하지만 몇몇 스크린 리더나 보조 기술이 title 속성을 label 태그의 대체제로 해석하지 못하기 때문에 추천하는 방법은 아닙니다. 

{% highlight html%}
<input title="Search" type="text" name="search">
<button type="submit">Search</button>
{% endhighlight %}

<br />

#### 3-1-3. 암시적으로 레이블 연관 짓기

레이블을 적용해야 하는 요소의 id를 모르거나 요소에 id를 설정하지 않은 경우가 있을 수 있습니다. 이 때는 label 태그를 레이블 텍스트와 폼 요소의 컨테이너로 사용합니다. 레이블과 폼 요소를 암시적으로 연관 지어 스크린 리더와 보조 기술의 지원을 받을 수 있습니다. 

{% highlight html%}
<label>
  First name:
  <input type="text" name="firstname">
</label>
<br>
<label>
  <input type="checkbox" name="subscribe">
  Subscribe to newsletter
</label>
{% endhighlight %}

<br />

#### 3-1-4. 버튼 레이블링

어떤 요소를 버튼으로 사용하는지에 따라 두 가지 경우로 나눌 수 있습니다. 
1. button 태그 : 레이블 텍스트를 자식 요소에 넣어서 사용합니다. 
2. input 태그 : 레이블 텍스트를 value 속성값으로 사용합니다. 

{% highlight html%}
<!-- button 태그 -->
<button type="submit">Submit</button>
<button type="button">Cancel</button>

<!-- input 태그 -->
<input type="submit" value="Submit">
<input type="button" value="Cancel">
{% endhighlight %}

<br />

#### 3-1-5. 레이블 텍스트의 위치 

레이블은 예측 가능하고 이해할 수 있는 위치에 존재하는 것이 중요합니다. 왼쪽에서 오른쪽으로 읽는 언어는 체크박스나 라디오 버튼의 오른쪽, 필드의 읜쪽에 레이블이 있는 것이 익숙합니다. 레이블이 필드 상단에 있을 경우 저시력자나 모바일 사용자가 수평 스크롤을 하지 않도록 도와줍니다. 중요한 것은 **레이블과 필드 요소가 가까워야 하고 시각적으로 명확한 관계를 유지해야 한다는 것입니다.** 

<br />

### 3-2. 제어 요소를 그룹으로 묶기

관련 있는 폼 요소를 그룹으로 묶는 것은 사용자가 폼을 더 잘 이해할 수 있게 만듭니다. 전체 폼을 한번에 받아들이기보다 작은 단위의 그룹에 집중하게 합니다. `<fieldset>`, `<legend>`를 사용하여 시각적으로, 코드 상으로 묶어줄 수 있습니다. 또한 `<optgroup>`으로 관련 있는 `<select>`들을 묶어줄 수 있습니다. 

#### 3-2-1. fieldset 태그로 관련 있는 제어 요소 연관 짓기 

fieldset 태그로 관련 있는 요소들을 묶어줄 수 있고, legend 태그는 해당 그룹의 제목 역할을 합니다. 아래 예시에는 동일한 필드 구조를 가진 두 개의 폼이 있는데 이러한 상황에서 fieldset 태그가 각각을 구별하는 데에 도움을 줍니다. 몇몇 스크린 리더는 모든 폼 요소를 읽을 때마다 legend를 읽어주기도 하지만 아예 읽지 않는 경우도 있습니다. 

따라서 legend와 label을 작성할 때 다음 두 가지 사항을 고려해야 합니다. 
1. 매번 레이블과 함께 legend를 읽을 수 있기 때문에 legend는 가능한 짧게 작성합니다.
2. legend를 아예 읽지 않을 수 있기 때문에 각 레이블에 충분한 설명을 작성합니다. 

{% highlight html%}
<fieldset>
	<legend>Shipping Address:</legend>
	<div>
		<label for="shipping_name">
      <span class="visuallyhidden">Shipping </span>Name:
    </label><br>
		<input type="text" name="shipping_name" id="shipping_name">
	</div>
  <div>
    <label for="shipping_street">Street:</label><br>
    <input type="text" name="shipping_street" id="shipping_street">
  </div>
	[…]
</fieldset>
<fieldset>
	<legend>Billing Address:</legend>
	<div>
		<label for="billing_name">
      <span class="visuallyhidden">Billing </span>Name:
    </label><br>
		<input type="text" name="billing_name" id="billing_name">
	</div>
  <div>
    <label for="billing_street">Street:</label><br>
    <input type="text" name="billing_street" id="billing_street">
  </div>
	[…]
</fieldset>
{% endhighlight %}

<br />

#### 3-2-2. WAI-ARIA로 관련 있는 제어 요소 연관 짓기 

role 속성에는 fieldset, legend와 비슷한 역할을 하는 `role='group'`이 있습니다. 이를 aria-labelledby와 함께 사용하면 그룹의 레이블도 지정할 수 있습니다. 아래 예시에서 최상위 div 태그에 두 속성이 모두 적용된 것을 확인할 수 있습니다. 

{% highlight html%}
<div role="group" aria-labelledby="shipping_head">
	<div id="shipping_head">Shipping Address:</div>
	<div>
		<label for="shipping_name">
      <span class="visuallyhidden">Shipping </span>Name:
    </label><br>
		<input type="text" name="shipping_name" id="shipping_name">
	</div>
	[…]
</div>
<div role="group" aria-labelledby="billing_head">
	<div id="billing_head">Billing Address:</div>
	<div>
		<label for="billing_name">
      <span class="visuallyhidden">Billing </span>Name:
    </label><br>
		<input type="text" name="billing_name" id="billing_name">
	</div>
	[…]
</div>
{% endhighlight %}

<br />

#### 3-2-3. select 태그에서 관련 있는 아이템 묶어주기

select 내부에서 관련 있는 option 태그를 묶어줄 때 optgroup 태그를 사용할 수 있습니다. optgroup의 label 속성은 해당 그룹의 레이블을 나타낼 때 사용합니다. 같은 종류로 분류할 수 있는 옵션이 매우 많을 경우 유용하게 쓸 수 있습니다.

{% highlight html%}
<select>
	<optgroup label="8.01 Physics I: Classical Mechanics">
		<option value="8.01.1">Lecture 01: Powers of Ten</option>
		<option value="8.01.2">Lecture 02: 1D Kinematics</option>
		<option value="8.01.3">Lecture 03: Vectors</option>
	</optgroup>
	<optgroup label="8.02 Physics II: Electricity and Magnestism">
		<option value="8.02.1">Lecture 01: What holds our world together?</option>
		[…]
	</optgroup>
	[…]
</select>
{% endhighlight %}

<br />

### 3-3. 폼 안내사항 

폼을 완성하기 위해 필요한 안내사항을 제공하여 사용자의 이해도를 높일 수 있습니다. 필수/선택 입력 필드에 대해서 알려주고 상황에 따라 데이터 형식을 알려주기도 합니다. 스크린 리더는 `<form>` 안에 있는 콘텐츠를 처리할 때 'Forms Mode' 로 변경하곤 하는데, 보통 input, select, textarea, legend, label 태그만 읽어줍니다. 따라서 해당 태그들과 적절한 속성을 활용하여 폼 안내사항 또한 읽을 수 있게 만드는 것이 중요합니다. 

#### 3-3-1. 전반적인 안내사항 

전체 폼에 적용할 수 있는 전반적인 안내사항을 제공합니다. 필수/선택 입력 필드, 데이터 형식, 시간 제한 등을 알려줍니다. 'Forms Mode'로 변경하기 전에 **스크린 리더가 읽을 수 있도록 form 태그 앞에 위치시킵니다.**

<br />

#### 3-3-2. 인라인 안내사항 

폼 제어 요소의 레이블 안에 관련된 안내사항을 제공하는 것도 중요합니다. 레이블 텍스트 안에 필드의 필수 입력 여부와 데이터 형식을 표시합니다. 

<br />

1️⃣ label 태그 안에 안내사항을 제공하기 

이 방식은 여러 웹 브라우저와 보조 기술의 지원을 받을 수 있습니다. 아래의 예시에서는 'Expiration date' 레이블 우측에 날짜 형식을 알려주고 있습니다. 

{% highlight html%}
<label for="expire">Expiration date (MM/YYYY): </label>
<input type="text" name="expire" id="expire">
{% endhighlight %}

<br />

2️⃣ label 태그 밖에 안내사항을 제공하기 

반대로 이 방식은 브라우저와 보조 기술의 지원을 받을 수 없지만 안내사항의 위치를 더 유연하게 설정할 수 있습니다. aria-labelledby 속성과 aria-describedby 속성을 이용할 수 있습니다. 이 둘의 차이는 **스크린 리더가 aria-describedby에 연관된 요소를 label과 다른 정보들을 모두 읽은 후 맨 마지막에 읽어준다는 것입니다.**  

{% highlight html%}
<!-- aria-labelledby -->
<label id="expLabel" for="expire">Expiration date:</label>
<span>
	<input type="text" name="expire" id="expire" aria-labelledby="expLabel expDesc">
	<span id="expDesc">MM/YYYY</span>
</span>

<!-- aria-describedby -->
<label id="expLabel" for="expire">Expiration date:</label>
<span>
	<input type="text" name="expire" id="expire" aria-labelledby="expLabel" aria-describedby="expDesc">
	<span id="expDesc">MM/YYYY</span>
</span>
{% endhighlight %}

<br />

#### 3-3-3. 플레이스홀더 

플레이스홀더에 안내사항이 있을 경우 사용자는 자신이 제대로 응답했는지 확인하기 어렵습니다. 플레이스홀더가 사용자에게 중요한 정보를 전달하긴 하지만 **플레이스홀더는 레이블의 대체제가 아닙니다.** 스크린 리더와 같은 보조 기술은 플레이스홀더를 레이블로 취급하지 않습니다. 따라서 플레이스홀더와 별개로 레이블을 렌더링하는 것이 좋습니다. 

<br />

### 3-4. 입력값 검증하기 

안내사항을 제공하는 것과 더불어 사용자의 입력값을 검증하여 실수를 방지합니다. HTML5는 이메일, 날짜 등 일반적인 input 타입을 검증하기 위한 내장 기능을 가지고 있습니다. 그 외에도 직접 검증 로직을 만들었다면 검증 결과를 사용자에게 알려줍니다. 

#### 3-4-1. 필수적인 input 검증하기 

label 태그에 'required' 라고 명시하기도 하고 input 태그에 required 속성을 사용하기도 합니다. label에 'required' 라고 명시하는 것은 required 속성을 지원하지 않는 구형 브라우저와 보조 기술을 사용하지 않는 사람들을 위함입니다. 비슷하게 required 속성을 사용하면 `aria-required='true'`를 적용한 것과 동일하지만 이를 지원하지 않는 브라우저에 대응하기 위해 중복으로 사용할 수 있습니다. 

{% highlight html%}
<label for="name">Name (required): </label>
<input type="text" name="name" id="name" required aria-required="true">
{% endhighlight %}

<br />

#### 3-4-2. 규칙있는 input 검증하기 

HTML5의 pattern 속성은 입력값을 정규표현식으로 검증할 수 있게 해줍니다. 전화번호, 우편번호 등 데이터에 특정한 규칙이 있을 때 유용합니다. 아래는 독일 차량 번호 검증을 위해 pattern 속성에 정규표현식을 사용한 예시입니다.

{% highlight html%}
<div>
	<input type="text" id="license"
		pattern="[A-ZÖÄÜ]{1,3} [A-Z]{2,4} [0-9]{1,4}"
	>
</div>
{% endhighlight %}

<br />

그 외에도 클라이언트/서버 검증 모두 진행하기, 사용자에게 입력한 값을 확인시키기, 실행 취소 기능 제공하기 등이 있습니다.  

<br />

### 3-5. 사용자 알림

성공 여부와 상관없이 사용자에게 폼 제출 결과에 대한 피드백을 제공합니다. 폼 제어 요소 근처에 인라인 피드백일 수도 있고, 폼 제출 이후에 전체 피드백일 수도 있지만 알림은 간결하고 명확해야 합니다. 특히 오류 메세지는 이해하기 쉽고 해결 방법에 대한 간단한 안내사항을 제공해야 합니다. 

#### 3-5-1. 전체 피드백 

폼이 제출되면 성공했는지, 오류가 발생했는지에 대해 사용자에게 알려주는 것이 중요합니다. 

<br />

1️⃣ heading 사용하기 

h1, h2 등으로 웹 페이지의 제목을 통해 알려주는 방법이 가장 일반적입니다. 이 때 오류가 발생했다면, '오류'라는 단어를 분명하게 사용하고 오류 개수도 알려주면 좋습니다. 

{% highlight html%}
<!-- 실패 -->
<h1>3 Errors – Billing Address</h1>
<!-- 성공 -->
<h1>Thank you for submitting your order.</h1>
{% endhighlight %}

<br />

2️⃣ title 태그 사용하기 

title 태그가 가장 먼저 읽히기 때문에 스크린 리더 사용자들은 바로 피드백을 받을 수 있습니다. 따라서 heading 요소가 콘텐츠 깊은 곳에 위치해 있을 경우 유용합니다. 

{% highlight html%}
<!-- 실패 -->
<title>3 Errors – Billing Address</title>
<!-- 성공 -->
<title>Thank you for submitting your order.</title>
{% endhighlight %}

<br />

3️⃣ 다이얼로그 사용하기 

사용자가 놓친 부분을 알려주는 가장 일반적인 방법입니다. 다음은 브라우저 내장 alert()를 사용한 예시입니다. 

{% highlight html%}
<button type="button" id="alertconfirm">Save</button>
{% endhighlight %}

{% highlight js%}
document.getElementById('alertconfirm')
	.addEventListener('click', function() {
		/* [… code saving data …] */
		alert('Thanks for submitting the form!');
	});
{% endhighlight %}

<br />

4️⃣ 오류 나열하기 

오류가 발생하면 form 태그 앞, 페이지 상단에 이를 나열합니다. heading 태그를 사용하여 쉽게 알아차릴 수 있게 합니다. 
각각의 오류 메세지는 
1. 오류가 발생한 폼 제어 요소의 레이블을 알려줍니다. 
2. 누구나 쉽게 이해할 수 있도록 오류를 간결하게 설명합니다. 
3. 오류 수정 방법을 알려주고, 입력 포맷을 다시 안내합니다.
4. 빠른 접근을 위해 폼 제어 요소에 대한 페이지 내 링크를 제공합니다.

또한 이러한 오류 목록을 감싸는 컨테이너에 `role='alert'` 속성을 명시해야 보조 기술의 지원을 받을 수 있습니다. 

{% highlight html%}
<div role="alert">
  <h4>There are 2 errors in this form</h4>
  <ul>
  	<li>
  		<a href="#firstname" id="firstname_error">
  			The First name field is empty; it is a required field and must be filled in.
  		</a>
  	</li>
  	<li>
  		<a href="#birthdate" id="birthdate_error">
  			The Date field is in the wrong format; it should be similar to 17/09/2013 (use a / to separate day, month, and year).
  		</a>
  	</li>
  </ul>
</div>
{% endhighlight %}

오류가 발생한 필드는 aria-describedby 속성으로 오류 메세지와 연결해줄 수 있습니다. 

{% highlight html%}
<label for="firstname">First Name:</label>
<input type="text" id="firstname" aria-describedby="firstname_error">
{% endhighlight %}

<br />

#### 3-5-2. 인라인 피드백 

제어 요소 주위에 피드백이 있을 경우 사용자에게 더 도움이 될 수 있습니다. 이는 오류가 발생했을 때 뿐만 아니라 정상적으로 입력했을 경우도 해당합니다. 통상적으로 메세지와 함께 시각적인 단서(아이콘)도 제공합니다. 

<br />

1️⃣ 제출한 뒤 피드백 

사용자가 폼을 완성할 수 있도록 적절한 성공, 오류 메세지를 표시합니다. **만약 오류가 발생했을 경우, 해당 input 태그를 포커스합니다.** 

<br />

2️⃣ 입력 중 피드백 

입력 중 즉각적인 피드백은 큰 도움을 줍니다. 단, 이러한 형태의 피드백이 모든 상황에서 옳은 것은 아니며 클라이언트에서의 오류 검증을 위한 코드가 필요합니다. 
아래 예시에서 span 태그 안에 `aria-live='polite'`를 사용하여 [live region](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/ARIA_Live_Regions)임을 스크린 리더에게 알려줍니다. 'polite'라는 값은 메세지의 중요성을 떨어트려 해당 메세지를 읽기 위해 스크린 리더가 현재 작업을 중단하지 않도록 합니다. 이로 인해 사용자가 키를 입력할 때마다 매번 읽는 것이 아니라 입력을 멈췄을 때만 한번 읽게 됩니다.  

<br />

- 메세지 피드백

{% highlight html%}
<div>
	<label for="username">Username:</label>
	<input type="text" name="username" id="username">
	<span id="username_feedback" aria-live="polite"></span>
</div>
{% endhighlight %}

{% highlight js%}
document.getElementById('username').addEventListener('keyup', function(){
	function setError(el, msg) {
		el.parentNode.querySelector('strong').innerHTML = "Error:";
		el.parentNode.className='error';
		el.parentNode.querySelector('span').innerHTML = msg;
	}
	function setSuccess(el) {
		el.parentNode.querySelector('strong').innerHTML = "OK:";
		el.parentNode.className='success';
		el.parentNode.querySelector('span').innerHTML = "&check;";
	}
	var val = this.value;
	if (val !== "") {
		if (taken_usernames.includes(val.trim())) {
			setError(this, '&cross; Sorry, this username is taken.');
		} else {
			setSuccess(this, '&check; You can use this username.');
		}
	} else {
		document.getElementById('username_feedback').innerHTML = '';
		document.getElementById('username_feedback')
			.parentNode.className = '';
		document.querySelector('[for="username"] strong').innerHTML = '';
	}
});
{% endhighlight %}

<br />

- 규모에 대한 피드백 

정상과 오류를 이분적으로 나누는 것이 아니라 비밀번호의 보안 수준을 표시하는 것처럼 단계별로 피드백을 주는 경우도 있습니다. 이럴 때는 색상, 바로미터, 단계를 알려주는 레이블 등이 필요합니다. 

<br />

- 포커스가 변경되었을 때 피드백 

날짜처럼 특정한 포맷으로 값이 입력되어야 하는 경우 사용자가 입력하는 도중에는 해당 포맷을 충족하지 못하기 때문에 계속 오류 메세지가 뜨게 됩니다. 이처럼 입력 중에 피드백을 주는 것이 의미 없는 경우도 있습니다. 이러한 경우에는 탭키를 누르거나 마우스로 다른 필드를 클릭해서 포커스가 변경되었을 때 입력을 확인하고 피드백을 줍니다. 

아래 예시에서 span 태그 안에 `aria-live='assertive'`를 사용하여 live region임을 스크린 리더에게 알려줍니다. 'assertive'라는 값은 메세지의 중요성을 강조하여 해당 메세지를 읽기 위해 스크린 리더가 현재 작업을 중단하도록 합니다. 이로 인해 포커스를 받는 다음 요소를 사용자에게 알려주기 전에 메세지를 읽어줍니다. 

{% highlight html%}
<div>
	<label for="expire"><strong></strong> Expiry date:</label>
	<input type="text" name="expire" id="expire" value="03.2015" aria-describedby="expDesc3">
	<span id="expDesc3" aria-live="assertive"></span>
</div>
{% endhighlight %}

<br />

### 3-6. 다중 페이지 폼 

폼이 길 경우 논리적인 단계로 구성된 여러 개의 작은 폼으로 나누는 것이 좋습니다. 이는 전체 폼을 이해하기 쉽게 만들어 주며, 특히 컴퓨터 사용 경험이 적거나 인지 장애가 있는 사람들에게 도움이 됩니다. 단계적 폼에 적용되어야 하는 기본 원칙은 다음과 같습니다. 
1. 모든 페이지에 '전반적인 안내사항'을 반복합니다. 
2. 제어 요소의 논리적인 흐름에 따라 폼을 나눕니다.
3. 선택적인 요소는 건너뛸 수 있게 하고 이를 쉽게 인지할 수 있게 만듭니다. 
4. 가능하면 시간 제한을 두지 않습니다. 시간 제한이 꼭 필요하다면, 사용자가 시간을 조절하거나 연장할 수 있는 기능을 제공합니다. 

#### 3-6-1. 단계 표시하기 

가능하면 첫번째 단계에서 이후 몇 단계가 더 이어지는지 설명합니다. 또한 각 단계에서 사용자가 현재 있는 단계를 알려줘야 합니다. 

<br />

1️⃣ title 태그 사용하기 

스크린 리더가 가장 먼저 읽는 태그이기 때문에 현재 몇번째 단계에 있는지를 가장 먼저 적습니다. 

{% highlight html%}
<title>Step 2 of 4: Shipping Address – Complete Purchase – Galactic Teddy Bears Shop</title>
{% endhighlight %}

<br />

2️⃣ heading 사용하기 

웹 페이지 제목에 단계를 표시합니다. 

{% highlight html%}
<h1>Shipping Address (Step 2 of 4)</h1>
{% endhighlight %}

<br />

3️⃣ HTML5 progress 태그 사용하기 

사용자 입력에 따라 남은 단계의 수가 결정되는 상황에서 사용할 수 있습니다.

{% highlight html%}
Survey <progress max="7" value="1">(Step 1 of circa 7)</progress><br>

Survey <progress max="7" value="2">(Step 2 of circa 7)</progress><br>

[…]

Survey <progress max="7" value="6">(Step 6 of circa 7)</progress><br>

Survey <progress max="7" value="7">(Finished)</progress>
{% endhighlight %}

progress 태그에는 기본적으로 애니메이션이 적용되어 있는데, 이는 [움직이는 콘텐츠에 대한 접근성](https://www.w3.org/WAI/test-evaluate/preliminary/#moving)을 준수하지 못한다는 문제가 있습니다. 아래와 같은 CSS를 사용하여 애니메이션을 멈출 수 있습니다.

{% highlight css%}
/* Microsoft IE */
progress {
  color: #036;
}

/* Apple Safari and Google Chrome */
progress::-webkit-progress-bar {
	background-color: #036;
}

/* Mozilla Firefox */
progress::-moz-progress-bar {
	background-color: #036;
}
{% endhighlight %}

<br />

4️⃣ 각 단계를 표시하기 

아래의 예시는 `<ol>`과 `<li>`로 각 단계를 나타내고 숨겨진 텍스트를 사용하여 현재 단계와 완료된 단계를 표시합니다. 가능하면 완료된 단계에 대한 링크를 제공하여 사용자가 폼을 다시 확인할 수 있게 합니다. 해당 링크로 이동했을 때는 현재 단계에서 입력한 값을 유지해줍니다.

{% highlight html%}
<div class="tlwrapper">
	<ol class="timeline">
		<li class="timeline-past">
				<span class="visuallyhidden">Completed: </span>
				<a href="billing.html">Billing Address</a>
		</li>
		<li class="timeline-current">
			<span class="visuallyhidden">Current: </span>
			<span>Shipping Address</span>
		</li>
		<li><span>Review Order</span></li>
		<li><span>Payment</span></li>
		<li><span>Finish Purchase</span></li>
	</ol>
</div>
{% endhighlight %}

<br />

## 4. 캐러셀 (Carousels)

W3C에서 제공하는 캐러셀 예시는 [여기](https://www.w3.org/WAI/tutorials/carousels/working-example/)에서 확인할 수 있습니다. 캐러셀에서 준수해야 하는 접근성은 다음과 같습니다. 

1. 움직이는 것을 멈출 수 있어야 합니다. 계속 움직이면 너무 산만할 수 있고 글자를 읽기 힘들게 만듭니다.
2. 네비게이션을 포함한 모든 기능은 키보드로 동작해야 합니다.
3. 아이템이 변경되는 것은 스크린 리더 사용자를 포함한 모든 사용자가 알 수 있어야 합니다. 
4. 포커스 되었을 때 키보드 위치는 합리적이고 이해하기 쉬운 방식으로 관리됩니다. 

### 4-1. 캐러셀 구조

#### 4-1-1. 일반적인 구조 

ul 태그를 사용하여 가장 잘 나타낼 수 있습니다. 맥락에 따라 li 태그 안에 필요한 요소를 사용할 수 있습니다. 아래 예시에서 aria-labelledby 속성을 가지고 있는 `<section>` 태그처럼 모든 사용자가 쉽게 찾을 수 있도록 레이블을 가진 영역 안에 있어야 합니다.  

{% highlight html%}
<section class="carousel" aria-labelledby="carouselheading">
  <h3 id="carouselheading" class="visuallyhidden">Recent news</h3>
  <ul>
    <li class="slide">…</li>
    <li class="slide">…</li>
    <li class="slide">…</li>
    …
  </ul>
</section>
{% endhighlight %}

<br />

#### 4-1-2. 캐러셀 아이템

캐러셀은 종종 이미지를 보여주기 위한 갤러리로 사용되기도 합니다. 하지만 보다 복잡하게 아티클이나 페이지 전체를 아이템으로 사용하기도 합니다. 모든 상황에서 콘텐츠의 의미를 명확하게 전달하기 위해 적절한 마크업을 사용해야 합니다.

{% highlight html%}
<!-- 아이템으로 이미지를 사용하는 경우 -->
<li class="slide">
  <img src="teddy1.jpg" alt="Space Teddy">
</li>
{% endhighlight %}

{% highlight html%}
<!-- 아이템으로 아티클을 사용하는 경우 -->
<li class="slide" style="background-image: url('teddy1.jpg');">
  <article>
    <h4>Space Teddy production reaches all-time high</h4>
    <p>Teddies in Space Inc. has released outstanding numbers for the last solar year. The production of Space Teddies increased by 17%. The new version, scheduled to be released in a few months, will likely be the biggest Space Teddy release ever.</p>
    …
  </article>
</li>
{% endhighlight %}

<br />

### 4-2. 기능

캐러셀 아이템을 선택하고, 아이템이 변경되었다는 것을 사용자에게 알려줍니다.

#### 4-2-1. 이전-다음 버튼 제공

아이템 간 이동할 수 있도록 버튼을 제공해야 합니다. button 태그를 사용하여 보조 기술의 지원을 받을 수 있도록 합니다.

<br />

#### 4-2-2. 현재 아이템 알려주기 

aria-live 속성을 사용하여 스크린 리더 사용자에게 현재 어떤 아이템이 보여지고 있는지 알려줍니다. 또한 이전-다음 버튼이 눌렸을 때 포커스가 변경되지 않게 합니다. 그렇지 않으면 캐러셀을 조종하는 것이 불편할 수 있습니다. 

{% highlight js%}
var liveregion = document.createElement('div');
liveregion.setAttribute('aria-live', 'polite');
liveregion.setAttribute('aria-atomic', 'true');
liveregion.setAttribute('class', 'liveregion visuallyhidden');
carousel.appendChild(liveregion);

if (announceItem) {
  carousel.querySelector('.liveregion').textContent = 'Item ' + (new_current + 1) + ' of ' + slides.length;
}
{% endhighlight %}

<br />

#### 4-2-3. 네비게이션 버튼 추가 

각 캐러셀 아이템에 대응하는 버튼을 표시하고 현재 아이템에 해당하는 버튼을 강조합니다. 이는 사용자가 캐러셀 콘텐츠에 대해 이해할 수 있게 해주고 바로바로 다른 아이템으로 이동할 수 있게 해줍니다. 다음 예시에서도 버튼들이 캐러셀 아이템 개수만큼 존재하고, 현재 아이템에 대응되는 버튼은 시각적으로 구분되는 스타일을 가지고 있을 뿐만 아니라 스크린 리더를 위한 숨겨진 텍스트(`(Current Slide)`)도 가지고 있습니다. 

{% highlight html%}
<ul class="slidenav">
  <li>
    <button class="current" data-slide="0">
      <span class="visuallyhidden">News</span> 1
      <span class="visuallyhidden">(Current Slide)</span> 
    </button>
  </li>
  <li>
    <button data-slide="1">
      <span class="visuallyhidden">News</span> 2
    </button>
  </li>
  <li>
    <button data-slide="2">
      <span class="visuallyhidden">News</span> 3
    </button>
  </li>
</ul>
{% endhighlight %}

<br />

#### 4-2-4. 선택된 캐러셀 아이템을 포커싱

사용자가 네비게이션 버튼을 사용했을 때 사용자의 포커스는 선택된 아이템에 가 있어야 합니다. 이는 키보드와 보조 기술 사용자가 상호작용을 더 쉽게 하도록 도와줍니다. 가령, (일반적인 구조로 작성했다면) li 태그에 포커스가 가야 합니다. 단, li 태그는 포커스를 받을 수 없기 때문에 tabindex를 -1로 설정하여 자바스크립트를 통해 포커스 받을 수 있게 합니다. 

<br />

### 4-3. 애니메이션

캐러셀에 애니메이션을 적용하지만, 콘텐츠를 읽을 시간이 필요한 사람을 위해 일시정지가 필요합니다. 

#### 4-3-1. 재생/정지 버튼 

애니메이션을 제어할 수 있는 버튼을 제공합니다. **버튼 요소 자체를 변경하는 것은 키보드 포커스를 잃을 수 있기 때문에 value, 속성값만 변경하는 것이 좋습니다.** 

{% highlight html%}
<!-- 재생 상태일 때 -->
<button data-action="stop"><span class="visuallyhidden">Stop Animation </span>￭</button>
<!-- 정지 상태일 때 -->
<button data-action="start"><span class="visuallyhidden">Start Animation </span>▶</button>
{% endhighlight %}

<br />

#### 4-3-2. 마우스 호버와 키보드 포커스 시 일시정지 

캐러셀 아이템이 마우스 호버된 상태이거나 키보드 포커스를 받고 있는 경우 애니메이션을 일시정지합니다. 이는 콘텐츠를 전부 읽지 못한 사용자에게 시간을 주고, 아이템 링크를 클릭하고 싶은 사람에게 유용합니다. 

<br />

#### 4-3-3. 보조 기술로부터 트랜지션 중인 요소를 숨기기 

트랜지션이 진행 중일 때 현재 아이템과 다음 아이템이 함께 보이는 시점이 있습니다. 이는 보조 기술에 의해 두 아이템이 모두 이용 가능하다고 해석될 수 있기 때문에 스크린 리더 사용자에게 혼란을 줄 수 있습니다. 다음 스크립트 예제에서는 아이템이 활성화되면 화면에 나타나도록 하는 'in-transition' 클래스를 추가하면서 보조 기술로부터 아이템을 숨기기 위해 `aria-hidden='true'`로 설정합니다. 트랜지션이 끝나면 해당 속성을 제거합니다. 

{% highlight js%}
slides[new_next].className = 'next slide'
  + ((transition == 'next') ? ' in-transition' : '');
slides[new_next].setAttribute('aria-hidden', 'true');

slides[new_prev].className = 'prev slide'
  + ((transition == 'prev') ? ' in-transition' : '');
slides[new_prev].setAttribute('aria-hidden', 'true');

slides[new_current].className = 'current slide';
slides[new_current].removeAttribute('aria-hidden');
{% endhighlight %}

<br />

### 4-4. 스타일링

버튼과 링크를 적절한 크기로 제공하고, 주위에 충분한 여백을 줍니다. 이는 운동 장애를 가지고 있는 사람, 모바일 기기와 같은 터치 스크린을 사용하는 사람에게 도움을 줍니다. 인라인이 아닌 버튼, 링크는 최소 44x44 픽셀이어야 합니다. 

#### 4-4-1. 색상 대조 
배경과 전경에 대해서 색상이 충분히 대조되어야 하지만 이미지 위에 버튼, 링크가 위치하면 적절한 값을 찾기 어려울 수 있습니다. 이 때, 버튼, 링크의 배경 값을 (반)투명하게 설정하면 사용된 이미지와 상관없이 대조를 유지할 수 있습니다. 

<br />

#### 4-4-2. 버튼 상태 표시

캐러셀에 네비게이션 버튼이 추가되어도 대체적으로 크기가 작기 때문에 버튼의 상태를 사용자가 알 수 있도록 색상과 모양으로 알려주는 것이 좋습니다. 

<br />


## 5. 맺으며

저번 글보다 이번 글에서 다룬 내용의 종류는 더 적었지만 파악해야 할 부분은 훨씬 많았던 것 같습니다. 특히 표와 폼은 개발할 때 매우 다양한 형태로 활용될 수 있다보니 일어날 수 있는 모든 경우에 대한 가이드를 제시하고 있어서 더 그렇게 느꼈던 것 같습니다. 여기서는 특정한 컴포넌트, 특정한 상황에 대해서 다루고 있지만 이러한 내용으로부터 더욱 일반적인 원칙들을 도출하여 개발할 때 접근성을 높일 수 있도록 해야겠습니다. 공식 문서에는 본 시리즈에서 다룬 내용보다 접근성과 관련된 정보들이 훨씬 더 많기 때문에 한번쯤 들어가서 관심 있는 내용을 읽어보는 것도 좋을 것 같습니다.


[참고]

- [Tables Tutorial](https://www.w3.org/WAI/tutorials/tables/)

- [Forms Tutorial](https://www.w3.org/WAI/tutorials/forms/)

- [Carousels Tutorial](https://www.w3.org/WAI/tutorials/carousels/)