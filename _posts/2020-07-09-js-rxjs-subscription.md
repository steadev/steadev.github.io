---
title: "[Javascript] RxJS - Subscription"
author: steadev
date: 2020-07-09 15:12:00 +0900
categories: [Javascript]
tags: [javascript, rxjs, subscription]
---


[https://rxjs.dev/guide/subscription](https://rxjs.dev/guide/subscription) 번역

오역 부분이나 조언(?)은 알려주시면 감사하겠습니다.

# Subscription

**구독이란?** 구독은 일회용 자원, 일반적으로 Observable의 실행을 나타내는 객체입니다. 구독은 unsubscribe라는 중요한 메소드를 가지고 있습니다. 이는 인자를 갖지 않고, 구독이 잡고있던 자원을 처리합니다. RxJS 이전버전에서는, 구독이 "일회용"으로 불렸습니다.

```javascript
import { interval } from 'rxjs';

const observable = interval(1000);
const subscription = observable.subscribe(x => console.log(x));
// Later:
// This cancels the ongoing Observable execution which
// was started by calling subscribe with an Observer.
subscription.unsubscribe();
```

> 구독은 자원 할당을 해제하거나 Observable 실행을 취소시기기 위해 필수적으로 unsubscribe() 함수를 가지고 있습니다. 

구독들은 같이 둘 수 있습니다. 그래서 한번의 unsubscribe() 함수 호출로 여러개의 구독들을 구독 해제할 수 있습니다. 한 구독에 다른 구독을 더함으로서 할 수 있습니다:

```javascript
import { interval } from 'rxjs';

const observable1 = interval(400);
const observable2 = interval(300);

const subscription = observable1.subscribe(x => console.log('first: ' + x));
const childSubscription = observable2.subscribe(x => console.log('second: ' + x));

subscription.add(childSubscription);

setTimeout(() => {
  // Unsubscribes BOTH subscription and childSubscription
  subscription.unsubscribe();
}, 1000);
```

실행하면 아래와 같은 결과를 볼 수 있습니다:

```
second: 0
first: 0
second: 1
first: 1
second: 2
```

또한, 추가했던 구독을 제거하기 위해서 remove(다른 구독) 메소드도 있습니다.