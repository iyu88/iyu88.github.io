---
layout: post
title: "[JavaScript] 모듈 시스템 (CJS, ESM)"
date: 2024-03-02 00:00:00
author: Hyunbin Lee
categories: JavaScript
tags: javascript moduleSystem commonJS ecmaScriptModules cjs esm 
cover: "/assets/instacode.png"
---

# 모듈 시스템 (CJS, ESM)

## 1. 모듈

코드를 짜다보면 프로그램의 규모가 점차 커지면서 하나의 파일에 작성했던 코드를 관련된 내용끼리 여러 파일에 분리하곤 합니다. 
JavaScript에서는 각 파일을 서로 다른 모듈로 취급하고, 각 모듈에서는 변수나 함수를 공유하기 위해 이를 내보내고 불러올 수 있어야 합니다. 
이 때, 프로젝트가 사용하고 있는 모듈 시스템과 사용하려는 모듈이 어떤 시스템을 따르고 있는지에 따라 사용 방식이 달라집니다. 
여러 가지 모듈 시스템에 대해 알아보고, 주로 사용되는 CJS, ESM을 살펴봅니다.

<br />

## 2. 여러 가지 모듈 시스템 

CJS, AMD, UMD, ESM에 대해서 순서대로 알아봅니다. 

### 2-1. CJS(CommonJS)

CJS는 NodeJS에서 지원하는 모듈 시스템으로 알려져 있습니다. 모듈을 불러올 때는 `require`를 통해 동기적으로 불러오고, 내보낼 때는 `module.exports`를 사용합니다. 

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=01_cjs_syntax.js"></script>

module.exports는 전역 스코프로 취급되어 모듈의 보안성을 해칠 수 있습니다. require로 불러온 모듈을 수정하면 해당 모듈을 사용하는 다른 곳에 영향을 끼칠 수 있으니 주의해야 합니다. 아래 예시에서 `cjsModule`의 속성을 수정했을 때 값이 반영되는 것을 확인할 수 있습니다. 

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=02_cjs_scope.js"></script>

<br />

### 2-2. AMD(Asynchronous Module Definition)

NodeJS가 아닌 브라우저 환경에 적합한 모듈 시스템의 필요성이 커지면서 AMD가 등장하게 되었습니다. AMD는 브라우저에서 실행되는 어플리케이션의 성능을 위해 모듈을 비동기적으로 불러옵니다. 모듈을 불러올 때는 `require`, 내보낼 때는 `define`을 사용합니다. 

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=03_amd_syntax.js"></script>

<br />

### 2-3. UMD(Universal Module Definition)

이제 NodeJS 환경과 브라우저 환경을 지원하는 모듈 시스템이 하나씩 생겼습니다. JavaScript 모듈을 제공하는 입장에서는 어느 환경에서 모듈을 실행할지 알 수 없기 때문에 각각의 형태로 모두 제공해주어야 했습니다. 이러한 불편함을 해소하기 위해 CJS와 AMD 방식 모두를 지원하는 UMD가 등장하게 되었습니다. 
JavaScript 유틸리티 라이브러리인 [Underscore](https://github.com/jashkenas/underscore/blob/master/underscore-umd.js)가 UMD 모듈 시스템을 사용하고 있습니다.

<br />

### 2-4. ESM(ECMAScript Modules)

마지막으로 소개할 ESM은 ECMAScript에서 지정한 표준 모듈 시스템으로, 브라우저 환경의 지원을 받아 별도의 설정없이 실행할 수 있습니다. 모듈을 불러올 때는 `import`를 통해 비동기적으로 불러오고, 내보낼 때는 `export`를 사용합니다. 앞선 다른 모듈 시스템들과는 달리 정적 분석을 통해 모듈을 관리합니다. 덕분에 모듈 간 의존 관계를 파악하여 빌드 단계에서 사용하지 않는 코드를 번들에서 제외하는 트리 쉐이킹(Tree Shaking)이 가능합니다.

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=04_esm_syntax.js"></script>

또한 ESM은 CJS처럼 module 전역 변수를 사용하지 않고 모듈마다 스코프를 따로 생성하여 보안에 유리합니다. 따라서 다음 예시에서 `esmModule`의 내용을 변경할 수 없습니다.

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=05_esm_scope.js"></script>

<br />

## 3. CJS와 ESM의 호환

위에서 살펴본 모듈 시스템 중 CJS와 ESM은 각각 NodeJS 환경과 브라우저 환경을 대표합니다. 만일 두 방식을 혼용한다면 어떻게 될까요? 두 가지 경우를 생각해볼 수 있습니다.

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=06_compatibility.js"></script>

CJS와 ESM은 각각 동기적, 비동기적으로 모듈을 불러옵니다. 따라서 1번 상황은 동기적인 모듈을 비동기적으로 불러오는 것이라고 할 수 있습니다. ESM이 ES2022부터 추가된 `Top-level Await`(async 함수가 아닌 최상위에서 await을 사용하는 것)을 지원하는 덕분에 **import를 사용하여 CJS 모듈을 정상적으로 불러올 수 있습니다.** 

반면 2번 상황은 정상적으로 동작하지 않습니다. 이는 CJS가 Top-level Await을 지원하지 않을 뿐더러 export 처리 방식에서 차이점을 보이기 때문입니다. 다음 예시에서 b.cjs 파일은 a.mjs 모듈을 불러올 때 default export 구문을 해석할 수 없습니다. **따라서 CJS 환경에서 ESM 모듈을 불러올 수 없습니다.**

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=07_default_export.js"></script>

<br />

ESM 모듈을 require로 불러오는 상황을 피하려면 **모듈을 제공할 때 CJS와 ESM 두 가지 방식을 모두 지원하면 됩니다.** package.json의 `exports` 필드를 활용하면 빌드 타임에 CJS 문법을 ESM 문법으로 (또는 그 반대로) 변경해줍니다. 

<script src="https://gist.github.com/iyu88/e8c8177869bc7938ccea8f9961358209.js?file=08_redux_package.js"></script>

위와 같이 exports 속성을 작성하면 CJS 환경에서는 `./dist/cjs/redux.cjs`에 있는 모듈을 불러오고 ESM 환경에서는 `./dist/redux.mjs`에 있는 모듈을 불러올 수 있습니다.

<br />

[참고]

- [CommonJS vs. ESM](https://www.hoeser.dev/blog/2023-02-21-cjs-vs-esm/)

- [Differences Between JavaScript Modules: CJS, AMD, UMD, and ESM](https://medium.com/@halilatilla/differences-between-javascript-modules-cjs-amd-umd-and-esm-f60124de131b)

- [ES Modules: All You Need To Know](https://konstantin.digital/blog/es-modules-all-you-need-to-know)

- [yuzu health / CJS vs ESM](https://yuzu.health/blog/cjs-vs-esm)

- [medium / CJS vs ESM](https://webreflection.medium.com/cjs-vs-esm-5f8b90a4511a)

- [CommonJS vs ECMAScript Modules (ESM). Choosing the Right Module System for Your JavaScript Project](https://blog.stackademic.com/commonjs-vs-ecmascript-modules-esm-choosing-the-right-module-system-for-your-javascript-project-ef4efa856554)
