---
title: "[Javascript] RxJS - Overview 번역"
author: steadev
date: 2020-07-01 20:47:00 +0900
categories: [Javascript]
tags: [javascript, rxjs]
---


Observable 객체를 다루는데 필수(?)적인 RxJS!!

rxjs 공식사이트 번역을 해봤습니다. ( [https://rxjs.dev/guide/overview](https://rxjs.dev/guide/overview) )

기분탓인지 모르겠지만 다른 개발문서들보다 어렵게(?) 쓰여있는거 같아서 의역도 좀 포함되어있습니다.

그렇지만.. 영어 실력이 부족해 오역이 있을 수 있습니다. (발견시, 알려주시면 감사하겠습니다.)

#### **소개**

RxJS는 Observable 시퀀스를 사용하여 비동기 및 이벤트 기반 프로그램을 작성하기위한 라이브러리입니다. [Observable](https://rxjs.dev/guide/observable), 위성 타입(Observer, Scheduler, Subjects), 그리고 [Array#extras](https://developer.mozilla.org/en-US/docs/Archive/Web/JavaScript/New_in_JavaScript/1.6) (map, filter, reduce, every 등)의 연산자, 이 셋 중 하나의 핵심 유형을 제공하여 비동기 이벤트를 콜렉션으로 처리 할 수 ​​있도록 해줍니다.

> _RxJS를 이벤트의 Lodash 정도로 생각하시면 됩니다._

ReactiveX는 [Observer 패턴](https://en.wikipedia.org/wiki/Observer_pattern)과 [Iterator 패턴](https://en.wikipedia.org/wiki/Iterator_pattern), 그리고 일련의 이벤트를 관리하는 이상적인 방법에 대한 요구를 충족시키기 위해 [컬렉션](https://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions)을 갖춘 [함수형 프로그래밍](https://martinfowler.com/articles/collection-pipeline/#NestedOperatorExpressions)을 결합했습니다.

비동기 이벤트 관리를 해결하는 RxJS의 필수 개념은 다음과 같습니다:

-   **Observable:** 호출할 수 있는 컬렉션(미래의 값이나 events)의 idea를 나타냅니다.
-   **Observer:** Observable이 전달한 값을 듣는 방법을 알고있는 콜백 모음입니다.
-   **Subscription:** Observable의 실행을 나타내며 주로 실행 취소에 유용합니다.
-   **Operators:** 함수형 프로그래밍 스타일을 가능하게하는 순수 함수들입니다. map, filter, concat, reduce 등과 같은 함수들을 다룹니다.
-   **Subject:** EventEmitter와 동일하며 value나 event를 여러 Observer에게 멀티 캐스팅하는 유일한 방법입니다.
-   **Schedulers:** 동시성을 제어하기위한 중앙 집중식 디스패처로, setTimeout, requestAnimationFrame 등의 계산이 수행되는 시점을 조정할 수 있습니다.

#### **첫 예제**

이벤트 리스너를 등록합니다.

```javascript
document.addEventListener('click', () => console.log('Clicked!'));
```

RxJS를 이용해서 observable 객체를 생성합니다.

```javascript
import { fromEvent } from 'rxjs';

fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```

#### **Purity**

RxJS를 강력하게 만드는 것은 순수 함수를 사용하여 value를 창출하는 능력입니다. 즉, 코드의 오류가 적습니다.

다른 부분의 코드가 state를 망칠 수 있는 비순수함수를 만듭니다.

```javascript
let count = 0;
document.addEventListener('click', () => console.log(`Clicked ${++count} times`));
```

RxJS를 이용하여 state를 떨어뜨립니다.

```javascript
import { fromEvent } from 'rxjs';
import { scan } from 'rxjs/operators';

fromEvent(document, 'click')
  .pipe(scan(count => count + 1, 0))
  .subscribe(count => console.log(`Clicked ${count} times`));
```

scan 함수는 array의 reduce 함수같은 역할을 합니다. 이는 콜백으로 넘어온 값으로 연산하고, 리턴 값은 다음 콜백의 값으로 전달됩니다.

#### **Flow**

RxJS는 이벤트의 흐름이 observable을 통과하는 방식을 컨트롤할 수 있게 도와주는 모든 범위의 연산자들을 가지고있습니다.

아래는 초당 최대 한번의 클릭을 허용하는 코드입니다. (순수 Javascript)

```javascript
let count = 0;
let rate = 1000;
let lastClick = Date.now() - rate;
document.addEventListener('click', () => {
  if (Date.now() - lastClick >= rate) {
    console.log(`Clicked ${++count} times`);
    lastClick = Date.now();
  }
});
```

RxJS:

```javascript
import { fromEvent } from 'rxjs';
import { throttleTime, scan } from 'rxjs/operators';

fromEvent(document, 'click')
  .pipe(
    throttleTime(1000),
    scan(count => count + 1, 0)
  )
  .subscribe(count => console.log(`Clicked ${count} times`));
```

다른 흐름제어 연산자들은  [filter](https://rxjs.dev/api/operators/filter), [delay](https://rxjs.dev/api/operators/delay), [debounceTime](https://rxjs.dev/api/operators/debounceTime), [take](https://rxjs.dev/api/operators/take), [takeUntil](https://rxjs.dev/api/operators/takeUntil), [distinct](https://rxjs.dev/api/operators/distinct), [distinctUntilChanged](https://rxjs.dev/api/operators/distinctUntilChanged) 등이 있습니다.

#### **Values**

observable을 통과해서 값을 바꿀 수 있습니다. 

아래는 매 클릭마다 현재 마우스의 x위치를 더하는 코드입니다. (순수 Javascript)

```javascript
let count = 0;
const rate = 1000;
let lastClick = Date.now() - rate;
document.addEventListener('click', event => {
  if (Date.now() - lastClick >= rate) {
    count += event.clientX;
    console.log(count);
    lastClick = Date.now();
  }
});
```

RxJS:

```javascript
import { fromEvent } from 'rxjs';
import { throttleTime, map, scan } from 'rxjs/operators';

fromEvent(document, 'click')
  .pipe(
    throttleTime(1000),
    map(event => event.clientX),
    scan((count, clientX) => count + clientX, 0)
  )
  .subscribe(count => console.log(count));
```

값을 생성하는 다른 연산자는 [pluck](https://rxjs.dev/api/operators/pluck), [pairwise](https://rxjs.dev/api/operators/pairwise), [sample](https://rxjs.dev/api/operators/sample) 등이 있습니다.

\- [RxJS - Observable](https://steadev.tistory.com/57)

\- [RxJS - Observer](https://steadev.tistory.com/58)

\- [RxJS - Operators](https://steadev.tistory.com/59?category=908072)

[\- RxJS - Subscription](https://steadev.tistory.com/60)

\- RxJS - Subjects ( ...ing)

\- RxJS - Scheduler ( ...ing)

\- RxJS - Testing ( ...ing)