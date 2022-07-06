---
title: "[Refactoring] Dealing with Inheritance(1)"
author: steadev
date: 2021-06-15 08:40:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 12과의 내용 정리(1)입니다.

## Pull Up Method(메서드 올리기) <-> Push Down Method

\- child class에서 중복된 함수를 사용한다면, 올리기.  
\- 만약 전체적인 흐름은 같지만 세부적인 내용이 다르다면 [Form Template Method](https://refactoring.com/catalog/formTemplateMethod.html)를 고려해보자.  
\- Form Template Method는 함수를 올리고 세부 내용만 따로 abstract function으로 빼내는 방법!  
  

```javascript
// from
class Employee {...}

class Salesperson extends Employee {
  get name() {...}
}
class Engineer extends Employee {
  get name() {...}
}
----------------------------------
// to
class Employee {
  get name() {...}
}

class Salesperson extends Employee {...}
class Engineer extends Employee {...}
```

## Pull Up Field(필드 올리기) <-> Push Down Field

\- 데이터 중복 선언 없앨 수 있다.  
\- 해당 필드를 사용하는 동작을 서브클래스에서 슈퍼클래스로 옮길 수 있다.  
  

```javascript
// from
class Employee {...} // typescript code
class Salesperson extends Employee {
  private name;
}
class Engineer extends Employee {
  private name;
}
----------------------------------
// to
class Employee {
  protected name;
}
class Salesperson extends Employee {...}
class Engineer extends Employee {...}
```

## Pull Up Constructor Body(생성자 본문 올리기)

\- 중복 제거. Similar with Pull Up Method  
  

```javascript
// from
class Party {...}
class Employee extends Party {
  constructor(name, id, monthlyCost) {
    super();
    this._id = id;
    this._name = name;
    this._monthlyCost = monthlyCost;
  }
}
-----------------------------------
// to
class Party {
  constructor(name) {
    this._name = name;
  }
}
class Employee extends Party {
  constructor(name, id, monthlyCost) {
    super(name);
    this._id = id;
    this._monthlyCost = monthlyCost;
  }
}
```

## Push Down Method(메서드 내리기) <-> Pull Up Method

\- 특정 서브클래스 하나(혹은 소수)와만 관련된 메서드는 슈퍼클래스에서 제거하고 해당 서브클래스(들)에 추가하는 편이 깔끔하다.  
  

```javascript
// from
class Employee {
  get name() {...}
}

class Salesperson extends Employee {...}
class Engineer extends Employee {...}
----------------------------------
// to
class Employee {...}

class Salesperson extends Employee {...}
class Engineer extends Employee {
  get name() {...}
}
```

## Push Down Field(필드 내리기) <-> Pull Up Field

\- 서브 클래스 하나(혹은 소수)에서만 사용하는 필드는 해당 서브클래스(들)로 옮긴다.  
  

```javascript
// from
class Employee { // typescript code
  protected name;
}

class Salesperson extends Employee {...}
class Engineer extends Employee {...}
----------------------------------
// to
class Employee {...}
class Salesperson extends Employee {...}
class Engineer extends Employee {
  private name;
}
```

## Replace Type Code with Subclasses(타입 코드를 서브클래스로 바꾸기) <-> Remove Subclass

\- 타입 코드에 따라 동작이 달라져야 하는 함수가 여러 개일 때 특히 유용하다.  
\- [Replace Conditional with Polymorphism](https://steadev.tistory.com/80) 참고  
  

```javascript
// from
function createEmployee(name, type) {
  return new Employee(name, type); 
}
-------------------------
// to
function createEmployee(name, type) {
  switch (type) {
    case 'engineer':    return new Engineer(name);
    case 'salesperson': return new Salesperson(name);
    case 'manager':     return new Manager(name);
  }
}
```

## Remove Subclass(서브클래스 제거하기) <-> Replace Type Code with Subclasses

\- 더이상 서브클래스의 역할을 제대로 하지 못한다면 과감하게 없애고 슈퍼클래스의 필드로 대체하기  
  

```javascript
// from
class Person {
  get genderCode() {return 'X';} 
}
class Male extends Person {
  get genderCode() {return 'M';} 
}
class Female extends Person {
  get genderCode() {return 'F';} 
}
-------------------------------
// to
class Person {
  get genderCode() {
    return this._genderCode; 
  }
}
```