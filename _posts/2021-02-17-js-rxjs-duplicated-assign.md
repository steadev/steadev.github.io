---
title: "Rxjs duplicated assign issue"
author: steadev
date: 2021-02-17 18:51:00 +0900
categories: [Javascript]
tags: [javascript, rxjs]
---


Observable을 구독하는 Subscription 변수에 한번 더 assign 한다면 어떤 일이 발생할까요?  
  
아래 코드를 보고 상황을 살펴보겠습니다.  
  

```javascript
class RxjsAssignment {
  constructor() {
      this.subscribeWindowScroll();
      this.subscribeWindowScroll();

      this.unsubscribeWindowScroll();
  }

  private windowScrollSubscription: Subscription;

  subscribeWindowScroll(): void {
      this.windowScrollSubscription = fromEvent(window, 'scroll').subscribe((event) => {
        this.onScroll(event);
      }); 
  }

  unsubscribeWindowScroll(): void {
        if (this.windowScrollSubscription !== undefined) {
            this.windowScrollSubscription.unsubscribe(); 
       }
  }

  onScroll(): void {
        console.log('onScroll');
  }
}
```

위 코드를 수행 후 스크롤을 하면 onScroll 함수의 콘솔이 찍힐까요?  
  
언뜻보면 Subscription 변수에 새로 assign 했고, unsubscribe도 해줬기에 아무 문제 없어보입니다.  
  
  
하지만..! **여전히 콘솔이 계속 찍히게 됩니다.** 사실 제가 별 생각없이 비슷한 로직으로 개발했다가 삽질을...흑..  
  
  
여튼 fromEvent Observable을 첫번째는 1번, 두번째는 2번으로 부르겠습니다.  
  
  
1번을 assign한 후, 2번을 assign하게 되면 1번은 이미 공간이 할당되었으므로 unsubscribe되지 않고 추적이 불가능한채로 남아있게 되고,  
  
  
2번을 아무리 unsubscribe해줘도 1번은 계속 메모리를 차지하게 됩니다.  
  
  
쉽게(?) 말하면, windowScrollSubscription 변수는 1번의 공간을 가리켰다가 2번을 가리키는 것일 뿐이지 1번을 없애고 2번을 가지는 것이 아닌 것입니다.  
  
  
따라서.. 저렇게 두번 assign하는 일이 일어나지 않게 조심해야하는 것은 물론이고, 혹시나 어쩔 수 없이 두번 하게 된다면..! 항상 unsubscribe를 해주고 assign 하자..  
  

```javascript
...
subscribeWindowScroll(): void {
    this.unsubscribeWindowScroll();
    this.windowScrollSubscription = fromEvent(window, 'scroll').subscribe((event) => {
      this.onScroll(event);
    }); 
}
...
```