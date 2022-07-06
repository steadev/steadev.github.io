---
title: "[Refactoring] Refactoring APIs(1)"
author: steadev
date: 2021-06-04 06:59:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 11과의 내용 정리(1)입니다.

## Separate Query from Modifier(질의 함수와 변경 함수 분리하기)

\-겉보기 부수효과(observable side effect)가 전혀 없이 값을 반환해주는 함수를 추구해야 한다.  
\-이런 함수는 호출 문장의 위치를 호출하는 함수 안 어디로든 옮겨도 되며 테스트하기도 쉽다. (신경 쓸 거리가 매우 적다)  
\-값을 반환하면서 부수효과도 있는 함수를 발견하면 무조건 분리를 시도하자!

```javascript
// from
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
  sendBill();
  return result;
}
---------------------------------------------
// to
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0); 
}
function sendBill() {
  emailGateway.send(formatBill(customer));
}
```

## Parameterize Function(함수 매개변수화하기)

\-둘 이상의 함수의 로직이 아주 비슷하고 단지 리터럴 값만 다르다면, 그 다른 값만 매개변수로 받아 처리하는 함수 하나로 합쳐서 중복을 없앨 수 있다.

```javascript
// from
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1); 
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05); 
}
--------------------------------------------
// to
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor); 
}
```

## Remove Flag Argument(플래그 인수 제거하기)

\-플래그 인수를 사용하면, 호출할 수 이쓴 함수들이 무엇이고 어떻게 호출해야 하는지를 이해하기가 어려워진다.  
\-함수들의 기능 차이가 잘 드러나지 않는다.  
\-사용할 함수를 선택한 후에도 플래그 인수로 어떤 값을 넘겨야 하는지를 또 알아내야 한다.  
\-Boolean 플래그 코드는 읽는 이에게 뜻을 온전히 전달하지 못하기 때문에 더욱 좋지 못하다. 전달한 true의 의미가 대체 뭐람?!

```javascript
// from
function setDimension(name, value) {
  if (name === 'height') {
    this._height = value;
    return;
  }
  if (name === 'width' {
    this._width = value;
    return;
  }
}
-----------------------------------
// to
function setHeight(value) {this._height = value;}
function setWidth(value) {this._width = value;}
```

## Preserve Whole Object(객체 통째로 넘기기)

\-통째로 넘기면 변화에 대응하기 쉽다. 만약 그 함수가 더 다양한 데이터를 사용하도록 바뀌어도 매개변수 목록은 수정할 필요가 없다.  
\-매개변수 목록이 짧아져서 일반적으로는 함수 사용법을 이해하기 쉬워진다.  
\-레코드에 담긴 데이터 중 일부를 받는 함수가 여러 개라면 그 함수들끼리는 같은 데이터를 사용하는 부분들이 있을 것이고, 그 부분의 로직이 중복될 가능성이 크다.  
\-객체로부터 값 몇 개를 얻은 후 그 값들로만 무언가를 하는 로직이 있다면 그 로직을 객체 안으로 집어넣으면 된다.  
\-한 객체가 제공하는 기능 중 항상 똑같은 일부만을 사용하는 코드가 많다면, 그 기능만 따로 묶어서 클래스로 추출할 수 있다.

```javascript
// from
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.widthinRange(low, high))
------------------------------
// to
if (aPlan.widthinRange(aRoom.daysTempRange))
```

## Replace Parameter with Query(매개변수를 질의 함수로 바꾸기) <-> Replace Query with Parameter

\-피호출 함수가 스스로 '쉽게' 결정할 수 있는 값을 매개변수로 건네는 것도 일종의 중복이다.  
\-하지만 매개변수 제거시, 피호출 함수에 원치 않는 의존성이 생긴다면 **이 리팩터링을 하면 안된다.**

```javascript
// from
availableVacation(anEmployee, anEmployee.grade);

function availableVacation(anEmployee, grade) {
  // 연휴 계산...
----------------------------------------
// to
availableVacation(anEmployee);

function availableVacation(anEmployee) {
  const grade = anEmployee.grade;
  // 연휴 계산...
```

## Replace Query with Parameter(질의 함수를 매개변수로 바꾸기) <-> Replace Parameter with Query

\-함수 안에서 전역 변수를 참조한다거나 제거하길 원하는 원소를 참조하는 경우 해당 참조를 매개변수로 바꿔 해결할 수 있다.(책임을 호출자로 옮기는 것)  
\-프로그램의 일부를 순수 함수로 바꿀 수 있으며, 결과적으로 그 부분은 테스트하거나 다루기가 쉬워진다.  
\-하지만 매개변수로 어떤 값을 제공할지를 호출자가 알아내야 한다. 결국 책임소재를 호출자에게 두느냐 함수에 두느냐의 문제로 귀결된다. 정답은 없으며, 프로젝트를 진행하면서 균형점이 이리저리 옮겨질 수 있으니 반대 리팩터링과는 친해져야 한다.

```javascript
// from 
targetTemperature(aPlan)

function targetTemperature(aPlan) {
  currentTemperature = thermostat.currentTemperature;
  // 생략
-----------------------------------------
// to
targetTemperature(aPlan, thermostat.currentTemperature)

function targetTemperature(aPlan, currentTemperature) {
  // 생략
```