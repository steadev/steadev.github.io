---
title: "[Refactoring] Simplifying Conditional Logic"
author: steadev
date: 2021-05-27 06:26:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 10과의 내용 정리입니다.

## Decompose Conditional(조건문 분해하기)

\-복잡한 조건부 로직은 프로그램을 복잡하게 만드는 가장 흔한 원흉에 속한다.  
\-무슨 일이 일어나는지는 이야기해주지만 '왜' 일어나는지는 제대로 말해주지 않을 때가 많은 것이 문제이다.  
  

```javascript
// from
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
--------------------------------------------------------------------
// to
charge = summer() ? summerCharge() : regularCharge();
```

## Consolidate Conditional Expression(조건식 통합하기)

\-비교하는 조건은 다르지만 그 결과로 수행하는 동작은 똑같은 코드들은 하나로 통합하는게 낫다.  
\-나뉜 조건들을 하나로 통합함으로써 내가 하려는 일이 더 명확해진다. 나눠서 수행해도 결과는 같지만, 읽은 사람은 독립된 검사들이 우연히 함께 나열된 것으로 오해할 수 있다.  
\-하지만, 하나의 검사라고 생각할 수 없는, 진짜로 독립된 검사들이라고 판단되면 **이 리팩터링을 해서는 안 된다**  
  

```javascript
// from
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled < 12) return 0;
if (anEmployee.isPartTime) return 0;
---------------------------------------
// to
if (isNotEligibleForDisability()) return 0;

function isNotEligibleForDisability() {
  return ((anEmployee.seniority < 2)
         || (anEmployee.monthsDisabled < 12)
         || (anEmployee.isPartTime)
}
```

## Replace Nested Conditional with Guard Clauses(중첩 조건문을 보호 구문으로 바꾸기)

\-이 리팩터링의 핵심은 의도를 부각하는 데 있다.  
\-보호 구문은 '이건 이 함수의 핵심이 아니다. 이 일이 일어나면 무언가 조치를 취한 후 함수에서 빠져나온다'라고 이야기 한다.  
  

```javascript
// from
function getPayAmount() {
  let result;
  if (isDead)
    result = deadAmount();
  else {
    if (isSeparated)
      result = separatedAmount();
    else {
      if (isRetired)
        result = retiredAmount();
      else
        result = normalPayAmount();
    }
  }
  return result;
}
------------------------------
// to
function getPayAmount() {
  if (isDead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if (isRetired) return retiredAmount();
  return normalPayAmount();
}
```

## Replace Conditional with Polymorphism(조건부 로직을 다형성으로 바꾸기)

\-타입을 기준으로 분기하는 switch문이 포함된 함수가 **여러 개** 보인다면, case별로 클래스를 하나씩 만들어 **공통 switch 로직의 중복을 없앨 수 있다.** 다형성을 활용하여 어떻게 동작할지를 각 타입이 알아서 처리하도록 하면 된다.  
  

```javascript
// from
switch (bird.type) {
  case '유럽 제비':
    return '보통이다';
  case '아프리카 제비':
       return (bird.numberOfCoconuts > 2) ? '지쳤다' : '보통이다';
  case '노르웨이 파랑 앵무':
    return (bird.voltage > 100) ? '그을렸다' : '예쁘다';
  default:
    return '알 수 없다';
}
...
switch (bird.type) {
  case '유럽 제비':
    return ...;
  case '아프리카 제비':
       return ...;
  case '노르웨이 파랑 앵무':
    return ...;
  default:
    return ...;
}
------------------------------------------------------
// to
switch (bird.type) {
  case '유럽 제비':
    return new EuropeanSwallow(bird);
  case '아프리카 제비':
       return new AfricanSwallow(bird);
  case '노르웨이 파랑 앵무':
    return new NorwegianBlueSwallow(bird);
  default:
    return new Bird(bird);
}
class EuropeanSwallow extends Bird {
  get Plumage() {
    return '보통이다'; 
  }
  ...
}
class AfricanSwallow extends Bird {
  get Plumage() {
       return (bird.numberOfCoconuts > 2) ? '지쳤다' : '보통이다';
  }
  ...
}
class NorwegianBlueSwallow extends Bird {
  get Plumage() {
    return (bird.voltage > 100) ? '그을렸다' : '예쁘다';
  }
  ...
}
```

## Introduce Special Case(특이 케이스 추가하기)

\-데이터 구조의 특정 값을 확인한 후 똑같은 동작을 수행하는 코드가 곳곳에 등장하는 경우, 코드 중복이 발생한다. 특정 값에 대해 똑같이 반응하는 코드가 여러 곳이라면 그 반응들을 한 데로 모으는 게 효율적이다.  
  

```javascript
// from
if (aCustomer === '미확인 고객') customerName = '거주자';
----------------------------------------------------
// to
class UnknownCustomer {
  get name() { return '거주자'; }
}
...
get customer() {
  return (this._customer === '미확인 고객') ? new UnknownCustomer() : this._customer; 
}
...
const customerName = aCustomer.name;
```

## Introduce Assertion(어서션 추가하기)

\-특정 조건이 참일 때만 제대로 동작하는 코드 영역이 있을 수 있다. 이런 가정이 코드에 항상 명시적으로 기술되어 있지는 않아서 알고리즘을 보고 연역해서 알아내야 할 때도 있다. 이럴 때 어서션 이용해서 코드 자체에 삽입해놓는 것이다.  
\-어서션은 항상 참이라고 가정하는 조건부 문장으로, 어서션이 실패했다는 건 프로그래머가 잘못했다는 뜻이다.  
  

```javascript
// from
if (this.discountRate)
  base = base - (this.discountRate * base);
-------------------------------------------
// to
assert(this.discountRate >= 0;
if (this.discountRate)
  base = base - (this.discountRate * base);
```

## Replace Control Flag with Break(제어 플래그를 탈출문으로 바꾸기)

```javascript
// from
for (const p of peaople) {
  if (!found) {
      if (p === '조커') {
      sendAlert();
      fount = true;
    }
  }
}
----------------------------------------
// to
for (const p of peaople) {
  if (p === '조커') {
    sendAlert();
    break;
  }
}
```