---
layout: post
title: "[A11y] WAI 문서로 접근성 이해하기 (1/2)"
date: 2023-12-24 00:00:00
author: Hyunbin Lee
categories: A11y
tags: Accessibility A11y
cover: "/assets/instacode.png"
---

# WAI 문서로 접근성 이해하기 (1/2)

> ### 💡 W3C에서 출판한 기술 사양을 읽어보며 접근성을 높일 수 있는 개발 요소들을 알아봅니다.

## 1. 웹 접근성이란

웹 접근성이란 노인, 장애인, 환자를 포함한 누구나 웹에 접근하여 웹 페이지를 탐색하거나 상호작용할 수 있게 지원하는 것을 말합니다. 

웹 접근성을 지원하기 위한 방법에는 대표적으로 WAI-ARIA(Web Accessibility Initiative – Accessible Rich Internet Applications)가 있습니다. 이는 웹 표준을 개발하는 조직인 W3C(World Wide Web Consortium)가 출판한 기술 사양으로 접근성을 증진하는 방법이 정리되어 있습니다. 

이 글에서는 [Easy Checks](https://www.w3.org/WAI/test-evaluate/preliminary/) 와 [Tutorials](https://www.w3.org/WAI/tutorials/)의 Page Structure, Menus, Images 파트에 있는 내용을 다룹니다. 

## 2. Easy Checks

대부분의 사용자들은 별도로 접근성을 지원하지 않아도 화면에 나타난 글자와 이미지, 스피커에서 나오는 소리, 마우스와 키보드를 사용하여 자유롭게 웹을 탐색할 수 있습니다. 하지만 시각적, 청각적 접근성을 필요로 하는 사람들을 위해서 시각 콘텐츠는 듣기 쉽게, 청각 콘텐츠는 보기 쉽게 제작되어야 합니다. 또한 적절한 시맨틱 태그를 사용하여 웹 페이지의 구조를 전달할 수 있어야 하며 키보드 접근성도 고려해야 합니다. 모든 것을 고려하기 이전에 W3C에서 제시하는 간단한 확인 사항들을 알아봅니다. 

- 개발자 도구에서 '접근성' 기능을 활용합니다. 

크롬 개발자 도구를 기준으로 '접근성(Accessibility)' 탭을 사용할 수 있습니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/8f11e7b9-19e3-414c-b966-8f35b01c21aa" alt="">
*Accessibility 탭에서 Enalble full-page accessibility tree 옵션을 체크하면 Element 탭에서 아이콘이 나타납니다*
<br />

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/66b99095-81a5-4198-af55-f6cf20bd1e24" alt="">
*해당 아이콘을 활성화하면 이제 DOM Tree가 아닌 접근성 트리를 구성하는 노드만 남길 수 있습니다*
<br />

*접근성 트리에 대한 설명은 [MDN 링크](https://developer.mozilla.org/ko/docs/Glossary/Accessibility_tree)을 참고해주세요.

- Web Content Accessibility Guidelines (WCAG) 를 따릅니다. 

말 그대로 W3C에서 제시하는 웹 접근성을 높이기 위한 가이드라인입니다. 가장 최신 버전은 2023년 10월에 나온 [2.2버전](https://www.w3.org/TR/WCAG22/)입니다. (3버전은 Exploratory 단계로 아직 준비 중입니다) 가이드라인에 있는 내용을 전부 파악하고 적용하는 것은 현실적으로 무리가 있을 것 같고, 문서에서 필요한 내용을 발췌해서 적용할 수 있을 것 같습니다. 

- 적절한 마크업을 사용합니다.

브라우저는 HTML 파일을 파싱하여 DOM Tree로 만들 뿐만 아니라 접근성 트리를 생성합니다. 이를 기반으로 웹 페이지가 스크린 리더나 보조 기술 (Assistive technology) 에 의한 지원을 받을 수 있기 때문에 적절한 HTML 태그와 속성을 명시하는 것이 중요합니다. 

- 키보드 지시사항을 준수합니다.

키보드로 웹을 탐색할 수 있게 하면서 동시에 여러 키 입력 조합이 필요한 경우 OS별 대응도 신경 써야 합니다. 대표적으로 Windows의 Ctrl과 MacOS의 cmd가 대응되야 합니다. 


## 3. 시맨틱 태그에서의 접근성 

위에서 설명한 것처럼 웹 페이지를 구성하는 각 요소에 적절한 시맨틱 태그를 사용하는 것은 중요합니다. 또한 각 태그를 사용할 때 상황에 맞게 접근성 관련 속성을 명시하는 것도 중요합니다. W3C 문서 내용을 토대로 각 태그를 사용할 때 신경써야 하는 점에 대해서 간추려봅니다. 

### 3-1. title element

- title 태그에 있는 텍스트는 스크린 리더가 가장 먼저 읽는 요소이기 때문에 **사용자의 현재 위치를 알려주기 위해 페이지 내용을 정확하고 간략하게 전달해야** 하며 다른 웹 페이지들과 구분될 수 있어야 합니다. 즉, 모든 페이지가 동일한 title 을 가지고 있는 상황은 지양해야 합니다. 
- 다음의 예시를 통해서 두괄식으로 요약된 페이지가 title 태그를 올바르게 사용했다고 볼 수 있습니다. 

{% highlight html%}
<!-- 👍 좋은 예시 -->
<title>[A11y] WAI 문서로 접근성 이해하기 (1/2) - 구멍을 외면하지 말자</title>

<!-- 👎 안 좋은 예시 -->
<title>구멍을 외면하지 말자 - [A11y] WAI 문서로 접근성 이해하기 (1/2)</title>
{% endhighlight %}

본 블로그 이름에 해당하는 '구멍을 외면하지 말자'라는 텍스트는 사용자의 현재 위치를 구분해주지 못하기 때문에 앞쪽에 있는 것은 안좋은 예시라고 할 수 있습니다. 

### 3-2. Image text alternatives

- 이미지를 볼 수 없는 사람들이 스크린 리더를 통해 이미지 내용을 파악할 때 사용되거나 이미지를 다운로드하지 못한 상태에서 유용합니다.
- 이미지가 어떤 맥락 속에서 사용된 것인지에 따라 alt 속성에 어떤 값을 명시하는 것이 좋은지가 결정됩니다. 
- 기본적으로 모든 이미지는 alt 속성을 가지고 있어야 하며, 이미지를 그대로 묘사하는 것보다 해당 **이미지를 사용함으로써 전달하고 싶은 의미가 무엇인지** 파악하고 활용하는 것이 중요합니다. 
- 자세한 내용은 `4-3. 이미지 (Images)` 에서 후술합니다. 

### 3-3. Headings

- 각 페이지에 최소 하나의 heading 이 있어야 하고 시각적으로 제목처럼 보이는 요소는 heading 태그를 사용해야 합니다. 
- 1에서 6까지의 위계가 있기 때문에 각기 다른 레벨의 heading 태그가 사용되었을 때 **실제로 의미 있는 위계로** 배치되었는지 고려해야 합니다. 

### 3-4. Contrast Ratio 

- 배경과 전경의 충분한 색상 대비가 이루어지지 않으면 노인이나 난독증이 있는 사람들이 콘텐츠를 읽을 수 없습니다. 이는 단순히 색상이 충분하게 대비를 이뤄야 한다는 의미뿐만 아니라 **색상의 밝고 어두움에 대한 휘도도 충분하게 대비되어야** 한다는 뜻입니다.
- 적절한 대조를 이루고 있는지 확인하는 방법에 대해서는 [W3C Easy Checks - Contrast Ratio](https://www.w3.org/WAI/test-evaluate/preliminary/#contrast)의 `What to check for`와 `Contrast Ratio `를 읽어보면 도움이 될 것 같습니다. 

### 3-5. Resize text 

- 사용자들이 자신에게 알맞은 글자 크기를 설정했을 때 웹 콘텐츠가 서로 겹치거나 수평 스크롤이 생겨서 가독성을 떨어트릴 수 있습니다.
- 글자 크기가 커져도 화면에서 **다른 모든 요소를 볼 수 있고, 상호작용할 수 있어야 하며 수평 스크롤은 최대한 생기지 않게** 하는 것이 좋습니다.

### 3-6. Keyboard access and visual focus

- 거동이 불편해서 보조 기술에 의존하거나 시각 장애가 있는 사용자는 키보드에 의존하여 웹 페이지를 탐색하기 때문에 키보드로 모든 웹 콘텐츠에 접근할 수 있어야 합니다. 이 때, 키보드로 포커스되고 있는 요소를 시각적으로 확인할 수 있어야 하고 논리적인 순서로 포커스가 이동해야 합니다. 
- 자신의 웹 페이지가 논리적인 순서로 탭 인터랙션을 지원하는지 알아보기 위해서 **주소창에 포커스를 둔 뒤 오로지 탭키만 눌러 탐색이 가능한지** 여부로 판단할 수 있습니다. 

키보드 인터랙션과 관련하여 다음의 목록을 체크리스트로 삼아도 좋을 것 같습니다. 

1. 모든 요소에 접근할 수 있고
2. 모든 요소에서 벗어날 수 있고
3. 논리적으로 읽는 순서에 맞게 이동하고 (위에서 아래로, 왼쪽에서 오른쪽으로)
4. 어떤 요소를 포커싱하고 있는지 시각적으로 알 수 있고
5. 키보드만으로 모든 기능을 사용할 수 있고
6. 드롭다운 리스트를 이용 가능하고
7. 이미지에 링크가 걸려있는 경우에 포커스가 명확하게 표시되고 다른 키를 눌렀을 때 이동한다.


### 3-7. 움직이고 깜빡거리는 콘텐츠 

- 일반 사용자 뿐만 아니라 주의력 결핍, 시각 처리 장애를 가지고 있는 사람들을 위해서 광고, 비디오처럼 움직이는 콘텐츠는 사용자가 제어할 수 있어야 합니다. 
- 제어가 불가능할 경우에는 움직이는 정보를 충분히 이해하지 못한 채 지나갈 수 있고, 콘텐츠 자체에 대한 집중력을 떨어트리는 문제가 있습니다. 
- 특히 다음 사항에 해당하는 요소는 **광과민성증후군 환자의 발작을 유발할 수 있기 때문에** 유의해야 합니다. 
  1. 1초에 3번 이상 깜빡이거나
  2. 화면의 넓은 부분을 차지하고 
  3. 충분히 밝은 경우 

따라서 웹 페이지를 구성할 때 반드시,

  1. 움직이고, 깜빡이는 정보가 자동으로 시작되고 5초 이상 지속되는지 (있다면 이를 제어할 수 있는지)
  2. 실시간으로 업데이트되는 정보가 있는지 (있다면 이를 제어할 수 있는지 또는 업데이트 주기를 조절할 수 있는지)
  3. 1초에 3번 이상 깜빡이는 콘텐츠가 있는지 

여부를 반드시 확인해야 합니다. 


## 4. 접근성을 위한 개발 

### 4-1. 페이지 구조 (Page Structure)

#### 4-1-1. Page Region 

웹 페이지를 구성하는 각 영역에 사용할 수 있는 시맨틱 태그를 말합니다. 잘 알려진 시맨틱 태그처럼 main, nav, aside 등이 있습니다. 그 중에서 WAI-ARIA 속성과 관련된 특징이 있는 header와 footer에 대한 정보는 다음과 같습니다.

- header : header는 웹 페이지 상단에 위치하여 전체 웹 페이지에 대한 정보를 담는 영역입니다. WAI-ARIA role로 보았을 때 [role="banner"](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/banner_role)을 수행하는데, article, section 태그 안에서 사용되면 해당 role을 잃고 보조 기술에 대한 지원을 받을 수 없습니다. 

- footer : footer는 웹 페이지 하단에 위치하여 header와 유사하게 사이트 정보를 담습니다. WAI-ARIA role로 보았을 때 [role="contentinfo"](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/contentinfo_role)을 수행하는데, article, section 태그 안에서 사용되면 해당 role을 잃고 보조 기술에 대한 지원을 받을 수 없습니다. 

대부분의 웹 브라우저들은 HTML5 요소를 지원하기 때문에 우리가 익히 아는 시맨틱 태그만 사용해도 괜찮지만, 브라우저와 보조 기술의 접근성을 극대화하기 위해서 HTML5 요소에 상응하는 role을 명시하는 것이 좋습니다.

{% highlight html%}
<header role="banner">…</header>
<main role="main">…</main>
<nav role="navigation">…</nav>
<footer role="contentinfo">…</footer>
{% endhighlight %}

#### 4-1-2. Labeling Regions

동일한 시맨틱 태그가 사용됐을 경우 각각을 구분하기 위해 label 속성을 사용하기도 합니다.

- aria-labelledby : 다른 요소의 id 값을 설정하면 다른 모든 요소를 라벨로 사용할 수 있습니다. 아래의 예시처럼 heading 태그가 다른 Page Region(nav)안에 있을 경우 h2는 nav를 설명하는 역할을 맡습니다.

{% highlight html%}
<nav aria-labelledby="regionheading">
  <h2 id="regionheading">On this Page</h2>
    …
</nav>
{% endhighlight %}

- aria-label : 여러 개의 nav가 있을 경우 각 시맨틱 태그의 역할을 더 구체화할 필요가 있습니다. 아래 예시의 nav 태그는 Main 네비게이션이라는 의미를 aria-label에 담고 있습니다. aria-labelledby와 달리 다른 요소를 사용하지 않기 때문에 라벨이 시각적으로 드러나지 않아야 할 때 적합합니다. 

{% highlight html%}
<nav aria-label="Main">
  …
</nav>
{% endhighlight %}

#### 4-1-3. Content Structure 

웹 페이지에 들어가는 실제 내용을 구성할 때 사용하는 시맨틱 태그를 말합니다. 잘 알려진 태그들은 최대한 간단하게 서술하겠습니다. 

- article : 웹 페이지 내에서 그 자체만으로 완전한 정보를 담고 있는 영역에 사용됩니다. 
- section : 콘텐츠를 나타내기 위해 가장 일반적으로 사용되며 콘텐츠를 주제별로 묶을 때 사용됩니다.
- paragraph : 글의 문단을 나타내는 데에 사용합니다. 일관적인 스타일이 적용되어 있을 경우 가독성이 올라갑니다. 
- list : 서로 다른 유형의 목록을 사용하여 정보를 목록화할 수 있습니다. ul, ol 이외에도 dl 태그도 존재합니다. dt, dd 태그와 함께 사용하여 용어를 설명할 때 사용합니다. 

{% highlight html%}
<dl>
  <dt>term</dt>
  <dd>description</dd>
</dl>
{% endhighlight %}

- quote : 인용된 문장에 사용하는 태그들입니다. 아래의 태그를 사용하면 보조 기술이 인용의 시작점과 끝점을 알려줍니다. 
  - blockquote :  길고 복잡한 인용을 block으로 담을 때 사용합니다. 
  - q : 짧은 인용을 inline으로 담을 때 사용합니다. 
  - cite : 인용의 출처를 언급할 때 사용합니다. 

- figure : 메인 콘텐츠에 대한 추가 정보를 제공할 때 사용합니다. 일반적으로 리스트, 이미지, 테이블 등을 포함하고 있고 figcaption 요소를 통해 figure에 라벨을 붙입니다. 아래 예시를 보면 복잡한 정보를 담고 있는 img 태그와 이에 대한 설명이 있는 a 태그를 figure 태그로 감싸고 있는 것을 볼 수 있습니다. 

{% highlight html%}
<p>The sales volume of our SpaceBear product was steadily the first three quarters but had a huge success in quarter four with the introduction of SuperBear in time for the holiday season. See graphic G3 for details.</p>

<figure role="group" aria-labelledby="fig-t3-capt">
    <figcaption id="fig-t3-capt">G3: SpaceBear sales volume</figcaption>
    <img src="…"
         alt="SpaceBear sales diagram, showing the huge success in Q4"
         longdesc="…">
    <a href="…">Long Description</a>
</figure>
{% endhighlight %}


### 4-2. 메뉴 (Menus)

#### 4-2-1. Structure 

네비게이션 메뉴를 표현할 때 리스트 (ul, ol)를 사용하면 보조 기술이 메뉴에 있는 아이템 개수를 보조 기기 사용자에게 알려주고 네비게이션 관련 기능을 제공합니다. 

메뉴 접근성을 위해 현재 활성화된 메뉴 아이템에 대한 처리도 중요합니다. 눈에 보이지 않는 (visibility : hidden이 적용된) 텍스트를 통해 스크린 리더 사용자에게 활성화된 아이템을 알려주는 방법이 있습니다. 

{% highlight html%}
<!-- visuallyhidden 클래스는 visibility: hidden 으로 보이지 않음 -->
<li>
	<span class="current">
		<span class="visuallyhidden">Current Page: </span>
		Space Bears
	</span>
</li>
{% endhighlight %}

한편, 활성화된 아이템에서는 a 태그를 제거하여 상호작용할 수 없게 해야 하고, 만일 불가능할 경우 활성화된 메뉴 아이템에 [aria-current='page'](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-current) 속성을 사용하여 활성화된 페이지임을 보조 기술에 알려줍니다. 

#### 4-2-2. Styling 

메뉴에 명확하고 일관된 스타일이 적용될수록 사용자들이 메뉴를 쉽게 탐색할 수 있습니다. 여러가지 고려사항이 있지만 간추린 항목은 다음과 같습니다.

1. 메뉴의 위치는 일반적으로 예상 가능한 위치에 있어야 합니다 (좌측 혹은 상단)
2. 가독성을 위해 메뉴 아이템에 (영어의 경우) 전부 대문자를 사용하거나 줄바꿈, 하이픈(-)을 사용하지 않습니다. 
3. 충분한 공백을 줘서 모바일 기기에서 터치할 수 있도록 합니다. 
4. 글자 크기를 키웠을 때 다른 요소를 가리지 않도록 합니다. 
5. 메뉴의 상태를 전달하기 위해 색상과 함께 반드시 다른 스타일 요소로 변경되어야 합니다. (크기, 아이콘 변경 등)

#### 4-2-3. Fly-out Menus

Fly-out 메뉴는 드롭다운 메뉴처럼 하위 메뉴를 담기 위해 별도의 DOM 요소가 나타났다 사라지는 것을 가리킵니다. 
접근성을 위해 하위 메뉴의 노출 여부를 WAI-ARIA 속성 중 [aria-expanded](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-expanded)에 명시해야 합니다. 

이런 Fly-out 메뉴는 마우스와 키보드 접근성 또한 고려해야 합니다. 

- 마우스 사용자 : 하위 메뉴가 너무 빨리 사라지게 될 경우 마우스 조작이 불편한 사용자가 쉽게 이용할 수 없기 때문에 메뉴가 사라지기까지 **약간의 딜레이를 주는 방법**이 있습니다. 

- 키보드 사용자 : 탭키를 사용해서 **모든 하위 메뉴를 탐색하지 않도록 합니다.** 즉, 탭키를 눌렀을 때 하위 메뉴가 열리며 모든 아이템에 포커스가 가는 방식은 좋지 않습니다. 이렇게 하면 키보드로 탐색하는 사용자가 불필요하게 너무 많은 메뉴에 접근해야 해서 원하는 정보에 접근하기 어려워지기 때문입니다.

#### 4-2-4. Application Menus

어플리케이션에서 메뉴를 제공할 때 하위 메뉴가 열리는 방식에 따라서 aria-expanded(드롭다운)나 aria-haspopup(모달) 중에서 적절한 WAI-ARIA 속성을 사용합니다. 또한 메뉴를 구성하는 각 요소에 적절한 role 속성도 명시합니다. 

- menubar: 수평으로 된 메뉴 막대
- menu: 메뉴 막대에 있는 링크나 명령어의 집합. 주로 Fly-out 메뉴에 사용
- menuitem: 개별 메뉴 아이템
- separator: 메뉴에서 두 메뉴 그룹을 구분하는 구분자

위의 WAI-ARIA 속성을 추가한다는 것이 자동으로 메뉴에 기능을 추가하거나 키보드 인터랙션이 더해진다는 것이 아니기 때문에 **반드시 개발자가 기능(페이지 이동 등)과 키보드 이벤트 핸들러를 추가해야 합니다.** 일반적으로 메뉴에서 사용되는 키와 기대되는 동작은 다음과 같습니다. 

- 좌우 키보드 : 메뉴 아이템 간 이동 
- 상하 키보드 : 하위 메뉴 아이템 탐색 
- Tab 키 : (메뉴 아이템 이동이 아닌) 다른 요소로 이동

이 때, **첫번째 메뉴 아이템은 tabindex 를 0으로 줘서 탭 포커싱을 받을 수 있도록 하고 나머지 아이템들은 -1로 설정하여 탭 포커싱을 받을 수 없도록 합니다.** 이렇게 하면 메뉴에 포커스가 있는 상태에서 다른 요소로 이동하고 싶을 경우 Tab 키를 누르면 되고, 하위 메뉴를 탐색하고 싶을 경우 화살표와 Enter 키를 사용해서 이어나갈 수 있기 때문입니다. 

### 4-3. 이미지 (Images)

#### 4-3-1. Informative Images 

이미지가 짧은 문장으로 표현할 수 있는 정보를 담고 있을 경우 해당 정보를 대체 텍스트로 사용합니다. 이는 이미지 자체에 대한 묘사를 의미하는 것은 아니지만 맥락에 따라서 묘사가 필요할 때도 있습니다. 

- 설명에 대한 보충을 목적으로 사용된 경우 
  - 이미 웹 콘텐츠에 정보가 들어있기 때문에 대체 텍스트에 설명을 적기보다 이미지에 대한 짧은 설명만 있으면 충분합니다. 

- 간단한 정보를 전달하고 있을 경우
  - 이미지가 전달하고 있는 정보를 대체 텍스트에서 자세하게 설명합니다.

- 인상이나 감정을 전달하고 있을 경우
  - 이미지가 사용되고 있는 맥락을 고려하여 의도한 인상을 묘사합니다.

- 파일 포맷을 나타내는 경우 
  - 대체 텍스트에도 파일 포맷만 적습니다.

#### 4-3-2. Decorative Images

정보 전달이 목적이 아니라 시각적으로 예쁘게 꾸미기 위한 목적으로 사용된 이미지에는 **null alt (alt='') 를 사용해야** 스크린 리더나 보조 기술이 그냥 지나칠 수 있습니다.
오히려 이런 이미지에 alt 값이 있을 경우 주변에 있는 설명과 내용이 다르면 스크린 리더를 사용하는 사람들에게 혼란을 줄 수 있기 때문입니다. 반대로 아예 alt 속성을 사용하지 않으면 몇몇 스크린 리더는 이미지 파일 이름을 읽어주기 때문에 부적절한 대응이 될 수 있습니다. 따라서 이미지를 사용하는 사람이 맥락을 잘 파악해서 이미지가 정보를 담고 있는지 판단하고 속성을 부여해야 합니다. 이러한 이유 때문에 가능하다면 장식용 이미지는 CSS background-image 속성으로 설명하는 편이 낫습니다.

#### 4-3-3. Functional Images

몇몇 이미지는 이미지를 통해 버튼, 링크 등 상호작용 가능한 요소에 사용자 행동을 이끌어내기 위해 사용됩니다. 이러한 이미지의 대체 텍스트에는 **상호작용으로 어떠한 일이 이루어질지에 대해** 설명해야 합니다. 기능성 이미지는 말 그대로 기능에 대한 의미를 담고 있기 때문에 alt 속성을 명시하지 않을 경우 큰 문제가 발생할 수도 있습니다. 

- 로고에 링크가 걸려있는 경우
  - 로고가 텍스트로 제공되지 않을 경우 alt 속성에 표시해주고, 있을 경우 null alt를 적어줍니다.

- 이미지가 기능을 의미하는 경우 
  - 대체 텍스트에 해당 이미지와 상호작용했을 때 어떤 동작이 실행되는지 명시합니다.

- 버튼에 사용된 이미지 
  - 대체 텍스트에 버튼의 목적을 설명합니다.

#### 4-3-4. Images of Text

사용자가 이미지에 있는 글자를 읽도록 의도적으로 배치된 경우 대체 텍스트에는 **이미지에 있는 글자와 동일한 내용**을 담습니다. 하지만 이렇게 사용자가 읽는 것을 목적으로 하는 글자에 디자인을 넣고 싶은 경우 이미지보다 텍스트에 CSS를 최대한 활용하는 편이 더 좋습니다. 

#### 4-3-5. Complex Images
짧은 문장으로 표현할 수 없을 정도로 중요하고 복잡한 정보가 들어있는 이미지 (그래프, 차트, 다이어그램, 삽화 등) 는 대체 텍스트를 두 부분으로 나누어서 제공합니다. 
1. 이미지를 식별 가능한 수준으로 짧게 설명하고, 이어서 나올 2번 설명의 위치를 언급합니다. 
2. 이미지에 있는 핵심적인 정보나 구체적인 수치를 글로 충분히 설명합니다. 

단, 이미지를 복잡하게 만든 다음 대체 텍스트로 설명하려고 하지 말고, 가능하면 최대한 이미지를 단순화하고 메인 콘텐츠에 충분히 자세한 설명을 포함하여 대체 텍스트에는 해당 위치만 가리킬 수 있도록 하는 것이 접근성을 높이는 데에 더 도움이 될 수 있습니다. 또한 이미지에 대한 간단한 요약을 추가하는 것도 접근성 증진에 도움될 수 있습니다. 

#### 4-3-6. Groups of Images

별점처럼 하나의 정보를 표현하기 위해 여러 개의 이미지가 사용되는 경우가 있습니다. 이 때는 모든 이미지에 대체 텍스트를 사용하지 말고, 하나의 이미지에만 전체 이미지 그룹이 의미하는 바를 대체 텍스트로 적어주면 됩니다. 동시에 나머지 이미지에는 null alt를 사용하여 보조 기술이 무시하게 합니다. 

{% highlight html%}
Rating:
<img src="star-full.jpg"  alt="3.5 out of 5 stars">
<img src="star-full.jpg"  alt="">
<img src="star-full.jpg"  alt="">
<img src="star-half.jpg"  alt="">
<img src="star-empty.jpg" alt="">
{% endhighlight %}

한편, 여러 개의 이미지가 관련된 하나의 주제로 묶여 이미지 모음을 표현할 때가 있습니다. (특정 시대의 예술 작품 등) 이 때는 각 이미지에 대체 텍스트를 설정하고, 시맨틱 태그를 사용하여 이미지들이 관계가 있음을 알려줘야 합니다. 아래 예시처럼 figure, figcaption 태그로 이미지들을 연관시키고 aria-labelledby 로 figcaption 요소가 올바른 figure에 대해 설명하도록 매핑합니다. 

{% highlight html%}
<figure role="group" aria-labelledby="fig1">
  <figcaption id="fig1">
    The castle through the ages: 1423, 1756, and 1936 respectively.
  </figcaption>

  <figure role="group" aria-labelledby="fig11">
    <img src="castle-etching.jpg"
      alt="The castle has one tower, and a tall wall around it.">
    <figcaption id="fig11">Charcoal on  wood. Anonymous, circa 1423.</figcaption>
  </figure>

   <figure role="group" aria-labelledby="fig12">
    <img src="castle-painting.jpg"
      alt="The castle now has two towers and two walls.">
    <figcaption id="fig12">Oil-based paint on canvas. Eloisa Faulkner, 1756.</figcaption>
  </figure>
</figure>
{% endhighlight %}


#### 4-3-7. Image Maps

map과 area 태그로 이루어진 이미지 맵은 다른 페이지로 이동할 수 있는 구성 요소와 상호작용이 가능합니다. 이 때, img 태그에서도 맥락을 전달하기 위해 대체 텍스트가 필요하고 area 태그에서도 링크 정보를 알려주기 위해 대체 텍스트가 필요합니다. 

{% highlight html%}
<img src="orgchart.png"
     alt="Board of directors and related staff: "
     usemap="#Map">
<map id="Map" name="Map">
  <area shape="rect"
        coords="176,14,323,58"
        href="[…]"
        alt="Davy Jones: Chairman">
  […]
  <area shape="rect"
        coords="6,138,155,182"
        href="[…]"
        alt="Harry H Brown: Marketing Director (reports to chairman)">
  […]
</map>
{% endhighlight %}

하지만 map, area 태그가 아직 몇몇 브라우저나 모바일 기기에서 불안정한 모습을 보이기 때문에 실제 사용하려면 주의사항을 충분히 숙지하거나 사용을 미루는 것이 좋아보입니다. 
