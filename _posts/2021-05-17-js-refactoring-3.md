---
title: "[Refactoring] Encapsulation(1)"
author: steadev
date: 2021-05-17 05:38:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 7과의 내용 정리(1)입니다.

## Encapsulate Record(레코드 캡슐화하기)

\-레코드의 단점은 계산해서 얻을 수 있는 값과 그렇지 않은 값을 명확히 구분해 저장해야 하는 점이다.  
\-때문에 가변 데이터를 저장하는 용도는 객체가 더 나을 수 있다. 값과 계산과정을 숨기고 메소드로 제공할 수 있다.

```javascript
// from
organization = {name: "애크미 구스베리", country: "GB"}
--------------------------------------------------------
// to
class Organization {
    constructor(data) {
         this._name = data.name;
          this._country = data.country;
    }
     get name()      {return this._name;}
      set name(arg) {this._name = arg;}
      get country()     {return this._country;}
      set country(arg) {this._country = arg;}
}
```

## Encapsulate Collection(컬렉션 캡슐화하기)

\-게터가 컬렉션 자체를 반환하도록 한다면, 그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀌어버릴 수 있다.  
\-add(), remove()와 같은 컬렉션 변경자 메서드를 만들어 해결할 수 있다.  
\-게터를 제공하되 내부 컬렉션의 복제본을 반환하여 예상치 못하게 바뀌는 문제를 해결할 수 있다.

```javascript
// from
class Person {
  get courses() {return this._courses;}
  set courses(aList) {this._courses = aList;}
}
---------------------------------------------
// to
class Person {
  get courses() {return this._courses.slice();}
  addCourse(aCourse) { ... }
  removeCourse(aCourse) { ... }
}
```

## Replace Primitive with Object(기본형을 객체로 바꾸기)

\-처음에는 숫자, 문자열 같은 간단한 데이터 항목으로 표현할 때가 많지만, 이후에 이 정보들이 더 이상 간단하지 않게 변할 수 있다.  
\-ex) 전화번호를 문자열로 표현 -> 포매팅이나 지역 코드 추출 같은 특별한 동작 추가

```javascript
// from
orders.filter(o => 'high' === o.priority
                  || 'rush' === o.priority);
-------------------------------------------
// to
orders.filter(o => o.priority.higherThan(new Priority('normal')));
```

## Replace Temp with Query(임시 변수를 질의 함수로 바꾸기)

\-비슷한 계산을 수행하는 다른 함수에서도 사용할 수 있어 코드 중복이 줄어들 수 있다.  
\-이번 리팩터링은 추출할 메서드들에 공유 컨텍스트를 제공하기에, 클래스 안에서 적용할 때가 효과가 가장 크다.

```javascript
// from
const basePrice = this._quantity * this._itemPrice;
return (basePrice > 1000) ? basePrice * 0.95 : basePrice * 0.98;
----------------------------------------------------------------
// to
get basePrice() {this._quantity * this._itemPrice;}
...
return (this.basePrice > 1000) ? this.basePrice * 0.95 : this.basePrice * 0.98;
```