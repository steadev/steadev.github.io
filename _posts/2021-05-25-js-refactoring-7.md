---
title: "[Refactoring] Organizing Data"
author: steadev
date: 2021-05-25 06:16:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 9과의 내용 정리입니다.

## Split Variable(변수 쪼개기)

\-역할이 둘 이상인 변수가 있다면 쪼개야만 한다. 코드를 읽는 이에게 커다란 혼란 초래..!  
  

```javascript
// from
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
---------------------------------
// to
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```

  
\-함수의 매개변수의 경우!

```javascript
// from
function discount(inputValue, quantity) {
  if (inputValue > 50) inputValue = inputValue - 2;
  if (quantity > 100) inputValue = inputValue - 1;
  return inputValue;
}
---------------------------------------------
// to
function discount(inputValue, quantity) {
  let result = inputValue;
  if (inputValue > 50) result = result - 2;
  if (quantity > 100) result = result - 1;
  return result;
}
```

## Rename Field(필드 이름 바꾸기)

\-이름은 매우 중요..!  
  

```javascript
get name() { ... } -> get title() { ... }
```

## Replace Derived Variable with Query(파생 변수를 질의 함수로 바꾸기)

\-가변 데이터는 서로 다른 두 코드를 이상한 방식으로 결합하기도 하는데, 한 쪽 코드에서 수정한 값이 연쇄 효과를 일으켜 다른 쪽 코드에 원인을 찾기 어려운 문제를 야기하기도 한다.  
\-가변 데이터를 완전히 배제하기란 현실적으로 불가능할 때가 많지만, 가변 데이터의 유효 범위를 가능한 한 좁혀야 한다.  
  

```javascript
// from
get production() { return this._production; }
applyAdjustment(anAudjustment) {
  this._adjustments.push(anAdujustment);
  this._production += anAdjustment.amount; // 이 누적 값은 함수에 직접 관련도 없고, 매번 갱신하지 않고도 계산할 수 있다.
}
---------------------------------------------
get production() { // 누적 계산을 분리
  return this._adjustments
      .reduce((sum, a) => sum + a.amount, 0);
}
applyAdjustment(anAudjustment) {
  this._adjustments.push(anAdujustment);
}
```

## Change Reference to Value(참조를 값으로 바꾸기)

\-값으로 바꾸면 불변이기에 자유롭게 활용하기 좋다. 서로간의 참조를 관리하지 않아도 된다.  
\-물론, 값을 공유해야 한다면 이 리팩터링은 적용하면 안된다.  
  

```javascript
// from
class Product {
  applyDiscount(arg) { this._price.amount -= arg; }
}
----------------------------------------------------
// to
class Product {
  applyDiscount(arg) { 
    this._price = new Money(this._price.amount - arg, this._price.money);
  }
}
```

## Change Value To Reference(값을 참조로 바꾸기)

\-참조를 값으로 바꾸기의 반대 리팩터링  
\-하나의 데이터 구조 안에 논리적으로 똑같은 제 3의 데이터 구조를 참조하는 레코드가 여러 개 있을 수 있다. 이 때, 참조로 다루면 좋을 수 있다.  
  

```javascript
// from
let customer = new Customer(customerData);
// to
let customer = customerRepository.get(customerData.id);
```

## Replace magic Literal(매직 리터럴 바꾸기)

\-매직 리터럴이란 소스코드에 등장하는 일반적인 리터럴 값을 말한다.  
  

```javascript
// from
function potentialEnergy(mass, height) {
  return mass * 9.81 * height;
}
---------------------------------------
// to
const STANDARD_GRAVITY = 9.81;
function potentialEnergy(mass, height) {
  return mass * STANDARD_GRAVITY * height;
}
```