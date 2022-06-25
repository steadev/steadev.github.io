---
title: "[Javascript] RxJS - Observable"
author: steadev
date: 2020-07-04 22:03:00 +0900
categories: [JAVASCRIPT]
tags: [javascript, rxjs, observable]
---


[https://rxjs.dev/guide/observable](https://rxjs.dev/guide/observable) 번역

오역 부분이나 조언(?)은 알려주시면 감사하겠습니다.

### **Observable**

Observable은 여러 값의 지연푸시 모음입니다. 그들은 다음 표에서 누락된 자리를 채웁니다.

|   | SINGLE | MULTIPLE |
| --- | --- | --- |
|   Pull | [Function](https://developer.mozilla.org/en-US/docs/Glossary/Function) | [Iterator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) |
|   Push | [Promise](https://developer.mozilla.org/en-US/docs/Mozilla/JavaScript_code_modules/Promise.jsm/Promise) | [Observable](https://rxjs.dev/class/es6/Observable.js~Observable.html) |

예제. 아래 예제는 구독(subscribe)됐을 때, 동기적으로 값 1,2,3을 즉시 넣고, 1초 뒤에 4를 넣고 끝내는 Observable입니다.

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

Observable을 호출하고 값들을 보기위해서는 구독(subscribe) 해야합니다.

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});

console.log('just before subscribe');
observable.subscribe({
  next(x) { console.log('got value ' + x); },
  error(err) { console.error('something wrong occurred: ' + err); },
  complete() { console.log('done'); }
});
console.log('just after subscribe');
```

위 실행 결과는 다음과 같습니다.

```
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

### **Pull versus Push**

Pull과 Push는 두개의 다른 프로토콜입니다. 이는 데이터 생산자가 데이터 소비자와 어떻게 소통하는지를 나타냅니다.

**Pull이란?** Pull 시스템에서는, 소비자가 생산자로부터 데이터를 언제 받을지 결정합니다. 생산자는 언제 소비자한테 전달될지 모릅니다.

모든 JavaScript 함수는 Pull 시스템입니다. 함수는 데이터 생산자이고, 그 함수를 부르는 코드는 함수의 리턴값을 pull 해서 소비합니다.

ES2015는 다른 Pull 시스템인, [generator functions and iterators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) (function\*) 를 소개했습니다. iterator.next()를 호출하는 코드가 소비자입니다. iterator(생산자)로부터 여러 값들을 가져옵니다(pulling).

|   | **PRODUCER** | ****CONSUMER**** |
| --- | --- | --- |
| **Pull** | **Passive:** 요청이 있을때 데이터 생성 | **Active:** 언제 데이터를 요청할지 정함 |
| **Push** | **Active:** 자신의 페이스대로 데이터 생성 | **Passive:** 받은 데이터에 반응 |

**Push란?** Push 시스템에서는, 생산자가 소비자에게 언제 데이터를 보낼지 결정합니다. 소비자는 언제 데이터를 받게 될지 모릅니다.

오늘날 JavaScript에서, Promise들은 가장 평범한 타입의 Push 시스템입니다. Promise(생산자)는 resolve된 값을 등록된 콜백들(소비자)로 전달합니다. 하지만 함수들과 다르게, 해당 값이 콜백에 "밀어 질 때"를 정확하게 결정합니다.

RxJS는 JavaScript를 위한 새로운 Push 시스템인 Observables를 소개합니다. Observable은 여러 값들의 생산자입니다. Observers(소비자들)에게 그 값들을 보냅니다(Pushing).

-   **Function**은 호출시에 동기적으로 한개의 값을 리턴하는 지연 계산(lazily evaluated computation)입니다.
-   **generator**는 iteration시에 동기적으로 0개부터 무한대까지의 값들을 리턴하는 지연 계산입니다.
-   **Promise**는 단일 값을 반환하거나 반환하지 않을 수 있는 계산입니다.
-   **Observable**은 호출된 이후로부터 동기적이거나 비동기적으로 0개부터 무한대가지의 값들을 리턴하는 지연계산입니다.

### **Observables as generalizations of functions**

대중적인 주장과 달리, Obervable들은 EventEmitter와 같지 않으며 여러 값에 대한 Promise들과도 같지 않습니다. Observable들은 몇몇 경우에 EventEmitter처럼 동작할 수 있습니다([RxJS Subjects를 이용해서 멀티캐스트 된 경우](https://rxjs.dev/guide/subject#multicasted-observables)). 하지만, 보통은 EventEmitter 처럼 동작하지 않습니다. 

> _Observable들은 인자가 없는 함수와 같습니다. 하지만 여러 값을 허용하도록 일반화 합니다._

다음을 고려해보겠습니다:

```javascript
function foo() {
  console.log('Hello');
  return 42;
}

const x = foo.call(); // same as foo()
console.log(x);
const y = foo.call(); // same as foo()
console.log(y);
```

아래와 같은 결과를 예상할 수 있습니다.

```
"Hello"
42
"Hello"
42
```

같은 동작을 Observable을 이용해보겠습니다.

```javascript
import { Observable } from 'rxjs';

const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
});

foo.subscribe(x => {
  console.log(x);
});
foo.subscribe(y => {
  console.log(y);
});
```

그리고 결과도 같습니다.

```
"Hello"
42
"Hello"
42
```

이렇게 되는 이유는 Function과 Observable이 둘다 지연계산이기 때문입니다. 만약 함수를 호출하지 않는다면, console.log('Hello')는 실행되지 않을 겁니다. Observable도 똑같이 호출하지 않으면 console.log('Hello')는 실행되지 않습니다. 게다가, 'calling' 또는 'subscribing'은 독립된 작업입니다: 2개의 함수 호출이 두개의 별도의 부작용를 일으킵니다. 그리고 2개의 Observable 구독은 두개의 별도의 부작용을 일으킵니다. 부작용들을 공유하고 구독의 존재와 상관없이 실행하는 EventEmitter들과 달리, Observable들은 실행을 공유하지 않고 게으릅니다.

Observable을 구독하는 것은 함수호출과 유사합니다.

몇몇 사람들은 Observable들이 비동기적이라고 주장합니다. 하지만 그것은 사실이 아닙니다. 만약 아래와 같이 함수 호출을 log들로 감쌌을 경우:

```javascript
console.log('before');
console.log(foo.call());
console.log('after');
```

아래와 같은 결과를 볼 수 있습니다. 

```
"before"
"Hello"
42
"after"
```

그리고 이것은 Observable로도 똑같이 동작합니다.

```javascript
console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

결과:

```
"before"
"Hello"
42
"after"
```

이것은 foo의 subscription이 함수처럼 완전히 동기적이라는 것을 증명합니다.

Obervable들은 동기적으로나 비동기적으로 값들을 전달할 수 있습니다. 

그러면 Observable과 함수와의 차이점은 무엇일까? **Observable은 시간이 지남에 따라 여러 값들을 리턴할 수 있습니다.** 이는 함수에서는 불가능합니다. 아래처럼 할 수 없습니다:

```javascript
function foo() {
  console.log('Hello');
  return 42;
  return 100; // dead code. will never happen
}
```

함수는 한 값만 리턴할 수 있습니다. 하지만 Observable은 이렇게 할 수 있습니다:

```javascript
import { Observable } from 'rxjs';

const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100); // "return" another value
  subscriber.next(200); // "return" yet another
});

console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

동기적 결과:

```
"before"
"Hello"
42
100
200
"after"
```

비동기적으로도 리턴할 수 있습니다:

```javascript
import { Observable } from 'rxjs';

const foo = new Observable(subscriber => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100);
  subscriber.next(200);
  setTimeout(() => {
    subscriber.next(300); // happens asynchronously
  }, 1000);
});

console.log('before');
foo.subscribe(x => {
  console.log(x);
});
console.log('after');
```

결과:

```
"before"
"Hello"
42
100
200
"after"
300
```

결론:

-   func.call()은 "동기적으로 한개의 값을 줘"를 의미합니다.
-   observable.subscribe()는 "동기적으로든, 비동기적으로든, 값이 몇개가 됐든 줘"를 의미합니다.

### **Anatomy of an Observable (Observable 해부)**

Observable들은 new Observable이나 생성자를 통해 **생성됩니다**. Observer에 의해 **구독됩니다**. Observer에게 next / error/ complete 알림들을 전달하는 작업을 **수행합니다**. 그리고 그들의 수행은 **처분될 수 있습니다**. 이 4가지 양상은 모두 Observable 인스턴스로 인코딩 됩니다. 하지만 몇 가지 양상은 다른 타입들(Observer, Subscription)과 연관되있습니다.

Observable의 핵심 관심사:

-   **생성(Creating)** Observables
-   **구독(Subscribing)** to Observables
-   **수행(Executing)** the Observable
-   **처분(Disposing)** Observables

### **Creating Observables**

Observable 생성자는 한개의 인자(구독 함수)를 받습니다.

아래의 예제는 관찰자(Observer)에게 매초마다 'hi'를 보내는 Observable을 생성합니다.

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next('hi')
  }, 1000);
});
```

> Observable들은 new Observable 로 생성할 수 있습니다. 가장 흔하게, Observable들은 생성함수(of, from, interval 등)를 통해 생성됩니다.

위 예제에서, 구독 함수는 Observable을 설명하는 가장 중요한 부분입니다. 구독하는 것이 무엇을 의미하는지 보도록 하겠습니다.

### **Subscribing to Observables**

예제 속의 observable은 아래와 같이 구독될 수 있습니다.

```javascript
observable.subscribe(x => console.log(x));
```

observable.subscribe와 new Observable(function subscribe(subscriber){...}의 subscribe가 같은 이름을 가지고 있는 것은 우연의 일치가 아닙니다. 둘은 다르지만, 실용적인 목적에서 개념적으로 동일하게 볼 수 있습니다.

이는 같은 Observable의 여러 Observer들 사이에서 어떻게 구독 호출이 공유되지 않는지 보여줍니다. Observer에서 observable.subscribe를 호출할 때, new Observable(function subscribe(subscriber){...} 속의 subscribe 함수가 주어진 관찰자를 위해 실행됩니다. obervable.subscribe 각각의 호출은 해당 관찰자에 대해 독립적인 셋업을 발생시킵니다.

> _Observable을 구독하는 것은 함수를 호출하는 것과 같습니다. 데이터가 전달되는 콜백을 제공합니다._

### **Executing Observables**

new Observable(function subscribe(subscriber) {...}) 속의 코드는 "Observable 실행"(각 관찰자들이 구독할때만 일어나는 지연 계산)을 나타냅니다. 실행은 동기적이거나 비동기적으로 여러 값을 생성합니다.

Observable 실행에 세 가지 유형의 값이 있습니다 :

-   "Next" notification: 숫자, 문자열, 객체 등과 같은 값을 보냅니다.
-   "Error" notification: Javascript 에러나 예외를 보냅니다.
-   "Complete" notification: 값을 보내지 않습니다.

"Next" notifications는 가장 중요하고 가장 일반적인 타입입니다. 이는 관찰자에게 전달되는 실제 데이터를 나타냅니다. 

"Error"와 "Complete" notifications는 Observable 실행동안 한번만 일어날 수 있습니다. 그리고 둘 중 하나만 수행될 수 있습니다.

이런 제약들은 정규식으로 표현된, 이른바 Observable 문법이나 규칙에 제일 잘 표현되어있습니다 : 

```javascript
next*(error|complete)?
```

> _Observable 실행에서, 0 ~ 무한대의 Next notification이 전달될 수 있습니다. 만약 Error나 Complete notification이 전달된다면, 이후에는 아무것도 전달되지 않습니다._

아래는 3개의 Next notification을 전달하고 complete 하는 예제입니다 : 

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});
```

Observables는 엄격하게 Observable 규칙을 엄격히 지킵니다. 그렇기에 아래의 코드는 '4'를 전달하지 않습니다 :

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
  subscriber.next(4); // Is not delivered because it would violate the contract
});
```

subscribe안의 코드들을 try/catch문으로 감싸는 것은 좋은 생각입니다. 이는 에러 발생시, Error notification을 전달할 수  것입니다 :

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.next(3);
    subscriber.complete();
  } catch (err) {
    subscriber.error(err); // delivers an error if it caught one
  }
});
```

### **Disposing Observable Executions**

Observable의 수행은 무한하고 관찰자가 유한한 시간내에 실행을 중단하려 하는 것이 일반적이기 때문에, 실행을 취소할 수 있는 API가 필요합니다. 각 실행은 하나의 관찰자에게만 독점되므로, 계산능력을 낭비하거나 메모리 낭비를 피하기 위해 관찰자가 값을 다 받으면 실행을 중지할 방법이 있어야 합니다.

observable.subscribe가 호출될 때, 관찰자는 새로 생성 된 Observable 실행에 연결됩니다. 이 호출은 또한 Subscription 객체를 리턴합니다 :

```javascript
const subscription = observable.subscribe(x => console.log(x));
```

이 Subscription은 진행중인 실행을 나타냅니다. 그리고 실행을 취소할 수 있는 최소한의 API를 가집니다. 자세한 것은 여기에서 [Subscription Type](https://rxjs.dev/guide/subscription)에 대해 읽어보세요. subscription.unsubscribe()로 실행을 중단할 수 있습니다 :

```javascript
import { from } from 'rxjs';

const observable = from([10, 20, 30]);
const subscription = observable.subscribe(x => console.log(x));
// Later:
subscription.unsubscribe();
```

> _구독 했을 때, Subscription을 돌려받을 수 있습니다. 이는 진행중인 실행을 나타냅니다. unsubscribe()만 호출하면 실행을 중단할 수 있습니다._

각 Observable은 create()를 사용하여 Observable을 만들 때 해당 실행 리소스를 처리하는 방법을 무조건 정의해야합니다. subscribe() 함수 내에서 사용자 지정 구독 취소 함수를 반환할 수도 있습니다.  

예를 들어, 이것은 setInterval을 clear 하는 방법입니다 :

```javascript
const observable = new Observable(function subscribe(subscriber) {
  // Keep track of the interval resource
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  // Provide a way of canceling and disposing the interval resource
  return function unsubscribe() {
    clearInterval(intervalId);
  };
});
```

observable.subscribe가 new Observable(function subscribe() {...})와 닮은 것처럼, subscribe에서 리턴한 unsubscribe는 개념적으로 subscribe.unsubscribe와 동일합니다. 실제로 이러한 개념을 둘러싼 ReactiveX 유형을 제거하면 다소 간단한 JavaScript가 남아 있습니다.

```javascript
function subscribe(subscriber) {
  const intervalId = setInterval(() => {
    subscriber.next('hi');
  }, 1000);

  return function unsubscribe() {
    clearInterval(intervalId);
  };
}

const unsubscribe = subscribe({next: (x) => console.log(x)});

// Later:
unsubscribe(); // dispose the resources
```

Observable, Observer, Subscription 과 같은 Rx 타입들을 사용하는 이유는 안전(Observable 규칙 같은)과 연산들의 조합 때문입니다.