---
title: "[Angular] Unsubscribe http request"
author: steadev
date: 2020-09-26 22:30:00 +0900
categories: [Angular]
tags: [angular, unsubscribe]
---


Observable을 unsubscribe 하는 것은 메모리 관리에 있어서 매우 중요한 이슈입니다.

그래서 저는 보통 rxjs 함수들로 자동으로 구독 해제하거나 (ex. take, takeUntil ... )

async pipe를 사용해서 필요한 경우가 아니라면 따로 unsubscribe를 해주지 않습니다.

하지만 이외에도 굳이 unsubscribe 해주지 않아도 되는 것이 있었는데..!

단발성 데이터에 대해서는 필요가 없는 것입니다.

http request의 경우에도 서버에서 한번 받아오면 끝나는 작업이기에 바로 complete가 호출되버립니다.

코드로 한번 비교해보도록 하겠습니다.

```javascript
const test = this.testService.getData().subscribe(
    data => console.log('data', data),
    err => console.log('error', err),
    () => {
          console.log('Complete!')
          console.log(test);
          setTimeout(()=>{
            console.log(test);
          })
      }      
)
```

\>> 결과

<img src="https://steadev.github.io/assets/images/angular/2020-09-26-1.png">

간단한 데이터를 angular의 **HttpClient**를 통해서 요청했고 그 결과 입니다.

결과를 받아오자마자 complete가 호출되는 것을 볼 수 있고

complete 호출되자마자 unsubscribe되는 것은 아니지만 call stack이 비워지고 setTimeout안의 콘솔이 실행되면

위 캡쳐처럼 **close: true, subscriptions: null** 인것을 확인할 수 있습니다.

이 뿐만 아니라 **of({ a: 1, b: 2}).subscribe()** 이런식으로 단순한 데이터를 Observable로 만들어 subscribe 했을 때도 마찬가지로 바로 complete로 가는 것을 확인할 수 있습니다.

반대로, 당연하겠지만 아래의 subscribe에서는 바로 complete가 호출되지 않습니다.

이 경우는 예시처럼 take를 사용한다던지, 직접 해제해줘야겠죠?

```javascript
interval(1000)
  .pipe(take(10))
  .subscribe(
     data => console.log('data', data),
     err => console.log('error', err),
     () => console.log('Complete!')
  )
```

<img src="https://steadev.github.io/assets/images/angular/2020-09-26-2.png">

**결론...**

**http response에 대해서는 굳이 unsubscribe를 해줄 필요는 없다! 하지만 해준다고 나쁠것도 없다..!**