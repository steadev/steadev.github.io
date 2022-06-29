---
title: "[Angular] 상태관리 with Mobx"
author: steadev
date: 2020-07-12 17:55:00 +0900
categories: [Angular]
tags: [angular, mobx]
---


Angular 개발을 하면서, 상태관리를 어떻게 하는게 제일 코드 양도 줄이고 효율적일까 고민을 많이 했습니다. 

기존에는 service와 rxjs를 활용해서 관리를 했고, Angular하면 대표적인 ngrx를 사용해보았습니다. 

하지만 boiler plate가 너무 많아지고 service + rxjs와 비교했을 때, 큰 장점을 느끼지 못했습니다. 

그러던 중에, boiler plate가 싹 빠진 redux 라고 불리는(?) mobx를 접하게 되었습니다..!

이하는 mobx 적용 후기입니다.

정리

\- mobx npm 버전 관리 문제 

\- service와 차이점이 없습니다. 그저 store를 따로 둬서 ajax 통신을 service에서, action은 store에서 할뿐

  @decoration만 붙어있는 service랑 똑같습니다. 그렇게 따지면 오히려 bolier plate가 늘어나서 굳이 angular에서는

  쓸 필요가 없지 않을까 싶습니다.