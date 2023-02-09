---
layout: post
title: "[AWS] Amazon EC2 를 사용해서 프로젝트 배포하기 - 사진으로 보는 튜토리얼"
date: 2023-02-09 00:00:00
author: Hyunbin Lee
categories: Deployment
tags: aws ec2 deployment
cover: "/assets/instacode.png"
---

# Amazon EC2 를 사용해서 프로젝트 배포하기 - 사진으로 보는 튜토리얼

> ### 💡 프로젝트 배포를 위해 AWS 인스턴스를 생성하는 과정을 사진으로 정리합니다.

## 1. 배포하기 

작년 12월을 끝으로 부스트캠프 7기 그룹 프로젝트 `함께 쓰는 공동 문서, Codocs`는 부스트캠프로부터 `NCP` (네이버 클라우드 플랫폼) 크레딧을 제공받아 안정적으로 배포할 수 있었습니다. 
크레딧 만료일이 1월말까지였기 때문에 어쩔 수 없이 얼마 전 배포를 중단하게 되었습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217187984-818ffd70-fc39-416c-9078-cebabcde3710.png">
*잘 있어요. NCP*
<br />

어딘가에 새로 배포하고 싶다는 생각이 들던 찰나, `AWS`에 관한 좋은 정보를 부스트캠프 7기였던 동료 개발자분으로부터 얻을 수 있었습니다. t4g 라는 인스턴스가 있는데 올해 말까지 무료로 제공된다는 것이었습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217189077-aa08c6d1-6d4c-48fe-b54c-cbb6790638c2.png">
*한 달에 750시간이니까 그냥 무료입니다.*
<br />

작년까지만해도 좋은 배포 수단이었던 `HEROKU`가 더 이상 무료가 아니게 되었기 때문에 바로 AWS로 배포해보기로 했습니다. 기록용이자 튜토리얼 성격의 글로 사진 위주로 따라하기 좋게 진행해보겠습니다. 

<hr>

## 2. 인스턴스 생성하기 

AWS에 로그인을 하고 EC2 메뉴로 들어오면 초기 인스턴스가 존재하지 않기 때문에 허허벌판에 `인스턴스 생성` 버튼만 덩그러니 있습니다. 눌러서 본격적으로 인스턴스를 생성합니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217190887-e30b10b9-a28c-4a44-b843-fd3616857551.png">
*빨간 박스 안에 버튼을 누르면*
<br />

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217191526-53964918-d5bb-43a8-9784-f266e7d0ca8a.png">
*다음 화면으로 넘어옵니다.*
<br />

처음에 말했던 t4g 인스턴스를 선택하려고 보니 <u> 이 인스턴스 유형은 선택한 AMI의 아키텍쳐(x86_64)를 지원하지 않습니다. </u> 라는 문구와 함께 비활성화되어 있습니다. 최상단의 서버 OS를 선택하는 부분에서 아키텍쳐가 기본값인 `64비트(x86)`으로 되어 있기 때문입니다. 이를 `64비트(Arm)`으로 바꿔줍니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217193148-69953f15-e34d-422c-8942-2e7ec5b468f0.png">
*t4g가 비활성화 되어 있기 때문에*
<br />

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217193244-95d5ff78-c923-4ce2-98b0-5e5d9c287bc4.png">
*운영체제 아키텍쳐를 바꿔줍니다.*
<br />

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217194012-0fd0364b-e2ac-4586-8129-a4a33d3f448f.png">
*arm 기반 M1 Mac도 잘 사용했으니까 괜찮을 것 같습니다.*
<br />

운영체제는 기본값으로 설정되어 있는 <u> Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type </u> 을 사용했습니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217194641-928737dc-a8bf-4eab-ac89-623697258dab.png">
<br />

다음으로 키 페어를 생성해줍니다. 추후에 SSH를 통해 서버에 접속하기 위해서 필요한 키를 생성하는 것입니다. `새 키 페어 생성` 버튼을 눌러줍니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217195057-a5a51db3-aa63-40ed-8270-82963ce2bb6c.png">
*버튼을 누르면 모달창이 열립니다.*
<br />

아래 사진에서 1번으로 표시된 칸에 키 페어 이름을 입력합니다. `codocs`로 키 이름을 설정했기 때문에 추후 생성되는 pem 파일은 `codocs.pem` 으로 생성됩니다. 입력한 뒤에 2번 버튼을 누릅니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217195644-401c05f1-45a4-40da-a075-989f4d668912.png">
<br />


다음으로 네트워크 설정입니다. 빨간 박스로 표시한 부분에서 HTTPS와 HTTP 트래픽을 허용하면 각각 사이트의 443번 포트와 80번 포트로 접속하는 것을 허용하게 됩니다. 여기서 설정하지 않아도 별도로 포트를 열 때 설정 가능합니다. 이 부분도 밑에서 자세히 다루겠지만 일단 두 개의 체크박스를 모두 선택하겠습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217196615-b99c970e-d864-45e2-9fc0-1c6f4784832f.png">
*체크하고 넘어가면 편합니다.*
<br />

스토리지는 기본값으로 8GB로 되어 있습니다. 안내 문구에 따르면 30GB까지 사용할 수 있다고 하니 넉넉하게 30으로 적어주었습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217198186-2d3fd4c3-4184-4117-bb24-54a60dec468e.png">
<br />

인스턴스 생성 버튼을 누르면 아래 사진과 같이 정상적으로 인스턴스가 생성됩니다. 정말 바로 진행되어서 깜짝 놀랬습니다. 👍

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217198532-9e0ffb7a-6ecc-4278-89f9-c8a9a3f7fd5f.png">
<br />

<hr>

## 3. 추가 설정하기 

인스턴스를 생성했을 때 IP 주소가 부여되었지만, 만일 실행이 잠시 중지되고 다시 실행되었을 때에도 새로운 IP 주소가 부여됩니다. 인스턴스에 새로운 IP 주소가 부여된다면 이후에 현재 IP 주소를 기준으로 진행할 여러 가지 설정들을 다시 해주어야 하는 번거로움이 있기 때문에 고정 IP 주소를 부여해야 합니다. 따라서 탄력적 IP 주소를 할당하여 IP 주소를 고정시켜주도록 합니다. 참고로 탄력적 IP를 만들고 **연결하지 않을 경우 과금이 된다**고 하니 주의해야합니다. 

아래 사진처럼 좌측 메뉴에서 1번을 눌러서 탄력적 IP를 설정하는 페이지로 이동합니다. 이후 2번으로 표시된 `탄력적 IP 주소 할당` 버튼을 누릅니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217212047-9de0f6dc-e5ad-4ea7-ad87-d959bf69b381.png">
<br />

페이지가 이동되면 `할당` 버튼을 눌러서 IP 주소를 생성합니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217212362-1fa9a40e-9389-464e-884a-c103fdcba669.png">
*바로 할당해줍니다.*
<br />

다시 원래 페이지로 돌아오게 되고 탄력적 IP 주소를 할당받았습니다. 
`이 탄력적 IP 주소 연결` 버튼을 눌러줍니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217212700-a0965bc5-aa9d-42db-8149-6a81523ed19a.png">
<br />

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217213503-69843fcf-baac-4128-b654-d9895b0a1996.png">
<br />

위 사진에 표시된 것처럼 1번과 2번을 눌러서 각각 방금 전 생성한 인스턴스와 프라이빗 IP 주소를 선택해줍니다. 이후 `연결` 버튼을 누르면 정상적으로 연결됩니다. 

<hr>

## 4. SSH 접속 

앞서 다운로드 받은 키 페어를 사용할 차례입니다. 서버를 설정하기 위해서 SSH 접속을 해보겠습니다. 아래 사진처럼 `작업`에서 `연결` 버튼을 누릅니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217221104-9fecaef3-1428-4f49-8b5d-982ea3367b85.png">
<br />

처음에 들어가면 `EC2 인스턴스 연결` 탭으로 되어 있습니다. `SSH 클라이언트` 탭으로 이동하면 SSH 접속을 위한 커맨드를 안내해줍니다. 로컬에서 터미널을 열고 키 페어가 저장되어 있는 경로로 이동하여 2번과 3번 명령어를 차례로 입력해줍니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217222445-167ac92d-3189-4bfe-92c0-af109c96a8fd.png">
<br />

SSH 접속이 가능한 터미널에서 차례대로 명령어를 입력하면 아래 사진과 같이 서버에 정상적으로 접속할 수 있습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217222704-98fc202e-c473-4401-9993-9a43e5748dd7.png">
<br />

<hr>

## 5. 포트 설정 (네트워크 접근 제어)

인스턴스를 생성할 때 네트워크 설정에서 HTTPS와 HTTP에 대한 트래픽을 허용해주었습니다. 프로젝트에서 사용하고자 하는 포트에 대한 추가적인 접근 허용이 필요합니다. 프로젝트에서는 3000(프론트엔드), 8000(API 서버), 8100(소켓 서버), 3306(MySQL) 포트를 사용했기 때문에 다음의 과정을 수행합니다. 

먼저 좌측 메뉴에서 `보안 그룹` 으로 이동합니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217223549-b697e9f8-584e-4ef9-8f05-2c633a71f772.png">
<br />

아래 사진의 1번 박스에서 보이는 것처럼 인스턴스에 적용되어 있는 보안 그룹을 선택합니다. 방금 전 생성한 인스턴스에서 사용하는 `launch-wizard-1` 를 선택했습니다. 이후 하단 메뉴에서 `인바운드 규칙`을 선택하고 3번 `인바운드 규칙 편집` 버튼을 누릅니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217224907-f1862bfd-b543-413f-b8b4-28ed7527a10e.png">
<br />

인바운드 규칙 편집 페이지에 들어오면 인스턴스를 생성할 때 체크했던 HTTPS와 HTTP 포트가 설정되어 있습니다. 1번 버튼을 눌러 새로운 규칙을 추가합니다. 2번 위치에 접근을 허용하고 싶은 포트 번호를 입력한 뒤, 3번 위치에 해당 포트로 접근할 수 있는 소스를 입력합니다. IPv4의 경우에는 `0.0.0.0`와 같은 형식으로 입력하고 IPv6는 `::/0`로 입력합니다. 모든 인바운드 규칙을 추가했다면 `규칙 적용` 을 누릅니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217227327-83e81103-f178-4cfd-a067-9dfe4caf276d.png">
<br />

보안 그룹 페이지로 돌아오면 다음과 같이 추가한 포트들이 정상적으로 등록된 것을 확인할 수 있습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217304911-1fd49d9e-0b58-4ada-8114-84695ce2f4a3.png">
<br />

<hr>

## 6. 도메인 연결 

현재 상태에서 웹 사이트에 접속하기 위해서는 퍼블릭 IP를 사용하여 접속하는 방법 밖에 없습니다. 예전에 NCP로 배포했을 때 사용했던 `codocs.site` 라는 주소를 사용하기 위해 EC2에서도 도메인 주소와 연결하기 위한 추가적인 설정을 해주겠습니다. 먼저 검색창에서 `Route 53` 이라는 메뉴를 찾습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217241759-cad6bdd5-5dca-478a-b7ef-5f9227611171.png">
<br />

좌측 메뉴에서 `호스팅 영역`으로 들어가서 2번으로 표시된 `호스팅 영역 생성` 버튼을 누릅니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217242244-73975732-8562-4c1e-bd81-c448ae3a812e.png">
<br />

두 개의 입력칸이 나타납니다. 1번 칸에는 구매했던 도메인 이름인 `codocs.site` 를 입력합니다. 2번 칸은 선택 사항이기 때문에 간단하게 프로젝트 설명을 적어주었습니다. 마지막으로 3번 `호스팅 영역 생성` 버튼을 누릅니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217242743-9eb0dd80-f51b-4f02-a42f-f17bd9effd8d.png">
<br />

정상적으로 호스팅 영역이 생성되면 이전 페이지로 돌아옵니다. 호스팅 영역 세부에서 `레코드 생성` 버튼으로 도메인 주소와 실제 IP 주소를 연결시켜주어야 합니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217245865-36f75fd3-6507-4460-9fe8-8e3ca822ec07.png">
<br />

`값`이라고 되어 있는 칸에 퍼블릭 IP 주소를 입력합니다.

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217246796-b661764f-2f9a-4bd6-876d-6277526dcc46.png">
<br />

즉시 레코드가 생성되며 아래와 같이 레코드에 라우팅 주소가 추가됩니다. 총 4개의 주소가 생성되었습니다. 해당 주소들을 도메인 네임서버에 연결시켜주겠습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217247731-8cf4c773-a8e4-4e3f-96e6-39ec2a8e38a2.png">
<br />

도메인을 구매했던 가비아 사이트로 이동하여 My가비아 -> 도메인 -> 관리로 이동하였습니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217248268-95cc9529-04fb-4631-83d4-0a3c22f9a5ad.png">
*도메인을 누르고*
<br />


<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217250999-82310b8d-8200-462d-8779-f347f7f32e72.png">
*사용하려는 도메인에 대한 관리 페이지로 들어갑니다.*
<br />

기존에 NCP로 연결했을 때의 주소가 남아있습니다. 수정하기 위해서 네임서버 설정을 누릅니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217248691-9c2907ad-6828-44cc-9464-52a778949e7a.png">
<br />

4개의 라우팅 주소를 차례대로 네임서버에 추가했습니다. 가비아에서 소유자 인증을 진행한 뒤에 적용 버튼을 누르면 모든 설정이 끝납니다. 

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217249488-bb518674-3a3e-47a2-a756-49fda80fe17a.png">
<br />

<hr>

## 7. 패키지 설치 

SSH 를 사용하여 서버에 접속한 뒤 프로젝트에 필요한 패키지를 다운로드 합니다. 이후 내용은 각자 진행하는 내용이 다를 수 있습니다. 따라서 개인적으로 프로젝트에 필요한 패키지를 설치한 과정과 문제 해결 링크 등을 담았습니다. 

<br />

### 1) NVM + Node.js

먼저 Node Package Manager를 사용하기 위해 NVM과 Node.js 를 설치했습니다. Node.js를 설치하는 과정은 AWS 공식 자습서에서 다루고 있습니다. 주의 사항에 나와 있는 것처럼 Amazon Linux2는 Node.js 18 버전을 지원하지 않기 때문에 16 버전을 사용했습니다.

- [자습서: Amazon EC2 인스턴스에서 Node.js 설정](https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

<br />

### 2) Redis 

프로젝트의 백엔드 부분에서 캐시 데이터를 저장하기 위한 Redis도 설치해주었습니다. 처음 설치해보았는데 꽤 단계가 많고 복잡했습니다. 개인적으로 백그라운드에서 실행하는 부분이 잘 되지 않아서 2번 정도 시도했습니다. 😅

- [[AWS] EC2에 Redis 설치](https://velog.io/@ssoop/AWS-EC2%EC%97%90-Redis-%EC%84%A4%EC%B9%98)

<br />

### 3) MySQL 

프로젝트에서 사용하고 있는 RDBMS인 MySQL도 설치해주었습니다. 백엔드 env 파일에서 사용자 비밀번호를 매우 간단하게 설정해놓았었는데, MySQL 비밀번호 정책 때문에 시간을 꽤 쏟았습니다. 정책 자체를 간단하게 (보안 레벨도 낮게, 길이도 짧게) 변경한 뒤에 적용할 수 있었습니다. 

- [[AWS] 10-1.EC2 MySQL 설치](https://goddaehee.tistory.com/292)
- [MySQL - password 정책 낮춰서 단순한 패스워드 사용하기](https://junho85.pe.kr/1484)

<br />

### 4) Git

마지막으로 Git을 다운로드 받은 뒤에 프로젝트 GitHub에서 저장소를 clone 받았습니다. 리눅스에서 사용할 수 있는 `nohup` 명령으로 npm 명령어들을 백그라운드에서 실행했습니다.

<br />

<hr>

## 8. 맺으며

<br />
<img alt="image" src="https://user-images.githubusercontent.com/31645195/217249608-802c63df-ad20-495a-959d-d5cc66f40ebc.png">
*codocs가 부활했습니다.*
<br />

위와 같이 정상적으로 사이트에 접속할 수 있게 되었습니다. 아직 부족한 부분이라고 한다면, NGINX 설정을 적용하지 않아 3000번으로 접속해야 한다는 점입니다. NGINX (을 학습하고) 설정까지 하면 원래 목적이였던 리팩토링과 꽤 거리가 멀어질 것 같아서 일단 이쯤 마치고 추후 수정하는 방향으로 계획을 세웠습니다. 추가적으로 HTTPS 설정도 필요하겠네요.

한편, 기존에 글을 작성했던 방식과 다르게 튜토리얼 형식의 글을 작성하는 것이 꽤 어색했습니다. 튜토리얼이니까 따라하면 손쉽게 목적을 이룰 수 있게 하는 것에 주안점을 두었지만, 중간중간 깊은 설명이 빠져있는 탓에 그런 것 같습니다. AWS를 자주 사용해보면서 각각의 기능들에 대해서 자세하게 설명할 수 있도록 발전하면 좋을 것 같습니다.

---

[참고]

- [[AWS]EC2 - 인스턴스 생성하기 (1/2)](https://codemonkyu.tistory.com/entry/AWSEC2-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4-%EC%83%9D%EC%84%B1%ED%95%98%EA%B8%B0-12)

- [[AWS] EC2 알아보기 + 인스턴스 생성하기](https://velog.io/@kyj311/AWS-EC2-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

- [자습서: Amazon EC2 인스턴스에서 Node.js 설정](https://docs.aws.amazon.com/ko_kr/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html)

- [[AWS] EC2 와 도메인 연결하기 (feat. 가비아)](https://developer-ping9.tistory.com/320)

- [[AWS] EC2에 Redis 설치](https://velog.io/@ssoop/AWS-EC2%EC%97%90-Redis-%EC%84%A4%EC%B9%98)

- [[AWS] Amazon Linux2 Redis 설치](https://small-stap.tistory.com/109?category=989595)

- [CentOS 7 MYSQL 5.7 설치 GPG keys 오류 해결 방법](https://change-words.tistory.com/entry/CentOS-MYSQL-GPGkeys)

- [[AWS] 10-1.EC2 MySQL 설치](https://goddaehee.tistory.com/292)

- [MySQL 8.0 패스워드 변경 방법](https://toytvstory.tistory.com/1617)

- [MySQL - password 정책 낮춰서 단순한 패스워드 사용하기. ERROR 1819 (HY000): Your password does not satisfy the current policy requirements.](https://junho85.pe.kr/1484)

- [nohup 으로 로그 남기면서 종료되지 않는 프로세스 실행](https://bcp0109.tistory.com/353)
