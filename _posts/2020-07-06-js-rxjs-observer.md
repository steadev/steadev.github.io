---
title: "[Javascript] RxJS - Observer"
author: steadev
date: 2020-07-06 11:51:00 +0900
categories: [Javascript]
tags: [javascript, rxjs, observer]
---


[https://rxjs.dev/guide/observer](https://rxjs.dev/guide/observer) 번역

오역 부분이나 조언(?)은 알려주시면 감사하겠습니다.

### **Observer**

**관찰자(Observer)란 무엇인가?** 관찰자는 Observable에서 전달된 값들의 소비자입니다. 관찰자들은 간단히 Observer에서 전달하는 각각의 알림 유형들(next, error, complete)의 콜백들의 모음입니다. 아래는 전형적인 Observer 객체입니다 :

```javascript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

관찰자(Observer)를 사용하기 위해서는, Observable 구독시에 인자로 제공해야 합니다.

```javascript
observable.subscribe(observer);
```

> 관찰자는 그저 Observer에서 전달하는 각각의 알림 유형들(next, error, complete) 3개의 콜백들을 가진 객체입니다.

RxJS에서의 관찰자는 부분적일수도 있습니다. 만약 저 콜백중 하나를 제공하지 않는다면, Observable의 실행은 일반적일 것입니다. 콜백 중 하나를 제공하지 않으면 Observable의 실행은 여전히 ​​정상적으로 발생하지만 일부 유형의 알림은 관찰자(Observer)에 해당 콜백이 없기 때문에 무시됩니다.

아래의 예제는 complete 콜백이 없는 관찰자입니다 :

```javascript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
};
```

Observable을 구독할 때, 콜백들을 관찰자 객체에 붙어있지 않은 상태로 인자로 넘길수도 있습니다. 예를 들면 아래와 같습니다 :

```javascript
observable.subscribe(x => console.log('Observer got a next value: ' + x));
```

내부적으로 observable.subscribe에서 첫 번째 콜백 인수를 다음 핸들러로 사용하여 Observer 객체를 만듭니다. 세 가지 유형의 콜백이 모두 인수로 제공 될 수 있습니다 :

```javascript
observable.subscribe(
  x => console.log('Observer got a next value: ' + x),
  err => console.error('Observer got an error: ' + err),
  () => console.log('Observer got a complete notification')
);
```