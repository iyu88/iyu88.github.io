---
layout: post
title: "[React] 디자인 시스템에 Compound Component Pattern 적용해보기"
date: 2023-03-25 00:00:00
author: Hyunbin Lee
categories: React
tags: React DesignSystem CompoundComponent Accessibility
cover: "/assets/instacode.png"
---

# 디자인 시스템에 Compound Component Pattern 적용해보기

> ### 💡 디자인 시스템을 만들어가면서 React의 합성 컴포넌트 패턴을 사용하여 Tabs 컴포넌트를 만들었던 내용을 정리합니다. 

## 1. 배경

지난 2월 초부터 부스트캠프를 수료한 동료 프론트엔드 개발자분들과 의기투합하여 함께 디자인 시스템을 만들자는 원대한 목표를 가지고 '차가운 디자인 시스템 (Cold Design System, 이하 `CDS`)'을 만들어 가고 있습니다. React 앱에서 편리하게 사용할 수 있도록 라이브러리 차원에서 지원해야 하는 각 컴포넌트의 역할을 정의하고 실제로 사용하는 상황을 고려하며 여러 컴포넌트를 완성해가고 있습니다.

프로젝트 초기에 구현한 컴포넌트들은 일반 텍스트를 표시하는 `Typography`, button 태그 역할을 하는 `Button`처럼 비교적 단순한 컴포넌트들과 여러 요소를 감싸는 데에 사용하는 `Container`, 'display: flex'가 적용된 요소를 선언적으로 사용할 수 있는 `Flexbox` 등의 Layout을 구현했습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227597943-548b5765-abc6-43f4-878f-fe13d83d4a7c.png">
*Typography 컴포넌트*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/227598499-a1611577-e63d-4485-8c4d-d0d48655a8da.png">
*Button 컴포넌트*
<br />

개발이 점차 진행되면서 디자인 시스템에서 제공해야 하는 컴포넌트들의 복잡도도 올라가게 되었습니다. 가령, 아래 Carousel처럼 비교적 복잡도가 높은 컴포넌트는 어떻게 구현해야 할지 고민하게 되었습니다.

<br />
<img src="https://user-images.githubusercontent.com/31645195/227600771-db6b4f93-5f96-4e4d-b519-999dd521555f.png">
*Carousel [출처:https://store.steampowered.com/]*
<br />

이러한 컴포넌트들을 구현하기 위해서 선택한 방법인 `Compound Component Pattern`(합성 컴포넌트 패턴)에 대해서 알아보고, 이를 사용하여 CDS의 Tabs 컴포넌트를 구현한 과정에 대해서 소개하겠습니다. 

*CDS 기술 스택: React, TypeScript, Emotion, Storybook, Vite

<br />

## 2. Compound Component Pattern

Compound Component Pattern을 소개하고 있는 수많은 글 중에서 [이 글](https://betterprogramming.pub/compound-component-design-pattern-in-react-34b50e32dea0)에서는 이렇게 설명하고 있습니다. 

> Compound components are a React pattern that provides an expressive and flexible way for a parent component to communicate with its children, while expressively separating logic and UI.

서브(자식) 컴포넌트들이 메인(부모) 컴포넌트 내부의 상태를 공유하면서 비지니스 로직과 사용자 인터페이스와 관련된 부분을 구분하는 React 패턴입니다. 다시 말해, 합성 컴포넌트 패턴은 여러 개의 작은 컴포넌트들이 각각의 역할을 분담하도록 하고 이를 조립하여 하나의 큰 컴포넌트를 만드는 것이라고 할 수 있습니다. 

합성 컴포넌트 패턴의 장점은 컴포넌트를 사용할 때 개발자가 필요로 하는 서브 컴포넌트만 합성하여 사용할 수 있기 때문에 개발자에게 자율성을 줄 수 있다는 것과 하나의 컴포넌트 안에 수많은 Props를 한꺼번에 전달하지 않고 서브 컴포넌트에 적절히 분배하여 관심사를 분리할 수 있다는 것입니다. 

반면, UI에 대한 자유도가 높은만큼 컴포넌트의 의도대로 합성되지 않을 수도 있고 합성 컴포넌트 패턴을 사용하지 않았을 때보다 JSX 소스 코드 길이가 길어질 수 있다는 단점도 있습니다. 

따라서 지금처럼 라이브러리를 만드는 입장에서는 사용자(개발자)에게 어떤 인터페이스를 열어줄 것인지, 어떤 옵션을 선택할 수 있게 할 것인지 충분히 고민하고 구현해야 합니다. 이어서 앞서 언급했던 것처럼 서브 컴포넌트들이 부모 컴포넌트의 상태를 공유하는 방법과 관련하여 React의 Context API에 대해서 간략하게 살펴보겠습니다.

<br />

## 3. Context API 

React에서 제공하는 Context API는 `Props Drilling`(상태를 공유하기 위해서 해당 상태가 필요하지 않은 컴포넌트들도 불필요하게 상태 값을 받아야 하는 문제)을 방지하면서 children 요소들이 상태를 공유할 수 있도록 해줍니다. 이를 합성 컴포넌트에서 사용하는 경우를 생각해보면 공유해야 하는 값은 Context를 사용하여 부모 컴포넌트 내부의 children들이 공유받도록 구현하고, 그렇지 않은 값은 각각의 자식 컴포넌트가 Props로 받아서 처리하게 됩니다.

만약 아래와 같은 select-option 태그를 합성 컴포넌트 패턴으로 만든다면 현재 선택된 option에 대한 정보는 전체 컴포넌트에서 공유해야 하기 때문에 Context API로 관리하고, 나머지 개별적인 option 정보는 자식 컴포넌트가 Props로 전달받아 자체적으로 관리하는 방식으로 구현할 수 있습니다.

{% highlight html%}
<select name="pets" id="pet-select">
    <option value="dog">Dog</option>
    <option value="cat">Cat</option>
    <option value="hamster">Hamster</option>
    <option value="parrot">Parrot</option>
    <option value="spider">Spider</option>
    <option value="goldfish">Goldfish</option>
</select>
<!-- 출처: https://developer.mozilla.org/ko/docs/Web/HTML/Element/select -->
{% endhighlight %}

<br />

## 4. Tabs 컴포넌트 

간단하게 Compound Component Pattern과 Context API에 대해서 알아보았습니다. 여기부터는 두 개념을 적용하여 Tabs 컴포넌트를 만들어 보겠습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227698050-cf8f5054-e630-4b28-bc9c-469ab8dc21f9.png">
*사진처럼 상단 메뉴를 누르면 하단에 내용이 바뀌는 컴포넌트를 만들 예정입니다.*
<br />

먼저 대략적으로 어떤 스펙으로 만들 것인지 생각해봅니다. 

#### 4-1. 스펙 정리 

크게 `기능`과 `디자인`으로 나누어 보았습니다. 

기능은 컴포넌트 내부에서 관리해야 하는 상태나 지원해야 하는 비지니스 로직과 관련한 부분이고, 디자인은 사용자에게 선택지를 제공하여 어떤 형태로 보여줄 것인지와 관련한 부분입니다.

1. 기능: Tabs에서 사용자(개발자)가 기대하는 요소들
  - 현재 선택된 Tab 정보
  - Tab 비활성화 기능
  - 마우스로 클릭하거나 키보드로 이동
  - Tab이 많을 경우 스크롤 가능하도록 구현 
  - Tab에 아이콘도 넣을 수 있도록 구현 

2. 디자인: Tabs에서 사용자(개발자)가 선택 가능한 옵션들
  - Tab의 기본 생김새
  - Tab의 너비를 꽉 차게 배치할 수 있는지 여부

위 요소를 생각하면서 컴포넌트를 구현해보겠습니다. 

<br />

#### 4-2. Context 생성 

먼저 Tabs의 각 부분을 나누어 살펴보겠습니다. 상단에는 사용자가 선택할 수 있는 Tab 선택지들이 존재하고 하단에는 선택한 Tab에 대응하는 내용이 표시되어야 합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227698297-a1812a18-591e-4c51-aedb-63548b1bcafa.png">
*상단과 하단이 각각 '선택할 수 있는 영역'과 '내용이 표시되는 영역'으로 구분됩니다.*
<br />

사용자가 선택한 Tab 정보가 상단과 하단에 모두 영향을 주기 때문에 해당 데이터는 Tabs 컴포넌트 내에서 공유되어야 하는 데이터가 됩니다. 선택한 Tab 정보를 담는 `selectedIndex 상태`를 공유하는 Tabs 컴포넌트를 Context API를 사용하여 만들어보겠습니다. 

*실제 코드와 달리 타입과 CSS 속성이 제외된 코드를 명기했습니다. 

{% highlight jsx%}
import { createContext, useState } from 'react';

const TabsContext = createContext(null);

const Tabs = ({ defaultValue, children }) => {
  const [selectedIndex, setSelectedIndex] = useState(defaultValue);

  const providerValue = { selectedIndex, setSelectedIndex };

  return (
    <TabsContext.Provider value={providerValue}>
      <Container>
        {children}
      </Container>
    </TabsContext.Provider>
  );
};
{% endhighlight %}

맨 처음에 활성화되어 있는 Tab 정보를 defaultValue로 받아서 useState로 관리합니다. 

이제 위에서 상단과 하단으로 나누었던 부분을 대략적인 JSX 구조로 나타내보겠습니다. 하위 컴포넌트의 이름은 MDN의 tab 접근성 가이드를 참고하여 명명했습니다.

{% highlight jsx%}
const App = () => {
  return (
    <Tabs>
      <Tabs.List>
        <Tabs.Trigger>첫번째 Trigger</Tabs.Trigger>
        <Tabs.Trigger>두번째 Trigger</Tabs.Trigger>
        <Tabs.Trigger>세번째 Trigger</Tabs.Trigger>
      </Tabs.List>
      <Tabs.Panel>첫번째 Panel</Tabs.Panel>
      <Tabs.Panel>두번째 Panel</Tabs.Panel>
      <Tabs.Panel>세번째 Panel</Tabs.Panel>
    </Tabs>
  )
}
{% endhighlight %}

이제 각각의 부분을 실제 컴포넌트로 분리하고 개별 컴포넌트가 관리해야 하는 데이터를 Props로 전달 받을 수 있도록 구현합니다. 

*이제부터 클릭 가능한 개별 Tab 영역을 `Trigger`라는 이름으로 부르겠습니다. 

<br />

#### 4-3. Tabs - 상위 컴포넌트 

상위 컴포넌트인 Tabs에서는 디자인적인 요소를 결정하는 Props를 추가적으로 받습니다. 

{% highlight jsx%}
const Tabs = ({
  defaultValue,
  variant = 'underline',
  isFitted = false,
  children,
}) => {
  const [selectedIndex, setSelectedIndex] = useState(defaultValue);

  const providerValue = {
    variant,
    isFitted,
    selectedIndex,
    setSelectedIndex,
  };

  // (생략)
};
{% endhighlight %}

`variant`는 underline과 rounded 중 하나를 선택할 수 있도록 하고 `isFitted`는 Trigger들의 너비를 부모 요소 너비에 딱 맞게 들어가도록 넓힐 것인지 여부를 boolean 값으로 결정합니다. 각각의 props에 대해서 기본값도 지정해줍니다.

<br />

#### 4-4. List - 하위 컴포넌트

List 컴포넌트는 내부에 Trigger들을 위치시키는 Wrapper 컴포넌트로 사용합니다. 따라서 CDS에서 배치를 도와주는 간단한 Layout 컴포넌트들만 가져와서 사용했습니다.

{% highlight jsx%}
const List = ({ children }) => {

  return (
    <Container overflowX="auto">
      <Flexbox justifyContent="flex-start">
        {children}
      </Flexbox>
    </Container>
  );
};
{% endhighlight %}

<br />

#### 4-5. Trigger - 하위 컴포넌트

가장 중요한 Trigger 컴포넌트입니다. 각각의 Trigger는 고유한 `value`를 받을 수 있도록 하여 Trigger가 선택되었을 때 동일한 value를 가진 Panel이 활성화되도록 구현할 것입니다. 또한 Trigger에 글자와 아이콘 등을 표시할 수 있도록 `text`와 `icon`을 받습니다. 추가로 비활성화된 Trigger임을 판단하기 위해서 `disabled` 값을 받습니다. 기본값은 false로 지정하여 비활성화되어야 하는 Trigger에서만 disabled 속성을 선언하도록 했습니다. 최종적인 Trigger의 Props는 아래처럼 정의했습니다. 

{% highlight jsx%}
const Trigger = ({ value, text, icon, disabled = false }) => {
  const context = useContext(TabsContext);
  const isActive = context.selectedIndex === value;

  return (
    <button disabled={disabled}>
      {icon && (
        <Icon
          source={icon}
          color={isActive ? primary100 : black}
        />
      )}
      {text && <Typography variant="body">{text}</Typography>}
    </button>
  )
}
{% endhighlight %} 

Trigger가 현재 선택되었는지 판단하기 위해서 Context의 selectedIndex와 value가 동일한지 여부를 `isActive` 변수에 저장하고 스타일을 적용할 때 사용할 수 있습니다. 

한편, Tabs에서 받았던 variant 값에 따라 Trigger에 서로 다른 스타일을 반영하기 위해 다음 객체들을 이용합니다. 

{% highlight jsx%}
// Trigger
const { color: themeColor } = useTheme(); // 테마를 가져오기 위한 커스텀 훅 
const { primary100, gray100, black, white } = themeColor;

const underlineStyle = {
  borderColor: `${context.selectedIndex === value ? primary100 : gray100}`,
  borderRadius: 0,
  borderBottomWidth: '2px',
  borderBottomStyle: 'solid',
  marginBottom: '-2px',
} 

const roundedStyle = {
  border: `${context.selectedIndex === value && `2px ${gray100} solid`}`,
  borderRadius: '10px',
  borderBottomLeftRadius: 0,
  borderBottomRightRadius: 0,
  borderBottomColor: `${white}`,
  marginBottom: '-2px',
} 

const triggerStyles = {
  underline: underlineStyle,
  rounded: roundedStyle,
};
{% endhighlight %} 

Props에 따라 속성들이 잘 적용된 것을 확인할 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227699903-a190ff72-6163-4204-aa9d-8b407ade3ce5.png">
*variant 값이 underline일 때(상단)와 rounded일 때(하단)*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/227699914-568aa00f-7b2c-4caa-b64b-b81c21b0a20d.png">
*isFitted 값이 true인 상태에서의 underline과 rounded*
<br />

<br />

구현 스펙의 기능 측면에 명시했듯이 disabled인 Trigger를 제외한 나머지 Trigger는 마우스로 클릭하거나 키보드의 특정 키를 눌러서 이동할 수 있어야 합니다. 이를 위해서 onClick에 등록할 onSelect 함수와 onKeyDown에 등록할 onPressArrow 함수를 정의합니다. 

키보드 방향키로 좌우 Trigger를 이동할 때, 현재 Trigger의 형제 요소를 탐색하기 위해서 `findFutureTrigger` 함수를 정의하여 호출합니다. 화살표 방향에 따라 Element의 `nextSiblingElement` 혹은 `previousSiblingElement` 필드에 접근하여 다음 Trigger를 찾아냅니다. disabled인 Trigger를 찾게 된다면 같은 방향으로 이어서 탐색합니다.

{% highlight jsx%}
// Trigger

const onSelect = () => {
  if (disabled) return; // disabled이면 early return
  setSelectedIndex(value); 
};

const onPressArrow = (e) => {
  e.preventDefault();
  const currentTrigger = e.target;

  // key 정보와 현재 Trigger 전달
  const futureTrigger = findFutureTrigger(e.key, currentTrigger); 

  if (!futureTrigger) return;

  const futureValue = futureTrigger.dataset['triggerValue'];

  if (futureValue === undefined) return;

  futureTrigger.focus();
  setSelectedIndex(futureValue);
};

const findFutureTrigger = (key, currentTrigger) => {
  const siblingProp = (function (key) {
    if (NEXT_KEY.includes(key)) return 'nextElementSibling';
    if (PREV_KEY.includes(key)) return 'previousElementSibling';
    return null;
  })(key);

  if (siblingProp === null) return;

  let futureTrigger = currentTrigger[siblingProp];

  while (futureTrigger && futureTrigger.hasAttribute('disabled')) {
    futureTrigger = futureTrigger[siblingProp];
  }

  return futureTrigger;
};

return (
  <button 
    disabled={disabled}
    onClick={onSelect}
    onKeyDown={onPressArrow}
    >
    <!-- (생략) -->
  </button>
)
{% endhighlight %}

<br />

#### 4-6. Panel - 하위 컴포넌트

다음은 선택된 Trigger에 대응하는 내용(컴포넌트)이 렌더링되는 Panel 부분입니다. List와 비슷하게 Panel은 children Props의 Wrapper 역할을 합니다. Trigger처럼 `value`를 받아서 selectedIndex와 동일한 Panel이 렌더링되도록 합니다.

{% highlight jsx%}
const Panel = ({ value, children }) => {
  const context = useContext(TabsContext);

  return (
    <Container
      css={css`
        padding: 1rem;
        display: ${context.selectedIndex === value ? 'block' : 'none'};
      `}
    >
      {children}
    </Container>
  );
};
{% endhighlight %}

<hr />

실제 동작을 확인하기에 앞서 Tabs 컴포넌트의 각 속성에 하위 컴포넌트들을 등록해줍니다. 

{% highlight jsx%}
Tabs.List = List;
Tabs.Trigger = Trigger;
Tabs.Panel = Panel;

export default Tabs;
{% endhighlight %}

최종적으로 아래와 같이 JSX를 구성하고 렌더링 결과를 확인해보겠습니다. 

{% highlight jsx%}
const App = () => {
  return (
    <Tabs defaultValue='1'>
      <Tabs.List>
        <Tabs.Trigger value="1" text="첫번째 탭" />
        <Tabs.Trigger value="2" text="두번째 탭" />
        <Tabs.Trigger value="3" text="세번째 탭" />
      </Tabs.List>
      <Tabs.Panel value="1">
        <div>{DUMMY_STR_1}</div>
      </Tabs.Panel>
      <Tabs.Panel value="2">
        <div>{DUMMY_STR_2}</div>
      </Tabs.Panel>
      <Tabs.Panel value="3">
        <div>{DUMMY_STR_3}</div>
      </Tabs.Panel>
    </Tabs>
  );
};
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/227709794-2c971a1c-f274-4fb8-a9de-7d9a5f3a4213.gif">
*마우스, 키보드 화살표로 움직일 수 있습니다.*
<br />

<br />

## 5. 기능 개선

Tabs의 기본적인 형태를 구현하긴 했지만 조금 더 완성도 있는 컴포넌트를 만들고 싶습니다. 

보완할만한 기능들을 추가로 정의하고 Tabs 컴포넌트를 개선하고자 합니다. 

<br />

#### 5-1. Trigger 선택 시 스크롤 애니메이션

아래 모바일 어플리케이션 GIF처럼 윈도우 너비가 좁거나 Trigger 개수가 많아 스크롤이 생긴 상황에서 사용자가 선택한 Trigger가 자동으로 좌측으로 이동하는 효과를 주고 싶습니다.

<br />
<img src="https://user-images.githubusercontent.com/31645195/227684976-c76646dd-f6ad-498a-b9a5-660acabe7edb.gif">
*상단 태그를 선택하면 좌측으로 자동 스크롤됩니다. [출처: 무신사]*
<br />

이를 Tabs에 적용하기 위해서 Element의 내장 함수인 `scrollIntoView`를 활용하기로 했습니다.
마우스 이벤트나 키보드 이벤트가 발생하여 focus할 Trigger를 정상적으로 찾았을 때 해당 Element로 scrollIntoView 메서드를 실행합니다. 이 때, 애니메이션을 주기 위해서 `behavior: smooth` 옵션을 포함한 scrollOptions를 인자로 전달합니다.

한편, onPressArrow 함수에서는 직접 DOM 요소를 찾기 때문에 문제없이 scrollIntoView를 호출할 수 있지만 onSelect에서는 그렇지 않습니다. 따라서 useRef를 사용하여 button 태그에 대한 reference를 저장합니다.

{% highlight jsx%}
const scrollOptions = {
  behavior: 'smooth',
  block: 'nearest',
  inline: 'start',
}

const onSelect = () => {
  if (disabled || !triggerRef.current) return;
  setSelectedIndex(value);
  triggerRef.current.scrollIntoView(scrollOptions); // 추가
};

const onPressArrow = (e: KeyboardEvent) => {
  e.preventDefault();
  const currentTrigger = e.target;

  const futureTrigger = findFutureTrigger(e.key, currentTrigger);

  if (!futureTrigger) return;

  const futureValue = futureTrigger.dataset['triggerValue'];

  if (futureValue === undefined) return;

  futureTrigger.focus();
  futureTrigger.scrollIntoView(scrollOptions); // 추가
  setSelectedIndex(futureValue);
};

return (
  <button 
    ref={triggerRef} // 추가
    disabled={disabled}
    onClick={onSelect}
    onKeyDown={onPressArrow}
    >
    <!-- (생략) -->
  </button>
)
{% endhighlight %} 

추가로 예시 GIF처럼 사용자가 좌측에 추가적인 Trigger가 있다는 것을 인지할 수 있도록 `scroll-margin` 속성을 추가하여 좌측에 약간의 여백을 확보할 수 있습니다. 

최종적으로 아래와 같은 모습으로 만들어졌습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227710008-445f8b75-a7b2-478b-ade3-72ee63f12cc2.gif">
*부드럽게 잘 스크롤 됩니다.*
<br />

<br />

#### 5-2. 접근성 지원 

Tabs 내부에 위치할 하위 컴포넌트들의 이름을 정할 때 MDN 접근성 가이드를 따라서 지은 바 있습니다. 이름만 접근성을 따르는 것이 아니라 실제로 `접근성을 지원하기 위한 여러 가지 HTML 속성`을 추가해주었습니다. 이를 위해 사용자가 직접 Tabs 컴포넌트 내부에서 공통적으로 사용할 수 있는 `label` 값을 입력할 수 있도록 상위 컴포넌트에서 Props를 추가로 받습니다.

{% highlight jsx%}
const Tabs = ({
  label, // 추가
  defaultValue,
  variant = 'underline',
  isFitted = false,
  children,
}) => {
  // (생략)
}
{% endhighlight %}

<br />

우선 List에서 적용할 수 있는 접근성 속성들은 다음과 같습니다. 
  1. `role`: List 컴포넌트가 tablist 역할이라는 것을 명시합니다. 
  2. `aria-label`: 현재 List가 무엇을 위한 List인지에 대한 값을 전달합니다. 
  3. `aria-orientation`: Tabs의 방향을 적어줍니다. 현재는 외부에서 방향을 제어할 수 없기 때문에 고정값으로 수평 방향을 의미하는 'horizontal'로 설정합니다. 

{% highlight jsx%}
const List = ({ children }) => {
  const { label } = useTabsContext();

  return (
    <Container
      role='tablist'
      aria-label={label}
      aria-orientation='horizontal'
      overflowX="auto"
    >
    <!-- (생략) -->
    </Container>
  )
}
{% endhighlight %}

<br />

다음으로 Trigger에서 적용할 수 있는 접근성 속성은 다음과 같습니다. 
  1. `role`: 사용자가 상호작용하게 되는 tab 역할이라는 것을 명시합니다. 
  2. `aria-selected`: 현재 선택되었을 경우 true, 아닐 경우 false를 줍니다. 
  3. `aria-controls`: 특정 요소가 다른 요소에 변화를 줄 경우, 변화가 일어나는 요소의 Id를 명시합니다. Trigger는 Panel에 변화를 일으키키 때문에 값을 `${label}-panel-${value}`로 설정하고 Panel의 Id도 동일하게 설정합니다. 
  (충분히 유일성이 보장될 수 있도록 임의로 Id를 설정했습니다.)
  4. `aria-disabled`: Trigger가 disabled 상태일 경우 true로 설정합니다.
  5. `tabIndex`: MDN 문서에 따르면 Trigger가 focus된 상태에서 키보드 Tab 키를 누르면 다음 Trigger를 focus하는 것이 아니라 Panel 내부로 이동하는 것이 좋다고 합니다. 따라서 선택된 Trigger는 tabIndex 값을 0으로 하고 다른 모든 Trigger들은 -1로 설정합니다. 

{% highlight jsx%}
const Trigger = ({ value, text, icon, disabled = false }) => {
  const context = useContext(TabsContext);
  const isActive = context.selectedIndex === value;

  return (
    <button 
      ref={triggerRef} 
      id=`${context.label}-trigger-${value}`
      role='tab'
      aria-selected={isActive}
      aria-controls=`${context.label}-panel-${value}`
      aria-disabled={disabled}
      tabIndex={isActive ? 0 : -1}
      disabled={disabled}
      onClick={onSelect}
      onKeyDown={onPressArrow}
      >
      <!-- (생략) -->
    </button> 
  )
}
{% endhighlight %}

<br />

마지막으로 Panel에서 적용할 수 있는 접근성 속성입니다. 
  1. `role`: tabpanel 역할이라는 것을 명시합니다. 
  2. `aria-labelledby`: 대응하는 Trigger의 Id를 명시하여 해당 Trigger가 Panel의 라벨 역할을 하도록 참조 관계를 설정합니다. 
  3. `tabIndex`: Tab 키를 눌렀을 때 Panel 내부로 focus가 이동할 수 있도록 값을 0으로 설정합니다. 

{% highlight jsx%}
const Panel = ({ value, children }) => {
  const context = useContext(TabsContext);

  return (
    <Container
      id=`${context.label}-panel-${value}`
      role='tabpanel'
      aria-labelledby=`${context.label}-trigger-${value}`
      tabIndex={0}
      css={css`
        padding: 1rem;
        display: ${context.selectedIndex === value ? 'block' : 'none'};
      `}
    >
      {children}
    </Container>
  );
};
{% endhighlight %}

<br />

#### 5-3. Context null 확인용 커스텀 훅 생성

createContext()에 전달하는 초기값을 null로 설정했기 때문에 useContext()를 사용할 때마다 불필요한 코드가 생겨나게 됩니다. TypeScript를 사용했기 때문에 useContext()의 반환값이 null이 아닌지 항상 확인해주어야 했습니다. 만약 null일 경우 빈 Fragment를 반환하도록 했습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227695909-be06deaa-530b-4b27-bca0-b0af26179454.png">
*Context를 사용하기 위해서 null 여부를 확인하는 반복적인 코드가 신경쓰입니다.*
<br />

이에 대한 해결법을 [다음 블로그](https://velog.io/@velopert/react-context-tutorial) 글에서 찾을 수 있었습니다. Context를 사용할 때 null을 확인하는 로직을 커스텀 훅으로 분리하는 방법입니다. 

{% highlight jsx%}
const useTabsContext = () => {
  const context = useContext(TabsContext);
  if (context === null) {
    throw new Error('useTabsContext should be used within Tabs');
  }
  return context;
};
{% endhighlight %}

이를 사용하면 Context 값을 사용하고자 하는 하위 컴포넌트에서 더 이상 반복적인 null 체크를 진행하지 않고 바로 Object destructuring으로 원하는 값을 가져올 수 있습니다. 

{% highlight jsx%}
const List = ({ children }) => {
  const { label } = useTabsContext(); // 추가

  return (
    // (생략)
  )
}
{% endhighlight %} 

<br />

## 6. 보완할 점

협업의 꽃, 코드 리뷰를 하는 도중에 tabIndex 값 설정과 관련된 질문을 받게 되었고 답변을 달기 위해 [MDN 사이트](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/tablist_role)에서 키 입력과 관련된 내용을 다시 읽다가 놓쳤던 내용을 발견하게 되었습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227697071-b425c590-8cf1-4bb4-8bbd-6705c82d1bbf.png">
*출처: https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/tablist_role*
<br />

Trigger를 이동할 때 맨 마지막 Trigger에서는 맨 처음으로 이동해야 하고 반대로 맨 처음 Trigger에서는 맨 마지막으로 이동해야 한다는 것입니다. 즉, 맨 처음과 맨 마지막이 이어져 있는 것처럼 동작해야 한다는 것입니다. 현재는 키보드 방향키로 양끝에 도달하게 되면 더 이상 움직이지 않는 구조입니다. findFutureTrigger 함수를 문서에 적힌 것처럼 작동하도록 업데이트해줍니다. 

{% highlight jsx%}
const findFutureTrigger = (key, currentTrigger) => {
  const siblingProp = (function (key) {
    if (NEXT_KEY.includes(key)) return 'nextElementSibling';
    if (PREV_KEY.includes(key)) return 'previousElementSibling';
    return null;
  })(key);

  if (siblingProp === null) return;

  let futureTrigger = currentTrigger[siblingProp];

  while (futureTrigger && futureTrigger.hasAttribute('disabled')) {
    futureTrigger = futureTrigger[siblingProp];
  }

  if (futureTrigger) return futureTrigger; 

  // 추가된 코드
  const triggerParent = currentTrigger.parentElement;

  if (triggerParent === null) return;

  futureTrigger =
    siblingProp === 'nextElementSibling'
      ? (triggerParent.firstElementChild)
      : (triggerParent.lastElementChild);

  while (futureTrigger.hasAttribute('disabled')) {
    futureTrigger = futureTrigger[siblingProp];
  }

  return futureTrigger;
};
{% endhighlight %}

수정한 코드를 적용하고 나서 테스트해보니 정상적으로 동작하는 것을 확인했습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227715573-78395f99-32b8-4843-add6-6f77d5e245b3.gif">
*시작과 끝을 이동하며 focus할 수 있습니다.*
<br />

<br />

## 7. 맺으며

Compound Component Pattern을 사용하여 Tabs 컴포넌트를 구현할 때 작은 단위로 나누고 각 컴포넌트를 조합하는 방식을 사용했습니다. 실제로 아래와 같이 다양한 형태의 Tabs 를 만들어 낼 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/227734557-d88f11f6-956c-476b-bd7a-8fef57b1d01e.png">
*아이콘만 사용한 Trigger*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/227734554-5f61c3af-4cbe-499b-bd2d-d1aae0e614f4.png">
*List를 하단에 배치한 Tabs*
<br />

앞으로도 CDS를 개발하면서 합성 컴포넌트 패턴이 필요한 곳에서 적절하게 도입하며 더 익숙해질 수 있도록 해야겠습니다. 개인적으로 합성 컴포넌트 패턴뿐만 아니라 접근성 관련 속성들을 자세하게 알아보고 적용해볼 수 있었다는 점에서 뜻깊었습니다. 

디자인 시스템을 만들면서 추상화에 대해서 생각해보고, 컴포넌트를 사용하는 개발자의 입장을 염두에 두고 개발하는 것이 색다른 경험인 것 같습니다. `CDS 1.0` 배포까지 모두 힘내서 개발하면 좋겠습니다. 😊  

---

[참고]

- [Compound Component Design Pattern in React](https://betterprogramming.pub/compound-component-design-pattern-in-react-34b50e32dea0)

- [5 Advanced React Patterns](https://javascript.plainenglish.io/5-advanced-react-patterns-a6b7624267a6)

- [chakra-ui / Tabs](https://chakra-ui.com/docs/components/tabs)

- [radix-ui / Tabs](https://www.radix-ui.com/docs/primitives/components/tabs)

- [MDN - HTML / select](https://developer.mozilla.org/ko/docs/Web/HTML/Element/select)

- [ARIA: tab role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/tab_role)

- [ARIA: tablist role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/tablist_role)

- [ARIA: tabpanel role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/tabpanel_role)

- [다른 사람들이 안 알려주는 리액트에서 Context API 잘 쓰는 방법](https://velog.io/@velopert/react-context-tutorial)

- [cds (cold design system)](https://github.com/c-h-w-h/cds)
