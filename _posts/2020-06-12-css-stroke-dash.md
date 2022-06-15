---
title: "[CSS] stroke-dasharray, stroke-dashoffset 활용하여 animation 만들기"
author: steadev
date: 2020-06-12 16:48:00 +0900
categories: [CSS]
tags: [css, stroke-dasharray, stroke-dashoffset]
---

stroke-dasharray와 stroke-dashoffset은 svg를 가지고 animation을 만드는데 활용되곤 합니다.

여기서 다뤄볼 것은 css로만 하는 방법과 javascript로 하는 방법입니다.

먼저, stroke-dasharray와 stroke-dashoffset에 대한 개념부터 확인하고 가겠습니다.

\- stroke-dasharray는 점선을 만들어주는 놈이고

\- stroke-dashoffset은 어디부터 시작할 것인지 정해주는 놈입니다.  

<img src="https://steadev.github.io/assets/images/css/2020-06-12-1.png" style="width:100px; height:100px" />

위 circle을 예로 봐보겠습니다.

원의 전체 길이(둘레)는 1359.7597... 입니다. 그리고 시작점은 시계방향 90도 지점부터 입니다.

<img src="https://steadev.github.io/assets/images/css/2020-06-12-2.png" style="width:100px; height:100px" />

그리고 이 원은 stroke-dasharray: 100 입니다.

그러므로 dasharray 값은 점선을 만드는 간격을 의미하고 그 간격대로 점선을 만들어주는 것입니다. 

<img src="https://steadev.github.io/assets/images/css/2020-06-12-3.png" style="width:100px; height:100px" />

사이 간격을 따로 설정할 수도 있는데, 위 원은 stroke-dasharray: 100 20 으로 설정되었습니다. 

뒤의 숫자(20)가 점선의 간격을 의미합니다.

<img src="https://steadev.github.io/assets/images/css/2020-06-12-4.png" style="width:100px; height:100px" />

이 원은 stroke-dasharray: 100, stroke-dashoffset: 100 입니다.

offset이 시작점을 정해주는 것이므로, 100만큼 띄운 이후에 100씩 간격을 두고 점선을 만듭니다.

그래서 두번째 원과 비교해보면 흰 부분과 파란부분이 뒤바뀌어 있는 것을 볼 수 있습니다.

그래서, 이것을 가지고 어떻게 animation을 만드는지 봐보겠습니다.

<img src="https://steadev.github.io/assets/images/css/2020-06-12-5.gif" style="width:100px; height:100px" />

1\. css로 animation 만들기

dasharray를 원 둘레로 설정해놓고 

animation 속성을 적용하면 끝입니다.

샘플 코드에서는 offset을 둘레의 2배로 설정해두어 animation이 끊기지 않도록 했습니다.

아래는 관련 코드입니다.

```html
<head>
  <style>
    .moving-outline circle {
      stroke-dasharray: 1360;
      animation: dash 2s linear infinite;
    }
    @keyframes dash {
      to {
        stroke-dashoffset: 2720;
      }
    }
  </style>
</head>
<body>
  <svg
    class="moving-outline" width="453" height="453" 
    viewBox="0 0 453 453" fill="none" xmlns="http://www.w3.org/2000/svg"
  >
    <circle
      cx="226.5" cy="226.5" r="216.5" 
      stroke="#018EBA" stroke-width="20"
    />
  </svg>
</body>
```

2\. js로 animation 만들기

원리는 간단합니다. 

dasharray를 원 전체의 둘레로 해놓고 offset을 전체부터 시작해서 조금씩 줄여가는 것입니다. 

dasharray를 설정해놓지 않으면 offset이 먹히지 않습니다. 

1) 원 둘레 구하기

2) dasharray를 원 둘레로 설정

3) 시간 흐름에 따라 dashoffset 값 늘리기(코드에서는 10ms 마다)

아래는 관련 코드입니다. 

```html
<body>
  <svg class="moving-outline" width="453" height="453" viewBox="0 0 453 453" 
       fill="none" xmlns="http://www.w3.org/2000/svg">
    <circle cx="226.5" cy="226.5" r="216.5"
            stroke="#018EBA" stroke-width="20"/>
  </svg>
  <script>
    const outline = document.querySelector(".moving-outline circle");
    const outlineLength = outline.getTotalLength();
    outline.style.strokeDasharray = outlineLength;
    outline.style.strokeDashoffset = 0;
 
    let duration = 60;
    let elapsed = 0;
    animate(elapsed);
    function animate(offset) {
      setTimeout(() => {
        elapsed++;
        if (elapsed > duration * 2) elapsed = 0;
        animate((elapsed / duration) * outlineLength);
      }, 10);
      outline.style.strokeDashoffset = offset;
    }
  </script>
</body>
```