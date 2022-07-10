---
title: "[Ionic] DevOps Solution - AppFlow"
author: steadev
date: 2021-07-04 15:51:00 +0900
categories: [Ionic]
tags: [Ionic, Appflow]
---


React Native에 Code Push가 있다면, Ionic에는 AppFlow가 있습니다!!

만약 cordova를 사용중이라면 Code Push를 활용하면 되지만,

capacitor라면 AppFlow를 활용하면 됩니다!

AppFlow 유료 버전을 사용하면 Native 빌드와 배포까지 할 수 있지만, 

이 글에서는 무료로 활용할 수 있는 유용한 Live Update에 대해서 설명하고자 합니다!

Live Update란, 스토어에 배포하지 않고도 실시간으로 앱을 업데이트 시키는 것입니다.

우선 방법을 순서대로 정리하면,

1.  **[https://ionic.io/appflow](https://ionic.io/appflow) 계정 생성**
2.  **앱 생성**
3.  **Git, GitLab과 같은 저장소 연결**
4.  **AppFlow관련 Plugin 설치 및 코드 작성**
5.  **웹 빌드 및 배포**
6.  **앱 빌드**
7.  **버전 관리**
8.  **참고사항**

우선, 1 ~ 3번은 간단하기도 하고, 하라는대로만 하면 문제 없이 하실 수 있을 것입니다!

설명에 들어가기 앞서, Live Update는 **Binary Compatible Changes**만 가능합니다.

여기서 Binary Compatible Changes란 무엇인가??

간단히 말하자면, Native를 제외한 웹 앱 부분의 변경을 의미하며, 아래 부분의 변경을 뜻합니다.

-   JavaScript / TypeScript logic
-   SASS / CSS styling
-   HTML layout
-   Assets (Images, Icons, static JSON files, etc)

즉, 어찌보면 당연하지만 Native 코드까지는 Live Update는 불가능하다는 것입니다.

#### **4\. AppFlow관련 Plugin 설치 및 코드 작성**

Destinations > Install Instructions를 클릭하면 Setup 명령어가 나와있습니다.   
(만약 없다면, 5번 웹 빌드부터 하시면 보일 것입니다..!)

세가지 옵션이 있는데 그 중 원하는 것을 선택하시면 됩니다.

그대로 입력하면 cordova-plugin-ionic 플러그인이 설치됩니다!

<img src="https://steadev.github.io/assets/images/ionic/2021-07-04-1.png" />
<img src="https://steadev.github.io/assets/images/ionic/2021-07-04-2.png" />

update-method 옵션 동작 방식은 다음과 같습니다.

**\[background\]**

\- 앱 실행시 Background에서 앱 동작과 상관없이 업데이트 된 코드를 다운로드 받습니다.

\- **앱 종료 후** 다시 실행시 다운로드 된 버전으로 앱이 실행됩니다.

\- Appflow에서 권장하는 옵션입니다.

**\[auto\]**

\- splashscreen 상태에서 업데이트 된 코드를 다운로드 받아서 적용시킵니다.

\- 때문에, 업데이트 사항이 많을 경우 초기 로딩시간이 많이 길어질 수 있습니다.

**\[none\] - **현재 프로젝트에 사용중****

\- programmatically 커스터마이징 가능.

\- 위 두가지 옵션은 앱이 새로 켜질 때만 업데이트가 되는데, 해당 옵션은 원하는대로 시점을 커스터마이징 할 수 있습니다.

\- 현재 제가 개발중인 프로젝트에서는 앱을 background로 내렸다가 올렸을 때(pause - resume)에도 앱을 업데이트하도록 했습니다.

```javascript
// 새로 업데이트 된 내용 있는지 체크 
const update = await Deploy.checkForUpdate(); 
if (update.available) { 
    // We have an update! 
    // download update 
    await Deploy.downloadUpdate((progress) => { 
    	// download progress(percent)
    }); 
    // 다운로드 내용 적용 
    await Deploy.extractUpdate((progress) => { });
    // 다운로드 내용 적용 후, 앱 새로고침 (원하는 위치에서 가능)
    await Deploy.reloadApp();
 }
```

**\* background와 auto 옵션은 따로 코드 작성이 필요하지 않습니다.**

#### **5\. 웹 빌드 및 배포**

우선 Commits에 커밋 목록이 보이고, 빌드를 원하는 커밋 선택시 아래처럼 보이게 될 것입니다.

<img src="https://steadev.github.io/assets/images/ionic/2021-07-04-3.png" />

여기서 Channel은 빌드 카테고리라고 보시면 될 것 같습니다.

한 예로, 아래 코드를 통해 채널을 바꾸고 해당 채널의 최신 코드로 업데이트 할 수 있습니다.

```javascript
// set channel
await Deploy.configure({channel: 'Production'});
// download, extract, reload
...
```

Live update 오른쪽의 toggle바를 **enabled**로 바꾸면 빌드가 끝남과 동시에 배포가 됩니다. 

빌드만 해놓고 배포는 나중에 하고 싶다면 **disabled**로 하면 됩니다!

#### **6\. 앱 빌드(로컬 빌드)**

반드시 아래 순서대로 빌드 해야 합니다.

따로 package.json에 아래 빌드를 모아 두는 것을 추천드립니다.

```
ionic build --prod // 웹 빌드
ionic deploy manifest // pro-manifest.json 파일 생성
npx cap sync // 앱 빌드
```

위 순서대로 해야하는 이유를 설명드리기 전에 manifest파일에 대해 먼저 알아야 합니다.

Appflow에서 빌드한 내용과 로컬에서 빌드한 내용를 비교할 수 있게 하는 것이 바로 manifest.json 파일입니다.

manifest.json 파일이 있어야 **기존 빌드 버전과 다른 부분만 파악해서 다운로드** 할 수 있게 됩니다.

만약 없다면, asset파일까지 죄다 매번 가져오기 때문에 시간이 엄청 오래걸립니다..   
(저희 프로젝트에서는 3분 조금 넘게 걸렸네요..)

그렇다면 위 순서대로 해야하는 이유는?!

미리 manifest 생성 후, 단순히 **ionic cap sync 한다면,**

**ionic build시 www폴더에 manifest파일이 사라지게 되고,**

**capacitor sync하면서 각 native 부분에도 manifest파일이 사라지게 됩니다.**

따라서 **ionic build 후 manifest파일 생성해주고 마지막으로 sync 맞춰주는 방식으로 빌드해야 manifest파일이 남아있게 됩니다**!!

#### **7\. 버전 관리**

예시 상황을 먼저 한 번 보겠습니다.

1.0.0 -> 2.0.0 으로 앱 업데이트하면서, 강제 업데이트를 원합니다.

이럴 경우 어떤 일이 발생할까요??!

**스토어에서 2.0.0 버전을 다운로드 받고서도 1.0.0 버전의 앱이 보이게 됩니다..!** 

왜 이렇게 되냐?!

1.0.0 버전의 웹 빌드를 로컬에 가지고 있습니다. 그래서 앱 실행시 1.0.0 버전을 먼저 보여주고, 

어? 최신 버전이 있네? 다운 받아야지!

이런 상황이 펼쳐지게 됩니다..! 

**새로운 앱을 받고 나서도 그 앱의 빌드 내용을 무시하고 로컬 캐싱된 1.0.0 빌드를 가져오는 것입니다.**

이것 때문에 유료로 Appflow를 써야 해결되는 문제인지.. 딱히 정보도 잘 없는 것 같고 엄청 삽질을 했는데요..

아래 사진처럼 Versioning으로 이 문제를 해결할 수 있습니다.

 위 예시를 해결하자면, 1.0.0 버전의 웹 빌드의 max를 1.1.0으로 했다쳤을 때

2.0.0 버전을 새로 받게 되면 

**어? 지금 로컬에 있는 버전은 max가 1.1.0이네? 무시해야지!**

그 결과 2.0.0 버전의 앱이 보이게 됩니다..!

참고로 Equivalent는 앱이 해당 버전일 경우, Live Update를 안합니다.  
자세한 것(?)은 아래 캡처를 잘 보시면 될 것 같습니다..!

<img src="https://steadev.github.io/assets/images/ionic/2021-07-04-4.png" />

**8\. 참고사항**

로컬 빌드 테스트시,

웹에서 빌드한 버전이 로컬에 남아있어서

아무리 새롭게 빌드한다고 해도 이전 버전의 앱이 보이게 됩니다.

앱 버전을 올려놓고 테스트 하는 방법도 있겠지만,

capacitor.config.json파일에 아래 코드를 추가해도 됩니다.

```json
"cordova": {
  "preferences": {
    "DisableDeploy": "true"
  }
}
```

**하지만 주의해야할 점!!**

이를 까먹고 그대로 빌드하게 되면.. 라이브 업데이트가 불가능한 대참사가 발생하게되니

배포시 반드시 지워줘야 합니다..!

**9\. 한계점**

너무 비쌉니다..

무료는 Live Updates가 10000건/mo

$499/mo 정책의 경우 25000/mo 밖에 안됩니다.

그런데 microsoft AppCenter의 codepush는 무료입니다..!

만약 native 배포까지 처리하려면 app flow를 사용해야겠지만

아니라면 AppCenter를 활용하는 것이 좋아보입니다.

다음에는 AppCenter를 통해 Ionic앱 codepush하는 방법에 대해 포스팅 하겠습니다.

이상 Ionic의 Live Update에 대한 설명이었습니다!

혹시 질문이나 부족한 부분이 있다면 댓글 부탁드립니다 :)

참고)

[https://ionic.zendesk.com/hc/en-us/articles/360003567694-How-to-restrict-Live-Update-by-native-version](https://ionic.zendesk.com/hc/en-us/articles/360003567694-How-to-restrict-Live-Update-by-native-version)

[https://ionic.zendesk.com/hc/en-us/articles/360002243614-What-Are-Binary-Compatible-Changes-](https://ionic.zendesk.com/hc/en-us/articles/360002243614-What-Are-Binary-Compatible-Changes-)