---
title: "[Angular] Lazy-loading"
author: steadev
date: 2020-06-20 12:33:00 +0900
categories: [Angular]
tags: [angular, lazy loading]
---


SPA의 대표적인 단점이 초기 구동시간이 오래걸린다는 점입니다. 

이를 보완하기 위한 방법이 Lazy Loading입니다.

기능모듈들을 초기에 로드하지 않고 요청이 있을 때 로드하는 방법입니다. 

아래는 앵귤러 공식 사이트에서의 예제를 작성한 코드입니다.

[https://github.com/steadev/angular-lazyloading-testing-app](https://github.com/steadev/angular-lazyloading-testing-app)

Lazy loading을 하는 방법은 라우팅 객체의 loadChildren 안에서 모듈을 import 합니다. 

app routing module에서 아래와 같이 customers, orders의 라우팅 설정을 해놓고 

customer 버튼을 눌렀을 시, localhost:4200/customers로 라우팅이 되면서 customer 모듈을 import 하는 방식입니다.

```
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

const routes: Routes = [
  {
    path: 'customers',
    loadChildren: () =>
      import('./customers/customers.module').then((m) => m.CustomersModule),
  },
  {
    path: 'orders',
    loadChildren: () =>
      import('./orders/orders.module').then((m) => m.OrdersModule),
  },
  {
    path: '',
    redirectTo: '',
    pathMatch: 'full',
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

우선 첫 구동시의 네트워크 상태 입니다. 

<img src="https://steadev.github.io/assets/images/angular/2020-06-20-1.png" />

차례대로 차이점을 봐보도록 하겠습니다.

**1\. 기능모듈( Customers, Orders )을 전부 초기 구동때 로드**

<img src="https://steadev.github.io/assets/images/angular/2020-06-20-2.png" />

**2\. 기능모듈 요청시에 로드** 

<img src="https://steadev.github.io/assets/images/angular/2020-06-20-3.png" />

1번의 경우는 모듈이 이미 전부 로드된 상태이기 때문에 customers 버튼을 눌러도 네트워크에 아무 반응이 없습니다. 새로 로드할 필요가 없기 때문이죠.

2번의 경우는 처음 버튼을 눌렀을 경우에 Customers 모듈을 가져오고 그 이후에는 이미 가져왔기에 눌러도 네트워크에는 반응이 없습니다. 

ref) [https://angular.io/guide/lazy-loading-ngmodules](https://angular.io/guide/lazy-loading-ngmodules)