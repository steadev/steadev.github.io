---
title: "Chrome DevTools 활용법 (개발자 도구)"
author: steadev
date: 2020-12-20 19:37:00 +0900
categories: [Chrome]
tags: [Chrome, developer tools]
---


많이들 기본적으로 사용하는 기능에 대해서는 언급하지 않고 알아두면 유용한 부분들 위주로 추려봤습니다.

## **\[Elements\]**

#### **Node에 쉽게 접근 (without querySelector)**

$0~ $4 까지 최근 클릭한 5개의 Node가 저장됩니다. 가장 최근이 $0, 가장 마지막이 $4 입니다.

아래 사진을 보면, <input>이 클릭된 상태에서 $0을 했더니 input element가 잡히는 것을 확인할 수 있습니다.

<img src="https://steadev.github.io/assets/images/chrome/2020-12-20-1.png" />

#### **Element Breakpoint**

sources탭에서 javascript breakpoint를 할 수 있지만, dom의 변화에 breakpoint를 설정할 수도 있습니다. 

아래 그림에서 보면, (하위 Nodes 변경 / 현재 Node 변경 / Node 삭제) 케이스에 대해 breakpoint를 지정할 수 있습니다.

이 외에도 force state 탭에서 element의 상태를 focus, hover 등으로 강제시킬 수 있습니다.

<img src="https://steadev.github.io/assets/images/chrome/2020-12-20-2.png" />

## **\[Consoles\]**

#### **Live Expression**

매번 console을 찍어가며 확인하던 부분을 저 눈처럼 생긴 버튼을 클릭하여 expression을 적어 실시간으로 확인해볼 수 있습니다.

해당 부분의 값이 무엇인지, element의 사이즈나 스타일 등 여러가지를 굳이 콘솔을 찍어보지 않더라도 이를 통해 간단히 확인해 볼 수 있습니다.

Live Expression 버튼을 여러번 눌러 expression을 여러개 추가할 수 있습니다.

<img src="https://steadev.github.io/assets/images/chrome/2020-12-20-3.png" />

## **\[Sources\]**

#### **Snippets**

간단한 script를 작성해 놓고 활용할 수 있습니다. 

아래의 예시를 보면, add 함수 snippet을 작성해 두었고, 아래 콘솔에서 정의한 함수를 사용해서 결과를 내는 것을 보실 수 있습니다. 

주의하실 점은 중간 빨간 박스의 시작 버튼을 눌러야 한다는 점입니다. 

자주 쓰이는 테스트용 함수들을 정의해 놓으면 유용할 것입니다 ㅎㅎ 

<img src="https://steadev.github.io/assets/images/chrome/2020-12-20-4.png" />