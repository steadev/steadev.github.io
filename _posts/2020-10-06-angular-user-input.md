---
title: "[Angular] User Input - 사용자 입력"
author: steadev
date: 2020-10-06 18:35:00 +0900
categories: [Angular]
tags: [angular, Template reference variables]
---


앵귤러의 사용자 입력에서 불필요한 코드를 제거하고 디펜던시를 제거할 수 있는 부분을 살펴보는 글입니다.

아래 코드들과 정보는 [앵귤러 공식 문서](https://angular.io/guide/user-input)를 참조했습니다.

angular에서 사용자 입력(input, click 등)을 받을 때 아래 코드처럼 $event를 통째로 넘기고 그 이벤트에 맞게 타입을 정의해서 사용할 수 있습니다.

아래는 **keyup**이라는 이벤트가 발생했기에 onKey() 함수에서의 파라미터 타입을 KeyboardEvent로 명시하여 사용합니다. 

```javascript
@Component({
  selector: 'app-keyup',
  template: `
    <input (keyup)="onKey($event)">
    <p>{{values}}</p>
  `
})
export class KeyUpComponent_v1 {
  values = '';


  onKey(event: KeyboardEvent) { // with type info
    this.values += (event.target as HTMLInputElement).value + ' | ';
  }
}
```

하지만!! 이는 너무 많은 정보를 컴포넌트로 넘기는 것은 물론, html에서 어떤 이벤트가 발생했는지를 알아야만 넘어온 이벤트 데이터를 활용할 수 있다는 치명적인 단점이 있습니다. 즉, html과 component가 분리되지 않고 의존성을 가지게 되는 문제가 있는 것입니다.

그래서 angular 공식 문서에서는 아래의 방식을 추천합니다. 

```javascript
@Component({
  selector: 'app-loop-back',
  template: `
    <input #box (keyup)="0">
    <p>{{box.value}}</p>
  `
})
export class LoopbackComponent { }
```

**Template reference variables(#box)**를 통해 component에는 필요한 정보를 주는 방식입니다.

아래는 value 정보만 컴포넌트로 넘겨 활용하는 예시 코드입니다.

```javascript
@Component({
  selector: 'app-key-up2',
  template: `
    <input #box (keyup)="onKey(box.value)">
    <p>{{values}}</p>
  `
})
export class KeyUpComponent_v2 {
  values = '';
  onKey(value: string) {
    this.values += value + ' | ';
  }
}
```

또한, form을 submit할때에도 비슷하게 활용할 수 있습니다.

```html
<form #itemForm="ngForm" (ngSubmit)="onSubmit(itemForm)">
  <label for="name"
    >Name <input class="form-control" name="name" ngModel required />
  </label>
  <button type="submit">Submit</button>
</form>

<div [hidden]="!itemForm.form.valid">
  <p>{{ submitMessage }}</p>
</div>
```

컴포넌트간의 dependency에 집중해왔었는데 html은 별로 고려하지 않았었다는 생각이 많이 드네요..!