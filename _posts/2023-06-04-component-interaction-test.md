---
layout: post
title: "[Cypress/Storybook] 컴포넌트 인터랙션 테스트"
date: 2023-06-04 00:00:00
author: Hyunbin Lee
categories: Cypress
tags: Cypress E2E_Test Storybook Component_Test Interaction_Test PlayFunction
cover: "/assets/instacode.png"
---

# 컴포넌트 인터랙션 테스트

> ### 💡 컴포넌트 인터랙션을 테스트해보기 위해서 Storybook에서 Cypress와 Play function을 적용한 내용을 정리합니다.

## 1. 들어가며

이전에 작성했던 글([디자인 시스템에 Compound Component Pattern 적용해보기](https://iyu88.github.io/react/2023/03/25/react-compound-component-pattern.html), [스토리북의 Docs 기능으로 컴포넌트 라이브러리 문서화하기](https://iyu88.github.io/storybook/2023/04/07/storybook-docs.html))에 이어서 CDS(Cold Design System)와 관련된 세번째 글을 작성하게 되었습니다. 

CDS를 개발할 때 담당했던 컴포넌트 중 하나인 Slider 컴포넌트는 `<input type='range' />`처럼 사용자가 값을 조절할 수 있는 UI를 제공합니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/09b25e65-a816-426e-9899-3c09d8f8d187" alt="HTML의 <input tpye='range' />와 CDS의 <Slider />">
*HTML의 &lt;input tpye='range' /&gt;와 CDS의 &lt;Slider /&gt;*
<br />

Slider 컴포넌트를 구현하고 PR을 올릴 당시 팀원들에게 항상 했던 말은 '실제로 잘 움직이는지 스토리북에서 꼭 테스트해주세요' 였습니다. 

Slider를 구성하고 있는 요소들이 사용자 이벤트에 의해서 정상적인 위치에 렌더링 되어야 했고, 이를 확인하기 위해서는 직접 검수하는 방법 밖에 없었기 때문입니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/f8fab42f-7cfe-4892-a804-a43a29ae306e" alt="Slider 구성 요소">
*Slider 구성 요소*
<br />

처음에는 마우스 이벤트만 구현했기 때문에 직접 Thumb를 움직이며 값이 정상적으로 변하는지 확인하면 됐지만, 이후 [MDN에서 다루는 키보드 접근성](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/slider_role#keyboard_interactions)을 지원하기 위해 키보드 이벤트도 추가하다보니 확인해야 할 부분이 많아졌습니다. 

Storybook에서 각 스토리마다 인터랙션을 일일이 확인해야 하는 불편한 점을 개선하기 위해 테스트 프레임워크 `Cypress`와 Storybook의 `Play function`을 각각 적용해보고 간단하게 비교해보려고 합니다. 

## 2. Cypress 

### 2-1. Cypress란

Cypress는 실제 사용자가 어플리케이션을 사용하는 브라우저 환경에서 테스트를 수행하는 E2E(End-to-End) 테스트 도구로 많이 알려져 있습니다. 

Cypress 공식 홈페이지에서는 시각적으로 디버깅하고, CI 단계에서 자동으로 실행되는 테스트를 작성할 수 있다고 소개하고 있습니다. 

> With Cypress, you can easily create tests for your modern web applications, debug them visually, and automatically run them in your continuous integration builds.

즉, 어떤 테스트를 진행해야 하는지 코드로 작성하면 이를 브라우저에서 자동으로 실행하면서 오류를 검사해줍니다.

Cypress에서는 [E2E 테스트](https://docs.cypress.io/guides/end-to-end-testing/writing-your-first-end-to-end-test) 이외에도 Component 단위로 테스트할 수 있는 [Component 테스트](https://docs.cypress.io/guides/component-testing/overview)도 제공하고 있습니다. 

E2E 테스트는 URL로 특정 사이트에 방문(visit)하여 테스트를 진행하는 것이고, Component 테스트는 특정 컴포넌트를 생성(mount)하여 테스트합니다. 

여기서는 Storybook으로 작성한 각 스토리에 대해서 Cypress를 적용하는 것이므로 E2E 테스트 방식을 사용합니다.

### 2-2. 환경 설정 

먼저 Cypress를 설치하고 실행합니다. 

<pre>
<code class="hljs shell">yarn add cypress --dev
npx cypress open
</code>
</pre>

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/49d3e49f-c046-4b74-b5b7-062aa3ad5fa5" alt="Cypress 실행 화면">
*Cypress가 열리면 위에서 언급한 것처럼 E2E Testing을 선택합니다.*
<br />

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/38159049-0068-422e-9a25-13be56117f35" alt="Cypress Configuration 파일 목록">
*기본 설정 파일 목록이 나옵니다. Continue를 눌러 계속합니다.*
<br />

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/1c88f677-6ced-427c-bfc9-dc32f2be23fe" alt="Browser 선택 화면">
*테스트가 실행될 브라우저를 선택합니다. Chrome으로 선택한 뒤 초록색 버튼을 누르면 새로운 Chrome 창이 실행됩니다.*
<br />

이후 작성할 테스트 파일에서 에디터가 발생시키는 에러를 방지하기 위해 아래처럼 eslint 플러그인을 설치하고 `.eslintrc`를 수정합니다.

<pre>
<code class="hljs shell">yarn add eslint-plugin-cypress --dev
</code>
</pre>

<pre>
<code class="hljs json">// .eslintrc
{
  "extends": [
    // (생략)
    "plugin:cypress/recommended" // 추가  
  ],
  "plugins": ["react", "@typescript-eslint", "import", "jsx-a11y", "cypress"], // "cypress" 추가
  // (생략)
}
</code>
</pre>

또한 TypeScript를 사용하고 있기 때문에 새롭게 생성된 `cypress` 폴더 내부에서 작성하는 .ts 파일도 인식할 수 있도록 `tsconfig.json` 파일을 수정합니다. 

<pre>
<code class="hljs json">// tsconfig.json
{
  // (생락)
  "include": ["src", "**/*.ts"], // "**/*.ts" 추가
}
</code>
</pre>

Storybook의 스토리를 테스트 대상으로 삼으려면 로컬에서 Storybook과 Cypress가 함께 실행되어야 합니다. 

Storybook을 실행하는 명령어와 `npx cypress open` 명령어를 동시에 실행하기 위해서 `concurrently`와 `wait-on`을 추가로 설치합니다.

- concurrently : 터미널에서 여러 개의 커맨드를 함께 실행할 수 있게 도와주는 라이브러리
- wait-on : 특정 파일, 포트, 소켓이 준비될 때까지 기다려주는 유틸리티 라이브러리 

<pre>
<code class="hljs shell">yarn add concurrently wait-on --dev
</code>
</pre>

package.json에서 다음과 같이 커맨드를 수정하여 `yarn test`만 입력하면 Storybook과 Cypress가 한번에 실행될 수 있도록 합니다. 

<pre>
<code class="hljs json">// package.json
"scripts": {
  // ...
  'dev': "yarn storybook",
  "storybook": "start-storybook -p 6006",
  "test:cy": "cypress open",
  "test": "concurrently 'yarn dev' 'wait-on http://localhost:6006 && yarn test:cy'"
},
</code>
</pre>

`yarn test`를 입력하면 의도한대로 동작하는 것을 확인할 수 있습니다. 

<!-- 커맨드 입력했을 때 gif -->

### 2-3. 간단한 테스트 코드 작성해보기

간단한 테스트 파일을 생성하여 Cypress 동작을 파악해보겠습니다. 

<pre>
<code class="hljs ts">// cypress/init.cy.ts
describe('Initial cypress test', () => { // (1)
  beforeEach(() => { // (2)
    cy.visit('http://localhost:6006'); // (3)
  });

  it('First test case', () => { // (4)
    cy.get("a[title='Storybook']").should('be.visible'); // (5);
  });
});
</code>
</pre>

1. describe() : 어떤 테스트 코드인지 하나로 묶어주는 함수입니다.
2. beforeEach() : 테스트 케이스 하나를 실행하기 전에 매번 실행할 코드를 정의할 수 있습니다.
3. cy.visit() : 인자로 들어간 사이트(테스트를 실행할 URL)를 방문합니다. 
4. it() : 하나의 테스트 코드를 작성합니다.
5. cy.get() : 선택자를 사용하여 DOM 요소를 가져올 수 있습니다. 최상위 요소부터 탐색합니다.
   cy.should() : 체이닝 대상이 어떤 상태가 되어야 하는지 명시합니다. 

즉, 위 코드는 `'http://localhost:6006'으로 접속해서 "a[title='Storybook']" 선택자를 가지고 있는 DOM 요소가 화면에 있는지` 확인하는 테스트입니다. 

Cypress로 실행된 브라우저에서 테스트 파일을 선택하면 Storybook 로고를 찾아서 케이스를 통과한 것을 확인할 수 있습니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/de7a328e-029c-494f-b78b-15852b53e02e" alt="Storybook 로고에 대한 테스트 코드를 통과한 모습">
*빨간색 박스로 표시된 부분에 있는 로고가 정상적으로 렌더링 된 것을 확인해줍니다.*
<br />

### 2-4. DOM 요소 가져오기

Slider 컴포넌트에서 테스트하고자 하는 이벤트와 테스트 케이스를 다음과 같이 정리해보았습니다.

- 마우스 이벤트 
  1. Thumb가 좌측으로 이동했을 때 최솟값으로 설정되는가 
  2. Thumb가 우측으로 이동했을 때 최댓값으로 설정되는가 
- 키보드 이벤트 
  1. 방향키 ⬅️ / ⬇️ 를 눌렀을 때 값이 step만큼 감소하는가
  2. 방향키 ➡️ / ⬆️ 를 눌렀을 때 값이 step만큼 감소하는가
  3. PageDown 키를 눌렀을 때 값이 10% 감소하는가 
  4. PageUp 키를 눌렀을 때 값이 10% 증가하는가
  5. Home 키를 눌렀을 때 최솟값으로 설정되는가 
  6. End 키를 눌렀을 때 최댓값으로 설정되는가

먼저 Slider의 `Default` 스토리를 대상으로 마우스 이벤트 테스트 코드를 작성해보겠습니다.

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
describe('Slider component test', () => {
  beforeEach(() => {
    cy.visit('http://localhost:6006/?path=/docs/components-slider--default'); // (1)
  });

  describe('Default slider', () => {
    it('Click min value', () => {
      cy.get("#story--components-slider--default").should('be.visible'); // (2)
    });
  });
});
</code>
</pre>

(1) localhost:6006에서 실행 중인 스토리에 접근하여 (2) Slider의 DOM 요소를 가져오는 흐름으로 코드를 작성했지만 결과는 실패했습니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/78fbb3f5-55f7-4a33-a8a2-fa7185514551" alt="cy.get()에서 DOM 요소에 접근하지 못해 테스트 케이스가 실패한 모습">
*cy.get()에서 DOM 요소에 접근하지 못해 테스트 케이스가 실패합니다.*
<br />

cy.visit()에서 사용하고 있는 링크에서 DOM 구조를 확인해보면 실제 Slider 컴포넌트가 위치한 곳이 iframe으로 둘러쌓여 있는 것을 확인할 수 있습니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/2ee86086-277b-4190-8011-c153e1986984" alt="개발자 도구에서 DOM 구조를 확인하니 스토리들이 iframe 내부에 있는 모습">
*개발자 도구에서 DOM 구조를 확인하니 스토리들이 iframe 내부에 존재합니다.*
<br />

iframe으로 처리되어 있는 부분을 가져오기 위해서 Cypress 메서드들을 체이닝하여 다소 복잡한 코드를 만들어냈습니다. 

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
describe('Slider component test', () => {
  beforeEach(() => {
    cy.visit('http://localhost:6006/?path=/docs/components-slider--default');
  });

  describe('Default slider', () => {
    it('Click min value', () => {
      cy.get("#storybook-preview-iframe").its('0.contentDocument.body').should('not.be.empty').then(body => cy.wrap(body).find('#story--components-slider--default')).should('be.visible');
    });
  });
});
</code>
</pre>

- cy.its() : 체이닝된 대상으로부터 매개변수에 해당하는 속성 값을 가져옵니다.
- cy.then() : 다음에 수행할 작업을 연속적으로 선언합니다.
- cy.wrap() : cypress 체이닝을 적용하고 싶은 객체를 감싸줍니다. 매개변수가 Promise일 경우 resolve될 때까지 기다립니다. 
- cy.find() : cy.get()처럼 DOM 요소를 찾지만, 최상위가 아닌 현재 체이닝된 대상부터 탐색합니다. 

위 메서드들을 사용하여 `#storybook-preview-iframe`에 해당하는 부분을 찾아서 iframe 내부에 DOM 요소를 불러왔을 때 `#story --components-slider--default`를 id로 하는 Slider가 존재하는지 확인합니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/fdd2dbab-314e-4334-8226-2c7ab7bfa157" alt="Default Slider가 렌더링 되었는지 확인하는 테스트가 통과한 모습">
*Default Slider가 렌더링 되었는지 확인하는 테스트가 통과했습니다.*
<br />

결과적으로는 테스트 대상이 되는 DOM을 불러올 수 있었지만 복잡하고 불필요한 정보가 코드에 노출되어 있습니다. 

이를 Cypress의 `Custom Commands` 기능을 활용하여 간단하게 변경하고, 추가적으로 스토리에 접근하는 cy.visit() 부분도 별도의 커맨드로 정의했습니다. 

cypress/support 폴더에 있는 `commands.ts` 파일에서 추가합니다. 

<pre>
<code class="hljs ts">// cypress/support/commands.ts
Cypress.Commands.add("visitStory", (componentName, storyName) => {
  cy.visit(`http://localhost:6006/iframe.html?id=components-${componentName}--${storyName}&viewMode=story`)
})

Cypress.Commands.add("getStory", (componentName, storyName) => {
  return cy.get("#storybook-preview-iframe").its('0.contentDocument.body').should('not.be.empty').then(body => cy.wrap(body).find(`#story--components-${componentName}--${storyName}`));
})
</code>
</pre>

이렇게 추가한 커맨드들의 타입을 정의해주기 위해 동일한 경로에 `index.d.ts`을 생성합니다. 

<pre>
<code class="hljs ts">// cypress/support/index.d.ts
declare namespace Cypress {
  interface Chainable {
    visitStory(path: string): void,
    getStory(componentName: string, storyName: string): Chainable<JQuery<HTMLElement>>
  }
}
</code>
</pre>

이를 통해서 앞으로 `cy.visitStory()`를 사용하여 보다 짧은 코드로 스토리에 접근하고, `cy.getStory()`를 사용하여 간편하게 각 스토리 DOM에 접근할 수 있습니다. 

추가적으로 cy.as() 메서드를 사용해서 cy.getStory()로 찾은 DOM 요소에 alias를 지정해주었습니다. 

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
describe('Slider component test', () => {
  beforeEach(() => {
    cy.visitStory('components-slider');
    cy.getStory('slider', 'default').as('slider');
  });

  describe('Default slider', () => {
    it('Click min value', () => {
      // ...
    });
  });
});
</code>
</pre>

.as()에 인자로 `slider`를 전달했기 때문에 이후부터 `#story--components-slider--default`에 접근할 때 `@slider`라는 간단한 이름을 사용할 수 있습니다. 

### 2-5. 마우스 이벤트 테스트하기 

이제 Slider 하위에 있는 `track`과 `thumb`를 찾아서 실제로 track의 가장 왼쪽을 클릭했을 때 thumb에 표시되는 값이 최솟값이 되는지 확인해보겠습니다. 

두 요소를 가져오기 위해서 id 속성 값을 상수로 선언하고 alias도 지정합니다. 

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
const SLIDER_PREFIX = '#cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

describe('Slider component test', () => {
  beforeEach(() => {
    cy.visitStory('components-slider');
    cy.getStory('slider', 'default').as('slider');
    cy.get('@slider').find(TRACK_ID).as('track');
    cy.get('@slider').find(THUMB_ID).as('thumb');
  });

  describe('Default slider', () => {
    it('Click min value', () => {
      cy.get('@track').click('left');
      cy.get('@thumb').invoke('text').should('eq', '0');  
    });
  });
});
</code>
</pre>

click()의 인자로 `left`를 전달하여 DOM 요소의 가장 왼쪽 좌표를 클릭하도록 했고, 이후 thumb 내부에 있는 text가 최솟값인 0으로 설정되었는지 확인합니다. cy.invoke('text')를 사용해서 텍스트 노드 값을 얻을 수 있습니다. 

다음과 같이 Cypress에서 성공적으로 실행된 것을 확인할 수 있습니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/3bb451ff-62cf-4163-acd5-b123f29e2123" alt="'Click Min Value' 테스트 케이스를 통과한 모습">
*'Click Min Value' 테스트 케이스를 통과했습니다.*
<br />

추가적으로 마우스 이벤트를 실행하고 나서 실제로 `filled`와 thumb의 위치가 일치하는지 확인하기 위해 `shouldBeEqualPercentage()` 함수를 정의했습니다. 

더불어서 alias를 진행하는 코드 역시 `initAlias()` 함수로 분리합니다.

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
const SLIDER_PREFIX = '#cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const FILLED_ID = `${SLIDER_PREFIX}-filled`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

describe('Slider component test', () => {
  const initAlias = (storyName: string) => {
    cy.getStory('slider', storyName).as('slider');
    cy.get('@slider').find(TRACK_ID).as('track');
    cy.get('@slider').find(FILLED_ID).as('filled');
    cy.get('@slider').find(THUMB_ID).as('thumb');
  }

  // 추가
  const shouldBeEqualPercentage = (dimension: FilledDimension, position: ThumbPosition) => {
    cy.get('@filled').then(([ filledElement ]) => {
      cy.get('@thumb').then(([ thumbElement ]) => {
        const filledStyle = window.getComputedStyle(filledElement)[dimension];
        const thumbStyle = window.getComputedStyle(thumbElement)[position];
        expect(filledStyle).equal(thumbStyle);
      });
    });
  }

  beforeEach(() => {
    cy.visitStory('components-slider');
  });

  describe('Default slider', () => {
    beforeEach(() => {
      initAlias('default');
    })

    // 추가 
    afterEach(() => {
      shouldBeEqualPercentage('width', 'left');
    })

    it('Click min value', () => {
      cy.get('@track').click('left');
      cy.get('@thumb').invoke('text').should('eq', '0');
    });
  });
});
</code>
</pre>

shouldBeEqualPercentage() 함수에서 각각 filled와 thumb에서 확인하고 싶은 CSS 속성 이름을 받아서 두 값이 동일한지 확인합니다. 

가령 thumb가 track의 정중앙에 위치한다면 thumb는 `left: 50%` 인 상태이기 때문에 filled는 `width: 50%`여야 합니다. 

left와 width에 대해서 두 속성 값이 50%로 같은지 확인하기 위해서 window.getComputedStyle()로 브라우저 상에서 CSS 속성 값을 가져와 서로 같은지 확인합니다. 

이 함수는 각각의 테스트 케이스가 실행된 뒤에 공통적으로 실행해야 하는 함수이기 때문에 `afterEach()` 라는 테스트 사이클에서 실행합니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/d0face00-1709-4a5b-8201-73bc9f157665" alt="Cypress 실행 결과 shouldBeEqualPercentage() 함수까지 통과한 모습">
*전체적인 테스트 흐름을 완성했습니다.*
<br />

### 2-6. 키보드 이벤트 테스트하기 

키보드 이벤트 테스트는 cy.trigger()를 통해서 진행할 수 있습니다. 

첫번째 인자로 키보드 이벤트 종류, 두번째 인자로 keyCode를 전달합니다. 

다음 코드는 오른쪽 방향키를 눌러서 Slider 값이 step만큼 증가한 것을 테스트하는 코드입니다. (여기서 step 값은 기본 값으로 1입니다)

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
describe('Slider component test', () => {
  // (생락)

  describe('Default slider', () => {
    beforeEach(() => {
      initAlias('default');
    })

    // (생략)

    it('Increase with right arrow', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 39 });
      cy.get('@thumb').invoke('text').should('eq', '51');
    });
  });
});
</code>
</pre>

이렇게 추가한 키보드 이벤트도 Cypress에서 정상적으로 통과합니다. 나머지 테스트 케이스 (최댓값 클릭, 방향키 입력, 특수키 입력)도 추가한 최종적인 Default Slider에 대한 테스트 코드는 다음과 같습니다. 

<pre>
<code class="hljs ts">// cypress/e2e/components/slider.cy.ts
const SLIDER_PREFIX = '#cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const FILLED_ID = `${SLIDER_PREFIX}-filled`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

describe('Slider component test', () => {
  const initAlias = (storyName: string) => {
    cy.getStory('slider', storyName).as('slider');
    cy.get('@slider').find(TRACK_ID).as('track');
    cy.get('@slider').find(FILLED_ID).as('filled');
    cy.get('@slider').find(THUMB_ID).as('thumb');
  }

  const shouldBeEqualPercentage = (dimension: FilledDimension, position: ThumbPosition) => {
    cy.get('@filled').then(([ filledElement ]) => {
      cy.get('@thumb').then(([ thumbElement ]) => {
        const filledStyle = window.getComputedStyle(filledElement)[dimension];
        const thumbStyle = window.getComputedStyle(thumbElement)[position];
        expect(filledStyle).equal(thumbStyle);
      });
    });
  }

  beforeEach(() => {
    cy.visitStory('components-slider');
  });

  describe('Default slider', () => {
    beforeEach(() => {
      initAlias('default');
    });

    afterEach(() => {
      shouldBeEqualPercentage('width', 'left');
    });

    it('Click min value', () => {
      cy.get('@track').click('left');
      cy.get('@thumb').invoke('text').should('eq', '0');
    });

    it('Click max value', () => {
      cy.get('@track').click('right');
      cy.get('@thumb').invoke('text').should('eq', '100');
    });

    it('Increase with right arrow', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 39 });
      cy.get('@thumb').invoke('text').should('eq', '51');
    });

    it('Increase with up arrow', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 38 });
      cy.get('@thumb').invoke('text').should('eq', '51');
    });

    it('Decrease with left arrow', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 37 });
      cy.get('@thumb').invoke('text').should('eq', '49');
    });

    it('Decrease with down arrow', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 40 });
      cy.get('@thumb').invoke('text').should('eq', '49');
    });

    it('Increase with page up', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 33 });
      cy.get('@thumb').invoke('text').should('eq', '60');
    });

    it('Decrease with page down', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 34 });
      cy.get('@thumb').invoke('text').should('eq', '40');
    });

    it('Set min with home key', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 36 });
      cy.get('@thumb').invoke('text').should('eq', '0');
    });

    it('Set max with end key', () => {
      cy.get("@thumb").trigger('keydown', { keyCode : 35 });
      cy.get('@thumb').invoke('text').should('eq', '100');
    });
  });
});
</code>
</pre>

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/ab250d2b-c027-4865-ad8d-ce866dc186c0" alt="Default Slider에 대해서 작성한 10가지 테스트 케이스를 모두 통과한 모습">
*Default Slider에 대해서 작성한 10가지 테스트 케이스를 모두 통과했습니다.*
<br />

Default Slider 스토리를 제외한 다른 스토리에도 테스트 코드를 적용해줍니다. 단, 모든 케이스를 작성하진 않고 동작을 명확하게 하고 싶은 테스트 케이스가 있을 경우에만 적용했습니다. 예를 들어 초기값이 0으로 설정되어 있는 `Start from Zero Slider`라는 스토리에서는 thumb를 좌측으로 움직여도 최솟값 이하로 값이 줄어들지 않는지 확인하기 위해서 관련된 테스트 케이스들만 적용했습니다. 

이렇게 작성한 모든 테스트 케이스를 전부 통과한 것을 확인할 수 있습니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/e8b494c0-45fc-463b-9dc3-c4cd34d02446" alt="Slider의 8가지 스토리에 대해 추가한 인터랙션 테스트가 모두 통과한 모습">
*Slider의 8가지 스토리에 대해 추가한 인터랙션 테스트가 모두 통과했습니다.*
<br />

## 3. Storybook Play function

이렇게 Cypress를 적용하던 중, 동료 팀원 분께서 Storybook에서 제공하는 [Play function](https://storybook.js.org/docs/react/writing-stories/play-function#page-top)으로 인터랙션 테스트를 진행할 수 있다고 알려주셨습니다. 

Play function은 컴포넌트 스토리 파일에서 testing-library, jest 문법을 사용하여 인터랙션 시나리오를 테스트할 수 있도록 도와줍니다. 

Cypress에서 진행했던 테스트 케이스들을 동일하게 수행해보았습니다. 


### 3-1. 환경 설정

[Storybook Addon Interactions](https://storybook.js.org/addons/@storybook/addon-interactions)을 설치하면 더욱 수월하게 디버깅을 진행할 수 있습니다. 아래의 라이브러리들을 설치합니다. 

<pre>
<code class="hljs shell">yarn add -D @storybook/addon-interactions @storybook/jest @storybook/testing-library
</code>
</pre>

설치한 addon을 사용하기 위해서 main.cjs 파일에서 addons 배열에 추가합니다. 

<pre>
<code class="hljs js">// .storybook/main.cjs
module.exports = {
  addons: [
    '@storybook/addon-interactions', // 추가
  ],
  // (생략)
};
</code>
</pre>

작성한 테스트를 터미널에서 실행시킬 수 있도록 [Storybook Test Runner](https://storybook.js.org/addons/@storybook/test-runner)도 설치해줍니다. 

<pre>
<code class="hljs shell">yarn add -D @storybook/test-runner
</code>
</pre>

test-runner로 테스트를 진행하기 위해서는 로컬에서 Storybook이 실행되고 있어야 합니다. 따라서 위에서 Cypress를 설치할 때 사용했던 concurrently와 wait-on을 활용해서 명령어를 만들어 줍니다. 

<pre>
<code class="hljs json">// package.json
"scripts": {
  // ...
  "dev": "yarn storybook",
  "storybook": "start-storybook -p 6006",
  "test-storybook": "test-storybook",
  "test:play": "concurrently 'yarn dev' 'wait-on http://localhost:6006 && yarn test-storybook'"
},
</code>
</pre>

이제 테스트 코드를 작성한 뒤에 `yarn test:play` 명령어를 사용해서 테스트 결과를 확인할 수 있습니다. 

### 3-2. Play function 작성하기 

Cypress에서는 별도의 파일을 생성하고 describe(), it()과 같은 메서드를 사용해서 각 테스트 케이스를 정의했지만, 여기서는 각 스토리에 대해서 `play` 속성을 정의하여 테스트하고자 하는 인터랙션 로직을 한꺼번에 작성합니다. 

Cypress에서는 iframe 내부에 있는 요소에 접근하기 위해서 별도의 Custom Commands를 작성했지만, play function을 사용한다면 아래 코드처럼 함수에 매개변수로 전달되는 `canvasElement`로 손쉽게 접근할 수 있습니다. 

해당 값과 `@storybook/testing-library`에서 제공하는 getByRole, getByText 등의 다양한 메서드를 조합하여 DOM 요소를 가져올 수 있습니다. 

<pre>
<code class="hljs tsx">// src/components/Dummy/Dummy.stories.tsx
import { getByText } from '@storybook/testing-library';

// (생략)

const Template: ComponentStory&lt;typeof Dummy&gt; = (args) => &lt;Dummy {...args} /&gt;;

export const Default = Template.bind({});
Default.args = { ...DEFAULT_PROPS };

Default.play = async ({ canvasElement }) => {
  const dummy = getByText(canvasElement, "dummy"); // text가 "dummy"인 DOM 요소를 가져옵니다. 

  // 테스트 코드 작성
};
</code>
</pre>

여기서 Storybook 버전에 따라서 테스트 코드를 작성하는 방법이 두 가지로 나뉩니다. 

첫번째로 Storybook 7버전부터 지원하는 CSF3 포맷을 따르는 스토리 파일에서는 다음과 같이 `step` 을 사용해서 각각의 테스트 케이스를 작성할 수 있습니다. 

[참고](https://storybook.js.org/docs/react/essentials/interactions)

<pre>
<code class="hljs tsx">// src/components/Dummy/Dummy.stories.tsx
import { getByText } from '@storybook/testing-library';

// (생략)

const Template: ComponentStory&lt;typeof Dummy&gt; = (args) => &lt;Dummy {...args} /&gt;;

export const Default = Template.bind({});
Default.args = { ...DEFAULT_PROPS };

Default.play = async ({ canvasElement, step }) => { // step 추가
  const dummy = getByText(canvasElement, "dummy");

  await step('테스트 제목', async () => {
    // 테스트 코드 작성
  });
};
</code>
</pre>

이렇게 하면 마치 테스팅 라이브러리에서 describe()를 사용하는 것처럼 각각의 테스트 케이스에 대한 라벨을 Storybook의 `Interactions` 탭에서 확인할 수 있습니다. 

반면, Storybook 7 이전 버전을 사용하고 있을 경우 CSF2 포맷을 따르기 때문에 step을 사용할 수 없습니다. 

따라서 별도로 테스트 케이스를 감싸주는 함수를 사용하지 않습니다. 

[참고](https://storybook.js.org/docs/6.5/react/essentials/interactions#snippet-storybook-interactions-play-function)

<pre>
<code class="hljs tsx">// src/components/Dummy/Dummy.stories.tsx
import { getByText } from '@storybook/testing-library';

// (생략)

const Template: ComponentStory&lt;typeof Dummy&gt; = (args) => &lt;Dummy {...args} /&gt;;

export const Default = Template.bind({});
Default.args = { ...DEFAULT_PROPS };

Default.play = async ({ canvasElement }) => {
  const dummy = getByText(canvasElement, "dummy");

  // 테스트 코드 작성
};
</code>
</pre>

현재 프로젝트에서는 Storybook 6.5 버전을 사용하고 있기 때문에 각 테스트 케이스를 구분하기 위해서 주석을 사용했습니다. 

Cypress에서 진행했던 것과 동일하게 Default Slider를 대상으로 하여 마우스 이벤트부터 테스트해보겠습니다. 

### 3-3. 마우스 이벤트 테스트하기

먼저 Slider를 구성하는 DOM 요소를 가져오기 위한 코드부터 작성합니다. 

Slider의 하위 요소를 id 속성으로 가져오기 위해서 canvasElement 속성과 `queryByAttribute()` 함수를 사용합니다.

<pre>
<code class="hljs tsx">// src/components/Slider/Slider.stories.tsx
import { queryByAttribute } from '@storybook/testing-library';

// (생략)

const SLIDER_PREFIX = 'cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

Default.play = async ({ canvasElement }) => {
  const thumb = queryByAttribute('id', canvasElement, THUMB_ID);
  const track = queryByAttribute('id', canvasElement, TRACK_ID);

  if (!thumb || !track) return;

  // 마우스 이벤트 작성 
};
</code>
</pre>

인터랙션 이벤트를 발생시키기 위해서 `@storybook/testing-library`에서 제공하는 함수 중 userEvent와 fireEvent를 사용할 수 있습니다. 

useEvent와 달리 fireEvent는 마우스 이벤트를 발생시킬 정확한 좌표를 옵션으로 지정할 수 있기 때문에 fireEvent를 사용합니다.

다음은 thumb를 track의 가장 좌측 좌표(left)로 옮겼을 때 최솟값으로 설정되는지 확인하는 코드입니다. 

<pre>
<code class="hljs tsx">// src/components/Slider/Slider.stories.tsx
import { expect } from '@storybook/jest';
import { queryByAttribute, fireEvent, waitFor } from '@storybook/testing-library';

// (생략)

const SLIDER_PREFIX = 'cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

Default.play = async ({ canvasElement }) => {
  const thumb = queryByAttribute('id', canvasElement, THUMB_ID);
  const track = queryByAttribute('id', canvasElement, TRACK_ID);

  if (!thumb || !track) return;

  const { x, y } = thumb.getBoundingClientRect();
  const { left, right } = track.getBoundingClientRect();

  // Drag min value
  await fireEvent.mouseDown(thumb, { clientX: x, clientY: y });
  await fireEvent.mouseMove(thumb, { clientX: left, clientY: y });
  await fireEvent.mouseUp(thumb);
  await waitFor(() => expect(thumb.textContent).toEqual('0'));
};
</code>
</pre>

fireEvent를 연속으로 호출할 경우 각 이벤트의 순서가 보장되지 않을 수 있기 때문에 await 키워드를 붙여서 실행합니다. 

최종적으로 값을 검증하는 expect()문은 `waitFor()`로 감싸서 mouseUp 이벤트 이후에 실행될 수 있도록 했습니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/2b37199e-c3aa-4822-a4c0-74da15377a70" alt="'Drag min value'를 수행하기 위한 이벤트들이 기록된 Interactions 탭의 모습">
*'Drag min value'를 수행하기 위한 이벤트들을 Interactions 탭에서 확인할 수 있습니다.*
<br />

drag 동작을 위해 반복되는 여러 개의 fireEvent를 하나로 묶어주기 위해 `simulateDragEvent()` 함수로 분리합니다.

<pre>
<code class="hljs tsx">// src/components/Slider/Slider.stories.tsx
import { expect } from '@storybook/jest';
import { queryByAttribute, fireEvent, waitFor } from '@storybook/testing-library';

// (생략)

const SLIDER_PREFIX = 'cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

// 타입 추가 
type KeyName = 'ArrowRight' ;

// 추가 
const simulateDragEvent = async (target: HTMLElement, fromX: number, fromY: number, toX: number, toY: number) => {  
  await fireEvent.mouseDown(target, { clientX: fromX, clientY: fromY });
  await fireEvent.mouseMove(target, { clientX: toX, clientY: toY });
  await fireEvent.mouseUp(target);
}

Default.play = async ({ canvasElement }) => {
  const thumb = queryByAttribute('id', canvasElement, THUMB_ID);
  const track = queryByAttribute('id', canvasElement, TRACK_ID);

  if (!thumb || !track) return;

  const { x, y } = thumb.getBoundingClientRect();
  const { left, right } = track.getBoundingClientRect();

  // Drag min value
  await simulateDragEvent(thumb, x, y, left, y);
  await waitFor(() => expect(thumb.textContent).toEqual('0'));
};
</code>
</pre>

### 3-4. 키보드 이벤트 테스트하기

다음으로 키보드 인터랙션을 추가합니다. 

특정 키를 입력하는 동작은 fireEvent.keyDown()과 fireEvent.keyUp()을 연달아 실행하여 구현합니다.

simulateDragEvent()를 정의한 것처럼 `simulateKeyboardEvent()` 함수를 적용하여 오른쪽 방향키를 입력했을 때의 테스트 코드를 작성했습니다. 

<pre>
<code class="hljs tsx">// src/components/Slider/Slider.stories.tsx
import { expect } from '@storybook/jest';
import { queryByAttribute, fireEvent, waitFor } from '@storybook/testing-library';

// (생략)

const SLIDER_PREFIX = 'cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

// 타입 추가 
type KeyName = 
  'ArrowRight' 
  | 'ArrowUp' 
  | 'ArrowLeft' 
  | 'ArrowDown'
  | 'PageUp' 
  | 'PageDown'
  | 'Home' 
  | 'End';

// 함수 추가 
const simulateKeyboardEvent = async (target: HTMLElement, key: KeyName) => {
  await fireEvent.keyDown(target, { key });
  await fireEvent.keyUp(target, { key });
}

Default.play = async ({ canvasElement }) => {
  const thumb = queryByAttribute('id', canvasElement, THUMB_ID);
  const track = queryByAttribute('id', canvasElement, TRACK_ID);

  if (!thumb || !track) return;

  const { x, y } = thumb.getBoundingClientRect();
  const { left, right } = track.getBoundingClientRect();

  // Increase with right arrow
  await simulateKeyboardEvent(thumb, 'ArrowRight');
  await waitFor(() => expect(thumb.textContent).toEqual('51'));
};
</code>
</pre>

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/b637bf29-08a0-497d-a95c-a8ac66a9dfb1" alt="'Increase with right arrow'를 수행하기 위한 이벤트들이 Interactions 탭에 나타난 모습">
*'Increase with right arrow'를 수행하기 위한 이벤트들을 Interactions 탭에서 확인할 수 있습니다.*
<br />

나머지 방향키와 특수키를 입력했을 때의 테스트 코드를 추가한 코드는 다음과 같습니다. 

(자세한 테스트 케이스에 대한 설명은 Cypress에서 작성했던 것과 동일하기 때문에 부연 설명을 줄이겠습니다)

<pre>
<code class="hljs tsx">// src/components/Slider/Slider.stories.tsx
import { expect } from '@storybook/jest';
import { queryByAttribute, fireEvent, waitFor } from '@storybook/testing-library';

// (생략)

const SLIDER_PREFIX = 'cds_Slider-slider';
const TRACK_ID = `${SLIDER_PREFIX}-track`;
const THUMB_ID = `${SLIDER_PREFIX}-thumb`;

const simulateDragEvent = async (target: HTMLElement, fromX: number, fromY: number, toX: number, toY: number) => {  
  await fireEvent.mouseDown(target, { clientX: fromX, clientY: fromY });
  await fireEvent.mouseMove(target, { clientX: toX, clientY: toY });
  await fireEvent.mouseUp(target);
}

type KeyName = 
  'ArrowRight' 
  | 'ArrowUp' 
  | 'ArrowLeft' 
  | 'ArrowDown'
  | 'PageUp' 
  | 'PageDown'
  | 'Home' 
  | 'End';

const simulateKeyboardEvent = async (target: HTMLElement, key: KeyName) => {
  await fireEvent.keyDown(target, { key });
  await fireEvent.keyUp(target, { key });
}

Default.play = async ({ canvasElement }) => {
  const thumb = queryByAttribute('id', canvasElement, THUMB_ID);
  const track = queryByAttribute('id', canvasElement, TRACK_ID);

  if (!thumb || !track) return;

  const { x, y } = thumb.getBoundingClientRect();
  const { left, right } = track.getBoundingClientRect();

  // Increase with right arrow
  await simulateKeyboardEvent(thumb, 'ArrowRight');
  await waitFor(() => expect(thumb.textContent).toEqual('51'));

  // Increase with up arrow
  await simulateKeyboardEvent(thumb, 'ArrowUp');
  await waitFor(() => expect(thumb.textContent).toEqual('52'));

  // Decrease with left arrow
  await simulateKeyboardEvent(thumb, 'ArrowLeft');
  await waitFor(() => expect(thumb.textContent).toEqual('51'));

  // Decrease with down arrow
  await simulateKeyboardEvent(thumb, 'ArrowDown');
  await waitFor(() => expect(thumb.textContent).toEqual('50'));

  // Increase with page up
  await simulateKeyboardEvent(thumb, 'PageUp');
  await waitFor(() => expect(thumb.textContent).toEqual('60'));

  // Decrease with page down
  await simulateKeyboardEvent(thumb, 'PageDown');
  await waitFor(() => expect(thumb.textContent).toEqual('50'));

  // Set min with home key
  await simulateKeyboardEvent(thumb, 'Home');
  await waitFor(() => expect(thumb.textContent).toEqual('0'));

  // Set max with end key
  await simulateKeyboardEvent(thumb, 'End');
  await waitFor(() => expect(thumb.textContent).toEqual('100'));

  // Drag min value
  await simulateDragEvent(thumb, x, y, left, y);
  await waitFor(() => expect(thumb.textContent).toEqual('0'));

  // Drag max value
  await simulateDragEvent(thumb, x, y, right, y);
  await waitFor(() => expect(thumb.textContent).toEqual('100'));
};
</code>
</pre>

나머지 스토리에 대해서도 필요한 테스트 케이스만 추가한 뒤 처음에 정의했던 yarn test:play 명령어를 실행합니다.

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/dfe397b6-4971-4422-a693-06c51f3af28c" alt="터미널에서 프로젝트 폴더에 있는 모든 stories 파일에 대해 테스트를 실행한 모습">
*프로젝트 폴더에 있는 모든 스토리 파일에 대해 테스트를 실행합니다.*
<br />

이렇게 Storybook에서 제공하는 play function 기능도 활용하여 Slider 컴포넌트에서 마우스와 키보드 인터랙션에 대한 테스트를 작성해보았습니다. 


## 4. Cypress와 Storybook interaction addon 간단 비교 

간단한 기능들만 사용해보았지만 짧게 Cypress와 Storybook Addon Interaction을 비교해보았습니다. 

- Cypress 


- Storybook Addon Interaction


## 5. Cypress 수정사항 

### 5-1. 오류 발생 

Cypress 관련 PR을 올렸는데 이런 코멘트가 달렸습니다. 

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/16d1904e-9743-4dbf-8420-55cd53feb868" alt="실행할 때마다 실패하는 테스트 케이스가 달라진다는 내용의 GitHub PR 코멘트">
*실행할 때마다 실패하는 테스트 케이스가 달라진다는 말씀*
<br />

여러 팀원들이 동일한 증상을 제보해주셔서 코드에 수정이 필요했습니다. 

의심되는 부분은 iframe 내용을 불러오는 시간이 Cypress에서 기본적으로 비동기 응답을 기다리는 시간보다 오래 걸려서 iframe 내부에 있는 DOM 요소를 불러오지 못해 테스트에 실패하는 것이라고 생각했습니다. 

즉, iframe 내부 DOM 요소를 가져오지 못한 채로 initAlias() 함수를 실행하니 오류가 발생하는 것이라고 판단했습니다.


### 5-2. visitStory 커맨드 수정

[이 글](https://ui.toast.com/posts/ko_20220111#e2e-%ED%85%8C%EC%8A%A4%ED%8C%85-%EB%8F%84%EA%B5%AC%EC%99%80-%ED%95%A8%EA%BB%98)에 있는 내용에서 URL로 각 스토리에 접근할 수 있다는 것을 알 수 있었습니다. 

기존 코드에서는 컴포넌트 스토리 페이지의 iframe 내부에서 인터랙션 테스트를 진행했다면, 이제는 iframe 내부를 URL로 직접 접근하여 인터랙션 테스트를 진행하도록 변경해야겠다고 생각했습니다.

이렇게 하면 원하는 스토리가 확실하게 렌더링된 URL에 접근하여 DOM 요소를 발견하지 못하는 결과가 발생하지 않을 것이기 때문입니다.

<pre>
<code class="hljs ts">// cypress/support/commands.ts
// 기존 코드
Cypress.Commands.add("visitStory", (path) => {
  cy.visit(`http://localhost:6006/?path=/docs/${path}`);
})

// 변경된 코드
Cypress.Commands.add("visitStory", (componentName, storyName) => {
  cy.visit(`http://localhost:6006/iframe.html?id=components-${componentName}--${storyName}&viewMode=story`)
})
</code>
</pre>

새로 변경한 URL로 접속해서 개발자 도구로 DOM Tree를 살펴보면 iframe 내부에 있던 `#root`가 존재하는 것을 확인할 수 있습니다. 

이로 인해 initAlias() 함수에서는 이제 #root 요소만 찾으면 되기 때문에 별도의 매개변수를 받을 필요가 없어졌습니다. 

또한 테스트 파일에서 visitStory()를 호출하는 위치도 변경했습니다. 

<pre>
<code class="hljs ts">// slider.cy.ts
describe('Slider component test', () => {
  const initAlias = () => { // 함수 수정
    cy.get('#root').as('root'); 
    cy.get('@root').find(TRACK_ID).as('track');
    cy.get('@root').find(FILLED_ID).as('filled');
    cy.get('@root').find(THUMB_ID).as('thumb');
  }

  const shouldBeEqualPercentage = (dimension: FilledDimension, position: ThumbPosition) => {...}

  // 공통 beforEach 삭제

  describe('Default slider', () => {
    beforeEach(() => {
      cy.visitStory('slider', 'default'); // 추가
      initAlias();
    });

    afterEach(() => {
      shouldBeEqualPercentage('width', 'left');
    });

    it('Click min value', () => {
      cy.get('@track').click('left');
      cy.get('@thumb').invoke('text').should('eq', '0');
    });

    // (생략)
    });
});
</code>
</pre>

<br />
<img src="https://github.com/iyu88/Algorithm/assets/31645195/3b7ac296-06f8-43a1-bbd5-067a91bd35f2" alt="테스트 실행 시간이 66초에서 17초로 줄어든 모습">
*동일하게 모든 테스트 케이스를 통과하면서 실행 시간이 크게 줄어들었습니다.*
<br />

변경된 결과로 인해서 간헐적으로 테스트가 실패했던 점을 고칠 수 있었고, 더불어 전체 테스트 코드를 실행 시간을 기존 66초에서 17초로 단축할 수 있었습니다. 

## 6. 맺으며 

컴포넌트에 대한 인터랙션을 테스트해보는 두 가지 방법을 체험해보면서 느꼈던 점을 간단하게 정리하며 글을 마치겠습니다. 

- 테스트 코드에서의 추상화에 대해서 생각해보게 되었습니다. 함수만으로 테스트 내용을 감싸는 것은 반복되는 테스트 케이스를 간편하게 작성할 수 있도록 도와주지만 테스트 케이스를 읽는 데에 방해가 되어 직관적으로 파악할 수 없을 것 같다는 생각이 들었습니다.

- data-attribute들이 테스트 코드를 작성할 때 도움이 많이 될 것 같습니다. 가령 Slider에 props로 전달한 min, max, defaultValue, step 등의 정보들을 테스트 코드에서 가져올 때 가장 좋은 방법이 data-attribute에 미리 저장해놓고 불러오는 방법인 것 같습니다. 현재 작성된 컴포넌트에서는 data-attribute를 적극적으로 활용하지 않았는데, 테스트 코드를 위해서 설정해두면 좋을 것 같습니다. 

- CI 단계까지는 작성하지 못했는데 관련된 GitHub Actions도 찾아보고 적용해보면 좋을 것 같습니다. 

- 어플리케이션에 대한 E2E 테스트를 작성한 것은 아니다보니 실제로 API 요청이 필요한 경우에도 적용해보면 도움이 될 것 같습니다.


[참고]

- [Table of Contents / Cypress Documentation](https://docs.cypress.io/api/table-of-contents)

- [Custom Commands / Cypress Documentation](https://docs.cypress.io/api/cypress-api/custom-commands#Child-Commands)

- [UX Testing with Cypress + Storybook](https://medium.com/coverwallet-engineering/ux-testing-with-cypress-storybook-a34a8ca506fd)

- [Writing UI Component Tests with Storybook and Cypress](https://medium.com/@nhashizume.ca/writing-ui-component-tests-with-storybook-and-cypress-b53cf357b1ae)

- [Working with iframes in Cypress](https://www.cypress.io/blog/2020/02/12/working-with-iframes-in-cypress/)

- [should("have.css") returns a pixel value when using inline percentage](https://github.com/cypress-io/cypress/issues/26540#issuecomment-1515301592)

- [Interaction tests](https://storybook.js.org/docs/react/writing-tests/interaction-testing)

- [Play function](https://storybook.js.org/docs/react/writing-stories/play-function#page-top)

- [Interactions Addon / Storybook: Frontend workshop for UI development](https://storybook.js.org/addons/@storybook/addon-interactions)

- [Test runner Addon / Storybook: Frontend workshop for UI development](https://storybook.js.org/addons/@storybook/test-runner)

- [Component Story Format (CSF)](https://storybook.js.org/docs/react/api/csf)

- [[Bug]: composeStories from @storybook/react does not work with interactions that use the step function](https://github.com/storybookjs/storybook/issues/21562)

- [Firing Events / Testing Library](https://testing-library.com/docs/dom-testing-library/api-events/)

- [스토리북으로 인터랙션 테스트하기](https://ui.toast.com/posts/ko_20220111#e2e-테스팅-도구와-함께)