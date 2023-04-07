---
layout: post
title: "[Storybook] 스토리북의 Docs 기능으로 컴포넌트 라이브러리 문서화하기"
date: 2023-04-07 00:00:00
author: Hyunbin Lee
categories: Storybook
tags: Storybook Docs DesignSystem
cover: "/assets/instacode.png"
---

# 스토리북의 Docs 기능으로 컴포넌트 라이브러리 문서화하기

> ### 💡 UI 컴포넌트 렌더링을 확인할 때 사용했던 Storybook으로 각 컴포넌트 정보를 문서화한 과정을 정리해봅니다. 

## 1. 문서화 

[지난 포스트](https://iyu88.github.io/react/2023/03/25/react-compound-component-pattern.html)에서 다루었던 CDS(Cold Design System)의 목표 중 하나로 라이브러리 배포를 준비하고 있습니다. 배포 전에 각 컴포넌트들을 리팩토링할 뿐만 아니라 [chakra](https://chakra-ui.com/), [NextUI](https://nextui.org/), [Radix](https://www.radix-ui.com/)같은 라이브러리들처럼 각 컴포넌트에 대한 설명을 담은 문서에 대해서도 신경쓰기로 했습니다. 컴포넌트 라이브러리인 만큼 다른 개발자들이 쉽게 UI를 사용할 수 있게 하려면 컴포넌트와 용례에 대한 정돈된 정보를 제공해야 하는 것이 중요하다고 생각했기 때문입니다. 

처음에는 '직접 소개 홈페이지를 만들까?'에 대한 생각도 했지만, 팀 내부적인 논의 끝에 UI 컴포넌트 렌더링을 확인하기 위해 사용 중이던 `Storybook` 라이브러리에서 제공하는 Docs 기능을 활용하여 컴포넌트를 소개하기로 결정했습니다. 이를 위해서 대대적인 Story 파일 수정 작업이 필요했습니다. 왜냐하면 이전까지는 각자가 개발했던 컴포넌트의 동작을 확인하기 위한 목적으로 각각의 스토리를 작성했기 때문에 전반적으로 최소한의 형식만 지켜진 상태였습니다. 또한 컴포넌트나 스토리에 대한 설명을 고려하지 않은 채로 작업했습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230436959-7ff2f4a2-88cf-46ac-8255-a9060aa309ac.png">
*기존 스토리는 정말 날것 그대로입니다.*
<br />

이제부터 Storybook이 제공하는 기능을 활용하여 각 컴포넌트의 스토리를 정리하고 문서화 작업을 진행한 과정을 되짚어 보겠습니다. 

<br />


## 2. 작업 목표 

먼저 Docs를 수정하기 전에 작업을 진행하면서 신경써야 할 점과 그렇지 않은 점을 구분했습니다.

#### [신경써야 할 점]
  + 각 컴포넌트의 props 타입과 설명을 명시합니다.
  + 컴포넌트 자체에 대한 설명과 각 스토리 설명 추가합니다.
  + 스토리에서 수정할 부분이 있을 경우 수정합니다.

#### [신경쓰지 않을 점]
  + 만약 컴포넌트 자체에 문제가 있을 경우 일단 무시합니다.
    - 발견한 문제를 컴포넌트 차원에서 해결해야 한다면 별도의 PR에서 작업합니다.
  + 현재까지 개발한 컴포넌트에 대해서만 작업합니다. 
    - 가령 Color Palette 등의 새로운 스토리는 추가하지 않습니다.
  + 리팩토링을 진행 중인 컴포넌트는 수정하지 않습니다.

대략적인 작업 목표를 설정했으니 [Storybook](https://storybook.js.org/tutorials/design-systems-for-developers/react/ko/document/)을 참고하면서 코드를 수정해보겠습니다.

<br />

## 3. 작업 내용 


### 3-1. ArgsTable 수정하기 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230456770-448e91ad-54cd-40ab-809f-36575f7cece4.png">
*Docs 탭에서 확인할 수 있는 ArgsTable*
<br />

컴포넌트의 ArgsTable은 각 스토리에서 `args`로 넘겨주고 있는 값을 토대로 props가 자동으로 추론되어 채워집니다. 이렇게 자동으로 추론되는 값을 사용해도 좋지만, 현재 스토리 별로 props를 전달하는 방식이 제각기여서 때론 props의 타입이 정확하게 추론되지 않는 경우도 존재합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230457509-0f761af4-bd79-4a7b-b4b4-aaf68ca357fe.png">
*args에 size가 있는 객체를 명시적으로 전달하면 props와 타입이 잘 추론됩니다.*
<br />

따라서 ArgsTable을 직접적으로 설정하여 혼동을 줄이고자 합니다. 먼저 각각의 props를 명시하기 위해 스토리 객체에서 `argTypes` 속성을 추가합니다. 내부에 각 props의 이름을 Key로 하는 객체를 포함시킵니다. 이해를 돕기 위해 `Spacing.stories.tsx`의 코드를 살펴보겠습니다.

{% highlight jsx%}
// Spacing.stories.tsx
export default {
  title: 'Spacing',
  component: Spacing,
  parameters: {
    layout: 'fullscreen',
  },
  argTypes: {
    size: {}, // 추가
  },
} as ComponentMeta<typeof Spacing>;
{% endhighlight %}

size의 value인 객체 내부에서 size props에 대한 설정을 진행합니다. 사용할 수 있는 값은 다음과 같습니다. 

- ① name : props 이름입니다. 
- ② description : props에 대한 세부 설명을 적습니다. 
- table
  - ③ type : props 타입을 설정합니다. 
  - ④ defaultValue : 초기값이 있으면 명시합니다. 
- ⑤ control : props 값을 수정해보고 싶을 때 어떤 방식으로 제공할 것인지 결정합니다. 
  - type : control의 타입을 설정합니다. 구체적인 옵션은 [공식문서](https://storybook.js.org/docs/react/essentials/controls)를 참고합니다. (매우 다양합니다)

<br />
<img src="https://user-images.githubusercontent.com/31645195/230440569-cbb8543c-f182-4e1d-8f63-11b4008953ee.png">
*실제 ArgsTable에 표시되는 위치*
<br /> 

위 사진을 참고하여 아래 코드처럼 설정하고 ArgsTable을 확인해보겠습니다. 

{% highlight jsx%}
export default {
  // (생략)
  argTypes: {
    size: {
      description: '여백의 크기를 설정합니다.',
      table: {
        type: { summary: 'SpacingVariant' },
      },
      control: {
        type: 'select',
        options: [5, 10, 15, 20, 30, 40, 60, 80, 100],
      },
    },
  },
} as ComponentMeta<typeof Spacing>;
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230461196-fec34744-012b-44da-8ae0-b8b53a2ae253.png">
*Spacing의 수정된 ArgsTable입니다. control type이 select로 설정된 것을 확인할 수 있습니다.*
<br />

<br />

### 3-2. description 추가하기 

서두에 언급했던 다른 라이브러리들처럼 컴포넌트에 대한 설명과 각 용례에 대한 설명을 추가하고자 합니다. 먼저 컴포넌트 설명을 추가하기 위해 스토리 객체에서 다음 두 가지 속성을 사용합니다.

{% highlight jsx%}
// Example.stories.tsx
export default {
  title: 'Example',
  component: Example,
  parameters: {
    layout: 'fullscreen',
    componentSubtitle: // 추가
      'parameters-componentSubtitle은 이곳을 설명합니다.',
    docs: { 
      description: {
        component: // 추가
          `parameters-docs-description-component는 이곳을 설명합니다.`,
      },
    },
  },
} as ComponentMeta<typeof Example>;
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230537127-8085a5a4-6b53-4f4d-a8ed-db7b42f6a266.png">
*componentSubtitle과 description의 위치*
<br />

전체 스토리에서 공통적으로 componentSubtitle은 컴포넌트에 대해 간단하게 `한 줄로 소개하는 역할`로 활용하고, description은 `자세하게 부연설명하는 역할`로 활용했습니다. 

특히, description은 컴포넌트의 props 중에서 사용자(개발자)가 선택할 수 있는 값의 종류를 명시하거나 Compound Component Pattern을 적용한 경우 children에서 사용할 수 있는 컴포넌트에 대해 설명하는 용도로 활용했습니다. 아래 사진을 보면 조금 더 와닿을 것 같습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230537106-b90d3df5-bdcb-4323-8557-60a4a661e15e.png">
*Toast 컴포넌트에서 사용하는 kind, vertical, horizontal에서 선택 가능한 값을 명시하고 있습니다.*
<br /> 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230537110-f2d60056-dbc6-4605-99e3-9377180e2952.png">
*Tabs처럼 합성 패턴을 사용한 컴포넌트에서는 하위 컴포넌트에 대해 설명해줍니다.*
<br /> 

<br /> 

이제 컴포넌트에 대한 소개가 끝났으니 스토리 설명을 추가해보겠습니다. 이 부분도 위 과정과 비슷하게 스토리의 `parameters`에 객체를 할당하는 방식으로 완료할 수 있습니다. 

{% highlight jsx%}
// Example.stories.tsx
export const StoryDescription = Template.bind({});
StoryDescription.args = {};

// parameters 속성에 객체를 할당
StoryDescription.parameters = {
  docs: {
    storyDescription:
      'parameters-docs-storyDescription에서 각 스토리를 설명합니다.',
  },
};
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230537136-db2954c0-7f97-4578-a4ec-d3a2e18d7058.png">
*storyDescription에 있는 문자열이 스토리 설명으로 들어갑니다.*
<br />

스토리 이름은 각 `스토리 변수`를 기반으로 정해집니다. 위 코드에서 `StoryDescription`이라는 변수를 사용했기 때문에 실제 Docs에는 `Story Description`으로 표시됩니다. 만약 변수의 이름과 스토리 Docs에 표시되는 이름을 다르게 설정하고 싶은 경우, 다음과 같이 `storyName` 필드에 값을 할당하여 변경할 수 있습니다. 실제로 이를 Carousel 컴포넌트의 스토리 이름을 변경할 때 사용했습니다. 

{% highlight jsx%}
export const StoryDescription = Template.bind({});
StoryDescription.args = {};

StoryDescription.storyName = 'Custom Story Name'; // 추가
StoryDescription.parameters = {
  docs: {
    storyDescription:
      'parameters-docs-storyDescription에서 각 스토리를 설명합니다.',
  },
};
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230537142-d4b109f9-7215-4249-abec-d8ac8a59e668.png">
*storyName에 할당한 값으로 이름이 변경된 것을 확인할 수 있습니다.*
<br />

<br />

### 3-3. decorators

스토리를 살펴보면 컴포넌트들이 스토리가 표시되는 영역의 중앙에 위치하면 더 좋을 것 같다는 생각이 들었습니다. 실제로 몇몇 컴포넌트에서는 스토리를 잘 확인할 수 있도록 이미 Flexbox나 Center같은 Layout 컴포넌트를 사용하고 있는 것을 확인할 수 있었습니다. 아래 Dropdown 코드처럼 말입니다.

{% highlight jsx%}
// Dropdown.stories.tsx
const Template: ComponentStory<typeof Dropdown> = (args) => (
  <Flexbox
    justifyContent='center'
    alignItems='center'
    css={css`
      width: 300px;
      height: 300px;    
    `}
  >
    <Dropdown {...args}>
      <Dropdown.Trigger>
        <Button text="팀원 목록" icon={MdPeople} variant="light" />
      </Dropdown.Trigger>
      <!-- 생략 -->
      </Dropdown>
  </Flexbox>
);
{% endhighlight %}

하지만 Dropdown과 무관한 코드를 모든 스토리에 적용하는 것은 불필요한 코드를 반복적으로 생성합니다. 이러한 문제를 `decorators` 속성을 활용하여 해결했습니다. decorators는 스토리에 공통적으로 적용하고 싶은 기본 설정을 추가할 수 있도록 도와줍니다. Dropdown을 개선해본다면, 다음과 같이 스토리 객체에 decorators 속성를 추가하고 모든 스토리가 중앙에 위치할 수 있도록 `{Story()}`를 Flexbox 컴포넌트로 감싸줍니다.

{% highlight jsx%}
// Dropdown.stories.tsx
export default {
  title: 'Dropdown',
  // (생략)
  decorators: [
    (Story) => (
      <Flexbox
        justifyContent='center'
        alignItems='center'
        css={css`
          height: 300px;
        `}
      >
        <Center>{Story()}</Center>
      </Flexbox>
    ),
  ],
} as ComponentMeta<typeof Dropdown>;
{% endhighlight %}

코드에서 {Story()}가 바로 \<Dropdown\>이 렌더링되는 위치가 됩니다. 이제 스토리 템플릿을 생성할 때 Layout과 관련된 코드 없이도 처음 코드와 동일하게 동작합니다. 

{% highlight jsx%}
// Dropdown.stories.tsx
const Template: ComponentStory<typeof Dropdown> = (args) => (
    <Dropdown {...args}>
      <Dropdown.Trigger>
        <Button text="팀원 목록" icon={MdPeople} variant="light" />
      </Dropdown.Trigger>
      <!-- 생략 -->
    </Dropdown>
);
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230606182-fa4d6853-94fe-40e7-b87f-00b68c748c9d.png">
*Dropdown 컴포넌트가 정중앙에 배치되었습니다.*
<br />

<br />

### 3-4. 스토리 계층 추가 및 Sidebar 정렬

Storybook을 실행하면 좌측 Sidebar에 확인할 수 있는 컴포넌트 스토리 목록이 나타납니다. 스토리 목록에서도 Layout과 Components로 컴포넌트가 구분되어 있다면 더 확인하기 편할 것 같습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230540741-5a4e4145-011d-4a54-a33f-e5104916e319.png">
*지금 구조에서 상위 계층이 하나 추가되면 좋을 것 같습니다.*
<br />

폴더 경로를 나타내듯이 스토리 객체의 title 속성에 '/'로 구분하여 계층을 추가할 수 있습니다. 앞서 언급한 대로 크게 Layout과 Components, 2가지로 나누도록 하겠습니다. Components도 세부적으로 나눌 수 있겠지만 아직 구체적으로 논의되지 않는 상태이므로 보류합니다. 

{% highlight jsx%}
// Container.stories.tsx
export default {
  title: 'Layout/Container',
  // (생략)
}as ComponentMeta<typeof Container>;
{% endhighlight %}

{% highlight jsx%}
// Table.stories.tsx
export default {
  title: 'Components/Table',
  // (생략)
}as ComponentMeta<typeof Table>;
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230446995-b04623d0-a107-4536-b5ab-65073063e4f8.png">
*의도한대로 잘 구분된 스토리들*
<br />

Layout과 Components로 잘 묶었지만 한 가지 불편한 점은 순서가 뒤죽박죽이라는 점입니다. 컴포넌트의 순서도 명시적으로 정해주겠습니다. 전체 스토리에 대한 설정을 할 수 있는 `preview.jsx`로 이동하여 아래 코드처럼 options 객체를 parameters의 속성으로 추가합니다. 

{% highlight jsx%}
// preview.jsx
export const parameters = {
  // (생략)
  options: {
    storySort: {
      method: 'configure',
      includeNames: true,
      order: [
        'Layout', ['Container', 'Flexbox', 'Center', 'List', '*'],
        'Components', ['Spacing', 'Typography', 'Badge', 'Image', 'Button', 'Input', 'Table', 'RangeSelector', 'Dropdown', 'Select', 'Tabs', 'Carousel', '*', 'Spinner', 'Modal', 'Toast'],
        '*'
      ],
    },
  },
};
{% endhighlight %}

`order`는 배열 내부에 컴포넌트 이름이 배치된 순서대로 Sidebar를 정렬합니다. 앞서 설정했던 스토리의 계층은 중첩된 배열 형태로 설정합니다. 위 코드처럼 `Layout/Container`는 `['Layout', ['Container']]`로 표현합니다. 중간에 `'*'`을 사용하면 순서가 정해지지 않은 컴포넌트들이 위치합니다. Docs를 확인하기 위해서 터미널에서 재실행합니다. (preview.jsx 파일을 수정하면 Storybook을 재실행해야 합니다)

<br />
<img src="https://user-images.githubusercontent.com/31645195/230447263-2fbdda05-51b7-4c03-9c3f-b721e1113245.png">
*코드대로 Sidebar가 정렬되었습니다.*
<br />

여기까지의 작업을 간단하게 정리하면 다음과 같습니다. 

- ArgsTable에 있는 props의 type과 description을 수정했습니다. 
- 컴포넌트와 스토리에 대한 description을 추가했습니다. 
- 공통적인 컴포넌트가 필요한 경우 decorators를 추가했습니다. 
- 스토리에 계층을 부여하고 Sidebar에 있는 컴포넌트에 배치 순서를 부여합니다. 

<br />

## 4. 추가 작업 

기본적인 Storybook 정리 작업은 끝났습니다. 이후에는 지금까지 진행한 작업들에서 조금 더 개선할 수 있는 점들을 고쳐보는 내용을 다루겠습니다. 

### 4-1. category 활용

앞서 `3-2. description 추가하기`에서 잠깐 등장했던 Tabs 컴포넌트는 Compound Component Pattern을 사용했기 때문에 하위 컴포넌트들이 받는 props에 대한 정보도 ArgsTable에 명시되어 있어야 합니다. 가령, Tabs.Trigger가 받을 수 있는 `value`, `text`, `icon`, `disabled`나 Tabs.Panel이 받을 수 있는 `value`에 대한 타입과 설명을 추가하고자 합니다. 일단 기존에 작업한 방식대로 argsType에 새로운 props를 추가해보았습니다. 

{% highlight jsx%}
// Tabs.stories.tsx
export default { 
  // (생략)
  argTypes: {
    value: {
      description: '각각의 Trigger와 Panel을 짝짓기 위한 값입니다.',
      table: {
        type: { summary: 'string', required: true },
      },
    },
    text: {
      description: 'Trigger에 표시할 제목을 설정합니다.',
      table: {
        type: { summary: 'string' },
      },
    },
    icon: {
      description: 'Trigger에 표시할 아이콘을 설정합니다.',
      table: {
        type: { summary: 'string' },
      },
    },
    disabled: {
      description: 'Trigger의 비활성화 여부를 결정합니다.',
      table: {
        type: { summary: 'boolean' },
        defaultValue: { summary: false },
      },
    },
  }
} as ComponentMeta<typeof Tabs>;
{% endhighlight %}

이렇게 하면 하위 컴포넌트들의 props를 추가할 수 있지만 순서가 섞여 있기 때문에 각 props가 어느 컴포넌트를 위한 값인지 알 수 없다는 문제가 있습니다.

<br />
<img src="https://user-images.githubusercontent.com/31645195/230448010-9af06a7b-0aff-45bf-a705-1142c998b9e4.png">
*props가 뒤죽박죽입니다.😵‍💫*
<br />

대안을 찾아보다가 `subcomponents`라는 필드를 스토리 객체에서 사용할 수 있다는 것을 찾았습니다. 하지만 Storybook Issue에 subcomponents가 정상적으로 작동하지 않는다는 [글](https://github.com/storybookjs/storybook/issues/21253)이 있습니다. [댓글](https://github.com/storybookjs/storybook/issues/21253#issuecomment-1476967911)에서는 그 이유를 다음과 같이 설명합니다. 

> subcomponents is deprecated in 7.0. The API is very limiting and hard to improve on. It is somewhat of a half-baked feature, that we want to rethink and do better.

라이브러리에서 subcomponents의 역할과 존재 의미에 대해서 고민하고 있는 것 같습니다. 프로젝트에서는 Storybook 6.5.16 버전을 사용하고 있지만 앞으로 deprecated될 기능을 사용하는 것은 지양하는 것이 좋겠다고 판단했습니다. 다른 방안을 찾아보다가 `category`를 활용하기로 결정했습니다.

먼저 Tabs의 ArgsTable을 Tabs, Tabs.Trigger, Tabs.Panel 이렇게 세 가지로 나누기 위해 다음과 같이 각 props에 category를 정해줍니다. 

{% highlight jsx%}
// Tabs.stories.tsx
export default {
  // (생략)
  argTypes: {
    label: {
      description: '고유한 값으로 접근성 속성에 사용됩니다.',
      table: {
        category: 'Tabs', // 추가
      },
    },
    defaultValue: {
      description: '초기에 활성화되어 있을 Trigger의 value를 입력합니다.',
      table: {
        category: 'Tabs', // 추가
      },
    },
    // (생략)
    text: {
      description: 'Trigger에 표시할 제목을 설정합니다.',
      table: {
        type: { summary: 'string' },
        category: 'Tabs.Trigger', // 추가
      },
    },
    icon: {
      description: 'Trigger에 표시할 아이콘을 설정합니다.',
      table: {
        type: { summary: 'string' },
        category: 'Tabs.Trigger', // 추가
      },
    },
    disabled: {
      description: 'Trigger의 비활성화 여부를 결정합니다.',
      table: {
        type: { summary: 'boolean' },
        defaultValue: { summary: false },
        category: 'Tabs.Trigger', // 추가
      },
    },
  },
} as ComponentMeta<typeof Tabs>;
{% endhighlight %}

Tabs.Trigger와 Tabs.Panel이 사용하는 props 중 value라는 속성 이름이 겹칩니다. value props의 category에 두 컴포넌트 이름을 함께 넣어줍니다.

{% highlight jsx%}
// Tabs.stories.tsx
// (생락)
value: {
  description: '각각의 Trigger와 Panel을 짝짓기 위한 값입니다.',
  table: {
    type: { summary: 'string', required: true },
    category: ['Tabs.Trigger', 'Tabs.Panel'], // 추가
  },
},
// (생락)
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230448451-9f14bfab-08c2-4aa5-aab1-adeaf8a49fc3.png">
*ArgsTable에 category가 생겼습니다.*
<br />

원래 category는 textColor와 backgroundColor를 Color라는 category 아래에 위치시키듯, 비슷한 props 속성들을 묶어주는 용도로 사용합니다. category를 하위 컴포넌트의 props를 표시하는 데에 사용해서 찝찝한 느낌이 들어 여러 자료를 찾아보다가 [이 Stotybook Issue](https://github.com/storybookjs/storybook/issues/20782)을 읽게 되었습니다. 댓글에서는 현재 subcomponents 기능에 대해서 논의하고 있기 때문에 대안으로 `category를 활용하는 방법`을 제시하고 있었습니다.

> ...you can specify categories when writing argTypes, which means you could create a category for each subcomponent and specify the arg types in the categories.

참고로 subcomponents가 deprecated된 이유도 설명하고 있는데,

1. subcomponents가 Docs일 경우에만 작동해서 Canvas나 테스트에서 역할이 없고, 
2. 현재의 구조로는 기능을 추가할 여지가 없다고 합니다.  

이렇게 category를 사용하여 Compound Component Pattern에서 props를 나타냈습니다. 

<br />

### 4-2. Plop JS로 스토리 템플릿 만들기

지금까지 작업한 스토리들은 사전에 약속된 방식으로 작업했다기보다 스스로 판단하여 '이렇게 작성하면 좋겠다'라고 생각하여 작업한 내용입니다. 결국 혼자서 임의로 작성한 형식이기 때문에 앞으로 새로 만들어질 컴포넌트의 스토리는 지금과 다른 형태로 작성될 수도 있습니다. 더불어 ArgsTable에 어떤 내용을 포함하고 어떻게 명시해야 하는지는 문서화 작업을 진행한 사람이 가장 잘 알고 있습니다. 따라서 스토리 작성에 도움이 될 수 있도록 하나의 템플릿을 만들어 형식을 지키게 하고 각 속성에 대한 설명에 간단하게 접근할 수 있으면 좋겠다고 생각했습니다.

겸사겸사 기존에 컴포넌트를 새로 만들 때 불편하게 생각했던 점 중 하나로 React 컴포넌트를 위한 index.tsx와 스토리를 위한 .stories.tsx 파일을 각각 생성해주어야 한다는 것이 있었습니다. 파일의 개수가 많진 않지만 반복적인 작업이 번거로웠습니다.

이를 해결하기 위해서 `Plop JS`를 도입했습니다. Plop JS는 micro-generator 프레임워크로 코드 구조를 간단하게 생성할 수 있도록 도와줍니다. 우선 다음과 같이 기본 설정 파일을 작성합니다. 

{% highlight jsx%}
// plopfile.js
export default function (plop) {
  plop.setGenerator("component", {
    description: "Create a component",
    prompts: [
      {
        type: "input",
        name: "name",
        message: "컴포넌트 이름을 입력해주세요~",
      }
    ],
    actions: [
      // index.tsx를 생성
      {
        type: "add",
        path: "src/components/{% raw %}{{pascalCase name}}{% endraw %}/index.tsx",
        templateFile: "templates/Component.tsx.hbs",
      },
      // stories.tsx를 생성
      {
        type: "add",
        path: "src/components/{% raw %}{{pascalCase name}}{% endraw %}/{% raw %}{{pascalCase name}}{% endraw %}.stories.tsx",
        templateFile: "templates/Story.tsx.hbs",
      },
    ],
  });
}
{% endhighlight %}

터미널에서 사용자(개발자)가 명령어를 입력했을 때 실행하는 동작을 정의합니다. \{\{pascalCase name\}\}으로 되어 있는 부분은 사용자가 CLI로 입력한 값을 파스칼 케이스로 사용하는 부분입니다. 해당 값을 사용하여 폴더를 자동으로 만들고, index.tsx와 .stories.tsx를 생성합니다. 각각의 파일은 templateFile로 설정한 파일을 기반으로 생성됩니다. 여기서는 스토리의 템플릿에 해당하는 `templates/Story.tsx.hbs` 파일만 살펴보도록 하겠습니다. 

{% highlight jsx%}
// Story.tsx.hbs
import { ComponentStory, ComponentMeta } from '@storybook/react';

import {% raw %}{{pascalCase name}}{% endraw %} from '.';

export default {
  title: 'Components/{% raw %}{{pascalCase name}}{% endraw %}',
  component: {% raw %}{{pascalCase name}}{% endraw %},
  parameters: {
    layout: 'fullscreen',
    componentSubtitle:
      '컴포넌트에 대한 간단한 설명을 적습니다.',
    // docs-description-component (Optional) 
    // 세부 설명을 적습니다. 
    docs: {
      description: {
        component: '마크다운 문법이 적용됩니다. \n' + '줄바꿈된 두번째 문장입니다.'
      }
    }
  },
  argTypes: {
    props1: {
      description: 'props1은 default 값이 있는 필수적인 속성입니다.',
      table: {
        // type
        // 필수가 아닐 경우 required를 삭제합니다.
        type: { summary: 'string', required: true },
        // defaultValue (Optional)
        // ex) true, 'string' 등등
        defaultValue: { summary: '기본 값이 있으면 여기에 적습니다.' },
        // category (Optional)
        // Props의 카테고리가 필요할 때 사용합니다.
        // 하나면 문자열, 여러 개면 배열로 전달합니다. 
        category: ['Compound1', 'Compound2'],
      },
      control: {
        // Args Table에서 사용자가 조작할 수 있는 타입입니다.
        // select일 경우 options를 배열로 제공합니다.
        // 그 외 text, number, boolean, color, check, radio 등 존재합니다.
        // control 불가능하게 하고 싶을 경우 객체가 아니라 false로 설정합니다.
        // ex) control: false, 
        // 참고 : https://storybook.js.org/docs/react/essentials/controls
        type: 'select',
        options: ['option1', 'option2'],
      },
    },
  },
  // decorators (Optional)
  // 공통적으로 적용하고 싶은 컴포넌트를 설정합니다.
  decorators: [
    (Story) => (
      <>
        {Story()}
      </>
    ),
  ],
} as ComponentMeta<typeof {% raw %}{{pascalCase name}}{% endraw %}>;

const Template: ComponentStory<typeof {% raw %}{{pascalCase name}}{% endraw %}> = (args) => {
  return (
    <{% raw %}{{pascalCase name}}{% endraw %} {...args}></{% raw %}{{pascalCase name}}{% endraw %}>
  );
};

// Default는 고정해주세요! 필요시 args를 전달합니다.
export const Default = Template.bind({});


export const Example = Template.bind({});
Example.args = {};
Example.storyName = '표시되는 스토리 이름을 변경할 수 있습니다. (변수 이름이 너무 꼬이면 사용해요)';
Example.parameters = {
  docs: {
    storyDescription:
      '스토리에 대한 설명을 적습니다.',
  },
};
{% endhighlight %}

이 글에서 작업했던 내용들이 총출동했습니다. 간단하게 설명하면 컴포넌트의 이름이 필요한 곳에 \{\{pascalCase name\}\}을 사용하고 Storybook 기능과 관련된 부분에 주석으로 간략하게 설명을 남겼습니다. CLI로 `yarn plop`을 실행하여 새로운 컴포넌트를 만드는 상황을 가정해보겠습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230587390-5d5c867e-0917-4e06-aa9a-439205a176c5.gif">
*Hyunbin 컴포넌트를 위한 파일들이 잘 만들어집니다.*
<br />

이제 팀원들을 위한 스토리 템플릿을 제공할 수 있게 되었습니다. Plop JS는 처음 적용해보았는데 다른 곳에서도 유용하게 사용할 수 있을 것 같습니다.

<br />

### 4-3. preview.jsx 옵션 추가 

마지막은 간단한 옵션들을 추가하는 작업입니다. Storybook 전체 설정을 위해 preview.jsx에서 부가적인 옵션을 설정합니다. 가장 먼저 필수적인 props들이 ArgsTable 상단에 위치하도록 하기 위해 controls 객체에 `sort: 'requiredFirst'`로 값을 추가하고 Toast의 ArgsTable을 살펴보겠습니다.

{% highlight jsx%}
// preview.jsx
export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    sort: 'requiredFirst', // 추가
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230589012-00b8a4a3-110f-4c5a-ada5-280da2bfac55.png">
*설정 전에는 required들이 따로 있었습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/230589068-663210b4-f3d0-4292-b9a9-bf36b4d15d99.png">
*requiredFirst로 설정하면 사진처럼 필수값이 먼저 표시됩니다.*
<br />

다음으로 Storybook에 처음 들어왔을 때 Canvas가 아닌 Docs가 나오도록 하고, Canvas는 선택할 수 없도록 변경합니다. 이를 위해 viewMode와 previewTabs를 아래 코드처럼 변경합니다. 

{% highlight jsx%}
// preview.jsx
export const parameters = {
  // (생략)
  previewTabs: {
    canvas: {
      hidden: true, // Canvas는 선택할 수 없도록 막습니다. 
    },
  },
  viewMode: 'docs', // 접속하면 Docs로 진입합니다. 
  // (생략)
};
{% endhighlight %}

<br />
<img src="https://user-images.githubusercontent.com/31645195/230448737-b9bc73b0-d7af-46f0-9234-0ca85a9c117c.png">
*이제 Docs만 보입니다.*
<br />

진행한 추가 작업을 정리하면, 

- category 활용하여 Compound Pattern의 하위 컴포넌트 props를 명시합니다.
- Plop JS로 스토리 템플릿을 생성할 수 있도록 설정합니다. 
- preview.jsx에서 sort, previewTabs, viewMode 값을 설정합니다. 

<br />

## 5. 맺으며 

이전에는 사용하지 않았던 Storybook 기능들을 찾아보며 적용할 수 있는 부분을 최대한 적용하려고 했던 것 같습니다. Storybook 문서에는 이런 말이 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/230592451-6011b5de-c95d-40a6-9ed3-342d8a1adf0d.png">
*맞습니다. 문서화는 힘든 작업이었습니다.*
<br />

Storybook이 도와준 부분이 많지만, 약 15개 정도의 컴포넌트를 작업하다보니 각 컴포넌트에서 빠진 점이 없는지 계속 확인하고 부연설명을 생각해내는 과정이 예상했던 것보다 더 많은 시간이 걸리도록 만든 것 같습니다. 긍정적으로 보면 더 큰 문서화 부채가 쌓이기 전에 진행하여 다행이었습니다. 한편으로는 '만약 개수가 더 많아진다면 어떻게 관리해야 할까?'에 대한 생각도 하게 됩니다. 

진행한 작업이 컴포넌트에 대해서 아주 자세한 설명을 추가하는 것은 아니여서 차차 보완해야 하겠지만, 앞으로의 개발 과정에 도움이 되도록 방향을 잡았다고 생각합니다. 그래서 그런지 작업한 내용들을 보면 보람차고 뿌듯합니다. 

이제는 정말 배포뿐이네요. 😊

---

[참고]

- [Storybook / Tutorials](https://storybook.js.org/tutorials/design-systems-for-developers/react/ko/document/)

- [Storybook / ArgTypes](https://storybook.js.org/docs/react/api/argtypes)

- [Storybook / Controls](https://storybook.js.org/docs/react/essentials/controls)

- [Storybook / Description](https://storybook.js.org/docs/react/api/doc-block-description)

- [Storybook / Decorators](https://storybook.js.org/docs/react/writing-stories/decorators)

- [question / How to order top-level sections?](https://github.com/storybookjs/storybook/issues/6327)

- [Feature Request / Subcomponents support](https://github.com/storybookjs/storybook/issues/20782)

- [Bug / Types for subcomponents in Meta are broken (React)](https://github.com/storybookjs/storybook/issues/21253)

- [Plop: Consistency Made Simple](https://plopjs.com/documentation/)

- [Configure default tab](https://github.com/storybookjs/storybook/issues/12111)