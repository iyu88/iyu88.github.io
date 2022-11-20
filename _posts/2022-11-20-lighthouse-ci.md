---
layout: post
title: "[Lighthouse] Lighthouse CI 적용기 - GitHub Actions 만들기"
date: 2022-11-20 00:00:00
author: Hyunbin Lee
categories: Lighthouse
tags: Lighthouse CI GitHub_Actions 
cover: "/assets/instacode.png"
---

# Lighthouse CI 적용기 - GitHub Actions 만들기

> ### 💡 웹 성능 측정 도구인 Lighthouse 를 적용하고 GitHub Actions 를 만들어봅니다.

## 1. Lighthouse 도입 배경 

부스트캠프 그룹 프로젝트 기간 첫 주차에 저희 조 주제를 바탕으로 마스터께서 `테크 스펙`을 작성해주셨습니다. 이 때, 임팩트 측정에서 눈길을 사로 잡는 한 단어가 있었습니다. 바로 `라이트하우스` 입니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202853153-e3071593-0a9d-4bc9-ab64-1adf8557b0c5.png">
*임팩트 측정에서 Lighthouse 에서 제공하는 기준을 적용합니다.*
<br />

우리의 제품이 좋다고 말하기 위해서 성능을 측정할 때 라이트하우스를 사용하는 것이 좋다는 조언을 듣고서 이를 적용해보기로 결정했습니다. 어렴풋이 이름을 들어본 기억이 있으나 실제로 사용해본 적이 없기 때문에 먼저 무엇인지 알아봐야겠다고 생각했습니다. 

<br />

## 2. Lighthouse 가 뭐에요?

Lighthouse 는 구글에서 개발한 웹 성능 측정 오픈 소스 툴입니다. 개발자 도구 탭에서 `Lighthouse` 탭을 선택하여 현재 페이지의 품질을 측정할 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202853412-df35c041-98ee-493f-b464-5d9add2ed89f.png">
*'Analyze page load' 버튼을 누르면 도구를 실행합니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/202853538-e24e875f-a9ff-489e-a217-fc8b1a66e80b.png">
*블로그 성능을 측정한 결과입니다. 여러 가지 지표가 상단에 나타납니다. 스크롤을 내리면 세부정보를 확인할 수 있습니다.*
<br />

## 3. 지표 소개 

상단에 나타나는 5가지 지표에 대한 간단한 설명과 몇 가지 세부 지표에 대해서 알아보겠습니다. 

<br />

### 3-1. Performance

말 그대로 웹 페이지의 성능입니다. 웹 페이지 요소들이 얼마나 빨리 화면에 렌더링되는지에 대해서 측정합니다. 방금 전 블로그 성능을 측정한 개발자 도구에서 Performance 를 누르면 다음과 같이 어떠한 요소들이 계산에 반영되었는지 확인해볼 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202853910-19cb9e0a-a0e5-4993-a9d7-59c38a3a8544.png">
*See calculator 를 누르면 새로운 창이 열립니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/202853925-aada9bd9-47b0-4a3c-a177-f5c238d65f68.png">
*성능을 측정하기 위해서 위에서 확인할 수 있는 6가지 지표가 활용되었음을 알 수 있습니다.*
<br />

6가지 중 FCP 와 LCP 는 다음과 같습니다. 
- First Contentful Paint ( FCP ) : 브라우저가 의미 있는 첫 번째 콘텐츠를 렌더링 할 때까지 걸리는 시간을 의미합니다. 웹 사이트를 이용자들이 바로 느낄 수 있는 부분이기 때문에 사용자 경험과 연관이 있습니다. 또한 렌더링 성능이 SEO 에도 영향을 주기 때문에 아래에서 설명할 SEO 점수와도 관계가 있습니다. 
- Largest Contentful Paint ( LCP ) : 화면에서 가장 큰 요소가 화면에 렌더링 될 때까지 걸리는 시간을 의미합니다. 단, 뷰포트를 벗어난 요소는 포함하지 않습니다. 

<br />

### 3-2. Accessibility 
웹 접근성과 관련된 지표입니다. 웹 접근성을 위해서 '적절한 HTML 태그를 사용했는지', '올바른 Attribute 에 key 와 value 가 들어가 있는지' 와 같은 것을 측정합니다. 또한 색상 대비도 측정 요소에 포함됩니다. 

<br />

### 3-3. Best Practices
웹 페이지가 오류를 발생하는지 확인합니다. 콘솔 창에 생기는 오류, HTTPS 지원 여부, 소스 코드가 실제 동작에 활용되는지 여부가 점수에 반영됩니다. 실제로 블로그 성능 평가에서 Best Practices 가 75점으로 저조한 모습을 보이는데, 콘솔 창에 warning 과 issue 들이 존재하기 때문임을 알 수 있습니다.  

<br />
<img src="https://user-images.githubusercontent.com/31645195/202854567-120c45be-14fd-4e18-b38a-b002be89a451.png">
*무수히 많은 노란색 로그들이 보입니다.*
<br />

### 3-4. SEO

Search Engine Optimization 의 준말인 SEO 는 웹 페이지가 기본적인 검색 엔진 최적화를 얼마나 잘 준수하고 있는지 수치화합니다. 검색 엔진에 대해서 최적화가 잘 되어 있을수록 검색 결과의 상단에 위치하게 됩니다. 다만, Lighthouse SEO 수치에는 검색 결과 순위에 영향을 주는 몇몇 요소들이 반영되지 않을 수 있습니다. 

<br />

### 3-5. PWA
Progressive Web App 의 준말인 PWA 는 웹 사이트를 어플리케이션처럼 사용할 수 있도록 하는 기술입니다. Lighthouse 에서 측정하는 기준은 바로 PWA 의 특징들과 관련이 있습니다. manifest 파일이 존재하여 앱을 다운로드 했을 때의 설정이 있는지 확인하거나 PWA 의 핵심 기능인 service-worker 를 사용하고 있는지 또는 캐싱을 어떻게 하고 있는지 등을 확인합니다. 추가적으로 다양한 화면 크기를 지원하는지와, 오프라인 접속 가능 여부, 설치 가능 여부 등도 지표에 포함됩니다. 

<br />

## 4. 설치 및 실행 

본격적으로 프로젝트에 설치해보겠습니다. React 프로젝트 내부로 이동하여 다음의 커맨드를 입력합니다. 

{% highlight shell%}
npm i -g @lhci/cli
{% endhighlight %}

같은 경로에 `.lighthouserc.json` 파일을 생성합니다.


{% highlight json%}
{
  "ci": {
    "collect": {
      "staticDistDir": "./build",
      "url": ["http://localhost:3000"],
      "numberOfRuns": 5
    },
    "upload": {
      "target": "temporary-public-storage"
    }
  }
}
{% endhighlight %}

- staticDistDir : 성능 측정에 사용할 HTML 파일이 위치하는 경로를 입력합니다. React 를 빌드 후 성능 측정하기 위해서 './build' 경로로 설정했습니다. 
- url : 성능을 측정할 url 주소를 명시합니다. 한 개 이상 들어갈 수 있습니다. 
- numberOfRuns : 성능 측정 횟수를 정할 수 있습니다. 기본값은 3입니다. 
- target : Lighthouse 보고서가 저장되는 유형을 선택합니다. 위의 
방식은 임시 스토리지에 저장하는 방식입니다. 

<br />

json 파일을 작성한 뒤에 React 를 빌드하고 Lighthouse 보고서를 확인해봅니다. 아래의 명령어를 차례대로 실행합니다. 

{% highlight shell%}
npm run build 
lhci autorun
{% endhighlight %}


<br />
<img src="https://user-images.githubusercontent.com/31645195/202856974-1d894bbc-8d47-437e-aa1c-d5823d0a8f24.png">
*성능 측정 중...*
<br />


<br />
<img src="https://user-images.githubusercontent.com/31645195/202857238-288d882f-4867-458d-bde1-5d55e7ce19b0.png">
*콘솔 창에 있는 포트 번호로 접속하면 개발자 도구에서 봤던 보고서를 확인할 수 있습니다.*
<br />


## 5. Lighthouse CI - GitHub Actions

위에서 Lighthouse 를 사용하는 방법에 대해서 알아보았습니다. 하지만, 이렇게 매번 명령어를 입력해서 결과를 확인하는 작업은 너무나 번거롭습니다. 따라서 이번에는 GitHub Actions 를 사용하여 Lighthouse 를 자동화해보겠습니다. 

<br />

먼저 GitHub 에 PR 이 올라왔을 때 Lighthouse 를 실행하는 yaml 파일 구성입니다. 

{% highlight yaml%}
{% assign LHCI_GITHUB_APP_TOKEN = "{{ secrets.LHCI_GITHUB_APP_TOKEN }}" -%}
# .github/workflows/lighthouse-ci.yaml
name: Run lighthouse CI When Pull Request

on:
  pull_request:
    branches: ['dev'] # dev 브랜치에서만 작동하도록 합니다.
    paths:
      - './frontend/**' # frontend 폴더 아래에 변경 사항이 있을 때에만 실행합니다. 
  # 옵션을 추가하여 GitHub Repo 의 Actions 탭에서 수동으로 실행할 수 있게 합니다. 
  workflow_dispatch: 

jobs:
  lhci:
    name: Lighthouse CI
    runs-on: ubuntu-latest # 본 동작을 실행할 운영체제를 명시합니다. 
    steps:
      - name: Checkout
      - uses: actions/checkout@v3 # 소스 코드를 다운로드합니다. 

      - name: Use Node.js 18.x # Node 버전을 지정할 수 있습니다. 
        uses: actions/setup-node@v3
        with:
          node-version: 18.x
      
      # Actions 의 실행 위치가 root 이기 때문에 frontend 폴더로 이동하여 
      # 패키지 다운로드 & 빌드를 실행합니다. 
      # Lighthouse CLI 를 설치하고 autorun 명령어를 실행합니다.
      - name: Install packages
        run: |
          cd ./frontend
          npm ci
      - name: Build
        run: |
          npm run build
      - name: Run Lighthouse CI
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ LHCI_GITHUB_APP_TOKEN }}
        run: |
          npm install -g @lhci/cli
          lhci autorun || echo "🚨 Fail to Run Lighthouse CI!"
{% endhighlight %}

<br />

> 처음에는 yaml 파일 내부에 바로 시크릿 키를 명시했었습니다. 나중에 커밋을 강제로 삭제하는 작업까지 수행했기 때문에 시크릿 키를 등록하는 부분은 사진을 많이 첨부해서 설명하겠습니다. 

<br />

마지막 동작에서 `secrets.LHCI_GITHUB_APP_TOKEN` 라는 값이 있습니다. 이는 LHCI_GITHUB_APP_TOKEN 이라는 이름의 변수를 env (환경 변수) 로 사용하고 있는 것입니다. Lighthouse 를 사용하기 위한 토큰은 다음의 [링크](https://github.com/apps/lighthouse-ci) 에서 다운로드 받을 수 있습니다. 위의 GitHub Actions 를 등록하고 싶은 Repo 를 찾아서 비밀 토큰을 발급 받습니다. 해당 값을 환경 변수로 저장하기 위해서 Repo 의 Settings 탭으로 이동합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202857992-19e36607-7080-42d7-82f6-8d7fe8388f66.png">
*Settings*
<br />

좌측의 사이드바에서 `Security - Secrets - Actions` 로 이동합니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202858412-d54bf761-4b24-4b66-89e7-8ebbe380e4df.png">
*Actions*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/202858483-62e2bc75-b4bb-493f-9a03-2cdc8a52fd16.png">
*새로운 비밀 환경 변수를 추가합니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/202858484-eff4b97d-39d2-4945-82c5-7654e96a5268.png">
*Actions 에서 사용할 변수 이름과 위에서 발급 받은 Lighthouse 비밀 토큰을 key-value 로 저장합니다.*
<br />


## 7. 결과값을 댓글로 추가하기

결과를 댓글로 추가하기 위해서는 두 가지 작업이 필요합니다. 
첫 번째로 `.lighthouserc.json` 파일을 수정해서 보고서 결과를 별도의 파일에 저장합니다. 
두 번째로 `lighthouse-ci.yaml` 파일에 보고서 결과를 추출하는 동작과 댓글에 추가하는 동작을 더해야 합니다. 

<br />

### 7-1. .lighthouserc.json 수정 

기존에 결과를 임시 스토리지에 저장하는 방식에서 폴더를 추가 생성하여 정적인 파일로 저장하도록 변경합니다.  

{% highlight json%}
{
  "ci": {
    "collect": {
      "staticDistDir": "./build",
      "url": ["http://localhost:3000"],
      "numberOfRuns": 5
    },
    "upload": {
      # 파일로 저장하기 위해서 target 값을 변경합니다. 
      "target": "filesystem", 
      # 보고서 결과가 위치할 폴더의 이름을 정합니다. 
      "outputDir": "./lhci_reports",
      # 각 보고서 파일 이름 규칙을 지정합니다. 
      "reportFilenamePattern": "%%PATHNAME%%-%%DATETIME%%-report.%%EXTENSION%%"
    },
    "assert": {
      "assertions": {
        "first-contentful-paint": ["warn", { "minScore": 0.75 }],
        "largest-contentful-paint": ["warn", { "minScore": 0.75 }]
      }
    }
  }
}
{% endhighlight %}


위에서 `assert` 라는 속성이 추가된 것을 확인할 수 있습니다. 내부에서 선택한 값이 지정한 기준을 만족하지 못할 경우 실행할 동작을 고를 수 있습니다. 예를 들어 위의 경우에는 `first-contentful-paint 가 최소 점수 0.75 를 넘지 못하면 경고 (warn)` 하는 옵션을 주었습니다. 설정해준 값에 따라서 기준을 만족하지 못하면 에러를 반환하도록 설정할 수도 있습니다. 

<br />

### 7-2. lighthouse-ci.yaml 수정 

이제 Actions 에 보고서 결과를 요약하고 댓글에 추가하는 동작을 정의하겠습니다. 


{% highlight yaml%}
{% assign GITHUB_TOKEN = "{{ secrets.GITHUB_TOKEN }}" -%}
- name: Format lighthouse score
    id: format_lighthouse_score
    uses: actions/github-script@v6
    with:
      github-token: ${{ GITHUB_TOKEN }}
      script: |
        const fs = require('fs');
        const results = JSON.parse(fs.readFileSync("./frontend/lhci_reports/manifest.json"));
        let comments = "";
        results.forEach((result) => {
          const { summary, jsonPath } = result;
          const details = JSON.parse(fs.readFileSync(jsonPath));
          const { audits } = details;
          
          const formatScore = (res) => Math.round(res * 100);
          Object.keys(summary).forEach(
            (key) => (summary[key] = formatScore(summary[key]))
          );
          const scoreEmoji = (res) => (res >= 90 ? "🟢" : res >= 50 ? "🟠" : "🔴");
          const reportRow = (label, score) => `| ${scoreEmoji(score)} ${label} | ${score} |`;
          const detailRow = (label, score, displayValue) => `${reportRow(label, score)} ${displayValue} |`;
          const report = [`⚡️ Lighthouse Report!`,
          `| Category | Score |`,
          `| -------- | ----- |`,
          `${reportRow('Performance', summary.performance)}`,
          `${reportRow('Accessibility', summary.accessibility)}`,
          `${reportRow('Best practices', summary['best-practices'])}`,
          `${reportRow('SEO', summary.seo)}`,
          `${reportRow('PWA', summary.pwa)}`].join('\n');
          const detail = [`🔎 Performance Details`,
          `| Category | Score | DisplayValue |`,
          `| -------- | ----- | ------------ |`,
          `${detailRow("First Contentful Paint", audits['first-contentful-paint'].score * 100, audits['first-contentful-paint'].displayValue)}`,
          `${detailRow("Largest Contentful Paint", audits['largest-contentful-paint'].score * 100, audits['largest-contentful-paint'].displayValue)}`].join('\n');
          comments += report + "\n" + detail + "\n";
          console.log(comments);
        });
        core.setOutput('comments', comments)
{% endhighlight %}

위의 스크립트는 root 주소에서 Lighthouse 의 보고서 파일이 있는 경로로 이동하여 해당 파일로부터 원하는 지표만을 추출하여 Markdown Table 형식으로 가공하는 작업을 합니다. 

핵심적인 부분은 보고서를 생성하면 manifest.json 이라는 파일을 생성하게 되는데, 해당 파일 내용을 읽어서 안에 있는 요약 정보와 세부 지표를 가져온다는 것입니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202860127-7a895edf-3540-42b8-b2f3-a5be54bd7948.png">
*.lighthouserc.json 에서 정의한 경로와 이름대로 보고서가 생성되어 배열 형태로 저장된 것을 확인할 수 있습니다.*
<br />

`numberOfRuns` 로 설정한 횟수만큼 수행한 측정 결과에 대해 summary 에서 5가지 주요 지표를 확인할 수 있습니다. 다른 key 에 저장된 값은 무엇일까요? jsonPath 에 있는 경로로 들어가 보았습니다. 


<br />
<img src="https://user-images.githubusercontent.com/31645195/202860399-28c01d86-ccd9-4a9b-ae2c-4bbae8d02fb7.png">
*가독성이 심히 안 좋은 json 파일이 있습니다.*
<br />

무엇인지 확인해보기 위해서 Web json formatter 로 정돈해보았습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202860551-9ea9a8ec-eb6d-4f1a-aa20-0a061608e4de.png">
*깔끔하게 정리되니까 가져다 쓸 정보가 보입니다.*
<br />

해당 json 파일 내부에 audits 라는 속성 아래에 `first-contentful-paint` 와 `largest-contentful-paint` 라는 글자가 보입니다. 또한 그 하위에는 `score`, `displayValue` 등의 지표가 저장되어 있습니다. 따라서 이러한 값들을 위 yaml 파일에 정의한 스크립트에서 읽어서 표로 가공하고 있는 것입니다. 

<br />

### 7-3. 댓글에 추가하는 액션 추가 

앞서 한 번 언급했지만 위의 스크립트는 Markdown Table 형식의 문자열을 반환합니다. 따라서 해당 값을 다음 작업에서 전달받아 댓글에 추가하면 됩니다. 이를 위해서 `unsplash/comment-on-pr@v1.3.0` 라는 액션을 사용했습니다. 

{% highlight yaml%}
{% assign GITHUB_TOKEN = "{{ secrets.GITHUB_TOKEN }}" -%}
{% assign comments = "{{ steps.format_lighthouse_score.outputs.comments }}" -%}
- name: comment PR
    uses: unsplash/comment-on-pr@v1.3.0
    env:
      GITHUB_TOKEN: ${{ GITHUB_TOKEN }}
    with:
      msg: ${{ comments}}
{% endhighlight %}


PR 을 올렸을 때 정상적으로 댓글에 테이블이 표시되는지 확인해보겠습니다. 


<br />
<img src="https://user-images.githubusercontent.com/31645195/202861006-ac1a1a40-8e46-4abb-a737-2e6869f99140.png">
*5개의 주요 지표와 2개의 세부 지표가 표로 정상 출력됩니다.*
<br />


## 8. 커스텀 액션 적용기 

하지만 여기서 문제점이 있습니다. 바로 보고서를 문자열로 가공하는 스크립트가 너무 길다는 점입니다. 이를 짧게 표현할 수 있는 방법이 없을까 하다가 마스터 클래스에서 들었던 커스텀 GitHub Actions 를 만들어 보기로 했습니다. 다음 [링크](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)의 튜토리얼을 참고했습니다. 


먼저 새로운 Repo 를 Public 으로 생성하고 root 에 action.yml 이라는 파일을 생성합니다. 파일 내부를 다음과 같이 작성했습니다. 

{% highlight yaml%}
name: 'lighthouse-report-formatter'
description: 'Format Lighthouse report into Markdown Table'
author: "iyu88"
inputs:
  lh_directory:  
    description: 'directory path to which the lighthouse is applied'
  manifest_path:
    description: 'path to which the reports manifest.json is saved'
    required: true
    default: '/lhci_reports'
runs:
  using: 'node16'
  main: 'index.js'
branding:
  icon: 'sun'
  color: 'orange'
{% endhighlight %}

각 속성의 역할은 다음과 같습니다. 

- name: GitHub 의 Actions 탭에서 해당 액션을 식별할 수 있는 이름입니다. 
- description: 액션에 대한 간단한 소개를 적습니다. 
- author: 액션을 만든 사람을 적습니다. 제 GitHub 아이디를 입력했습니다. 
- inputs: 액션을 실행하기 위해서 입력 받아야 하는 변수를 적습니다. 함수의 매개변수 개념입니다. 
  - description: 각 input 에 대한 짧은 소개입니다. 
  - required: 필수적인 요소인지 boolean 값으로 정합니다. 
  - default: 사용자가 값을 명시하지 않았을 경우에 사용할 기본값을 적습니다. 
- runs: 해당 액션이 실행되는 환경을 말합니다. Node.js 환경에서 실행합니다. 
  - using: 구체적인 Node 버전을 명시합니다. 
  - main: Node 환경에서 실행할 파일 이름입니다. 
- branding: Marketplace 에 릴리즈했을 때 나타나는 아이콘 이미지를 설정합니다. 

<br />

위에서 확인할 수 있듯이 사용자는 `Lighthouse 가 설치된 위치`를 입력해야 하고, `.lighthouserc.json 파일에 정의된 outputDir` 을 입력값으로 받습니다. 실제 동작은 index.js 안에 작성합니다. 앞서 lighthouse.yaml 에 있던 스크립트를 가져옵니다. GihHub 에서 공개한 인터페이스를 사용하기 위해서는 다음의 패키지를 설치해야 합니다. 

{% highlight shell%}
npm i @actions/core
{% endhighlight %}

앞서 action.yml 에서 `inputs` 으로 받았던 값을 스크립트 내부에서 사용하기 위해서 다음과 같이 코드를 작성합니다. 

{% highlight javascript%}
const lh_directory = core.getInput('lh_directory');
const manifest_path = core.getInput('manifest_path');
{% endhighlight %}

또한 index.js 의 반환값을 다음 actions 에 전달하기 위해서 setOutput() 이라는 메서드를 사용해야 합니다. 다음과 같이 첫 번째 인자로 입력된 문자열로 두 번째 인자로 전달된 값에 접근할 수 있습니다. 

{% highlight javascript%}
const comments = formatLighthouseReportTable(lh_directory, manifest_path);
core.setOutput("comments", comments);
{% endhighlight %}

> 여기까지 진행한 뒤에 바로 Marketplace 에 릴리즈하고 적용을 하니 아예 Actions 가 적용되지 않는 문제가 생겼습니다. 배포 전에 해야할 일이 하나 더 있다는 것을 알아차리지 못했습니다. 

이제 index.js 와 의존성이 있는 node 패키지들을 하나의 파일로 합쳐야 합니다. 왜냐하면 node_modules 전체를 저장소에 올리는 것은 비효율적이기 때문입니다. 따라서 @zeit/ncc 라는 라이브러리를 사용하여 index.js 와 종속성이 있는 패키지를 하나의 파일로 만듭니다. 

{% highlight shell%}
npm i -g @zeit/ncc
ncc build index.js
{% endhighlight %}

위의 build 명령어를 사용하면 dist/index.js 파일이 새롭게 생깁니다. 따라서 action.yml 에서 실행하는 파일 경로도 수정합니다. 

{% highlight yaml%}
runs:
  using: 'node16'
  main: 'dist/index.js'
{% endhighlight %}


이제 Marketplace 에 배포를 진행합니다.

<br />
<img src="https://user-images.githubusercontent.com/31645195/202894399-1dc52d36-4390-4134-99c9-6784a6ab5b11.png">
*GitHub Repo 의 우측 사이드바에서 'Create a new release' 버튼을 클릭합니다.*
<br />

<br />
<img src="https://user-images.githubusercontent.com/31645195/202894612-674d6493-738f-4d70-aa5e-b6025fd625e3.png">
*사진에 있는 입력 폼을 하나씩 채워줍니다.*
<br />

1. 간단한 계정 인증 절차를 거치고 action.yml 에 필요한 값이 있는지 확인합니다. 
2. README 파일을 필수적으로 작성해야 합니다. 
3. 해당 액션의 카테고리를 지정합니다. 
4. 버전 태그와 브랜치를 지정합니다. 
  - 여기서 정한 버전 태그를 추후 해당 액션을 사용할 때 적어주어야 합니다. 
5. 간단한 설명을 적습니다. 

이렇게 작성을 완료하면 아래의 사진처럼 등록이 완료되고 Marketplace 에서 확인할 수 있습니다. 

<br />
<img src="https://user-images.githubusercontent.com/31645195/202894769-b2c75ab0-0e08-4692-8886-07c2baf7db0a.png">
*Marketplace 등록 완료!*
<br />

실제 lighthouse-ci.yaml 의 전문은 다음과 같습니다. 파일이 매우 짧아집니다! 성공적으로 배포를 끝마쳤습니다.


{% highlight yaml%}
{% assign GITHUB_TOKEN = "{{ secrets.GITHUB_TOKEN }}" -%}
{% assign LHCI_GITHUB_APP_TOKEN = "{{ secrets.LHCI_GITHUB_APP_TOKEN }}" -%}
{% assign comments = "{{ steps.format_lighthouse_score.outputs.comments }}" -%}
name: Run lighthouse CI When Pull Request

on:
  pull_request:
    branches: ['dev']
    paths:
      - './frontend/**'
  workflow_dispatch:

jobs:
  lhci:
    name: Lighthouse CI
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Use Node.js 18.7.0
        uses: actions/setup-node@v3
        with:
          node-version: 18.7.0

      - name: Install packages
        run: |
          cd ./frontend
          npm ci

      - name: Build
        run: |
          cd ./frontend
          npm run build
        env: 
          CI: false

      - name: Run Lighthouse CI
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ LHCI_GITHUB_APP_TOKEN }}
        run: |
          cd ./frontend
          npm install -g @lhci/cli
          lhci autorun || echo "🚨 Fail to Run Lighthouse CI!"

      - name: Format lighthouse score
        id: format_lighthouse_score
        uses: iyu88/lighthouse-report-formatter@1.0.0
        with:
          lh_directory: ./frontend/
          manifest_path: lhci_reports

      - name: comment PR
        uses: unsplash/comment-on-pr@v1.3.0
        env:
          GITHUB_TOKEN: ${{ GITHUB_TOKEN }}
        with:
          msg: ${{ comments }}

{% endhighlight %}

<br />

## 9. 결론

- 웹 성능 측정 도구인 Lighthouse 를 설치하고 자동으로 PR 댓글에 보고서 결과 요약을 볼 수 있도록 GitHub Actions 를 사용했습니다. 
- 추가적으로 Marketplace 에 자체적으로 만든 액션을 등록했습니다. 
- 앞으로 PR 을 올릴 때마다 프론트엔드에서의 성능 측정을 자동화할 수 있습니다. 
- GitHub Actions 를 처음 써보지만 직접 만들어보는 경험을 통해서 익숙해질 수 있었습니다. 

<br />

## 10. 생각해볼 점
- 직접 만든 액션을 개선할 수 있습니다. 
  - 현재 댓글에 5번의 결과 모두 표시되는데 \<details\> 태그를 사용하여 짧게 줄일 수 있습니다.
    - 또는 inputs 로 표시 횟수를 조절할 수 있게 합니다.  
  - 세부 정보는 FCP 와 LCP 만 표시되고 있는데 inputs 로 원하는 값을 입력할 수 있게 합니다. 
  - manifest.json 파일을 찾는 방식을 더 간단하게 할 수 있습니다. 
    - glob 이라는 라이브러리를 사용하여 경로만 입력하면 파일을 손쉽게 찾을 수 있습니다. 

<br />

[출처 & 참고]
- [Lighthouse CI를 알아보고 Github Actions에 적용하기](https://fe-developers.kakaoent.com/2022/220602-lighthouse-with-github-actions/)

- [Lighthouse 사용법](https://cloud-library.tistory.com/entry/Lighthouse-%EC%82%AC%EC%9A%A9%EB%B2%95)

- [Google Lighthouse (SEO)](https://velog.io/@w-hyacinth/Google-Lighthouse-SEO)

- [PWA(Progressive Web Apps)란 무엇이고 장단점은 무엇인가](https://ux.stories.pe.kr/224)

- [라이트하우스 성능 지표 살펴보기](https://medium.com/jung-han/%EB%9D%BC%EC%9D%B4%ED%8A%B8%ED%95%98%EC%9A%B0%EC%8A%A4-%EC%84%B1%EB%8A%A5-%EC%A7%80%ED%91%9C-%EC%82%B4%ED%8E%B4%EB%B3%B4%EA%B8%B0-83df3dc96fb9)

- [Lighthouse 사용법](https://velog.io/@dell_mond/Lighthouse-%EC%82%AC%EC%9A%A9%EB%B2%95)

- [What makes a good Progressive Web App?](https://web.dev/pwa-checklist/)

- [[Github]깃허브 액션이란](https://jinmay.github.io/2020/05/13/git/github-action/)

- [[React] github action을 이용한 자동 배포 환경 구성](https://satisfactoryplace.tistory.com/361)

- [Lighthouse meet GitHub Actions - I CAN MAKE THIS WORK](https://blog.johnnyreilly.com/2022/03/20/lighthouse-meet-github-actions#add-lighthouse-stats-as-comment)

- [Always fails when opening a PR #7](https://github.com/unsplash/comment-on-pr/issues/7)

- [macOS에 nvm설치하는 방법! ( feat. brew )](https://somjang.tistory.com/entry/macOS%EC%97%90-nvm%EC%84%A4%EC%B9%98%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95-feat-brew)

- [Creating a JavaScript action - GitHub Docs](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)

- [🎬 Github Action을 마켓에 등록해보자](https://medium.com/jung-han/github-action%EC%9D%84-%EB%A7%88%EC%BC%93%EC%97%90-%EB%93%B1%EB%A1%9D%ED%95%B4%EB%B3%B4%EC%9E%90-7a181a0b4a8f)

- [직접 만든 Github Action을 Marketplace에 등록해보자.](http://www.kwangsiklee.com/2022/03/%EC%A7%81%EC%A0%91-%EB%A7%8C%EB%93%A0-github-action%EC%9D%84-marketplace%EC%97%90-%EB%93%B1%EB%A1%9D%ED%95%B4%EB%B3%B4%EC%9E%90/)

- [나만의 GitHub Actions 만들기(JS를 이용하여)](https://velog.io/@adam2/%EB%82%98%EB%A7%8C%EC%9D%98-GitHub-Actions-%EB%A7%8C%EB%93%A4%EA%B8%B0JS%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%98%EC%97%AC)

[코드 링크]
- [Gist - .lighthouserc.json](https://gist.github.com/iyu88/a1b23f315a57b0b48f91ccde8adb8f19)
- [Gist - lighthouse.yaml](https://gist.github.com/iyu88/050ae60d4dc217dd7b92020fc8b8c0ef)
- [Gist - lighthouse-ci.yaml](https://gist.github.com/iyu88/49bebfafb1711778ad2c6f6768936006)
- [GitHub - lighthouse-report-formatter](https://github.com/iyu88/lighthouse-report-formatter)