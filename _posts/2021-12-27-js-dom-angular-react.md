---
title: "React와 Angular의 DOM"
author: steadev
date: 2021-12-27 11:54:00 +0900
categories: [Javascript]
tags: [Javascript, DOM, Angular, React]
---


### **DOM이란?**

-   Document Object Model
-   HTML, XML 문서의 프로그래밍 interface(API)
-   document
-   BOM중의 하나

```javascript
document.querySelector('.woot');
```

-   DOM 구성 - Render Tree

<img src="https://steadev.github.io/assets/images/js/2021-12-27-1.png" />
<img src="https://steadev.github.io/assets/images/js/2021-12-27-2.png" />

### **그렇다면, SPA에서 상태가 변할 때 DOM은 어떤 방식으로 변경될까?**

\[ virtual DOM vs incremental DOM \]

<img src="https://steadev.github.io/assets/images/js/2021-12-27-3.png" />
<img src="https://steadev.github.io/assets/images/js/2021-12-27-4.png" />

### **virtual DOM**

-   React, Vue에서는 DOM의 html 요소가 바뀔 때, 새로운 virtual DOM을 만들어 놓고 기존의 virtual DOM과 비교해서 (diffing) 필요한 부분만 바꿔준다.
-   컴포넌트의 렌더링 결과를 값으로 받을 수 있고, 이를 테스트나 디버깅 등에 사용할 수 있다.
-   virtual DOM을 사용하기에, 메모리 효율성이 떨어지고 개발자의 컴포넌트 구성에 따라 성능의 차이가 크다.  
    → 구성에 따라 render를 더 많이해서 virtual DOM을 더 많이 생성하게 되면..!  
    \-> react의 PureComponent, memo 사용으로 비용을 최소화 시켜야..!
-   Svelte에서는 이 가상돔을 만들어서 비교하는 과정을 없애서 빠르다. compile하기 때문.  
    React, Vue는 브라우저단에서 바로바로 처리하기 때문에 불가능.

### **incremental DOM**

-   모든 컴포넌트는 일련의 명령으로 컴파일 된다. 이 명령들은 데이터가 변경될 때 그 자리에서 DOM 트리를 만들고 업데이트 한다.
-   Incremental DOM을 사용할 때, 프레임워크는 컴포넌트를 해석하지 않는다.  
    대신, 컴포넌트는 명령들을 참조한다. 사용되지 않는 명령들은 컴파일러단에서 생략이 가능! (so called - Tree Shaking)
-   virtual DOM은 인터프리터가 필요하고, 이는 실시간으로 동작하기 때문에 뭐가 필요한지 아닌지 알 수 없다. 때문에 모든 것을 브라우저에 보내야 한다.
-   incremental DOM은 가상 DOM이 필요 없기에 메모리를 많이 절약할 수 있다. DOM node가 추가되거나 삭제될 때만 메모리를 할당한다.

<img src="https://steadev.github.io/assets/images/js/2021-12-27-5.png" />
<img src="https://steadev.github.io/assets/images/js/2021-12-27-6.png" />
<img src="https://steadev.github.io/assets/images/js/2021-12-27-7.png" />

### **그래서 뭐가 더 좋은거지?**

```
💡 Incremental DOM은 메모리의 효율성에서 훨씬 뛰어나지만, 속도면에서는 Virtual DOM 방식이 더 빠르다.
```

앱의 성격에 맞는 것을 선택하면 된다.

<img src="https://steadev.github.io/assets/images/js/2021-12-27-8.png" />

### **구글팀에서 Incremental DOM을 선택한 이유?**

```
💡 모바일 기기에서의 메모리 최적화를 위해 선택
```

→ 어플리케이션은 반드시 모바일 기기에서 문제 없이 작동해야 한다. 그리고 이는 어플리케이션 번들의 용량(Tree shaking)과 메모리 점유율에 대한 최적화를 의미한다.

### **결론..**

리액트가 더 빠르다 메모리가 어떻다 비교하는 것은 많지만,  
개인적으로는 사실 체감 속도는 거의 차이 나지 않는 것 같다. 

메모리도 요즘 기기 성능이 워낙 좋아야지...  
결국 진짜 속도나 메모리에 민감한 서비스가 아닌 이상 취향에 맞는 것을 사용하면 될 듯 싶다

참고)

-   [https://m.blog.naver.com/magnking/220972680805](https://m.blog.naver.com/magnking/220972680805)
-   [https://www.youtube.com/watch?v=1ojA5mLWts8](https://www.youtube.com/watch?v=1ojA5mLWts8)
-   [https://css-tricks.com/an-introduction-and-guide-to-the-css-object-model-cssom/](https://css-tricks.com/an-introduction-and-guide-to-the-css-object-model-cssom/)
-   [https://na27.tistory.com/228](https://na27.tistory.com/228)
-   [https://blog.ninja-squad.com/2019/05/07/what-is-angular-ivy/](https://blog.ninja-squad.com/2019/05/07/what-is-angular-ivy/)
-   [https://angular.kr/guide/ivy](https://angular.kr/guide/ivy)
-   [https://medium.com/@yeon22/angular-incremental-dom-56e1ee9fa3d8](https://medium.com/@yeon22/angular-incremental-dom-56e1ee9fa3d8)
-   [https://blog.nrwl.io/understanding-angular-ivy-incremental-dom-and-virtual-dom-243be844bf36](https://blog.nrwl.io/understanding-angular-ivy-incremental-dom-and-virtual-dom-243be844bf36)
-   [https://blog.bitsrc.io/incremental-vs-virtual-dom-eb7157e43dca](https://blog.bitsrc.io/incremental-vs-virtual-dom-eb7157e43dca) (incremental vs virtual DOM)
-   [https://heropy.blog/2019/09/29/svelte/](https://heropy.blog/2019/09/29/svelte/) (svelte)
-   [https://browserperson.medium.com/from-view-engine-to-ivy-rendering-in-angular-a81d9eb8199b](https://browserperson.medium.com/from-view-engine-to-ivy-rendering-in-angular-a81d9eb8199b) (view engine vs ivy)
-   [https://auth0.com/blog/face-off-virtual-dom-vs-incremental-dom-vs-glimmer/](https://auth0.com/blog/face-off-virtual-dom-vs-incremental-dom-vs-glimmer/) (compare speed)BOM이란?