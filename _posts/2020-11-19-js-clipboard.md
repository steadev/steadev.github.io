---
title: "Clipboard copy on Safari using Javascript"
author: steadev
date: 2020-11-19 12:17:00 +0900
categories: [Javascript]
tags: [javascript, clipboard]
---


웹개발을 할때, 링크를 복사한다든지 그 외의 다른 텍스트를 클립보드에 복사시키려면 크롬의 경우는 매우 간단합니다.

하지만... 그놈의 애플..이 항상 문제죠;; 후

아래 코드는 safari건 chrome이건 다 동작합니다. 

임의의 textarea를 생성하고 값을 세팅하고 textarea영역을 선택(복사할 글자 선택).

보통 여기까지하면 textarea의 모든 범위의 값이 선택됩니다. 

이후 범위 설정하면 설정한 범위까지의 text가 선택됩니다.

마무리로 execCommand('copy') 해준다면 완료!......

```javascript
iosCopyToClipboard(textToCopy): boolean {
    const tmpTextarea = document.createElement('textarea');
    tmpTextarea.value = textToCopy;
    // tmpTextarea.contentEditable = true;
    // tmpTextarea.readOnly = false;

    document.body.appendChild(tmpTextarea);
    tmpTextarea.select();
    tmpTextarea.setSelectionRange(0, 9999);  // 셀렉트 범위 설정
    const result = document.execCommand('copy');
    document.body.removeChild(tmpTextarea);
    return result;
 }
```

... 라면 좋겠지만! ios에는 한가지 함정이 있습니다.

safari에서는 무조건!! **user action에 의해 trigger 된 것만 복사를 허용**해줍니다.

이를 몰라서 문제 없는 코드만 수정하며 삽질을 많이 했는데요,,, 

user action에 의해 trigger 되는 것이란, 말 그대로 유저가 버튼 누르거나 직접 꾹 눌러서 복사 하는 것입니다. 

그래서 safari console에서 직접 복사 커맨드 입력해봤자 false만 return 해줄 뿐입니다.

저같은 경우는, 클릭했을때 수행하는 함수에서 api call을 하나 수행한 후에 copy로직을 수행했었습니다.

유저가 직접 클릭한 함수이긴 했으나 그 안에서 서버로부터 값을 가져오는 작업을 수행하는 바람에 

'이건 유저에 의한 이벤트가 아니구나!' 라고 지멋대로 판단하는 것 같습니다..

실제로, 해당 api call 작업 바로 직전에 copy하면 잘 동작을 하지만, 작업 후로 로직을 옮기면 실패하게 됩니다.

safari가 개발자 친화적이 되길 기원합니다...!!