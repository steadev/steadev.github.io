---
title: "Dependency Injection"
author: steadev
date: 2022-07-27 23:33:00 +0900
categories: [Javascript]
tags: [javascript, angular, DI]
---


## Dependency Injection이란?

Dependency Injection 말은 많이 들어보고 실제로 엄청 쓰고 있긴 한데

이게 뭔 말일까?? 의존성 주입? 의존성은 뭐고 주입은 또 뭐지?

일단 정의를 한번 보겠습니다.

> Dependency injection is a technique in which an object receives other objects that it depends on. (wiki)
> 

위키피디아에서의 정의로, 의존성 주입은 객체가 의존하는 다른 객체를 받는 기술입니다.

...? 의존하는 다른 객체를 받는다는게 뭔 소리일까?

> Dependency injection, or DI, is a design pattern in which a class requests dependencies from external sources rather than creating them. (Angular Docs)
> 

앵귤러에서 말하는 DI는 내부에서 직접 의존성을 만들기보다는 외부에 의존성들을 요청한다는 좀더 직관적인(?) 설명을 하고 있습니다.

일단 Dependency(의존)에 대해서 알아보겠습니다.

### Dependency

말 그대로 한 객체가 다른 객체에 의존한다는 말!

```javascript
class A {
	private b = new B();
}

```

위 코드를 보면 A는 B가 있어야 기능을 수행할 수가 있습니다.

이를 A 클래스가 B 클래스에 의존성을 가진다고 표현합니다.

위 케이스를 더 정확히 표현하면 Object Dependency라고 합니다.

### 그렇다면 Injection이란 뭘까?

외부에서 값이나 객체를 생성해서 넣어주는 것을 의미합니다. 바로 예시를 보면 알 수 있다!

```javascript
class A {
  constructor(private n: number) {}
}
...
const a = new A(10);

```

그렇다면, Dependency Injection이란 의존성을 가진 객체를 주입하는 것임을 알 수 있습니다.
다시 처음 DI 정의들을 보면 이해가 갑니다.

그리고 추가적으로 ioc라는 개념이 있습니다.

### IoC(Inversion of Control)

말그대로 컨트롤이 거꾸로 됐다는 것으로, 제어의 역전이라고 합니다..

class A에서 직접 B를 만들어 사용하는 것은 본인이 직접 다 하는 것이지만,
그와 반대로 DI는 알아서 만들어오라고 요청하는 것이기에 IoC라고 합니다.

비유하자면,
김장하는데 속재료들을 직접 만들어 쓰는게 아니라,
속재료 내놔! 김치 먹고싶으면.
이런 느낌일 것 같습니다.

그런데, Angular에서 DI를 쓰는거에 의문이 하나 생깁니다.

### 내가 외부에서 객체를 생성해서 직접 넣어주지도 않았는데 객체가 주입될까?

@Injectable() 데코레이터를 통해, 혹은 providers에 등록이 되어 있다면
프레임워크가 알아서 생성해서 주입 시켜준다.
React는 프레임워크가 아니기에 수동으로 처리해줘야 합니다.

이를 **IoC Container**라고 한다. 제어권을 컨테이너(framework)가 가져가는 것.
프레임워크가 객체의 생성을 책임지고, 의존성을 관리해줍니다.

### @Injectable()의 providedIn에 'root'말고도 다른 옵션들이 있는데 차이점은?

- root	- application-level에서 singleton 인스턴스가 주입된다.
- platform - 모든 application에서 공유되는 singleton 인스턴스가 주입된다.
- any - 모든 lazy loaded 모듈마다 새로운 인스턴스가 주입된다. 대신 앱 실행 전에 로드된 모듈들에는 하나의 인스턴스가 공유된다.
- null - 아무 값도 넣지 않았을 때이고, scope이 없기 때문에 직접 providers에 넣어줘야 한다.

### Dependency Injection이 뭔지는 알겠는데 그러면 왜 사용하는거지?

- 의존 관계 설정이 [컴파일](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8C%8C%EC%9D%BC)시가 아닌 [실행](https://ko.wikipedia.org/wiki/%EC%8B%A4%ED%96%89)시에 이루어져 모듈들간의 [결합도](https://ko.wikipedia.org/wiki/%EA%B2%B0%ED%95%A9%EB%8F%84)를 낮출 수 있다.
- [코드 재사용](https://ko.wikipedia.org/wiki/%EC%BD%94%EB%93%9C_%EC%9E%AC%EC%82%AC%EC%9A%A9)을 높여서 작성된 모듈을 여러 곳에서 소스코드의 수정 없이 사용할 수 있다.
- [모의 객체](https://ko.wikipedia.org/wiki/%EB%AA%A8%EC%9D%98_%EA%B0%9D%EC%B2%B4) 등을 이용한 [단위 테스트](https://ko.wikipedia.org/wiki/%EC%9C%A0%EB%8B%9B_%ED%85%8C%EC%8A%A4%ED%8A%B8)의 편의성을 높여준다.

라고 위키에 쓰여있습니다. 

그렇지만 확 와닿지는 않는...

아래 케이스를 보면 어떤 점이 좋은지 파악하기 쉬울 것 같습니다.

```javascript
abstract class Bird {...}

class Sparrow extends Bird {...}
class Pigeon extends Bird {...}

class DI {
	constructor() {
		// case 1
		this.bird = new Sparrow();
		// case 2
		this.bird = new Pigeon();
	}
}

------------ to

abstract class Bird {...}

class Sparrow extends Bird {...}
class Pigeon extends Bird {...}

class DI {
	constructor(private bird: Bird) {
	}
}
```

이렇듯 객체 생성을 직접 하지 않고 떠넘김(?)으로 결합도를 낮출 수 있고 코드의 양도 많이 줄어들게 됩니다.





ref)

- [https://medium.com/@jang.wangsu/di-dependency-injection-이란-1b12fdefec4f](https://medium.com/@jang.wangsu/di-dependency-injection-%EC%9D%B4%EB%9E%80-1b12fdefec4f)
- [https://ko.wikipedia.org/wiki/의존성_주입](https://ko.wikipedia.org/wiki/%EC%9D%98%EC%A1%B4%EC%84%B1_%EC%A3%BC%EC%9E%85)
- [https://edykim.com/ko/post/understanding-angular-dependency-injection-inject-injectable-tokens-and-providers/](https://edykim.com/ko/post/understanding-angular-dependency-injection-inject-injectable-tokens-and-providers/)
- [https://medium.com/@jang.wangsu/di-inversion-of-control-container-란-12ecd70ac7ea](https://medium.com/@jang.wangsu/di-inversion-of-control-container-%EB%9E%80-12ecd70ac7ea)
- [https://velog.io/@jojo_devstory/DIDependency-Injection에-대해-알아보자](https://velog.io/@jojo_devstory/DIDependency-Injection%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90)
- [https://www.youtube.com/watch?v=_GjfTDECWOg](https://www.youtube.com/watch?v=_GjfTDECWOg)
- [https://velog.io/@wlsdud2194/what-is-di](https://velog.io/@wlsdud2194/what-is-di)