---
title: "[Refactoring] Moving Features(2)"
author: steadev
date: 2021-05-24 06:10:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 8과의 내용 정리(2)입니다.

## Replace Inline Code with Function Call(인라인 코드를 함수 호출로 바꾸기)

\-함수는 여러 동작을 하나로 묶어준다.  
\-함수의 이름이 코드의 동작 방식보다는 목적을 말해주기 때문에 코드를 이해하기 쉬워진다.  
\-함수는 중복을 없애는 데도 효과적이다. 그리고 동작을 변경할 때도 한 곳만 수정하면 된다.  
  

```javascript
// from
let appliesToMass = false;
for (const s of states) {
  if (s === 'MA') appliesToMass = true;
}
------------------------------------
// to 
let appliesToMass = states.includes('MA');
```

## Slide Statements(문장 슬라이드하기)

\-관련된 코드들을 가까이 뭉쳐놓기  
  

```javascript
// from
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
-------------------------------------
// to
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```

## Split Loop(반복문 쪼개기)

\-반복문 하나에서 두 가지 일을 수행하는 경우, 두 가지 일 모두를 잘 이해하고 진행해야 하는 단점이 있다.  
\-반복문을 두 번 실행해야 하는게 병목이라 밝혀지면 그때 다시 합치면 된다. 하지만, 대부분 성능에 저하는 없다.  
  

```javascript
// from
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}
averageAge = averageAge / people.length;
---------------------------------------
// to
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
  averageAge += p.age;
}
averageAge = averageAge / people.length;
```

## Replace Loop with Pipeline(반복문을 파이프라인으로 바꾸기

\-컬렉션 파이프라인을 이용하면 처리 과정을 일련의 연산으로 표현할 수 있다.  
\-대표적으로 map, filter를 활용할 수 있다.  
\-논리를 파이프라인으로 표현하면 이해하기 훨씬 쉬워진다. 객체가 파이프라인을 따라 흐르며 어떻게 처리되었는지 읽을 수 있기 때문!  
  

```javascript
// from
const names = [];
for (const i of input) {
  if (i.job === 'programmer')
    names.push(i.name);
}
----------------------------
// to
const names = input
  .filter(i => i.job === 'programmer')
  .map(i => i.name);
```

## Remove Dead Code(죽은 코드 제거하기)

\-사용되지 않는 코드가 있다면 소프트웨어의 동작을 이애하는 데 커다란 걸림돌이 될 수도 있다.  
  

```javascript
// from
if (false) {
  doSomethingThaeUsedToMatter(); 
}
---------------------------
// to
```