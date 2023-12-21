---
title: "[칭찬일기 개발기 #2] 폴더 구조"
author: steadev
date: 2023-11-26 12:41:00 +0900
categories: [React Native]
tags: [React Native, Expo]
---

### expo cli로 프로젝트 생성시 최초 Directory

<img src="https://steadev.github.io/assets/images/compliment-diary/2023-11-26-1.png" />

내가 주로 작업할 코드 관련 파일들만 따로 src폴더로 묶었다.

App.tsx파일도 src안으로 옮겼는데 그러면 오류가 발생한다.

package.json에 entry 파일 경로가 node_modules/expo로 정의되어있기때문..!
`"main": "node_modules/expo/AppEntry.js"`

그래서 해당 js파일을 수정했다.

```

```

// AppEntry.js
import registerRootComponent from 'expo/build/launch/registerRootComponent';

// as-is
// import App from '../../App';

// to-be
import App from '../../src/App';

registerRootComponent(App);

```

마무리로 `patch-package`로 변경사항 패치파일 생성!

node_modules의 파일을 수정한 것이기에

이렇게 해야 나중에 node_module 다시 설치할 때의 시행착오를 없앨 수 있다!

`npx patch-package expo`

```

// expo patch file

diff --git a/node_modules/expo/AppEntry.js b/node_modules/expo/AppEntry.js
index 9e1b283..5f66b1d 100644
--- a/node_modules/expo/AppEntry.js
+++ b/node_modules/expo/AppEntry.js
@@ -1,5 +1,5 @@
import registerRootComponent from 'expo/build/launch/registerRootComponent';

-import App from '../../App';
+import App from '../../src/App';

registerRootComponent(App);

```


### 최종 폴더 구조

<img src="https://steadev.github.io/assets/images/compliment-diary/2023-11-26-2.png" />

개발을 처음 할 때는 폴더 구조를 어떻게 가져가는게 베스트일지 많이 찾아보고 고민도 해봤다.
하지만 사람마다의 의견도 다 다르고 '이게 정석이다' 하는 것도 없고
결국 프로젝트 성격이나 개인 취향에 따라 천차만별인듯하다.

그리고 어짜피 개인 프로젝트이기에 이런게 좋다 저런게 좋다 따라가기보다는 취향대로 하고자 한다.

##### components, hooks

전역에서 사용될 공용 components, hooks
##### interfaces

공용으로 사용될 타입들을 정의.

공용이 아닌 것들은 각 service에서 따로 정의해 처리할 예정
##### navigations

routing 정의.

따로 산발되어 있는 구조보다는 한곳에서 관리하는게 개인적으로 편했음..!
##### screens

Angular 사용 당시, 폴더 구조를 최대한 flat하게 가져가야 가독성이 좋다는 공식문서를 보고

처음엔 최대한 그렇게 해보려 했으나 개인적으로는 프로젝트가 커지면 커질수록 가독성이 오히려 많이 떨어지고

컴포넌트명도 안겹치게 하기 위해 폴더명이 엄청 길어지는 등 취향에 맞지 않았다..

그래서 screen폴더에 이제 이것저것 많아질 텐데, flat하기보다는 depth를 두고자 한다.

```

ex)

- screens
  - Home
    - components
    - hooks

```

##### services

MVVM의 model 담당.
##### store

redux 관련

아직 redux를 제대로 써본적이 없기에 내부 구조는 시행착오를 겪지 않을까 싶다.

##### styles

color 등의 전역 스타일 상수

##### utils

유틸성 함수 관련.
```
