---
layout: post
title: "[React] React Concurrent Mode - Data fetching, Suspense, React hook"
date: 2022-11-01 00:00:00
author: Hyunbin Lee
categories: React
tags: React concurrentMode dataFetching Suspense useTransition useDeferredValue  
cover: "/assets/instacode.png"
---

# React Concurrent Mode - Data fetching, Suspense, React hook

> ### 💡 React 의 동시성 모드와 관련한 기능들에 대해서 알아봅니다. 

## 1. React 

React 가 등장하기 전에 동적인 웹 프론트엔드 개발을 위해서 주로 사용되던 라이브러리는 `jQery` 였습니다. jQuery 는 브라우저에 있는 DOM 을 제어하고 이벤트를 등록하는 작업들을 편리하게 수행할 수 있도록 도와주었습니다. 하지만, jQery 는 DOM 을 다룰 때 많은 단계를 거쳐야 했기 때문에 성능 상에서 바닐라 자바스크립트보다 낮은 퍼포먼스를 보였습니다. 또한 브라우저가 다양해지면서 각 브라우저마다 지원하는 DOM API 스펙이 달라졌고, 이로 인한 웹 프론트엔드 개발의 복잡성은 증가했습니다. 

<br />

React 는 이러한 문제들을 해결해주었습니다. 합성 이벤트 ( Synthetic Event ) 로 브라우저의 기본 이벤트를 감싸서 개발자가 다양한 환경에 일일이 대응하지 않아도 React 에서 알아서 처리해주는 것처럼 개발의 복잡성을 줄여주었습니다. 이제 React 는 웹 프론트엔드를 개발할 때 널리 사용되고 있습니다. Single Page Application 으로 만들어서 페이지 이동 간에 별도의 reload 가 필요하지 않습니다. 이를 통해서 페이지 구성 요소를 빠르게 불러오는 웹 사이트를 만들 수 있고 이용자들에게 긍정적인 사용자 경험을 제공합니다. 

<hr />

## 2. 데이터를 불러오는 문제 

앞서 페이지 이동 간에 별도의 reload 가 필요하지 않다고 했지만 서버로부터 정보를 받아서 화면에 표시하는 작업은 필수적입니다. React 로 구현한 웹 사이트의 구성 요소인 컴포넌트에서 필요한 데이터를 불러오고 이를 사용자의 화면에 DOM 을 통해서 동적으로 표시해주는 데에서 문제가 발생할 수 있습니다. 

<br />

일반적으로 웹 페이지에 표시할 데이터를 불러오는 작업은 컴포넌트가 마운트된 이후에 진행합니다. 이를 `Fetch on render` 라고 부릅니다. 데이터를 화면에 표시하기 위해서 클라이언트에서 모든 데이터를 받은 뒤에 DOM 을 업데이트하게 됩니다. 이러한 방식은 다음과 같은 문제 상황을 발생시킬 수 있습니다. 
- 데이터를 불러오는 시간동안 컴포넌트 내부에 있는 데이터에 의존하는 DOM 이 존재하지 않을 수 있습니다. 따라서 사용자가 데이터와 관련된 이벤트를 발생시키지 못하거나 발생시키더라도 이를 DOM에 정상적으로 반영하지 못할 수 있습니다. 
- 또한 데이터를 다 불러온 뒤에 업데이트 되는 DOM 때문에 기존에 화면에 위치한 요소들의 좌표가 바뀌면서 Layout 을 다시 계산하는 연산을 한 번 더 수행하게 됩니다. 

<br />

이처럼 데이터를 어느 시점에서 불러오는지를 결정하는 것이 또 다른 중요한 과제가 되었습니다. 더 자세한 내용은 `4. Data Fetching` 에서 살펴보도록 하겠습니다. 

<hr />

## 3. React Concurrent Mode 

데이터를 불러오는 작업을 수행하면서 React 가 주는 긍정적인 사용자 경험과 성능 상의 이점이라는 두 마리 토끼를 모두 잡아야 하는 상황이 되었습니다. 이를 위해 `React Concurrent Mode (동시성 모드)` 가 등장했습니다.  

<br />

`동시성`이란 독립적으로 실행될 수 있는 작업들을 여러 조각으로 세분화하고 번갈아 가면서 실행하는 것을 말합니다. 자바스크립트는 싱글 스레드 언어이기 때문에 한 번에 하나의 작업을 수행합니다. 하지만 브라우저에서 실행되는 자바스크립트는 DOM Tree 구성, 네트워크 통신, 사용자 이벤트 처리 등 여러 가지 작업을 처리해야 합니다. 만일 하나의 작업만 수행할 수 있다면 낮은 퍼포먼스로 인해 웹 브라우저가 매우 느리다는 느낌을 받아야 하지만 실제로는 전혀 그렇지 않습니다. 브라우저는 문제 없이 위의 작업들을 동시에 처리하는 것처럼 보입니다. 

<br />

브라우저는 `이벤트 큐`와 `이벤트 루프`를 통해서 동시성을 유지합니다. 특정한 작업들은 자바스크립트가 처리하는 작업이 저장되는 `콜스택`에서 처리되지 않고 이벤트 큐로 이동됩니다. 이후 콜스택이 비어있는 `Idle` 상태일 때 이벤트 큐에 있는 작업들이 이벤트 루프에 의해서 콜스택으로 다시 이동하여 처리됩니다. 따라서 브라우저는 위에서 언급한 작업들을 비동기 콜백으로 실행하여 동시성을 달성합니다. 

<br />

React 는 렌더링 과정에서 일어나는 일들을 작은 단위로 나누고 빠른 속도로 번갈아 가면서 작업을 처리하는 동시성 모드를 지원합니다. 동시성 모드를 기존의 React 와 다른 특별한 모드로 받아들이기보다는, 문제를 해결하기 위해서 등장한 React 의 기능들이라고 이해하면 됩니다. 이와 관련한 기능과 React Hook 에 대해서 알아보기 전에 Data Fetching 방식에 대해서 짚고 넘어가겠습니다.  

<hr />

## 4. Data Fetching 

동시성 모드 기능들에 대해서 다루기 전에 일반적인 Data Fetching (데이터를 불러오는) 의 유형에 대해서 간단하게 알아보겠습니다. 다음과 같은 3가지 유형으로 나눌 수 있습니다. 

1. Fetch on render
2. Fetch then render
3. Render as fetch 

*API 는 [jsonplaceholder](https://jsonplaceholder.typicode.com/photos) 사이트를 사용했습니다. 

### 4-1) Fetch on render

`Fetch on render` 는 컴포넌트가 모두 마운트된 뒤에 데이터를 불러오는 작업을 수행하는 방식입니다. React 에서 useEffect Hook 을 사용하여 데이터를 모두 불러온 뒤에 state 를 변경시키고 이를 DOM 에 렌더링합니다. 

<br />

#### 4-1-1) 데이터만 불러올 때 

API 서버로부터 받아온 정보만 화면에 렌더링하는 경우입니다. `/photos` 경로로 요청을 보내면 배열에 5천 개의 결과값을 받아옵니다. 

<pre>
<code class="hljs javascript">useEffect(() => {
  const dataUrl = "https://jsonplaceholder.typicode.com/photos";
  const fetchOnRender = async () => {
    try {
      const response = await axios.get(dataUrl);
      setDummyData(response.data);
    } catch (err) {
      console.log(err);
    }
  };
  fetchOnRender();
}, []);
</code>
</pre>

컴포넌트가 마운트되었을 때 useEffect 안에서 `setDummyData()` 를 통해서 state 를 업데이트합니다. 데이터가 있을 경우에는 "데이터 로딩 중" 이라는 문자를 표시하고 데이터를 모두 받아왔을 때에는 해당 정보들을 표시하는 방식으로 구현한 결과입니다.

<br />

![useEffect_fetch_on_render](https://user-images.githubusercontent.com/31645195/198887484-a8a302e8-a3a4-4dba-b15b-f4e5c08b8e4e.gif)

<br />

다음은 크롬 확장 프로그램으로 존재하는 `React Developer Tools` 에서 `Profiler` 탭에서 확인할 수 있는 타임라인입니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/198887674-ac9fe2a8-3367-4203-93c0-b8a0333bc98a.png">
*4-1-1) Fetch on render / 데이터만 불러올 때*
<br />

<br />

<strong> [4-1-1) Fetch on render / 데이터만 불러올 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 13ms  | render          | 3ms   | 13ms     |
| 2      | 16ms  | commit          | 1ms   | 13ms     |
| 3      | 24ms  | passive-effects | 2ms   | 13ms     |
| 4      | 85ms  | render          | 49ms  | 335ms    |
| 5      | 239ms | commit          | 239ms | 335ms    |
| 6      | 419ms | passive-effects | 1ms   | 335ms    |

<br />

Profiler 에서 제공하는 각 이벤트 종류는 다음과 같은 의미를 가지고 있습니다. 

- render : DOM 에 어떠한 변화가 일어나야 하는지 결정합니다. 이전의 렌더링 결과와 비교합니다. 
- commit : React 에서 실제로 DOM 을 조작하고 변경 사항을 적용하는 단계입니다. 
- passive-effects : useEffect 에서 정의한 작업을 동기적으로 실행하기 위해서 새로운 자바스크립트 작업 단위를 발생시킵니다. commit 을 실행한 후에 발생합니다. 

1번에서 3번 이벤트까지는 초기 컴포넌트가 렌더링되었을 때를 의미하고 4번에서 6번 이벤트까지는 useEffect 를 통해서 API 요청 결과가 state 에 반영되었을 때를 의미합니다. 

<br />

#### 4-1-2) 데이터를 불러오고 이미지를 상단에 배치했을 때  

동일한 API 서버로부터 정보를 받아와서 표시하지만 상단에 이미지 태그를 배치하고 난 뒤에 차이점을 살펴보았습니다. 

<pre>
<code class="hljs javascript"><xmp>
return (
  <div>
    <img src="./image.JPG" alt="테스트 이미지" width="100px" />
    {dummyData.length ? (
      dummyData.map((d) => <h4 key={d.id}>{JSON.stringify(d)}</h4>)
    ) : (
      <h1> 데이터 로딩 중...</h1>
    )}
  </div>
);
</xmp>
</code>
</pre>

JSX 반환값에서 데이터 목록을 표시하기 전에 이미지 태그를 먼저 위치시킵니다. 

<br />

![useEffect_fetch_on_render_img_first](https://user-images.githubusercontent.com/31645195/198888673-ad72c76d-8a2c-403d-9185-6aa63056f8e1.gif)

<br />

위와 같이 이미지가 최상단에 위치해있고 데이터를 불러오면 이미지 아래에 렌더링하는 것을 확인할 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/198888755-87e75e95-8ca0-4f0e-b561-36e3f4508391.png">
*4-1-2) Fetch on render / 데이터를 불러오고 이미지를 상단에 배치했을 때*
<br />

<br />

<strong> [4-1-2) Fetch on render / 데이터를 불러오고 이미지를 상단에 배치했을 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 24ms  | render          | 4ms   | 14ms     |
| 2      | 27ms  | commit          | 2ms   | 14ms     |
| 3      | 36ms  | passive-effects | 2ms   | 14ms     |
| 4      | 109ms | render          | 94ms  | 341ms    |
| 5      | 203ms | commit          | 216ms | 341ms    |
| 6      | 450ms | passive-effects | 1ms   | 341ms    |

<br />

#### 4-1-3) 데이터를 불러오고 이미지를 하단에 배치했을 때  

`4-1-2)` 와 동일하지만 이번에는 이미지 태그를 데이터 목록 하단에 위치시키고 테스트했습니다. 

<br />

![useEffect_fetch_on_render_img_last](https://user-images.githubusercontent.com/31645195/198888928-f4e15e35-98ba-470f-ace4-75d35e5a8f30.gif)

<br />

초기에는 이미지가 최상단에 위치해있고 데이터가 렌더링되면 이미지가 아래로 밀려나서 화면에서 사라지는 것을 확인할 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/198889013-09905889-bc8c-496e-b5cd-675654637223.png">
*4-1-3) Fetch on render / 데이터를 불러오고 이미지를 하단에 배치했을 때*
<br />

<br />

<strong> [4-1-3) Fetch on render / 데이터를 불러오고 이미지를 하단에 배치했을 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 13ms  | render          | 4ms   | 17ms     |
| 2      | 18ms  | commit          | 2ms   | 17ms     |
| 3      | 27ms  | passive-effects | 2ms   | 17ms     |
| 4      | 88ms  | render          | 68ms  | 342ms    |
| 5      | 156ms | commit          | 228ms | 342ms    |
| 6      | 430ms | passive-effects | 1ms   | 342ms    |

<br />

#### 4-1-4) Fetch on render 정리 

각각의 경우에 대해서 4번 render 이벤트와 5번 commit 이벤트에 대해서 비교해보겠습니다. 

<br />

| 테스트 번호 | 이벤트 순서 | 이벤트 유형 | 실행시간  | 일괄 처리 시간 |
|:--------:|:--------:|:--------:|:-------:|:----------:|
| 4-1-1  | 4      | render | 49ms  | 335ms    |
|        | 5      | commit | 239ms | 335ms    |
| 4-1-2  | 4      | render | 94ms  | 341ms    |
|        | 5      | commit | 216ms | 341ms    |
| 4-1-3  | 4      | render | 68ms  | 342ms    |
|        | 5      | commit | 228ms | 342ms    |

<br />

눈여겨 볼 점은 JSX 가 이미지 태그를 반환한 경우 (`4-1-2`, `4-1-3`) 가 그렇지 않은 경우보다 render 의 실행시간이 더 오래걸렸다는 것입니다. 이는 화면에 그려야 할 DOM 의 개수가 더 많기 때문이라고 해석할 수 있습니다. 일괄 처리 시간이 약 6-7ms 정도 더 걸렸다는 점도 동일한 내용으로 해석할 수 있습니다. 

<br />


이미지 태그가 상단에 위치한 경우와 하단에 위치한 경우의 commit 실행시간이 각각 216ms 와 228ms 로 12ms 정도 차이나는 것을 알 수 있습니다. 이미지가 하단에 위치했을 경우 데이터 목록이 상단에서부터 렌더링되면서 기존의 이미지 위치가 재계산되는 과정이 commit 단계에서 추가되었기 때문이라고 해석할 수 있습니다. Fetch on render 방식은 글 도입부에 설명했던 것처럼 느린 DOM 렌더링을 통해서 사용자가 웹 페이지와 상호작용하는 것을 방해하거나 Layout Shift 때문에 불필요한 레이아웃 좌표 계산 등의 단점이 존재합니다. 

<hr />

### 4-2) Fetch then render

앞서 발생한 문제들은 모두 데이터를 느리게 불러올 경우 발생했습니다. 따라서 `Fetch then render` 방식에서는 데이터를 조금 더 빠르게 불러오는 방향으로 접근합니다. 바로 컴포넌트를 생성하기 전에 데이터를 미리 불러오기 시작하는 방식입니다. 따라서 이번에는 컴포넌트의 반환값은 동일하지만 데이터를 불러오는 순서가 다른 경우를 테스트합니다.

<br />

#### 4-2-1) 데이터만 불러올 때 

<pre>
<code class="hljs javascript">const photoData = fetchThenRender();

export default function ChildComponent() {
  const [dummyData, setDummyData] = useState([]);

  useEffect(() => {
    photoData.then((response) => {
      setDummyData(response.data);
    });
  }, []);

  // (생략)
}
</code>
</pre>

컴포넌트 바깥에서 데이터를 불러오는 `fetchThenRender()` 함수를 호출합니다. useEffect 에서는 해당 변수에 Promise 객체가 결과를 반환할 때 state 를 업데이트해줍니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/199045526-455cced8-497f-4803-bfa8-1eafe9807d7d.png">
*4-2-1) Fetch then render / 데이터만 불러올 때*
<br />

<br />

<strong> [4-2-1) Fetch then render / 데이터만 불러올 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 20ms  | render          | 3ms   | 8ms      |
| 2      | 23ms  | commit          | 2ms   | 8ms      |
| 3      | 27ms  | passive-effects | 1ms   | 8ms      |
| 4      | 73ms  | render          | 46ms  | 292ms    |
| 5      | 119ms | commit          | 214ms | 292ms    |
| 6      | 364ms | passive-effects | 1ms   | 292ms    |

<br />

#### 4-2-2) 데이터를 불러오고 이미지를 배치했을 때

`4-1-2)` & `4-1-3)` 과 동일하게 이미지 태그를 배치했을 경우에 대해서도 테스트해보았습니다. 해당 태그를 데이터 목록의 각각 상단과 하단에 위치시켰을 때의 결과값은 다음과 같습니다. 

<br />

<strong> [이미지가 데이터 목록 상단에 있을 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 25ms  | render          | 3ms   | 17ms     |
| 2      | 28ms  | commit          | 2ms   | 17ms     |
| 3      | 41ms  | passive-effects | 1ms   | 17ms     |
| 4      | 94ms  | render          | 42ms  | 241ms    |
| 5      | 135ms | commit          | 166ms | 241ms    |
| 6      | 335ms | passive-effects | 1ms   | 241ms    |

<br />

<strong> [이미지가 데이터 목록 하단에 있을 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 25ms  | render          | 3ms   | 17ms     |
| 2      | 28ms  | commit          | 2ms   | 17ms     |
| 3      | 42ms  | passive-effects | 1ms   | 17ms     |
| 4      | 111ms | render          | 42ms  | 210ms    |
| 5      | 153ms | commit          | 155ms | 210ms    |
| 6      | 321ms | passive-effects | 1ms   | 210ms    |

<br />

#### 4-2-3) Fetch then render 정리 

<br />

| 테스트 번호     | 이벤트 순서 | 이벤트 유형 | 실행시간  | 일괄 처리 시간 |
|:------------:|:--------:|:--------:|:-------:|:----------:|
| 4-2-1      | 4      | render | 46ms  | 292ms    |
|            | 5      | commit | 214ms | 292ms    |
| 4-2-2 (상단) | 4      | render | 42ms  | 241ms    |
|            | 5      | commit | 166ms | 241ms    |
| 4-2-2 (하단) | 4      | render | 42ms  | 210ms    |
|            | 5      | commit | 155ms | 210ms    |

<br />

주목할 부분은 위의 세 가지 경우의 render 와 commit 실행시간이 모두 `4-1) Fetch on render` 의 실행시간보다 짧게 걸렸다는 점입니다. render 의 경우에는 적게는 3ms 에서 많게는 약 48ms 정도 빠릅니다. 또한 commit 의 경우에도 적게는 2ms 에서 많게는 약 84ms 정도 실행시간이 줄어든 것을 확인할 수 있습니다. 

<br />

#### 4-2-4) 데이터를 두 개 불러올 때 

Fetch then render 는 데이터 로딩을 빠르게 시작한다는 점에서 Layout Shift 가 발생할 가능성이 적습니다. 하지만 또 다른 문제 상황이 존재합니다. 바로 데이터를 여러 개 불러오는 경우입니다. 데이터를 미리 불러온다는 개념을 유지하면서 여러 개의 데이터를 받아와야 하는 경우도 살펴보겠습니다. 

이전까지 photos 데이터를 받아오기 위해서 API 요청을 한 번만 보냈다면, 5백 개의 comments 데이터까지 받아오기 위해서 추가적인 API 요청을 보냅니다.

<pre>
<code class="hljs javascript">const fetchDummyData = () => {
  return Promise.all([
    fetchThenRender("photos"),
    fetchThenRender("comments"),
  ]).then(([photos, comments]) => [photos, comments]);
};

const dummyData = fetchDummyData();

export default function ChildComponent() {
  const [dummyPhotoData, setDummyPhotoData] = useState([]);
  const [dummyCommentData, setDummyCommentData] = useState([]);

  useEffect(() => {
    dummyData.then(([photos, comments]) => {
      setDummyPhotoData(photos.data);
      setDummyCommentData(comments.data);
    });
  }, []);

  // (생략);
}
</code>
</pre>

Promise.all() 을 사용하여 photos 와 comments 에 대한 응답을 하나의 변수에 저장합니다. useEffect 에서 각 state 를 업데이트합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/199139903-381bf13d-2eda-454d-87c5-588034ad30d6.png">
*4-2-3) Fetch then render / 데이터를 두 개 불러올 때*
<br />

<br />

<strong> [4-2-3) Fetch then render / 데이터를 두 개 불러올 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간  | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:-------:|:----------:|
| 1      | 14ms  | render          | 3ms   | 16ms     |
| 2      | 17ms  | commit          | 2ms   | 16ms     |
| 3      | 30ms  | passive-effects | 1ms   | 16ms     |
| 4      | 151ms | render          | 54ms  | 244ms    |
| 5      | 205ms | commit          | 177ms | 244ms    |
| 6      | 394ms | passive-effects | 1ms   | 244ms    |

<br />

이러한 방식은 컴포넌트에 필요한 모든 데이터를 받을 때까지 기다려야 한다는 단점이 존재합니다. comments 데이터의 개수가 적기 때문에 데이터를 상대적으로 일찍 받아서 빠르게 DOM 을 업데이트하는 것을 기대할 수 있습니다. 하지만 Promise.all() 을 통해서 모든 데이터를 받아와야 값을 반환할 수 있기 때문에 아무리 comments 를 먼저 불러왔어도 photos 데이터를 다 받아오기 전까지 DOM 이 업데이트 되지 않습니다. 

<br />

한편 Promise.all() 을 사용하지 않고 개별적인 함수로 데이터를 받아오는 방식을 사용할 수도 있습니다. 다만 서로 다른 데이터 종류가 여러 개 존재할 경우 컴포넌트 선언 전에 작성하는 코드가 길어지거나 관리해야 할 함수가 많아진다는 단점이 존재합니다. 

<hr />

### 4-3) Render as fetch 

지금까지 컴포넌트를 먼저 불러오면 DOM 이 느리게 업데이트되고 데이터를 먼저 불러오면 여러 개의 요청을 보낼 때 문제가 발생할 수 있다는 것을 확인했습니다. 이러한 딜레마를 해결하기 위한 방법으로 React Concurrent Mode 에서는 `Suspense` 라는 기능을 제공합니다. Suspense 를 사용하면 컴포넌트 렌더링과 동시에 데이터를 받아오기 시작합니다. 또한 Suspense 의 fallback 속성을 사용하여 데이터를 로딩하는 시간동안 화면에 표시할 DOM 을 정의할 수 있습니다. 

<br />

#### 4-3-1) Suspense 를 사용할 때 

<pre>
<code class="hljs javascript">const wrapPromise = (promise) => {
  let status = "pending";
  let response;

  const suspender = promise.then(
    (res) => {
      status = "success";
      response = res;
    },
    (err) => {
      status = "error";
      response = err;
    }
  );

  const read = () => {
    switch (status) {
      case "pending":
        throw suspender;
      case "error":
        throw response;
      default:
        return response;
    }
  };

  return { read };
};

const fetchDummyData = () => {
  const requestURL = "https://jsonplaceholder.typicode.com/photos";
  try {
    const response = axios.get(requestURL);
    return wrapPromise(response);
  } catch (err) {
    console.log(err);
  }
};
</code>
</pre>


위 코드에서 각 함수의 역할은 다음과 같습니다. 

- fetchDummyData : axios 를 사용하여 API 요청을 보냅니다. 
- wrapPromise : fetchDummyData() 함수가 반환한 Promise 와 다음의 두 함수를 Lexical Scope 에 저장합니다. 
  - suspender : Promise 의 결과에 따라서 상태 (status) 와 응답 (response) 를 지정합니다. 
  - read : Promise 가 정상적으로 반환되었을 경우 값을 반환하고, 그렇지 않을 경우 오류를 발생시킵니다. 

<pre>
<code class="hljs javascript"><xmp>
function App() {
  return (
    <Suspense fallback={<h1>데이터 로딩 중...</h1>}>
      <ChildComponent />
    </Suspense>
  );
}
</xmp>
</code>
</pre>

Suspense 의 fallback 속성을 통해서 데이터가 DOM 에 표시되기 전까지 "데이터 로딩 중..." 이라는 문자를 출력합니다. 

<pre>
<code class="hljs javascript">const response = fetchDummyData();

export default function ChildComponent() {
  const dummyData = response.read();
  // (생략)
</code>
</pre>

위와 같이 Suspense 로 둘러싸인 컴포넌트에서는 데이터를 컴포넌트 외부에서 불러오고 실제 값은 내부에서 사용합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/199047638-52d93f94-e076-450a-8e4f-c8c673e69f0f.png">
*4-3-1) Render as fetch / Suspense 를 사용할 때*
<br />

<br />

<strong> [4-3-1) Render as fetch / Suspense 를 사용할 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형       | 실행시간 | 일괄 처리 시간 |
|:--------:|:-------:|:--------------:|:------:|:----------:|
| 1      | 14ms  | render       | 4ms  | 5ms      |
| 2      | 16ms  | during-mount | 55ms | -        |
| 3      | 18ms  | commit       | 2ms  | 5ms      |
| 4      | 72ms  | render       | 13ms | 172ms    |
| 5      | 89ms  | render       | 5ms  | 172ms    |
| 6      | 94ms  | render       | 5ms  | 172ms    |
| 7      | 99ms  | render       | 5ms  | 172ms    |
| 8      | 104ms | render       | 5ms  | 172ms    |
| 9      | 109ms | render       | 5ms  | 172ms    |
| 10     | 114ms | render       | 5ms  | 172ms    |
| 11     | 119ms | render       | 3ms  | 172ms    |
| 12     | 121ms | commit       | 4ms  | 172ms    |

<br />

`during-mount` 라는 이벤트를 통해서 API 요청을 받기까지 약 55ms 가 걸린 것을 확인할 수 있습니다. 또한 총 2번의 commit 이 발생하는데 첫 번째 commit 은 fallback UI 를 렌더링하는 이벤트이고 두 번째 commit 은 최종적인 데이터 목록을 렌더링하는 이벤트임을 짐작할 수 있습니다. 

<br />

#### 4-3-2) 데이터를 두 개 불러올 때 

Suspense 를 사용하여 데이터를 여러 개 불러올 때도 fallback UI 를 사용할 수 있습니다.

<pre>
<code class="hljs javascript"><xmp>return (<>
  <Suspense fallback={<h1>데이터 로딩 중...</h1>}>
    <ChildPhotoList />
  </Suspense>
  <Suspense fallback={<h1>데이터 로딩 중...</h1>}>
    <ChildCommentList />
  </Suspense>
</>);
</xmp>
</code>
</pre>

여러 개의 Suspense 를 사용할 때에는 다음과 같이 순차적으로 배치하면 됩니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/199050041-5726631d-c3dc-4fa6-9a6e-9a7152795eb3.gif">
*두 개의 데이터 중 로딩이 완료된 데이터 (하단) 부터 표시되는 것을 확인할 수 있습니다.*
<br />


<br />
<img src="https://user-images.githubusercontent.com/31645195/199048186-659790b3-2f0b-4163-8aa9-2ef29774463a.png">
*4-3-2) Render as fetch / 데이터를 두 개 불러올 때*
<br />

<br />

<strong> [4-3-2) Render as fetch / 데이터를 두 개 불러올 때] </strong>

| 이벤트 순서 | 타임스탬프 | 이벤트 유형          | 실행시간 | 일괄 처리 시간 |
|:--------:|:-------:|:-----------------:|:------:|:----------:|
| 1       | 29ms  | render          | 5ms  | 6ms      |
| 2       | 33ms  | during-mount    | 34ms | -        |
| 3       | 34ms  | commit          | 2ms  | 6ms      |
| 4       | 68ms  | render          | 5ms  | 40ms     |
| 5       | 75ms  | render          | 4ms  | 40ms     |
| 6       | 79ms  | commit          | 2ms  | 40ms     |
| 7       | 108ms | passive-effects | 1ms  | 40ms     |
| 8       | 108ms | render          | 12ms | 61ms     |
| 9       | 121ms | render          | 5ms  | 61ms     |
| 10     | 127ms | render          | 5ms  | 61ms     |
| 11     | 132ms | render          | 5ms  | 61ms     |
| 12     | 138ms | render          | 5ms  | 61ms     |
| 13     | 144ms | render          | 5ms  | 61ms     |
| 14     | 152ms | render          | 9ms  | 61ms     |
| 15     | 161ms | render          | 5ms  | 61ms     |
| 16     | 166ms | commit          | 3ms  | 61ms     |

<br />

앞서 `4-3-1) Suspense 를 사용할 때` 에서 2번의 commit 이 일어났다면, 이번에는 추가로 한 번 더 commit 이 발생합니다. 이는 두 개의 Suspense 내부에서 데이터 로딩을 완료한 각 하위 컴포넌트가 렌더링되기 때문입니다. 한편 각 render 이벤트의 실행시간은 한 번의 API 요청을 보냈을 때와 큰 차이가 없는 것을 확인할 수 있습니다. 

<hr />

## 5. Concurrent Mode Hooks

React 의 함수형 컴포넌트 안에서 반환값에 사용하고 싶은 변수에 useState 를 사용할 수 있습니다. state 가 변하면 새로운 state 를 반영하기 위해서 컴포넌트가 리렌더링 됩니다. 하지만 해당 state 를 참조하는 DOM 노드가 너무 많이 존재하거나 state 의 변화가 지나치게 빈번하게 일어날 때에는 이를 실시간으로 반영하면서 발생하는 Reflow 나 Repaint 가 브라우저에 부담이 되어 렌더링 퍼포먼스를 저하시킬 수 있습니다. 

<br />

이를 해결하기 위해서 React 의 동시성 모드는 useTransition 과 useDeferredValue 라는 두 가지 Hook 을 제공합니다. 이러한 Hook 을 사용했을 때와 사용하지 않았을 때의 코드 결과를 테스트해보겠습니다. 1만 개의 <div\> 태그 안에 0으로 초기화된 state 를 표시하고 1초마다 1씩 증가하는 코드를 10초동안 실행하는 방식으로 테스트를 진행했습니다. 

<br />

### 5-1) useState 

<pre>
<code class="hljs javascript">export default function BasicUseState() {
  const [inputNumber, setInputNumber] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setInputNumber((value) => value + 1);
    }, 1000);

    setTimeout(() => {
      clearInterval(interval);
    }, 10000);
  }, []);

  // (생략)
};
</code>
</pre>

컴포넌트가 마운트되면 setInterval 내부의 setInputNumber() 함수가 1초마다 실행됩니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/199054432-841f7572-3744-4dbb-b764-068408a0fb9d.gif">
*useState 로 state 가 1초마다 바뀌면서 리렌더링이 일어나는 것을 확인할 수 있습니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/199229576-70117a0f-0986-423f-a29d-3fdc98000d3b.png">
*5-1) useState*
<br />

<br />

### 5-2) useTransition

useTransition 은 컴포넌트 내 state 의 업데이트 우선순위를 낮춰서 리렌더링을 늦게 발생시키는 역할을 합니다. 다음의 두 가지 값을 반환합니다. 
- 초기값으로 전달한 state 가 업데이트 되었는지를 나타내는 boolean 값
- 업데이트를 지연시키고 싶은 state 를 감싸는 startTransition() 함수

<pre>
<code class="hljs javascript">export default function Transition() {
const [inputNumber, setInputNumber] = useState(0);
const [pending, startTransition] = useTransition();

useEffect(() => {
  const interval = setInterval(() => {
    startTransition(() => {
      setInputNumber((value) => value + 1);
    });
  }, 1000);

  setTimeout(() => {
    clearInterval(interval);
  }, 10000);
}, []);
</code>
</pre>

`5-1)` 과 동일한 구조로 되어 있지만 startTransition() 함수가 inputNumber 의 Setter 함수를 감싸고 있습니다. 

<pre>
<code class="hljs javascript"><xmp><p>아래에 입력값이 표시됩니다.</p>
    {pending ? (
      <div>
        Pending is <strong>True</strong>
      </div>
    ) : (
      DUMMY_ARRAY.map((_, index) => <div key={index}>{inputNumber}</div>)
    )}
</xmp>
</code>
</pre>

또한 JSX 의 반환값에서 변수 `pending` 을 사용하여 바뀐 state 가 DOM 에 반영되지 않았을 때 표시할 fallback UI 를 정의합니다. 


<br />
<img src="https://user-images.githubusercontent.com/31645195/199054184-bf17fab8-1e77-4133-961c-be1f3f4a4475.gif">
*1초마다 state 가 바뀌지만 업데이트되기 전에 pending 이 true 가 되면서 fallback UI 가 나타납니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/199229581-b9b5907f-23f4-446e-a94f-8112686db10c.png">
*5-2) useTransition*
<br />


<br />

### 5-3) useDeferredValue

useDeferredValue 도 앞서 useTransition 과 동일하게 state 의 업데이트 우선순위를 낮추어서 자주 발생하는 state 변화에 대응하는 Hook 입니다. useTransition 이 startTransition() 과 같은 함수를 제공하여 우선순위를 낮추고자 하는 state 를 감싸는 방식으로 작성되었다면, useDeferredValue 는 선언 시에 state 를 초기값으로 전달합니다. 이후 해당 값의 업데이트 우선순위가 낮아지게 됩니다. 다만, state 의 업데이트 여부를 감지하는 변수는 제공되지 않습니다. 

<pre>
<code class="hljs javascript">export default function DeferredValue() {
  const [inputNumber, setInputNumber] = useState(0);
  const deferredValue = useDeferredValue(inputNumber);

  useEffect(() => {
    const interval = setInterval(() => {
      setInputNumber((value) => value + 1);
    }, 1000);

    setTimeout(() => {
      clearInterval(interval);
    }, 10000);
  }, []);

  // (생략)
} 
</code>
</pre>

useDeferredValue Hook 에서 inputNumber 를 초기값으로 지정하여 렌더링 우선순위를 낮춥니다. 이후 해당 값을 사용하는 DOM 노드들은 업데이트가 지연됩니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/199229591-f59d2b6f-4c3c-44bf-8b65-924fad3a00e3.png">
*5-3) useDeferredValue*
<br />


### 5-4) Concurrent Mode Hooks 정리 

Concurrent Mode Hooks 테스트에서는 다른 테스트 결과처럼 별도의 render 나 commit 의 실행 시간에 대해서 다루지 않았습니다. useTransition 과 useDeferredValue 에 대해서 설명할 때 업데이트 우선순위를 낮춘다는 표현을 사용했습니다. Profiler 에서는 이와 관련하여 실행 시간보다 더 중요한 정보인 `lanes` 라는 데이터를 제공합니다. 

<br />

lanes 는 React 의 `재조정` (Reconciliation : DOM 을 재구성하는 과정을 가리킴) 엔진인 `fiber` 에서 사용하는 값으로, 업데이트의 우선순위를 나타내는 숫자입니다. 각 작업 유형에 따라서 도로에 있는 차선같이 다양한 lanes 가 있고 각각의 우선순위가 존재합니다. 재조정이나 fiber 에 대해서 다루기에는 양이 매우 방대하기 때문에 lanes 에 대한 특징만을 간단하게 정리하면 다음과 같습니다. 

- 32비트로 작업의 우선순위를 나타냅니다.
- 자손에 대한 우선순위는 childLanes 에 저장됩니다. 
- lanes 값과 우선순위는 반비례합니다. 

<br />

따라서 이러한 lanes 가 1일 경우 가장 높은 우선순위를 가지게 됩니다. 다음은 동시성 모드 테스트 결과에서 확인할 수 있는 lanes 정보를 정리하여 나타낸 것입니다. 

<br />

| 테스트 번호 | Hook             | lanes (1) | lanes (2) | lanes (3) | lanes (4) |
|:--------:|:------------------:|:-----------:|:-----------:|:-----------:|:-----------:|
| 5-1    | useState         | 16        | 16        | 16        | 16        |
| 5-2    | useTransition    | 16        | 64        | 128       | 256       |
| 5-3    | useDeferredValue | 64        | 128       | 256       | 512       |

<br />

위의 표에서 확인할 수 있듯이 useState 만 사용한 경우에는 lanes 가 일정하게 16으로 고정되어 있는 것을 확인할 수 있었습니다. 나머지 Concurrent Mode 의 두 Hook 은 서로 값이 다르긴 하지만 점차 lanes 가 높아지는 것을 알 수 있습니다. 즉, lanes 값이 높아지기 때문에 우선순위가 낮아져 업데이트를 지연시키는 원리가 작동한다는 것을 확인할 수 있습니다. 

<hr />


## 6. 결론

- React Profiler 탭에서 React 와 각 컴포넌트의 이벤트를 확인할 수 있습니다. 
- Data Fetching 방식에는 세 가지 방법이 있습니다. 
  1. Fetch on render
      + 컴포넌트가 마운트된 뒤에 데이터를 불러옵니다. 
      + 늦은 DOM 업데이트를 통해서 사용자 경험을 해칠 수 있습니다.
      + Layout Shift 가 일어날 가능성이 높습니다. 
  2. Fetch then render
      + 컴포넌트 호출보다 데이터를 먼저 불러오기 시작합니다. 
      + 여러 개의 서로 다른 API 요청을 보낼 때 비효율적일 수 있습니다. 
  3. Render as fetch 
      + 컴포넌트와 데이터를 동시에 불러옵니다. 
      + React Suspense 에서는 데이터 로딩 중 렌더링 할 fallback UI 를 지정할 수 있습니다.
      + 여러 개의 Suspense 를 사용할 수 있습니다. 
- React 의 state 를 참조하는 DOM 노드가 많고 값이 빈번하게 바뀔 경우 성능이 저하됩니다. 
- 동시성은 작업을 잘게 나누고 빠르게 번갈아 가면서 수행하는 것을 말합니다. 
- React Concurrent Mode 에서는 두 가지 Hook 을 제공합니다. 
  1. useTransition
      + startTransition() 함수 내부에 정의된 state Setter 함수의 우선순위가 낮아집니다. 
  2. useDeferredValue
      + 초기값으로 전달한 state 의 업데이트 우선순위가 낮아집니다. 
- 위 두 Hook 이 우선순위를 낮추는 방법은 fiber 가 해석하는 lanes 의 값을 증가시키는 것입니다.

<hr />

## 7. 생각해볼 점

- 각 테스트 케이스 별 실행 시간 차이가 정말로 의미있는 차이를 보이는 것인지 생각해볼 필요가 있습니다.
- 테스트를 진행할 때 브라우저에 캐싱되어 있는 데이터를 불러오는 것은 아닌지 테스트하는 방법을 생각해볼 수 있습니다. 
- 다양한 Data Fetching 방법과 동시성 모드를 실제 구현 단계에서 어떻게 적재적소에 사용할 수 있을지 고민해보아야 합니다. 
- React 의 재조정 알고리즘과 fiber 의 내부 구조에 대해서 알아두면 작동 원리를 이해하는 데에 좋을 것 같습니다. 

<hr />

[출처 & 참고]

- [Javascript에서의 동시성과 Event loop](https://godsenal.com/posts/Javascript%EC%97%90%EC%84%9C%EC%9D%98-%EB%8F%99%EC%8B%9C%EC%84%B1%EA%B3%BC-Event-loop/)

- [What is React Concurrent Mode?](https://velog.io/@cadenzah/react-concurrent-mode)

- [Concurrent Mode API Reference (Experimental) - React](https://17.reactjs.org/docs/concurrent-mode-reference.html#suspenselist)

- [React Suspense 이해하기](https://developer-alle.tistory.com/428)

- [Experimental React: Using Suspense for data fetching - LogRocket Blog](https://blog.logrocket.com/react-suspense-data-fetching/#fetch-on-render)

- [How does SuspenseList work in React? React Source Code Walkthrough 25](https://jser.dev/react/2022/06/19/how-does-suspense-list-work.html)

- [What is "Lane" in React?](https://dev.to/okmttdhr/what-is-lane-in-react-4np7)

- [What is Lanes in React source code? - React Source Code Walkthrough 21](https://jser.dev/react/2022/03/26/lanes-in-react.html)

- [How to use Transitions in React 18](https://blog.shahednasser.com/how-to-use-transitions-in-react-18/)

- [React 18 에서 달라지는점](https://velog.io/@katanazero86/React-18-%EC%97%90%EC%84%9C-%EB%8B%AC%EB%9D%BC%EC%A7%80%EB%8A%94%EC%A0%90)

- [3분 리액트 - React18의 useTransition, useDeferredValue](https://itchallenger.tistory.com/654)

- [React 18 useDeferredValue로 성능 최적화하기](https://vroomfan.tistory.com/45)

- [리액트 useTransition, useDeferredValue 정리](https://velog.io/@brgndy/%EB%A6%AC%EC%95%A1%ED%8A%B8-useTransition-useDeferredValue-%EC%A0%95%EB%A6%AC)

- [Hooks API Reference - React](https://reactjs.org/docs/hooks-reference.html#usedeferredvalue)

<br />

[코드 링크]
- [https://github.com/iyu88/React-Concurrent-Mode](https://github.com/iyu88/React-Concurrent-Mode)

<br />
