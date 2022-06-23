---
title: "script 속성 async / defer"
author: steadev
date: 2020-06-30 21:29:00 +0900
categories: [JAVASCRIPT]
tags: [javascript, html, async, defer]
---


javascript를 불러올때, DOM이 구성되는 것을 기다리기 위해 

<body> 태그 제일 아래에서 script를 가져오기도 합니다.

하지만 async, defer 속성을 줘서 간편하게 이를 해결할 수 있습니다. 

만약 DOM과 크게 상관이 없다거나 다른 script와의 연관성이 없다면,

그리고 js파일이 작아서 DOM이 화면에 보이는 시간이 얼마 안걸린다면 굳이 저 두 속성을 넣을 필요는 없어보입니다. 

이게 무슨말인지는 아래 수행 순서를 먼저 보고 오도록 하겠습니다.

  방향  -----> 

#### **\[head안 속성X\]**

<img src="https://steadev.github.io/assets/images/js/2020-06-30-1.png">

#### **\[body맨 아래 속성X\]**

<img src="https://steadev.github.io/assets/images/js/2020-06-30-2.png">

#### **\[async\]**

<img src="https://steadev.github.io/assets/images/js/2020-06-30-3.png">

#### **\[defer\]**

<img src="https://steadev.github.io/assets/images/js/2020-06-30-4.png">

정리하자면, 

1\. 속성 없을때 script를 만나면 불러오고 실행까지 시킨 후에 나머지 작업을 합니다. 위에 보이는 것처럼 head 안에 있다면 HTML 파싱이 끝나기 전에 실행해버리고, 만약 script 파일이 크다면 그만큼 로딩이 오래걸립니다.(요즘은 워낙 사양이 좋아서 크게 상관은 없어보입니다만 미세한 차이라도...)

2\. async는 HTML파싱과 병렬적으로 동작하나, script를 다 가져왔다면 HTML 파싱을 멈추고 즉시 수행합니다.

즉시 수행하는 만큼, 만약 script가 여러 개이고, feching 속도가 다르다면, 수행 순서가 뒤죽박죽이 될 수 있습니다. 그렇기에 이를 고려해서 사용해야 합니다.

3\. defer는 async처럼 병렬로 동작하나, 파싱 후에 작업을 수행합니다. 파싱이 다 끝난뒤 실행되기에, 수행 순서가 중요하다면 특히나 defer를 활용해야겠습니다. 그리고 module 타입의 경우는 기본적으로 지연 로딩이 적용되서 굳이 defer 속성은 넣을 필요가 없습니다.

script의 DOM과의 연관성, 타 script와의 종속성, 화면 Load 시간

등 여러 요소를 고려해서 적절히 옵션을 써주면 되지만

기본적으로 head에서 defer 옵션을 쓰면 되지 않을까 싶습니다.