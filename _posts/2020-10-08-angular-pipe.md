---
title: "[Angular] Pipe"
author: steadev
date: 2020-10-08 00:16:00 +0900
categories: [Angular]
tags: [angular, Template reference variables]
---


view에 맞게 데이터를 변경할 때,

단순한? 방법은 component ts에서 view용 변수를 하나 선언하고

원하는대로 데이터를 변형해서 초기화하는 방법일 것입니다. 

하지만 앵귤러에서의 Pipe를 이용하면 굳이 컴포넌트를 거치지 않고

바로 뷰에서 원하는대로 데이터를 변형시킬 수 있습니다. 

아래 예시를 보면,

위에는 component에서 template용 변수를 생성한 경우고

아래는 pipe를 이용한 코드입니다. 엄청 간단해지는!!

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-uppercase',
  template: `<p>angular's uppercase is {{ angularUpper }}</p>`
})
export class HeroBirthdayComponent {
  angular = 'angular';
  angularUpper = angular.toUpperCase();
}
```

```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-uppercase',
  template: `<p>angular's uppercase is {{ angular | uppercase }}</p>`
})
export class HeroBirthdayComponent {
  angular = 'angular';
}
```

하지만 주의해야할 점이 있습니다. 

아래 코드의 경우는 (from. angular 공식문서) object array를 필터링 해주는 pipe입니다. 

이때 문제는, heroes 내용이 바뀌어도 pipe가 그 변화를 감지하지 못한다는 것입니다.

object의 참조가 바뀌지 않는 이상, 그 안의 값이 바뀐다 해도 화면엔 이전 상태로만 보이게 됩니다.

이는 pipe가 pure change만 감지하기 때문입니다. string, number, boolean, symbol 같은 기본 자료형의 값이 변경되거나 객체의 참조가 변경되는 것만 감지하는 것입니다. 

```javascript
// Pipe
import { Pipe, PipeTransform } from '@angular/core';

import { Flyer } from './heroes';

@Pipe({ name: 'flyingHeroes' })
export class FlyingHeroesPipe implements PipeTransform {
  transform(allHeroes: Flyer[]) {
    return allHeroes.filter(hero => hero.canFly);
  }
}

// Template
<div *ngFor="let hero of (heroes | flyingHeroes)">
  {{hero.name}}
</div>
```

그래서 이때는 아래와 같이 **pure: false** 옵션을 주어 해결할 수 있습니다. 

```javascript
// Pipe
import { Pipe, PipeTransform } from '@angular/core';

import { Flyer } from './heroes';

@Pipe({ 
  name: 'flyingHeroes',
  pure: false    
})
export class FlyingHeroesPipe implements PipeTransform {
  transform(allHeroes: Flyer[]) {
    return allHeroes.filter(hero => hero.canFly);
  }
}

// Template
<div *ngFor="let hero of (heroes | flyingHeroes)">
  {{hero.name}}
</div>
```

하지만, 이는 객체의 모든 변화에 반응하며 filter를 수행하기 때문에, 만약 pipe에서 복잡하고 오래걸리는 로직이 수행되는 경우라면 성능에 지대한 영향을 끼칠 수 있으므로 주의해야 합니다.

이하는 angular에서 기본적으로 제공하는 자주 사용되는 pipe 입니다.

-   [DatePipe](https://angular.io/api/common/DatePipe)
-   [UpperCasePipe](https://angular.io/api/common/UpperCasePipe)
-   [LowerCasePipe](https://angular.io/api/common/LowerCasePipe)
-   [CurrencyPipe](https://angular.io/api/common/CurrencyPipe)
-   [DecimalPipe](https://angular.io/api/common/DecimalPipe)
-   [PercentPipe](https://angular.io/api/common/PercentPipe)

여기에 추가로 매우 유용한 **async pipe**가 있지만

이는 observable과 뗄수 없는 사이이므로 여기서는 다루지 않겠습니다.