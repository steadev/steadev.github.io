---
title: "[Refactoring] Encapsulation(2)"
author: steadev
date: 2021-05-18 05:08:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 7과의 내용 정리(2)입니다.

## Extract Class(클래스 추출하기)

\-메서드와 데이터가 너무 많은 클래스는 이해하기가 쉽지 않으니 잘 살펴보고 적절히 분리하는 것이 좋다.  
\-특히 일부 데이터와 메서드를 따로 묶을 수 있다면 어서 분리하라는 신호!  
\-함께 변경되는 일이 많거나 서로 의존하는 데이터들도 분리!  
  

```javascript
// from
class Person {
  ...
  get officeAreaCode() {return this._officeAreaCode;}
  get officeNumber()   {return this._officeNumber;}
}
------------------------------------------------------
// to
class Person {
  ...
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  get officeNumber()   {return this._telephoneNumber.number;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  get number()   {return this._number;}
}
```

## Inline Class(클래스 인라인하기)

\-클래스 추출하기의 반대 리팩터링으로, 더 이상 제 역할을 못하는 분리된 클래스는 인라인해버린다.  
\-클래스 추출하고 나니 특정 클래스에 남은 역할이 거의 없을 때 이런 현상이 자주 생김  
  

```javascript
// from
class Person {
  ...
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  get officeNumber()   {return this._telephoneNumber.number;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  get number()   {return this._number;}
}
------------------------------------------------------
// to
class Person {
  ...
  get officeAreaCode() {return this._officeAreaCode;}
  get officeNumber()   {return this._officeNumber;}
}
```

## Hide Delegate(위임 숨기기)

\-서버 객체의 필드가 가리키는 객체(위임 객체)의 메서드를 호출하려면 클라이언트는 이 위임 객체를 알아야 한다. 이럴 경우 위임 객체의 인터페이스가 바뀌면 이 인터페이스를 사용하는 모든 클라이언트가 코드를 수정해야 한다.  
\-따라서, 서버 자체에 위임 메서드를 만들어서 위임 객체의 존재를 숨기면 된다!  
  

```javascript
// from
manager = aPerson.department.manager;
------------------------------------
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;} 
}
```

## Remove Middle Man(중재자 제거하기)

\-위임 숨기기의 반대 리팩터링  
\-클라이언트가 위임 객체의 또 다른 기능을 사용하고 싶을 때마다 서버에 위임 메서드를 추가해야 하는데, 이러다보면 단순히 전달만 하는 위임 메서드들이 점점 많아진다.  
\- 이렇게 되면, 서버 클래스는 그저 중개자 역할로 전락하여 차라리 클라이언트가 위임 객체를 직접 호출하는게 나을 수 있다.  
  

```javascript
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;} 
}
------------------------------------
manager = aPerson.department.manager;
```

## Substitute Algorithm(알고리즘 교체하기)

\-더 간명한 방법을 찾아낸다면, 교체하기  
\-거대하고 복잡한 알고리즘의 경우 교체하기란 상당히 어려우니, 알고리즘을 간소화하는 작업부터 해야 작업이 쉬워진다.  
  

```javascript
// from
function foundPerson(people) {
  for (let i = 0; i < people.length; i++) {
    if (people[i] === 'Don') {
      return 'Don';
    }
    if (people[i] === 'John') {
      return 'John';
    }
    if (people[i] === 'Kent') {
      return 'Kent';
    }
  }
  return '';
}
----------------------------------------
// to
function foundPerson(people) {
  const candidates = ['Don', 'John', 'Kent'];
  return people.find(p => candidates.includes(p)) || '';
}
```