---
title: "[CSS] position (static, absolute, relative, sticky, fixed)"
author: steadev
date: 2020-06-26 23:07:00 +0900
categories: [CSS]
tags: [css, position]
---


static absolute relative fixed 이렇게 네가지의 position이 존재합니다. 

저도 어설프게는 알고 있었지만 정확히 어떻게 활용해야 하는지 매번 헷갈려서 정리해봤습니다. 

### **\[static\]**

태그들의 default 값입니다. 빈공간에 차례대로 들어가게 됩니다. 위->아래 / 왼->오 

기본적으로 div 태그는 display: block이라 위->아래 입니다. 그래서 inline-block으로 일렬로~

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="XWXeRpE" data-user="steadev" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/steadev/pen/XWXeRpE">
  position-static</a> by steadev (<a href="https://codepen.io/steadev">@steadev</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

### **\[absolute\]**

static이 아닌 부모를 기준으로 위치 조절이 가능합니다. 전부다 static이라면 body가 기준이 됩니다.

예제에서, 두번째 박스만 absolute로 했고, 그의 부모인 container기준으로 top, left가 적용되는 것을 보실 수 있습니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="vYLemVO" data-user="steadev" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/steadev/pen/vYLemVO">
  position-absolute</a> by steadev (<a href="https://codepen.io/steadev">@steadev</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

### **\[relative\]**

현재 위치를 기준으로 위치 조절이 가능해집니다. absolute와의 차이점은, 

absolute는 다른 태그들이 absolute인 태그의 자리를 메꿔버리지만,

relative는 relative인 태그의 자리가 채워진 상태로 위치가 조절됩니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="jOWGmRY" data-user="steadev" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/steadev/pen/jOWGmRY">
  position-relative</a> by steadev (<a href="https://codepen.io/steadev">@steadev</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

### **\[fixed\]**

화면의 절대적 위치에 고정되며, 스크롤을 해도 화면에 고정되어 있습니다.

따로 top, left 등의 위치를 지정해주지 않으면 본래 위치에 고정되며, 그 자리를 주변 element가 차지합니다.

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="ExPwXgq" data-user="steadev" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/steadev/pen/ExPwXgq">
  position-fixed</a> by steadev (<a href="https://codepen.io/steadev">@steadev</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>

### **\[sticky\]**

스크롤을 해도 화면에 고정되는 것은 fixed와 똑같지만, 지정한 높이가 올때까지는 일반 흐름에 따릅니다.

그렇기 때문에 스크롤 방향에 맞게 top, bottom, right, left 중 하나는 꼭 지정해줘야 합니다. 

아래 예시를 스크롤해보면 이해가 확 될 겁니다~!

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="pogWwoK" data-user="steadev" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/steadev/pen/pogWwoK">
  position-sticky</a> by steadev (<a href="https://codepen.io/steadev">@steadev</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>