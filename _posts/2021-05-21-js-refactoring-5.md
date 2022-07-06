---
title: "[Refactoring] Moving Features(1)"
author: steadev
date: 2021-05-21 06:08:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 8과의 내용 정리(1)입니다.

## Move Function(함수 옮기기)

\-어떤 함수가 자신이 속한 모듈 A의 요소들보다 다른 모듈 B의 요소들을 더 많이 참조한다면 B로 옮겨야 한다. (캡슐화up, 의존성down)  
\-함수를 옮길지 말지를 정하기는 쉽지 않지만, 선택이 어려울수록 큰 문제가 아닌 경우가 많다.  
\-중첩 함수를 사용하다보면 숨겨진 데이터끼리 상호 의존하기가 아주 쉬우니 중첩함수는 되도록 만들지 말자.

```javascript
// from
function trackSummary(points) {
  const totalTime = calculateTime();
  const totalDistance = calculateDistance();
  const pace = totalTime / 60 / totalDistance;
  return {
    time: totalTime,
    distance: totalDistance,
    pace: pace
  }

  function calculateDistance() { // 총 거리 계산
       let result = 0;
    for (const i = 1; i < points.length; i++) {
      result += distance(points[i-1], points[i]); 
    }
    return result;
  }

  function distance(p1, p2) { ... } // 두 지점의 거리 계산
  function radians(degrees) { ... } // 라디안 값으로 변환
  function calculateTime() { ... } // 총 시간 계산
}
------------------------------------------------------
// to
function trackSummary(points) { ... }
function totalDistance() { ... }
function distance(p1, p2) { ... }
function radians(degrees) { ... }
function calculateTime() { ... }
```

## Move Field(필드 옮기기)

\-데이터 구조가 중요. 데이터 구조를 잘못 선택하면 아귀가 맞지 않는 데이터를 다루기 위한 코드로 범벅이 된다.  
\-이를 곧바로 수정하지 않으면 훗날 작성하게 될 코드를 더욱 복잡하게 만든다..

```javascript
// from
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this._discountRate;}
}
------------------------------------------------------
// to
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this.plan._discountRate;}
}
```

## Move Statements into Function(문장을 함수로 옮기기)

\-중복 제거. 추후 반복되는 부분에서 무언가 수정할 일이 생겼을 때 단 한 곳만 수정하면 된다.  
  

```javascript
// from
function renderPerson(outStream, person) {
  const result = [];
  ...
  result.push(`<p>제목: ${person.photo.title}</p>`); // 제목 출력(중복코드)
  result.push(emitPhotoData(person.photo);
  ...
}
function photoDiv(p) {
  return [
    '<div>',
    `<p>제목: ${p.title}</p>`, // 제목 출력(중복 코드)
    emitPhotoData(p),
    '</div>'
  ].join('/n');
}
function emitPhotoData(aPhoto) {
  const result = [];
  result.push(`<p>위치: ${aPhoto.location}</p>`);
  result.push(`<p>날짜: ${aPhoto.date.toDateString()}</p>`);
  ...
  return result.join('/n');
}
------------------------------------------------
// to
function renderPerson(outStream, person) {
  const result = [];
  ...
  result.push(emitPhotoData(person.photo);
  ...
}
function photoDiv(p) {
  return [
    '<div>',
    emitPhotoData(p),
    '</div>'
  ].join('/n');
}
function emitPhotoData(aPhoto) {
  const result = [];
  result.push(`<p>제목: ${aPhoto.title}</p>`);
  result.push(`<p>위치: ${aPhoto.location}</p>`);
  result.push(`<p>날짜: ${aPhoto.date.toDateString()}</p>`);
  ...
  return result.join('/n');
}
```

## Move Statements to Caller(문장을 호출한 곳으로 옮기기)

\-위 문장을 함수로 옮기기의 반대 리팩터링  
\-초기에는 응집도 높고 한 가지 일만 수행하던 함수가 어느새 둘 이상의 다른 일을 수행하게 바뀔 수 있고, 이럴 때 활용.  
\-예컨데 여러 곳에서 사용하던 기능이 일부 호출자에게는 다르게 동작하도록 바뀌어야 한다면, 달라진 동작을 함수에서 꺼내 해당 호출자로 옮겨야 한다.

```javascript
// from 
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${aPhoto.title}</p>`);
  outStream.write(`<p>위치: ${aPhoto.location}</p>`);
}
-------------------------------------------------
// to
emitPhotoData(outStream, person.photo);
outStream.write(`<p>위치: ${aPhoto.location}</p>`);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>제목: ${aPhoto.title}</p>`);
}
```