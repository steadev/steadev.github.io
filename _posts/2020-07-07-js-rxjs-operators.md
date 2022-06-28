---
title: "[Javascript] RxJS - Operators"
author: steadev
date: 2020-07-07 13:57:00 +0900
categories: [JAVASCRIPT]
tags: [javascript, rxjs, observer]
---


[https://rxjs.dev/guide/operators](https://rxjs.dev/guide/operators) 번역

오역 부분이나 조언(?)은 알려주시면 감사하겠습니다.

RxJS는 Observable이 기본이지만, 대부분 연산자에 유용합니다. 연산자는 복잡한 비동기코드를 선언적 방식으로 구성하는데 필수적인 부분입니다.

## 연산자란?

연산자는 함수입니다. 2종류의 연산자가 있습니다 :

**파이프형 연산자**들은 observableInstance.pipe(operator()) 구문을 사용해 Observable로 파이프 될 수 있습니다. 이는 [filter(...)](https://rxjs.dev/api/operators/filter)와 [mergeMap(...)](https://rxjs.dev/api/operators/mergeMap)을 포함합니다. 호출되면 Observable 객체를 변경하지는 않습니다. 대신, 새로운 Observable을 반환합니다. 구독 로직은 첫번째 Observable에 기반합니다.

> 파이프형 연산자는 Observable을 input처럼 취해서 다른 Observable을 반환하는 함수입니다. 이는 순수 연산입니다 : 이전 Observable은 수정되지 않습니다.  

파이프형 연산자는 필수적으로 순수함수입니다. Observable을 받아 다른 Observable을 생산합니다. 반환된 Observable을 구독하는 것은 input Observable 또한 구독하는 것입니다.

**생성 연산자**들은 다른 종류의 연산자입니다. 이는 새로운 Observable을 생성하기 위해 단독 함수로써 호출될 수 있습니다. 예를 들어, of(1, 2, 3) 은 1, 2, 3을 하나씩 차례로 내보내는 Observable을 생성합니다. 생성 연산자들은 이후의 섹션에서 더 상세하게 다루겠습니다.

예를들어, [map](https://rxjs.dev/api/operators/map) 이라는 연산자는 같은 이름의 배열 메소드와 유사합니다. \[1, 2, 3\].map(x => x \* x) 의 결과는 \[1, 4, 9\] 인 것처럼, Observable은 아래와 같이 생성됩니다:

```javascript
import { of } from 'rxjs';
import { map } from 'rxjs/operators';

map(x => x * x)(of(1, 2, 3)).subscribe((v) => console.log(`value: ${v}`));

// Logs:
// value: 1 
// value: 4
// value: 9
```

1, 4, 9의 결과가 나옵니다. 다른 유용한 연산자는 [first](https://rxjs.dev/api/operators/first) 입니다:

```javascript
import { of } from 'rxjs';
import { first } from 'rxjs/operators';

first()(of(1, 2, 3)).subscribe((v) => console.log(`value: ${v}`));

// Logs:
// value: 1
```

map은 논리적으로 즉석에서 구성되어야 합니다. 매핑 기능을 제공해야 하기 때문입니다. 대조적으로, first는 변함없는 값일 수 있습니다. 하지만 그럼에도 불구하고 즉석에서 구성되어야 합니다. 일반적으로 모든 연산자는 인수가 필요한지 여부에 관계없이 구성됩니다.

## Piping

파이프형 연산자들은 함수입니다. 그래서 보통의 함수처럼 사용될 수 있습니다. op()(obs) ㅡ 하지만 실제로는, 많은 연산자들이 함께 모이는 경향이 있습니다. 그리고 읽을수 없게 되어버립니다: op4()(op3()(op2()(op1()(obs)))). 이런 이유로, Observable은 .pipe()라는 메소드가 있습니다. 이는 똑같은 일이지만 훨신 가독성이 좋게 해줍니다: 

```javascript
obs.pipe(
  op1(),
  op2(),
  op3(),
  op3(),
)
```

op()(obs) 는 절대 쓰이지 않습니다. 심지어 하나의 연산자만 있더라도 obs.pipe(op()) 가 보편적으로 선호됩니다.

## Creation Operators

**생성 연산자란?** 파이프형 연산자와는 구분되게, 생성 연산자들은 흔한 사전 정의된 행동이나 다른 Observable들을 조합함으로써 Observable을 생성하는데 사용되는 함수입니다.

생성 연산자의 전형적인 예제는 interval 함수입니다. 이는 숫자를 인자로 받아 Observable을 생성해 반환합니다:

```javascript
import { interval } from 'rxjs';

const observable = interval(1000 /* number of milliseconds */);
```

[모든 정적 생성 연산자 리스트](https://rxjs.dev/guide/operators#creation-operators) 입니다.

## Higher-order Observables (고차 Observables)

Observable은 가장 보편적으로 string, number 같은 보통의 값들을 내보냅니다. 하지만 놀랍게도 종종, Observable의 Observable을 다뤄야 할 필요가 있습니다. 소위 고차 Observable이라 합니다. 예를 들어, 파일의 URL인 string들을 내보내는 Observable이 있다고 상상해봅시다. 코드는 아래와 같을 것입니다:

```javascript
const fileObservable = urlObservable.pipe(
   map(url => http.get(url)),
);
```

http.get()은 각각의 URL에 대해 (string 혹은 string 배열의) Observable을 반환합니다. 이제 Observable의 Observable(고차 Observable)이 생겼습니다.

하지만 고차 Observable로 어떻게 작업을 해야할까요? 일반적으로, 평탄화해서 사용합니다: 어떤 방법을 이용해서 고차 Observable을 하나의 보통의 Observable로 바꿔줍니다. 예시:

```javascript
const fileObservable = urlObservable.pipe(
   map(url => http.get(url)),
   concatAll(),
);
```

[concatAll()](https://rxjs.dev/api/operators/concatAll) 연산자는 바깥의 Observable에서 나온 안쪽 각각의 Observable들을 구독하고, 한 Observable이 끝날때까지 도출된 값들을 복사합니다. 그리고 다음으로 넘어갑니다. 모든 값들은 그런식으로 연결됩니다. 다른 유용한 평탄화 연산자(소위 join 연산자)들은 

-   [mergeAll()](https://rxjs.dev/api/operators/mergeAll) — 도착하는대로 안쪽의 Observable을 구독하고 값을 도출합니다.
-   [switchAll()](https://rxjs.dev/api/operators/switchAll) — 첫번째 안쪽 Observable을 도착하는대로 구독하고, 값을 도출합니다. 하지만 다음 안쪽 Observable이 도착하면, 이전의 것은 구독 해제하고 새로운 것을 구독합니다.
-   [exhaust()](https://rxjs.dev/api/operators/exhaust) — 첫번째 안쪽 Observable을 도착하는대로 구독하고 값을 도출합니다. 첫번째가 끝나기 전까지는 다른 새로운 Observable들은 무시합니다. 끝나면 다음 Observable을 기다립니다.

많은 배열 라이브러리가 [map()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)과 [flat()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) (혹은 flatten())을 [flatMap()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap) 하나로 합친것 처럼, 이에 상응하는 RxJS 평탄화 연산자들이 있습니다. [concatMap()](https://rxjs.dev/api/operators/concatMap), [mergeMap()](https://rxjs.dev/api/operators/mergeMap), [switchMap()](https://rxjs.dev/api/operators/switchMap), and [exhaustMap()](https://rxjs.dev/api/operators/exhaustMap).

## Marble diagrams(구슬 다이어그램)

연산자들의 동작을 설명하기에 종종 문자로는 충분치 않습니다. 많은 연산자들은 시간과 연관되있습니다. 예를들어, 연산자들은 여러가지 방법으로 값 배출을 지연, 샘플링, 조절, 또는 디바운스 할 수 있습니다. 다이어그램은 종종 이것에 더 나은 방법입니다. 구슬 다이어그램은 어떻게 연산자들이 동작하는지 시각적으로 보여주고, input Observable, 연산자와 파라미터, output Observable을 포함하고 있습니다.

> 구슬 다이어그램에서, 시간 흐름은 왼->오 입니다. 이 다이어그램은 Observable 실행에서 어떻게 값들(구슬)이 배출되는지를 묘사합니다.

아래에서 구슬 다이어그램을 설명하고 있습니다.

<img src="https://steadev.github.io/assets/js/2020-07-07-1.png" />

이 문서 사이트 전체에서 구슬 다이어그램을 광범위하게 사용하여 연산자의 작동 방식을 설명합니다. 화이트 보드 나 단위 테스트 (ASCII 다이어그램)와 같은 다른 상황에서도 유용 할 수 있습니다.

## Categories of operators

다른 목적의 연산자들이 있습니다. 그리고 이는 생성, 변환, 필터링, 결합, 멀티 캐스팅, 오류 처리, 유틸리티 등으로 분류 될 수 있습니다. 다음의 리스트에서는 모든 연산자가 카테고리별로 구성되어 있습니다. 

전체 개요는 [참조 페이지](https://rxjs.dev/api)를 참조하십시오

### Creation Operators

-   [ajax](https://rxjs.dev/api/ajax/ajax)
-   [bindCallback](https://rxjs.dev/api/index/function/bindCallback)
-   [bindNodeCallback](https://rxjs.dev/api/index/function/bindNodeCallback)
-   [defer](https://rxjs.dev/api/index/function/defer)
-   [empty](https://rxjs.dev/api/index/function/empty)
-   [from](https://rxjs.dev/api/index/function/from)
-   [fromEvent](https://rxjs.dev/api/index/function/fromEvent)
-   [fromEventPattern](https://rxjs.dev/api/index/function/fromEventPattern)
-   [generate](https://rxjs.dev/api/index/function/generate)
-   [interval](https://rxjs.dev/api/index/function/interval)
-   [of](https://rxjs.dev/api/index/function/of)
-   [range](https://rxjs.dev/api/index/function/range)
-   [throwError](https://rxjs.dev/api/index/function/throwError)
-   [timer](https://rxjs.dev/api/index/function/timer)
-   [iif](https://rxjs.dev/api/index/function/iif)

### Join Creation Operators

These are Observable creation operators that also have join functionality -- emitting values of multiple source Observables.

-   [combineLatest](https://rxjs.dev/api/index/function/combineLatest)
-   [concat](https://rxjs.dev/api/index/function/concat)
-   [forkJoin](https://rxjs.dev/api/index/function/forkJoin)
-   [merge](https://rxjs.dev/api/index/function/merge)
-   [partition](https://rxjs.dev/api/index/function/partition)
-   [race](https://rxjs.dev/api/index/function/race)
-   [zip](https://rxjs.dev/api/index/function/zip)

### Transformation Operators

-   [buffer](https://rxjs.dev/api/operators/buffer)
-   [bufferCount](https://rxjs.dev/api/operators/bufferCount)
-   [bufferTime](https://rxjs.dev/api/operators/bufferTime)
-   [bufferToggle](https://rxjs.dev/api/operators/bufferToggle)
-   [bufferWhen](https://rxjs.dev/api/operators/bufferWhen)
-   [concatMap](https://rxjs.dev/api/operators/concatMap)
-   [concatMapTo](https://rxjs.dev/api/operators/concatMapTo)
-   [exhaust](https://rxjs.dev/api/operators/exhaust)
-   [exhaustMap](https://rxjs.dev/api/operators/exhaustMap)
-   [expand](https://rxjs.dev/api/operators/expand)
-   [groupBy](https://rxjs.dev/api/operators/groupBy)
-   [map](https://rxjs.dev/api/operators/map)
-   [mapTo](https://rxjs.dev/api/operators/mapTo)
-   [mergeMap](https://rxjs.dev/api/operators/mergeMap)
-   [mergeMapTo](https://rxjs.dev/api/operators/mergeMapTo)
-   [mergeScan](https://rxjs.dev/api/operators/mergeScan)
-   [pairwise](https://rxjs.dev/api/operators/pairwise)
-   [partition](https://rxjs.dev/api/operators/partition)
-   [pluck](https://rxjs.dev/api/operators/pluck)
-   [scan](https://rxjs.dev/api/operators/scan)
-   [switchMap](https://rxjs.dev/api/operators/switchMap)
-   [switchMapTo](https://rxjs.dev/api/operators/switchMapTo)
-   [window](https://rxjs.dev/api/operators/window)
-   [windowCount](https://rxjs.dev/api/operators/windowCount)
-   [windowTime](https://rxjs.dev/api/operators/windowTime)
-   [windowToggle](https://rxjs.dev/api/operators/windowToggle)
-   [windowWhen](https://rxjs.dev/api/operators/windowWhen)

### Filtering Operators

-   [audit](https://rxjs.dev/api/operators/audit)
-   [auditTime](https://rxjs.dev/api/operators/auditTime)
-   [debounce](https://rxjs.dev/api/operators/debounce)
-   [debounceTime](https://rxjs.dev/api/operators/debounceTime)
-   [distinct](https://rxjs.dev/api/operators/distinct)
-   [distinctKey](https://rxjs.dev/class/es6/Observable.js~Observable.html#instance-method-distinctKey)
-   [distinctUntilChanged](https://rxjs.dev/api/operators/distinctUntilChanged)
-   [distinctUntilKeyChanged](https://rxjs.dev/api/operators/distinctUntilKeyChanged)
-   [elementAt](https://rxjs.dev/api/operators/elementAt)
-   [filter](https://rxjs.dev/api/operators/filter)
-   [first](https://rxjs.dev/api/operators/first)
-   [ignoreElements](https://rxjs.dev/api/operators/ignoreElements)
-   [last](https://rxjs.dev/api/operators/last)
-   [sample](https://rxjs.dev/api/operators/sample)
-   [sampleTime](https://rxjs.dev/api/operators/sampleTime)
-   [single](https://rxjs.dev/api/operators/single)
-   [skip](https://rxjs.dev/api/operators/skip)
-   [skipLast](https://rxjs.dev/api/operators/skipLast)
-   [skipUntil](https://rxjs.dev/api/operators/skipUntil)
-   [skipWhile](https://rxjs.dev/api/operators/skipWhile)
-   [take](https://rxjs.dev/api/operators/take)
-   [takeLast](https://rxjs.dev/api/operators/takeLast)
-   [takeUntil](https://rxjs.dev/api/operators/takeUntil)
-   [takeWhile](https://rxjs.dev/api/operators/takeWhile)
-   [throttle](https://rxjs.dev/api/operators/throttle)
-   [throttleTime](https://rxjs.dev/api/operators/throttleTime)

### Join Operators

Also see the [Join Creation Operators](https://rxjs.dev/guide/operators#join-creation-operators) section above.

-   [combineAll](https://rxjs.dev/api/operators/combineAll)
-   [concatAll](https://rxjs.dev/api/operators/concatAll)
-   [exhaust](https://rxjs.dev/api/operators/exhaust)
-   [mergeAll](https://rxjs.dev/api/operators/mergeAll)
-   [startWith](https://rxjs.dev/api/operators/startWith)
-   [withLatestFrom](https://rxjs.dev/api/operators/withLatestFrom)

### Multicasting Operators

-   [multicast](https://rxjs.dev/api/operators/multicast)
-   [publish](https://rxjs.dev/api/operators/publish)
-   [publishBehavior](https://rxjs.dev/api/operators/publishBehavior)
-   [publishLast](https://rxjs.dev/api/operators/publishLast)
-   [publishReplay](https://rxjs.dev/api/operators/publishReplay)
-   [share](https://rxjs.dev/api/operators/share)

### Error Handling Operators

-   [catchError](https://rxjs.dev/api/operators/catchError)
-   [retry](https://rxjs.dev/api/operators/retry)
-   [retryWhen](https://rxjs.dev/api/operators/retryWhen)

### Utility Operators

-   [tap](https://rxjs.dev/api/operators/tap)
-   [delay](https://rxjs.dev/api/operators/delay)
-   [delayWhen](https://rxjs.dev/api/operators/delayWhen)
-   [dematerialize](https://rxjs.dev/api/operators/dematerialize)
-   [materialize](https://rxjs.dev/api/operators/materialize)
-   [observeOn](https://rxjs.dev/api/operators/observeOn)
-   [subscribeOn](https://rxjs.dev/api/operators/subscribeOn)
-   [timeInterval](https://rxjs.dev/api/operators/timeInterval)
-   [timestamp](https://rxjs.dev/api/operators/timestamp)
-   [timeout](https://rxjs.dev/api/operators/timeout)
-   [timeoutWith](https://rxjs.dev/api/operators/timeoutWith)
-   [toArray](https://rxjs.dev/api/operators/toArray)

### Conditional and Boolean Operators

-   [defaultIfEmpty](https://rxjs.dev/api/operators/defaultIfEmpty)
-   [every](https://rxjs.dev/api/operators/every)
-   [find](https://rxjs.dev/api/operators/find)
-   [findIndex](https://rxjs.dev/api/operators/findIndex)
-   [isEmpty](https://rxjs.dev/api/operators/isEmpty)

### Mathematical and Aggregate Operators

-   [count](https://rxjs.dev/api/operators/count)
-   [max](https://rxjs.dev/api/operators/max)
-   [min](https://rxjs.dev/api/operators/min)
-   [reduce](https://rxjs.dev/api/operators/reduce)

## 커스텀 연산자 만들기

### pipe() 함수로 새로운 연산자 만들기

만약 흔히 사용되는 연산자들의 시퀀스가 있다면, pipe() 함수를 이용해서 그 시퀀스를 하나의 연산자로 추출할 수 있습니다. 만약 시퀀스가 흔하지 않더라도, 하나의 연산자로 바꾸는 것은 가독성을 개선시킬 수 있습니다.

예를들어, 아래와 같이 홀수를 버리고 짝수를 2배하는 함수를 만들 수 있습니다:

```javascript
import { pipe } from 'rxjs';
import { filter, map } from 'rxjs/operators';

function discardOddDoubleEven() {
  return pipe(
    filter(v => ! (v % 2)),
    map(v => v + v),
  );
}
```

(pipe() 함수는 Observable의 .pipe() 메소드와 유사하지만 같지는 않습니다.)

### scratch로 새로운 연산자 만들기

더 복잡하지만, 기존 연산자의 조합으로 만들 수없는 연산자를 작성해야하는 경우(드물게 발생하지만...) 아래와 같이 Observable 생성자를 사용하여 처음부터 연산자를 작성할 수 있습니다:

```javascript
import { Observable } from 'rxjs';

function delay(delayInMillis) {
  return (observable) => new Observable(observer => {
    // this function will called each time this
    // Observable is subscribed to.
    const allTimerIDs = new Set();
    const subscription = observable.subscribe({
      next(value) {
        const timerID = setTimeout(() => {
          observer.next(value);
          allTimerIDs.delete(timerID);
        }, delayInMillis);
        allTimerIDs.add(timerID);
      },
      error(err) {
        observer.error(err);
      },
      complete() {
        observer.complete();
      }
    });
    // the return value is the teardown function,
    // which will be invoked when the new
    // Observable is unsubscribed from.
    return () => {
      subscription.unsubscribe();
      allTimerIDs.forEach(timerID => {
        clearTimeout(timerID);
      });
    }
  });
}
```

Note that you must

1.  입력 Observable을 구독할 때, 세 가지 관찰자 함수 next(), error(), complete()를 모두 구현해야 합니다.
2.  Observable이 끝났을 때 정리하는(이 경우에는 구독 취소하고 timeout 초기화) "해체(teardown)" 함수를 구현해야 합니다.
3.  Observable 생성자에 전달된 함수에서 해체함수(teardown function)를 반환합니다.

물론, 이는 한 예시일 뿐입니다. delay() 연산자는 [이미 있습니다.](https://rxjs.dev/api/operators/delay)