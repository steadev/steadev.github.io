---
title: "[Refactoring] Dealing with Inheritance(2)"
author: steadev
date: 2021-06-15 08:42:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 12과의 내용 정리(2)입니다.

## Extract Superclass(슈퍼클래스 추출하기)

\- 메서드 올리기 리팩터링과 유사.  
\- 공통 부분을 superclass로 분리하기!  
  

```javascript
// from
class Department {
  get totalAnnualCost() {...}
  get name() {...}
  get headCount() {...}
}

class Employee {
  get annualCost() {...}
  get name() {...}
  get id() {...}
}
-----------------------------
// to
class Party {
  get name() {...}
  get annualCost() {...}
}

class Department extends Party {
  get headCount() {...}
}

class Employee Party {
  get id() {...}
}
```

## Collapse Hierarchy(계층 합치기)

\- 어떤 클래스와 그 부모가 너무 비슷해져서 더는 독립적으로 존재해야 할 이유가 사라지는 경우에 둘을 하나로 합치기!  
  

```javascript
// from
class Employee {...}
class Salesperson extends Employee {...}
--------------------------------------
// to
class Employee {...}
```

## Replace Subclass with Delegate(서브클래스를 위임으로 바꾸기)

\- 상속의 단점은 한 번만 쓸 수 있는 카드라는 것이다. 무언가가 달라져야 하는 이유가 여러 개여도 상속에서는 그중 단 하나의 이유만 선택해 기준으로 삼 을 수밖에 없다. ex) 사람 객체의 동작을 '나이대'와 '소득 수준'에 따라 달리 하고 싶다면 서브클래스는 젊은이와 어르신이 되거나, 혹은 부자와 서민이 되어야 한다. 둘다는 안된다.  
\- 또 다른 단점은 상속은 클래스들의 관계를 아주 긴밀하게 결합한다. 부모를 수정하면 자식들의 기능을 해치기가 쉽기에 조심해야 한다. 그래서 자식들이 슈퍼클래스를 어떻게 상속해 쓰는지를 이해해야 한다.  
\- 위임은 위 두 문제를 모두 해결해준다. 다양한 클래스에 서로 다른 이유로 위임할 수 있다.  
\- 디자인패턴 - State Pattern이나 Strategy Pattern과 유사하다.  
  

```javascript
// from
class Order {
  get daysToShip() {
    return this._warehouse.daysToShip; 
  }
}

class PriorityOrder extends Order {
  get daysToShip() {
    return this._priorityPlan.daysToShip; 
  }
}
---------------------------------------
// to
class Order {
  get daysToShip() {
    return (this._priorityDelegate)
        ? this._priorityDelegate.daysToShip
        : this._warehouse.daysToShip; 
  }
}

class PriorityOrderDelegate {
  get daysToShip() {
    return this._priorityPlan.daysToShip; 
  }
}
```

## Replace Superclass with Delegate(슈퍼클래스를 위임으로 바꾸기)

\- Java의 Stack class가 List를 상속하고 있는데, Stack에 필요하지 않은 연산들이 List를 상속함으로써 그대로 노출되는 문제가 있다.  
\- 위 경우에 Stack에서 List객체를 필드에 저장해두고 필요한 기능만 위임했다면 더 멋졌을 것이다!  
  

```javascript
// from
class List {...}
class Stack extends List {...}
-----------------------------
// to
class Stack {
  constructor() {
    this._storage = new List(); 
  }
}
class List {...}
```