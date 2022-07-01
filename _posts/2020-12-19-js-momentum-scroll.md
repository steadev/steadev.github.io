---
title: "[Javascript] ios Momentum Scrolling Issue(탄력스크롤)"
author: steadev
date: 2020-12-19 23:39:00 +0900
categories: [Javascript]
tags: [javascript, momentum scroll]
---


ios는 탄력스크롤이라는게 있죠?

스크롤에 관성도 있고 끝에 도달하면 바운스도 됩니다. 

아래 css를 통해 이를 적용하는데요..

**\-webkit-overflow-scrolling: touch;**

**하지만 문제가 있습니다.**

**스크롤 중일 때에는 scrollTop이 업데이트 되지 않는다는 것입니다..!**

제가 진행중인 프로젝트에서 element들이 추가됨에 따라 스크롤 높이를 계산해서 스크롤 위치를 재설정해줘야 하는 경우가 있었는데, 

```javascript
scrollElement.scrollTop = 0;
```

이렇게만 쓴다면, 스크롤 관성은 그대로 유지되고 element는 추가되서 뚝뚝 끊기는 현상이 발생할 수 있습니다.

(주로 역방향 스크롤시...)

위에 언급한 프로젝트에서 역방향 스크롤시, dom이 추가되고 추가된만큼 계산해서 스크롤 위치를 다시 잡아줘야 했습니다. 그런데, dom이 추가될때마다 스크롤이 끊기고 이리갔다 저리갔다 제멋대로였습니다.

처음에는 scrollTop이 업데이트 안된다는 생각을 못하고, 그저 이미지가 포함된 dom이 추가되면서 고정되지 않은 이미지 사이즈 때문에 스크롤이 난리법석인줄 알았..습니다 ㅠㅠ

쨋든 원인을 알았으니 해결책을 강구하던 중 설마 될까 하고 시도했던 방법이 성공을 했습니다...!

### 해결방법

```javascript
scrollElement.style.overflowY = 'hidden';
scrollElement.scrollTop = somewhere;
scrollElement.style.overflowY = 'scroll';
```

이 방법이 정답일지는 모르겠지만 끊김없이 스무스하게 스크롤 되는 모습을 확인할 수 있었습니다 :)

간단히, hidden으로 스크롤 아닌 것처럼 하고 scrollTop을 업데이트 시켜준 후에 다시 스크롤옵션을 주는 방법입니다.

hidden 옵션을 주면 그순간 관성이 뚝 끊길줄 알았지만 관성은 그대로 남아있고 아무일 없었다는 듯이 스크롤 되더군요..!

혹시나 같은 문제를 겪으신 분들은 이 방법을 써보셔도 좋을 것 같습니다 :)