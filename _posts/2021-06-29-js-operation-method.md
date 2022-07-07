---
title: "[Refactoring] Dealing with Inheritance(2)"
author: steadev
date: 2021-06-29 01:37:00 +0900
categories: [Javascript]
tags: [javascript]
---


**\[Javascript는 어떤 원리로 돌아가는 걸까?\]**

싱글 스레드? 콜스택? 말은 많이 들어 봤지만, 막상 설명하자니 입이 안떨어졌습니다..

<img src="https://steadev.github.io/assets/images/js/2021-06-29-1.png" />

자바스크립트는 웹 브라우저에 내장된 엔진으로 돌아가게 되는데, 엔진은 이와 같이 구성되어있습니다.

> Memory Heap - 변수, 함수 등의 메모리 할당이 일어나는 곳  
> Call Stack - 호출 스택이 쌓이는 곳

그리고 대표적인 자바스크립트 엔진은 구글 크롬에서 사용중인 V8!  
(8기통 엔진을 의미하는 명칭)

**\[그렇다면 v8엔진에서는 무슨 일을 하는 걸까?\]**

<img src="https://steadev.github.io/assets/images/js/2021-06-29-2.png" />

1.  자바스크립트 소스코드를 가져와 Parser에게 넘긴다.
2.  Parser는 파싱을 통해 AST(Abstract Syntax Tree)로 변환시킨다.
3.  AST를 인터프리터를 통해 byte code로 변환(Ignition)한다.
4.  bytecode를 실행함으로써 실제 작동하게 된다.
5.  그 중 자주 사용되는 코드는 TruboFan으로 보내진다.
6.  TruboFan은 이 코드를 Optimized Machine Code로 컴파일해놓고 사용한다.

**AST**

```javascript
function hello (name){ 
    return 'Hello,' + name; 
} 
--------------------------------- 
// 구조화 된 AST 
{ 
    type: 'FunctionDeclaration', 
    name: 'hello',
    arguments: [{
        type: 'Variable', 
        name: 'name' 
    }],
    ...
}
```

  
**Ignition**

Ignition은 자바스크립트 코드를 바이트 코드(ByteCode)로 변환하는 인터프리터이다. 원본 소스 코드보다 컴퓨터가 해석하기 쉬운 바이트 코드로 변환하여, 수시로 코드를 파싱(Parsing)하는 작업을 최소화하고 코드의 양도 줄임으로써 메모리 공간도 효율적으로 관리할 수 있게 된다.  
  
**TurboFan**

TurboFan은 V8 v5.9 버전 이전에 사용되었던 Crankshaft 컴파일러를 완전히 대체한 최적화 담당 컴파일러이다. TurboFan은 바이트 코드로 수시로 변환하는 과정을 최소화하기 위해 사용된다.

V8은 런타임 중에 Profiler라는 친구에게 함수나 변수들의 호출 빈도와 같은 데이터를 모으라고 시킨다. 이렇게 모인 데이터를 이용하여 TurboFan이 자기 기준에 맞는 코드를 가져와서 최적화를 시킨다.  
  
  
**\[엔진에서 언어 번역해서 실행시켜주는건 알겠는데, 어떻게 싱글스레드라면서 멀티스레드처럼 동작하는 거지?\]**

<img src="https://steadev.github.io/assets/images/js/2021-06-29-3.png" />

web APIs 에서 비동기 요청들을 수행하고 callback에 대해서는 callback queue에 넣어준다.

callback queue는 대기하다가 call stack이 비는 시점에 이벤트 루프를 돌려 해당 콜백 함수를 스택에 넣는다. **이벤트 루프의 기본 역할은 큐와 스택 두 부분을 지켜보고 있다가 스택이 비는 시점에 콜백을 실행시켜 주는 것이다.**

결과적으로 비동기 작업들은 따로 web API에서 처리를 해주기 때문에 멀티스레드 같이 느껴지는 것!  
  
**\[웹은 브라우저가 있어서 그 안에서 엔진이 돌아간다지만, Nodejs는 어떻게 돌아가는 거지?\]**

-   v8은 독립형으로 개발된 엔진. C++ 프로그램에 별도로 내장하여 실행시킬 수 있습니다!

  
**\[그러면, 웹은 Web APIs가 비동기 작업들을 처리해주는데, Nodejs는 비동기 작업을 어떻게 처리하지?\]**

-   libUv가 비동기 처리를 담당하는 라이브러리. 비동기 작업에 대해서는 시스템 api(ex. os에서 제공)를 활용하고, 동기작업에 대해서는 스레드 풀을 활용.
-   Thread Pool은 \*\*작업 처리에 사용되는 스레드를 제한된 개수만큼 정해 놓고 작업 큐(Queue)에 들어오는 작업들을 하나씩 스레드가 맡아 처리하는 것\*\*을 말한다. 기본은 4개인데, 직접 설정할 수 있다.

**\[스레드 풀을 통해서 멀티스레드 작업을 하는 것 아닌가? 그런데 왜 싱글 스레드라고 하는 거지?\]**

-   event loop는 한개의 thread에서만 돌아가기 때문에 싱글스레드처럼 동작하지만, 내부적으로 멀티 쓰레드를 활용할 수 있다.

  
**\[스레드 개수를 정할 수 있다면, 많이 할수록 성능이 좋은 거 아닌가?\]**

-   스레드가 너무 많으면 프로그램의 성능이 심각하게 떨어진다.
-   첫째, 정해진 분량의 일을 여러 스레드에 분배하면 스레드 하나하나는 할 일이 별로 없게 된다. 그래서 스레드를 시작하고 종료하는 비용이 실제로 하는 일의 양을 압도하게 된다.

둘째, 소프트웨어 스레드가 너무 많으면 한정된 하드웨어 자원을 공유하는데 드는 비용이 커진다.  
  
  
참고)  
[https://velog.io/@namezin/javascript-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC](https://velog.io/@namezin/javascript-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC)  
[https://darrengwon.tistory.com/953](https://darrengwon.tistory.com/953)  
[https://helloinyong.tistory.com/290](https://helloinyong.tistory.com/290)  
[https://m.blog.naver.com/pjt3591oo/221976414901](https://m.blog.naver.com/pjt3591oo/221976414901)  
[https://heowc.dev/2018/02/08/thread-pool/](https://heowc.dev/2018/02/08/thread-pool/)