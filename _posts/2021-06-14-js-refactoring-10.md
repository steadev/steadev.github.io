---
title: "[Refactoring] Refactoring APIs(2)"
author: steadev
date: 2021-06-14 09:49:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 11과의 내용 정리(2)입니다.

## Remove Setting Method(세터 제거하기)

\-무조건 접근자 메스드를 통해서만 필드를 다루려 할 때(심지어 생성자 안에서도) 세터를 제거해서 값이 바뀌면 안 된다는 뜻을 분명히  
\-생성 스크립트(creation script)를 사용해 객체를 생성할 때 설계자는 스크립트가 완료된 뒤로는 그 객체의 필드 일부(혹은 전체)는 변경되지 않으리라 기대한다. 이때 세터를 제거하여 의도를 명확히

```javascript
// from
class Person {
  get name() {...}
  set name(aString) {...}
}
-----------------------
// to
class Person {
  get name() {...}
}
```

## Replace Constructor with Factory Function(생성자를 팩터리 함수로 바꾸기)

\-생성자를 팩터리 함수로 바꾸면, 이름을 바꾸거나 여러 추가적인 로직을 처리하는데 제약이 없다.

```javascript
// from
leadEngineer = new Employee(document.leadEngineer, 'E');
// to
leadEngineer = createEngineer(document.leadEngineer);
```

## Replace Function with Command(함수를 명령으로 바꾸기)

\-함수를 그 함수만을 위한 객체 안으로 캡슐화하면 더 유용해지는 상황이 있다.  
\-보조 연산을 제공할 수 있으며, 수명주기를 더 정밀하게 제어하는 데 필요한 매게변수를 만들어주는 메서드도 제공할 수 있다.  
\-하지만, 유연성은 복잡성을 키우고 얻는 대가임을 잊지 말아야 함..!! 저자가 명령을 선택할 때는 명령보다 더 간단한 방식으로는 얻을 수 없는 기능이 필요할 때 뿐이다.

```javascript
// from
function score(candidate, medicalExam, scoringGuide) {
  let result = 0;
  let healthLevel = 0;
  // skip code
}
-----------------------------------------------------
// to
class Scorer {
  constructor(candidate, medicalExam, scoringGuide) {
    this._candidate = candidate;
    this._medicalExam = medicalExam;
    this._scoringGuide = scoringGuide;
  }

  execute() {
    this._result = 0;
    this._healthLevel = 0;
    // skip code
  }
}
```

## Replace Command with Function(명령을 함수로 바꾸기)

\-로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크니 평범한 함수로 바꿔주는 게 낫다.  
  

```javascript
// from
class ChargeCalculator {
  constructor(customer, usage) {
    this._customer = customer;
    this._usage = usage;
  }

  execute() {
    return this._customer.rate * this._usage;
  }
}
---------------------------------------------
// to
function charge(customer, usage) {
  return customer.rate * usage; 
}
```

## Return Modified Value(수정된 값 반환하기)

\-데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가장 어려운 부분 중 하나다.  
\-그래서 데이터가 수정된다면 그 사실을 명확히 알려주어서, 어느 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는 일이 대단히 중요하다.  
\-이 리팩터링은 값 하나를 계산한다는 분명한 목적이 있는 함수들에 가장 효과적이고, 반대로 값 여러 개를 갱신하는 함수에는 효과적이지 않다.  
  

```javascript
// from
let totalAscent = 0;
calculateascent();

function calculateAscent() {
  for (let i = 1; i< points.length; i++) {
    const verticalChange = points[i].elevation - points[i-1].elevation;
    totalAscent += (verticalChange > 0) ? verticalChange : 0;
  }
}
------------------------------------------------------------------
// to
let totalAscent = calculateascent();

function calculateAscent() {
  let result = 0;
  for (let i = 1; i< points.length; i++) {
    const verticalChange = points[i].elevation - points[i-1].elevation;
    result += (verticalChange > 0) ? verticalChange : 0;
  }
  return result;
}
```

## Replace Error Code with Exception(오류 코드를 예외로 바꾸기)

\-예외를 사용하면 오류 코드를 일일이 검사하거나 오류를 식별해 콜스택 위로 던지는 일을 신경 쓰지 않아도 된다.  
  

```javascript
// from
if (data)
  return new ShippingRules(data);
else
  return -23;
--------------------------
// to
if (data)
  return new ShippingRules(data);
else
  throw new OrderProcessingError(-23);
```

## Replace Exception with Precheck(예외를 사전확인으로 바꾸기)

\-예외가 과용되면 안되고 말 그대로 예외적으로 동작할 때만 쓰여야 한다.  
\-함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는 곳에서 조건을 검사하도록 해야 한다.  
  

```javascript
// from
double getValueForPeriod (int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
      return 0;
  }
}
------------------------------------------
// to
double getValueForPeriod (int periodNumber) {
  return (periodNumber >= values.length) ? 0 : values[periodNumber];
}
```