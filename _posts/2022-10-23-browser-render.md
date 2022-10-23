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

> ### 💡 브라우저의 Reflow 와 Repaint 를 개발자 도구로 확인해봅니다.

## 1. 브라우저 렌더링 프로세스 

일반적으로 웹을 개발할 때 세 가지 언어 HTML, CSS, Javascript 를 사용합니다. 작성된 각각의 파일들은 브라우저를 통해서 사용자들에게 보여집니다. 소스 코드들이 잘 갖춰진 형태의 웹 페이지로 만들어지는 과정을 브라우저 렌더링 프로세스를 통해 설명할 수 있습니다. 기본적인 흐름은 다음의 그림과 같습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197388724-e57fd66f-c537-4a42-9639-d8ab0c430157.jpeg">
*Rendering Process 를 도식화한 이미지*
<br />

1. 먼저 HTML 파일을 `DOM Tree` 로 변환하는 과정을 거칩니다. HTML 파일 내부에 작성된 여러 가지 태그들이 Tokenizing (작은 단위로 나누기), Lexing (의미를 가진 형태로 만들기) 과 Parsing (올바른 데이터인지 확인하기) 의 과정을 거쳐서 Tree 형태의 구조로 만들어집니다.

2. CSS 파일도 동일하게 Parser 를 거쳐서 `CSSOM Tree` 로 만들어집니다.

3. 앞서 만들어진 두 개의 트리인 DOM Tree 와 CSSOM Tree 를 합쳐서 `Render Tree` 를 만듭니다. CSSOM 에서 작성된 스타일이 적용되어야 할 부분을 DOM Tree 에서 찾아서 적용하는 방식으로 생성됩니다. 이 때, Render Tree 의 특징은 실제로 브라우저에서 나타나는 형태와 Render Tree 의 형태가 다를 수 있다는 점입니다. 예를 들어서 화면에 실제로 표시되지 않는 visibility: "hidden" 과 같은 속성이 적용된 태그의 경우에는 Render Tree 에 존재하지만 브라우저를 렌더링할 때는 제외됩니다.

4. Render Tree 를 기반으로 하여 브라우저에서 각 태그가 표시되어야 하는 위치와 크기를 계산하는 과정인 `Layout` 을 거칩니다. 주로 위치와 관련된 속성을 계산합니다. 

5. 태그들은 각자 다른 계층으로 분류됩니다. 태그가 속할 Layer 를 나누는 과정을 `Layer Tree` 라고 부릅니다. 

6. 실제 렌더링될 요소들을 브라우저에 그리는 과정을 `Paint` (draw) 라고 부릅니다.

7. 마지막으로 요소가 렌더링된 계층들이 하나로 합쳐진 것을 `Composite Layer` 라고 합니다. 여러 Layer 를 하나의 비트맵으로 만들어서 최종적인 페이지를 만듭니다. 

<hr />

## 2. Reflow 와 Repaint 

Javascript 는 웹 페이지의 UI를 동적으로 바꿀 수 있습니다. 이렇게 바뀐 부분을 브라우저에 반영하기 위해서 렌더링 프로세스를 다시 진행해야 합니다. 이처럼 바뀐 부분이 렌더링될 때 `Reflow`, `Repaint` 가 발생한다고 합니다. 이는 앞서 설명한 렌더링 과정에서 각각 `Layout` 과 `Paint` 가 다시 발생하는 것을 말합니다.

<br />
<img src="https://www.freecodecamp.org/news/content/images/size/w1600/2022/02/Twitter-post---55.png">
*Reflow 와 Repaint 가 발생하는 조건에 대해 간단하게 정리한 사진*
<br />

<hr />

## 3. Chrome dev Tools 살펴보기 

코드로 Reflow 와 Repaint 를 확인하기 전에 잠깐 Chrome 개발자 도구에 대해서 살펴보겠습니다. 

<br />

### 1) Performance 

개발자 도구의 Performance 탭에서 웹 페이지를 일정시간 녹화하고 발생한 이벤트를 모니터링할 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197391162-910e3dd3-d64d-413e-a9be-6cd8aa1c8788.png">
*Performance 탭의 구성*
<br />

1. 웹 페이지 녹화를 시작합니다. 녹화를 중단하고 싶을 때 멈출 수 있습니다. 
2. 웹 페이지를 새로고침하여 처음 로딩될 때부터 녹화를 시작합니다. 
3. 녹화된 기록을 삭제할 수 있습니다. 
4. 녹화된 시간동안 각 프레임마다 스냅샷을 확인할 수 있습니다. 
5. 각각의 탭을 통해서 원하는 성능 정보를 모니터링할 수 있습니다. 
6. 찾고 싶은 Activity 를 검색하여 필터링할 수 있습니다. 
7. 녹화된 시간동안 발생한 Activity 목록을 확인할 수 있습니다. 

<br />

### 2) Layers 

Performance 탭에서 Paint Profiler 를 통해서 확인할 수 있는 정보를 Layers 탭에서 더 자세하게 확인할 수 있습니다. 뒤에서 코드로 살펴볼 예제 코드를 사용하여 Layers 탭에 대해서 살펴보겠습니다. 탭에 들어가면 다음과 같은 화면이 나옵니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197393780-9e50f7db-03af-45fd-b978-14a5022dc4c3.png">
*Layers 탭의 구성*
<br />

1. 현재 윈도우의 Layers 를 나타냅니다. 
2. Layers 를 이동하거나 회전하면서 살펴볼 수 있는 컨트롤러입니다. 
3. Layers 구성을 확인할 수 있습니다. 
4. Details 탭에서 Layer 의 세부정보와 Profiler 에서 구체적인 렌더링 과정을 확인할 수 있습니다. 
5. 현재 페이지에서 렌더링된 요소들의 로그를 확인할 수 있습니다. 
6. 슬라이드로 범위를 지정하여 3번과 5번 영역을 필터링할 수 있습니다. 

<br />

### 3) Rendering 

마지막 개발자 도구인 Rendering 에 대해서 살펴보겠습니다. 해당 탭에서 옵션을 활성화하면 웹 페이지에서 실시간으로 변하는 요소를 더욱 편하게 찾을 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197394371-ef3393b8-40e7-493a-b528-7152c2222a05.png">
*Rendering 탭은 상단에 없는 경우가 있습니다. 다음 경로에서 찾을 수 있습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197394796-309bc21d-092e-40e9-81c6-ce453922a774.png">
*처음 들어가면 모든 체크박스가 해제되어 있습니다. 위에서 세 가지 도구만 활성화했습니다.*
<br />

- Paint flashing : Repaint 가 일어나는 부분을 초록색으로 표시합니다. 
- Layout Shift Regions : 전환된 부분을 파란색으로 표시합니다. 
- Layer borders : 레이어 테두리를 주황색이나 올리브색으로 표시합니다. 

기능을 활성화한 뒤 웹 페이지를 보면 설정한 옵션에 따라서 색상이 나타나는 것을 확인할 수 있습니다. 

![Rendering 탭 설정했을 때](https://user-images.githubusercontent.com/31645195/197394449-d78e7760-bdd3-4635-920e-244dd7a22703.gif)

<hr />

## 4. 최적화 방법 확인해보기 

앞서 소개한 개발자 도구 중에서 Performance 를 주로 활용하며, 네 가지 서로 다른 방식으로 코딩한 방식을 비교하면서 Reflow 와 Repaint 를 확인해보겠습니다. 

*요소는 HTML Element 를 의미합니다. 
*속성은 CSS Property 를 의미합니다. 

<br />

### 1) 여러 요소의 속성을 업데이트 

먼저 요소의 `width` 를 변경하는 경우에 대해서 테스트 해보았습니다. 

<br />

#### 1-1) 여러 요소의 속성을 따로 업데이트 

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

#### 1-2) 여러 요소의 속성을 한꺼번에 업데이트 

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

두 코드의 차이점은 `Element.style.Property` 를 설정하는 코드가 연속적으로 존재하는지 여부입니다. 이러한 소스 코드의 차이가 어떠한 점에서 다른지 Performance 탭에서 확인해보겠습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197396610-7bde0249-f447-48e8-995d-7e64a737e1db.png">
*1-1) 의 결과입니다. Layout 이벤트가 총 5번 발생한 것을 알 수 있습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197396664-fa5d85b1-bd09-42f3-afb2-13a74bd90741.png">
*1-2) 의 결과입니다. Layout 이벤트가 총 2번 발생한 것을 알 수 있습니다.*
<br />

위의 두 사진에서 확인할 수 있듯이 `1-2) 여러 요소의 속성을 한꺼번에 업데이트` 의 경우가 Layout 이벤트를 3번 덜 발생시키는 것을 알 수 있습니다. 이는 `Element.style.Property` 처럼 속성을 업데이트하는 코드가 모여 있기 때문에 Layout 을 재계산할 때 변경된 값이 한 번에 적용되기 때문입니다. 따라서 Layout 을 변경시키는 값을 코드 상에서 모아서 작성해야 Reflow 를 적게 발생시킨다는 것을 알 수 있습니다. 

<hr />

### 2) 한 요소의 속성을 업데이트  

다음으로는 아이디가 `first` 인 요소의 속성만 업데이트하는 경우에 대해서 테스트 해보았습니다.  

<br />

#### 2-1) 한 요소의 속성을 따로 업데이트

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

#### 2-2) 한 요소의 속성을 한꺼번에 업데이트

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

앞서 `1) 여러 요소의 속성을 업데이트` 의 예시에서 실험해봤던 점을 하나의 요소에서 동일하게 진행해보았습니다. 다른 점은 width 속성뿐만 아니라 `fontSize` 와 `backgroundColor` 속성까지 변경한다는 점입니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197397451-2b37fdb9-2a77-4fec-a7bb-f37f17e106d6.png">
*2-1) 의 결과입니다. Layout 이벤트가 총 3번 발생한 것을 알 수 있습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197397454-01140744-8da7-4b78-9eea-da4019224e88.png">
*2-2) 의 결과입니다. Layout 이벤트가 총 2번 발생한 것을 알 수 있습니다.*
<br />

width 와 더불어 fontSize 를 변경한다는 점 때문에 `2-1) 한 요소의 속성을 따로 업데이트` 에서 Reflow 가 1번 더 일어난다는 점을 알 수 있습니다.

<hr />

### 3) 노출 제어를 통한 속성 설정 

요소를 화면에 표시되지 않는 상태로 만든 뒤에 원하는 속성을 설정하고, 다시 이전처럼 화면에 나타나도록 속성을 복원합니다. 이를 통해서 화면에 렌더링되지 않는 상태에서 스타일을 변경할 수 있기 때문에 Paint 할 때 드는 연산을 줄일 수 있습니다. 이를 실험을 통해서 확인해보겠습니다. 

<br />

#### 3-1) Opacity 를 통한 노출 제어 

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

#### 3-2) Display 를 통한 노출 제어 

<pre>
<code class="hljs javascript">first.style.display = "none";
first.style.fontSize = firstFontSize;
first.style.width = firstWidth + step + "px";
first.style.backgroundColor = firstBackground;
first.style.display = "block";</code>
</pre>

<br />

#### 3-3) Visibility 를 통한 노출 제어 

<pre>
<code class="hljs javascript">first.style.visibility = "hidden";
first.style.fontSize = firstFontSize;
first.style.width = firstWidth + step + "px";
first.style.backgroundColor = firstBackground;
first.style.visibility = "visible";</code>
</pre>

세 코드는 각각 opacity, display, visibility 속성을 조절하여 요소를 화면에서 지웁니다. `2) 한 요소의 속성을 업데이트` 에서 진행한 것과 동일하게 3개의 속성을 수정한 뒤에 다시 화면에 렌더링한 것을 Performance 탭에서 녹화한 결과는 다음과 같습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197398277-7b7619c0-fb4f-4034-821f-17ca21638238.png">
*3-1) 의 결과입니다. Layout 은 0.3ms, Paint 는 0.4ms, Composite Layers 는 1.2ms 가 걸렸습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197398275-e2721df1-d890-4386-8caf-e3e007503d02.png">
*3-2) 의 결과입니다. Layout 은 0.3ms, Paint 는 0.5ms, Composite Layers 는 0.9ms 가 걸렸습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197399118-2553adbb-5604-4b45-a6c2-4e378aee2dba.png">
*3-3) 의 결과입니다. Layout 은 0.6ms, Paint 는 0.9ms, Composite Layers 는 1.4ms 가 걸렸습니다.*
<br />

앞의 두 경우에서 opacity 속성을 사용한 경우가 0.3ms만큼 더 Composite Layers 이벤트에 시간을 할애한 것을 확인할 수 있습니다. 또한 Layout 의 경우 절대적인 시간은 동일한 것으로 보이나 비율로 보았을 때, opacity 는 4.7%, display 는 6.1% 걸린 것을 확인할 수 있습니다. 이는 `2. Reflow 와 Repaint` 에서 첨부한 이미지에서 확인할 수 있듯이 opacity 속성은 Composite Layers 에 영향을 주고 display 속성은 Layout 에 영향을 주기 때문이라고 해석할 수 있습니다. 한편, 마지막 예제에서 Paint 가 0.9ms 가 걸렸습니다. 이는 visibility 속성이 Paint 에 영향을 주기 때문이라고 할 수 있습니다. 참고로 각 테스트 경우에서 Composite Layers 이벤트가 발생한 횟수는 92회, 52회, 53회였습니다. 

<hr />

### 4) 노드 복제를 통한 속성 설정 

마지막으로 노드를 복제하고, 복제한 노드의 속성을 수정하는 방식으로도 렌더링할 수 있습니다. 

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

스타일을 한꺼번에 업데이트 한 `2-2) 한 요소의 속성을 한꺼번에 업데이트` 의 결과와 비교해보겠습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/197400100-3396f28f-28eb-4b7a-b695-b121ce92209f.png">
*4) 의 결과입니다. Layout 이 0.3ms, Paint 가 0.4ms 가 걸렸습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/197400088-020b6924-08cf-41b1-86cd-b1ad5359fa93.png">
*2-2) 의 결과입니다. Layout 이 0.3ms, Paint 가 0.4ms 가 걸렸습니다.*
<br />

두 가지 최적화 방식 모두 Reflow 와 Repaint 가 동일한 시간이 걸린 것을 확인할 수 있습니다.

<hr />

## 5. 결론

- Reflow 와 Repaint 는 웹 페이지 변경 요소에 따라서 발생합니다. 
- 해당 이벤트들을 브라우저의 개발자 도구를 통해서 확인할 수 있습니다. 
- 렌더링 최적화를 위해서
  + 1) 스타일 속성을 업데이트하는 코드는 한 곳에 모아서 작성합니다. 
  + 2) 노출 제어를 통해서 최적화할 수 있습니다. opacity 속성이 가장 유효합니다. 
  + 3) 노드를 복사하는 방식으로도 최적화할 수 있습니다. 

<hr />

## 6. 생각해볼 점

- 단순한 구조로 테스트해서 눈에 띄는 차이를 확인하기 힘들었습니다. 더 복잡한 구조에서 실험해볼 필요가 있습니다. 

<hr />

[출처 & 참고]


- [https://www.webperf.tips/tip/measuring-paint-time/](https://www.webperf.tips/tip/measuring-paint-time/)

- [Detecting when the Browser Paints Frames in JavaScript](https://www.webperf.tips/tip/measuring-paint-time/)

- [Understanding Repaint and Reflow in JavaScript](https://medium.com/swlh/what-the-heck-is-repaint-and-reflow-in-the-browser-b2d0fb980c08)

- [DOM Performance (Reflow & Repaint) (Summary)](https://gist.github.com/faressoft/36cdd64faae21ed22948b458e6bf04d5)

- [Render-tree Construction, Layout, and Paint](https://web.dev/critical-rendering-path-render-tree-construction/)

- [[Browser] Reflow와 Repaint](https://beomy.github.io/tech/browser/reflow-repaint/)

- [브라우저 렌더링에 대한 이해와 최적화](https://velog.io/@yrnana/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80-%EB%A0%8C%EB%8D%94%EB%A7%81%EC%97%90-%EB%8C%80%ED%95%9C-%EC%9D%B4%ED%95%B4%EC%99%80-%EC%B5%9C%EC%A0%81%ED%99%94)

- [브라우저의 Reflow 와 Repaint](https://oyg0420.tistory.com/entry/%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%9D%98-Reflow-%EC%99%80-Repaint)

- [Analyze runtime performance - Chrome Developers](https://developer.chrome.com/docs/devtools/evaluate-performance/)

- [Profiling site speed with the Chrome DevTools Performance tab / DebugBear](https://www.debugbear.com/blog/devtools-performance)

- [Debug like a Pro in Chrome Dev Tools - How to use Rendering tab](https://tharakamd-12.medium.com/debug-like-a-pro-in-chrome-dev-tools-how-to-use-rendering-tab-ba25f5bb6264)

- [What is the Chrome Dev Tools "Hit Test" timeline entry?](https://stackoverflow.com/questions/33174286/what-is-the-chrome-dev-tools-hit-test-timeline-entry)

- [Render-tree Construction, Layout, and Paint](https://web.dev/critical-rendering-path-render-tree-construction/)

- [Web Animation Performance Fundamentals - How to Make Your Pages Look Smooth](https://www.freecodecamp.org/news/web-animation-performance-fundamentals/)

- [Dev-Docs/웹 브라우저의 작동 원리.md at master · im-d-team/Dev-Docs](https://github.com/im-d-team/Dev-Docs/blob/master/Browser/%EC%9B%B9%20%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%EC%9D%98%20%EC%9E%91%EB%8F%99%20%EC%9B%90%EB%A6%AC.md)

<br />

[코드 링크]
- [https://github.com/iyu88/Browser-Render-Test](https://github.com/iyu88/Browser-Render-Test)

<br />
