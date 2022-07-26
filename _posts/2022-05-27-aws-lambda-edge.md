<!-- ---
title: "Lambda@Edge를 통한 Dynamic OG Tags, SEO"
author: steadev
date: 2022-05-27 13:19:00 +0900
categories: [AWS]
tags: [AWS, Lambda@Edge]
---


<img src="https://steadev.github.io/assets/images/aws/2022-05-27-1.png" />

End user에게 데이터가 가는 중간 과정 각각에 lambda@edge를 통해서 관여할 수 있습니다.

**Viewer Request**   \- End User로부터 CloudFront로 요청이 도착한 직후

**Origin Request**  \- CloudFront에서 Origin Server로 요청을 보내기 직전  
**Origin Response**  \- Origin Server에서 보낸 응답이 도착한 직후  
**Viewer Response** \- CloudFront에서 End User로 Response 보내기 직전

## **주의사항**

-   버지니아 북부(us-east-1) 리전에서만 배포 가능
-   로그는 CloudWatch에 쌓임.
-   _But, us-east-1에 쌓이는 것이 아니고 접속한 각 region에서 쌓입니다._  
    (ex. 서울 region에서 접속했을 경우, 서울 region에서만 로그 확인 가능)

## **SEO 적용 방법**

#### **Basic Idea**

1.  **Viewer Request**에서 해당 요청이 crawler bot에 의한 요청인지 확인 및 header에 해당 정보 추가
2.  **Origin Response**에서 crawler bot에 의한 요청일 경우 og tag만 포함한 response 전달  
    \-> 단순하게 말하자면, bot이 요청했을 경우에 response를 조작하여 보내는 것입니다.

#### **함수 작성 및 배포 과정**

-   **함수 생성**

<img src="https://steadev.github.io/assets/images/aws/2022-05-27-2.png" />
<img src="https://steadev.github.io/assets/images/aws/2022-05-27-3.png" />

-   **코드 작성 및 배포**

<img src="https://steadev.github.io/assets/images/aws/2022-05-27-4.png" />

1.  파란 박스에서 코드를 바로 작성하거나 이미 작성된 코드 업로드
2.  빨간 박스를 통해 Lambda@Edge로 배포(**Add trigger** or **Deploy to Lambda@Edge**)
3.  **Distribution 및 edge 선택** 후 배포(아래사진)

<img src="https://steadev.github.io/assets/images/aws/2022-05-27-5.png" />

-   **배포 결과 확인**

<img src="https://steadev.github.io/assets/images/aws/2022-05-27-6.png" />

-   **로그 확인  
    **

<img src="https://steadev.github.io/assets/images/aws/2022-05-27-7.png" />

`us-east-1.xxx` 부분 함수 확인!

lambda지역과 무관하게 **request가 발생한 지역을 기준**으로 cloudwatch에 쌓입니다!

### **예시 코드**

**Viewer Request**

```javascript
const bot = /googlebot|bingbot|yandex|baiduspider|twitterbot|facebookexternalhit|rogerbot|linkedinbot|embedly|quora link preview|showyoubot|outbrain|pinterest|slackbot|vkShare|W3C_Validator|kakaotalk-scrap|yeti|naverbot|kakaostory-og-reader|daum/g;

exports.handler = (event, context, callback) => {
  const request = event.Records[0].cf.request;
  const user_agent = request.headers['user-agent'][0]['value'].toLowerCase();
  const referer = request.headers['referer'];
  request.headers['referer'] = [
    {
      key: 'referer',
      value: Array.isArray(referer) && referer[0] !== undefined ? referer[0].value : request.headers['host'][0].value
    }
  ]
  if (user_agent) {
    const found = user_agent.match(bot);
    request.headers['is-crawler'] = [
      {
        key: 'is-crawler',
        value: `${!!found}`,
      },
    ];
  }
  callback(null, request);
};
```

**Origin Response**

```javascript
'use strict';
// origin-response
exports.handler = async (event, context, callback) => {
  const { request, response } = event.Records[0].cf;
  const { headers, uri } = request || {};
  const isCrawlerHeader = request.headers['is-crawler'];
  let isCrawler = false;
  if (Array.isArray(isCrawlerHeader) && isCrawlerHeader[0].value !== undefined && isCrawlerHeader[0].value !== null) {
    isCrawler = isCrawlerHeader[0].value;
  };
  if (isCrawler === 'true') {
    // asset 가져오는 요청이라면 그대로 return;
    // google search console, naver search advisor html가져올 경우 그대로 return
    if (uri.includes('assets/') || uri.includes('.html') || uri.includes('robots.txt')) {
      callback(null, response);
      return;
    }

    const body = `<html><head>
                    <meta property="og:locale" content="ko_KR" />
                    <meta property="og:locale:alternate" content="en_US" />
                    <meta property="og:url" content="${uri}" />
                    <meta property="og:type" content="website" />
                    <meta property="og:title" content="${title}" />
                    <meta property="og:description" content="${description}️"/>
                    <meta property="og:image" content="${image}" />
                    <meta property="og:image:type" content="image/jpg" />
                    <meta name="twitter:card" content="summary_large_image" />
                    <meta name="twitter:title" content="${title}" />
                    <meta name="twitter:description" content="${description}️" />
                    <meta name="twitter:image" content="${image}" />
                  </head></html>`;

    response.headers = {
      ...response.headers,  
      "content-type": [{"key": "Content-Type", "value": "text/html"}]
    };
    response.status = '200';
    response.statusDescription = 'OK';
    response.body = body;
    callback(null, response);
  } else {
    callback(null, response);
  }
};
```

### **여기까지에서의 문제점!**

위 대로 따라하더라도 잘 동작하지 않을 것입니다..

-   header에 넣어준 is-crawler 헤더가 Origin response 에서 보이질 않습니다.
-   is-crawler가 Origin response에서 확인이 가능하다 하더라도 bot이 아닌 일반 요청에서도 조작된 html이 보이게 됩니다.  
    html파일이 캐싱되어 bot이 아닌데도 캐싱된 response가 가게 되는 것입니다.

### **해결..!**

**is-crawler 헤더 문제**

`CloudFront > 배포 > 동작 편집` 에서 정책 생성을 통해 필요한 헤더를 직접 세팅해줄 수 있습니다.

원본 요청 정책에 해당 정책을 세팅하면 Origin Response에서 해당 header를 확인할 수 있습니다.

<img src="https://steadev.github.io/assets/images/aws/2022-05-27-8.png" />
<img src="https://steadev.github.io/assets/images/aws/2022-05-27-9.png" />

**캐싱 문제**  
response headers를 초기화하는 부분에 cache를 disable하도록 작성해주어야 합니다!

```javascript
// 꼭 캐시를 사용하지 않도록 설정해야 합니다.
    response.headers = {
      ...response.headers,  
      "content-type": [{"key": "Content-Type", "value": "text/html"}],
      "cache-control": [{"key": "cache-control", "value": "no-cache, no-store, must-revalidate"}]
    };
```

한번 lambda함수를 바꾸면 배포시간이 꽤 걸리고

오류가 발생했을 때 console.log를 확인하려면 일일이 cloud watch에 들어가서...

어디 지역에서 request가 발생했는지 모르면 계속 지역 바꿔가면서...

이런 저런 삽질을 많이 했는데요 :(

삽질의 제일 큰 이슈가 위의 header와 캐싱 이슈였습니다;

혹시나 비슷한 문제를 겪는 분들께 도움이 되기를 바랍니다 :)

질문이나 부족한 부분이 있다면 댓글 부탁드립니다! -->