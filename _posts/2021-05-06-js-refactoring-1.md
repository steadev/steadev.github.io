---
title: "[Refactoring] A First Set Of Refactorings(1)"
author: steadev
date: 2021-05-06 07:14:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 6과의 내용 정리입니다.

## Extract Function (함수 추출하기)

\-목적과 구현을 분리  
\-코드가 5~6줄을 넘어가면 추출할 냄새가 나기 시작..!  
\-성능 느려질 걱정은 ㄴㄴ 요즘은 그럴일 거의 없다  
  

```javascript
// from
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  // 세부 사항 출력
  console.log(`고객명: ${invoice.customer}`);
  console.log(`채무액: ${outstanding}`);
}   
--------------------------------------------
// to
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();
  printDetails(outstanding);

  function printDetails(outstanding) {
    console.log(`고객명: ${invoice.customer}`);
    console.log(`채무액: ${outstanding}`);
  }
}
```

## Inline Function(함수 인라인하기)

\-함수 추출하기의 반대 리팩터링  
\-함수 본문이 함수명만큼이나 명확할 경우 굳이 함수를 나누기 보다는 합치기  
  

```javascript
// from
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}
---------------------------------------------
// to
function getRating(driver) {
  return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```

## Extract Variable(변수 추출하기)

\-표현식이 너무 복잡할 경우 단계마다 이름 붙이기(직관적으로 보이도록)  
\-중단점 지정하거나 상태 출력(console.log)을 하며 디버깅 할때도 좋다  
  

```javascript
// from
return order.quantity * order.itemPrice - 
      Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
    Math.min(order.quantity * order.itemPrice * 0.1, 100);
-----------------------------------------------
// to
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```

## Inline Variable(변수 인라인하기)

\-변수명이 표현식이랑 다를 바 없을 때는 그냥 inline 하는게 좋다.  
  

```javascript
// from
const basePrice = anOrder.basePrice;
return (basePrice > 1000);
-------------------------------------------------
// to
return anOrder.basePrice > 1000;
```

## Change Function Declaration(함수 선언 바꾸기)

\-함수명 변경 - 함수명만 보고도 유추되도록 변경! 축약어 멈춰!  
  

```javascript
function circum(radius) {...} -> function circumeference(radius) {...}
```

\-매개변수 객체를 속성으로 바꾸기.  
\-객체가 넘어가야 할 경우도 있지만, **의존성**이 높아지기 때문에..! 필요하지 않다면 속성값만 넘기기

```javascript
// from
function inNewEngland(aCustomer) {
    return ['MA', 'CT', 'ME', 'VT', 'NH', 'RI'].includes(aCustomer.address.state);  
}
const newEnglanders = someCustomers.filter(c => inNewEngland(c));
-------------------------------------------------
// to
function inNewEngland(stateCode) {
    return ['MA', 'CT', 'ME', 'VT', 'NH', 'RI'].includes(stateCode);  
}
const newEnglanders = someCustomers.filter(c => inNewEngland(aCustomer.address.state));
```

## Encapsulate Variable(변수 캡슐화하기)

\-데이터로의 접근을 독점하는 함수를 만듦으로써, 데이터 재구성이라는 어려운 작업을 함수 재구성이라는 더 단순한 작업으로 변환시킨다.  
\-데이터를 변경하고 사용하는 코드를 감시할 수 있는 확실한 통로가 될 수 있다.  
\-데이터 변경 전 검증이나 변경 후 추가 로직을 쉽게 끼워 넣을 수 있다.  
\-따라서, 데이터의 유효범위가 넓을수록 캡슐화 해야 한다. 그래야 자주 사용하는 데이터에 대한 **결합도가 낮아진다.**  
\-객체 데이터를 private하게 유지할 수 있다. (마음대로 변경 방지)  
  

```javascript
// from
const defaultOwner = { firstName: 'Martin', lastName: 'Fowler'};
-------------------------------------------------
// to
const defaultOwnerData = { firstName: 'Martin', lastName: 'Fowler'};
export function defaultOwner()          { return defaultOwnerData; }
export function setDefaultOwner(arg) { defaultOwner = arg; }

// if you wanna prevent object from being changed
export function defaultOwner() { return Object.assign({}, defaultOwnerData); }
```