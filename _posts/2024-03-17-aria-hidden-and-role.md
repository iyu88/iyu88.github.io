---
layout: post
title: "[A11y] aria-hidden='true'와 aria-role='none'"
date: 2024-03-17 00:00:00
author: Hyunbin Lee
categories: A11y
tags: Accessibility A11y
cover: "/assets/instacode.png"
---

# aria-hidden='true'와 aria-role='none'

## 1. aria-hidden='true'

aria-hidden 속성은 요소를 [접근성 트리](https://developer.mozilla.org/ko/docs/Glossary/Accessibility_tree)에서 제거할 지 여부를 결정합니다. `aria-hidden='true'`로 설정하면 해당 요소가 접근성 트리에 노출되지 않을 뿐만 아니라 자식 요소들도 모두 영향 받아 사라지게 됩니다. 이렇게 접근성 트리에서 사라지면 스크린 리더 등 접근성을 지원하는 보조 기술의 도움을 받을 수 없습니다. 

aria-hidden 속성은 시각적인 렌더와는 직접적인 연관이 없습니다. **다만 정상적으로 렌더링된 요소를 무턱대고 aria-hidden='true'로 설정하는 것은 지양해야 합니다.** 왜냐하면 시각적으로 파악할 수 있는 정보를 스크린 리더의 도움을 받을 수 없게 되어 보조 기술 사용자의 사용성을 저하시킬 수 있기 때문입니다. 따라서 불필요하다고 판단되거나 중복되는 정보에 한해서 설정해야 합니다. 

<br />

## 2. aria-role='none'

aria-role 속성은 요소에 역할을 명시하여 콘텐츠에 의미(semantic meaning)를 부여합니다. 이렇게 의미가 부여된 요소는 스크린 리더 등 보조 기술 사용자가 해당 요소의 의미를 파악하고 상호작용할 수 있게 도와줍니다. 

[다양한 role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles#aria_role_types)이 존재하지만 이 중에서 `aria-role='none'`(aria-role='presentation'과 동일)은 **요소에 있는 시맨틱 의미를 제거하는 데에 사용합니다.** 일반적으로 <header>, <footer>, <article>과 같은 시맨틱 태그와는 함께 사용하지 않습니다. 왜냐하면 이러한 경우에는 비시맨틱 태그인 <div>를 대신 사용하는 것이 더 적절하기 때문입니다. 다음 두 가지 상황에서 aria-role='none'을 유용하게 활용할 수 있습니다. 

#### 2-1. 레거시 마크업을 수정할 때 

CSS 레이아웃 관련 속성들이 발전하기 이전에는 페이지 레이아웃을 구성하는 목적으로 <table> 태그를 마크업에 많이 사용했습니다. 만일 이러한 사이트를 유지보수해야 하는 경우 aria-role='none'을 부여하여 오용된 <table>의 시맨틱 의미를 제거할 수 있습니다. 

#### 2-2. 마크업 구조에서 시맨틱 의미를 제거하는 것이 도움이 될 경우

오히려 aria-role='none'으로 시맨틱 의미를 제거하는 것이 접근성에 도움이 되는 경우가 있습니다. 아래는 role='tablist' 마크업 예시입니다. 
<script src="https://gist.github.com/iyu88/05caa5e430a638fe465d75d9613b5c21.js?file=01_tablist_example.html"></script>
tablist는 자식 요소로 바로 role='tab'에 해당하는 요소가 오는 것이 적절합니다. 이러한 경우 <li> 요소는 시맨틱 의미를 가질 필요가 없기 때문에 role='none'으로 제거합니다. 

<br />

## 3. 두 속성의 공통점 

위 두 속성은 **상호작용 가능한 요소에 설정하는 것을 지양해야 한다는 공통점**을 가지고 있습니다. 이는 접근성 보조 기술이 파악할 수 없는 요소(aria-hidden='true')나 의도적으로 시맨틱 의미가 제거된 요소(aria-role='none')에 키보드 focus 혹은 마우스 포인터 focus가 가능할 경우, 각각 웹 접근성을 떨어트리고 [role 관련 표준 스펙](https://www.w3.org/TR/wai-aria-1.2/#conflict_resolution_presentation_none)을 준수하지 못하는 문제가 있기 때문입니다. 

<br />

[참고]

- [MDN - 접근성 트리](https://developer.mozilla.org/ko/docs/Glossary/Accessibility_tree)
- [MDN - aria-hidden](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-hidden)
- [MDN - WAI-ARIA Roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles)
- [MDN - ARIA: presentation role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/presentation_role)
- [Know your ARIA: 'Hidden' vs 'None'](https://www.scottohara.me/blog/2018/05/05/hidden-vs-none.html)
