---
title: "[Refactoring] A First Set Of Refactorings(2)"
author: steadev
date: 2021-05-11 06:59:00 +0900
categories: [Javascript]
tags: [javascript, refactoring]
---


마틴파울러의 Refactoring 2판 6과의 내용 정리(2)입니다.

## Rename Variable(변수 이름 바꾸기)

\-명확한 프로그래밍의 핵심은 이름짓기..!

```javascript
let a = height * width; --> let area = height * width;
```

## Introduce Parameter Object(매개변수 객체 만들기)

\-데이터 뭉치를 데이터 구조로 묶으면 데이터 사이의 관계가 명확해진다  
\-매개변수 수가 줄어든다  
\-같은 데이터 구조를 사용하는 모든 함수가 원소를 참조할 때 항상 같은 이름 쓰기에 일관성도 높여준다  
\-나중에 동작까지 함께 묶기 좋도록 class로 만드는것 추천

```javascript
// from
function readingOutsideRange(station, min, max) {
  return station.readings
      .filter(r => r.temp < min || r.temp.max)
}

alerts = readingOutsideRange(station, 
                             operationgPlan.temperatureFloor,
                             operationPlan.temperatureCeiling);
--------------------------------------------------
// to
class NumberRange {
  constructor(min, max) {
    this._data = {min: min, max: max};
  }

  get min() {return this._data.min;}
  get max() {return this._data.max;}

  contains(arg) {return (arg >= this.min && arg <= this.max);}
}

function readingOutsideRange(station, range) {
  return station.readings
      .filter(r => !range.contains(r.temp))
}

const range = new NumberRange(operationgPlan.temperatureFloor,
                              operationPlan.temperatureCeiling);

alerts = readingOutsideRange(station, range);
```

## Combine Functions into Class(여러 함수를 클래스로 묶기)

\-함수들이 공유하는 공통 환경을 더 명확하게 표현할 수 있다.  
\-각 함수에 전달되는 인수를 줄여서 객체 안에서의 함수 호출을 간결하게 만들수 있다.  
\-객체를 시스템의 다른 부분에 전달하기 위한 참조를 제공할 수 있다.  
  

```javascript
// from
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}
-----------------------------------------------
// to
class Reading {
    base() {...}
    taxableCharge() {...}
    calculateBaseCharge() {...} 
}
```

## Combine Functions into Transform(여러 함수를 변환 함수로 묶기)

\-Combine Functions into Class 방법과는 유사할 수 있지만, 원본데이터가 코드 안에서 갱신될 경우는 클래스로 묶는 편이 훨씬 낫다. 변환 함수로 묶으면 가공한 데이터를 새로운 레코드에 저장하므로, 원본 데이터가 수정되면 일관성이 깨질 수 있기 때문.  
\-여러 함수를 묶는 이유는 도출 로직이 중복되는 것을 피하기 위해서다.  
\-원본데이터를 바꾸는 함수들을 한군데 묶어 변경된 원본데이터를 반환해준다.  
  

```javascript
// from
function base(aReading) {...}
function taxableCharge(aReaging) {...}
-----------------------------------------------
// to
function enrichReading(argReading) {
    const aReading = _.cloneDeep(argReading);
      aReading.baseCharge = base(aReading);
    aReading.taxableCharge = taxableCharge(aReaging);
      return aReading;
}
```

## Split Phase(단계 쪼개기)

\-서로 다른 두 대상을 한꺼번에 다루는 코드는 분리! 하나에만 집중!  
  

```javascript
// from
const orderData = orderString.split(/\s+/);
const productPrice = priceList(orderData[0].split("-")[1]];
const orderPrice = parseInt(orderData[1]) * productPrice;
---------------------------------------------------
// to
const orderRecord = parseOrder(order);
const orderPrice = price(orderRecord, priceList);

function parseOrder(aString) {
    const values = aString.split(/\s+/);
      return ({
        productID: values[0].split("-")[1],
          quantity: parseInt(values[1])
    })
}

function price(order, priceList) {
     return order.quantity * priceList[order.productID]; 
}
```

'

```javascript
// from
function priceOrder(product, quantity, shippingMethod) {
    const basePrice = product.basePrice * quantity;
      const discount = Math.max(quantity - product.discountThreshold, 0)
            * product.basePrice * product.discountRate;
      const shippingPerCase = (basePrice > shippingMethod.discountThreshold)
            ? shippingMethod.discountedFee : shippingMethod.feePerCase;
      const shippingCost = quantity * shippingPerCase;
      const price = basePrice - discount + shippingCost;
      return price;
}
--------------------------------------------------
// to
function priceOrder(product, quantity, shippingMethod) {
    const priceData = calculatePricingData(product, quantity);
      return applyShipping(priceData, shippingMethod);
}

function calculatePricingData(product, quantity) {
      const basePrice = product.basePrice * quantity;
      const discount = Math.max(quantity - product.discountThreshold, 0)
            * product.basePrice * product.discountRate;
    return {basePrice: basePrice, quantity: quantity, discount: discount};
}

function applyShipping(priceData, shippingMethod) {
     const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold)
            ? shippingMethod.discountedFee : shippingMethod.feePerCase;
      const shippingCost = priceData.quantity * shippingPerCase;
      return priceData.basePrice - priceData.discount + shippingCost;
}
```