---
title: "[Javascript] ios 한글 조합 문제"
author: steadev
date: 2020-12-19 20:42:00 +0900
categories: [Javascript]
tags: [javascript, ios korean]
---


ios 13이상에서는 받침이 완료되지 않은 문자에 대해서 buffer에 저장되버립니다.

받침까지 완료가 되어야만 제대로 출력이 되고, 아닌경우는 받침이 필요없지 않는한 이상한 현상을 겪게 됩니다.

ex) '아아' 라는 문자 전송 후, input = ''으로 입력값을 초기화 해줘도 다음에 'ㅇ'을 입력하면 '앙'이 출력됩니다.

이런 ~그지같은~ 현상 때문에 여러 방법을 시도해봤는데요..!

한가지 방법은 input의 focus를 없앴다가 다시 focus 시켜주는 방법입니다. 

입력이 완료되면 blur 후 다시 focus 해주는 방식인데,

해결이야 되지만 키보드가 내려갔다 다시 올라오는 불편한 상황이 연출됩니다..ㅜㅠ

그러다 아래 참고로 적어둔 포스트에서 답을 얻었습니다. (감사합니다..!)

간단히 해결책만 말하자면,

숨겨진 input을 하나 두고 focus를 먼저 줘버리면 해당 buffer가 숨겨진 input으로 가버리고

다시 원래 input을 focus하면 쓸데없는 버퍼가 사라지게 됩니다. 아래처럼..!!

```javascript
// 숨겨진 input focus 직후 원래 input focus
hiddenInputElem.focus();
inputElem.focus();
```

이렇게되면 키보드도 그대로 유지되면서 자연스럽게 해결이 가능합니다

참고) [https://kidk.kr/2019-12-07-clear-input-buffer-in-ios/](https://kidk.kr/2019-12-07-clear-input-buffer-in-ios/)