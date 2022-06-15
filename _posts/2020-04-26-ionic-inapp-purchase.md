---
title: "Ionic 인앱결제 (in app purchase 2)"
author: steadev
date: 2020-04-26 14:43:00 +0900
categories: [Ionic]
tags: [Ionic, InAppPurchase]
---


Ionic으로 인앱결제를 해보면서 native에 비해 자료도 적고 가려운 곳을 긁어주는 정보들이 별로 없었습니다.

그래서 이것이 제가 했던 것을 기록하는 의미도 있지만 누군가에게는 도움이 됬으면 합니다.

1.  Ionic
    -   Code
    -   billing.fovea.cc
2.  안드로이드
    -   상품 등록
    -   sandbox
3.  IOS
    -   상품 등록
    -   sandbox
    -   계약

들어가기에 앞서, 처음에 저는 코드상으로 상품을 등록하고 결제하는 과정을 전부 할 수 있을 줄 알았습니다.

DB만 바꿔주면 상품명과 가격 등의 정보도 바로바로 바꿀 수 있을 줄 알았습니다...ㅠㅠ

그래서 관련 부분을 찾느라 삽질을 했어서, 혹시나 저와 같은 분이 계시다면! 각 플랫폼에 상품을 미리 등록해놓은 다음 그 정보를 바탕으로 결제하는 것이니 참고하시기 바랍니다.

## 1\. Ionic

### **\[Code\]**

Ionic에는 'In App Purchase'와 'In App Purchase 2' 이렇게 두 종류의 플러긴을 제공합니다. '2' 버전을 사용하는 이유는 billing.fovea.cc를 활용하기 위해서 입니다. 결제 검증을 알아서 해주는 서비스로, backend에서 할 일을 줄여줍니다. 물론, 굳이 저기를 쓰지 않아도 상관은 없습니다. 

먼저 상품을 등록해 줍니다. 

```typescript
this.store.register({
    id: "my_consumable1",
    type: this.store.CONSUMABLE
});
...
const p = this.store.get("my_consumable1");
```

처음엔 등록한다는게 각 스토어에 등록한다는 얘긴줄 알고 이렇게 하면 되겠다 했지만... 그렇게 쉬울리 없죠

이때 id는 스토어에 등록한 product id와 동일해야 합니다.

그리고 type은 게임 아이템과 같이 소모품이라면 CONSUMABLE

비정기구독의 경우는(apple만 해당) NON\_PAID\_SUBSCRIPTION

정기구독의 경우는 PAID\_SUBSCRIPTION

이렇게 해주시면 됩니다. 하지만 개발 테스트시에는 그냥 전부 consumable로 통일하는게 낫다고 생각합니다. 

그리고 아래의 get은 해당 id의 상품 IAPProduct 객체를 가져오는 것인데, 저는 개인적으로 쓸일이 없었습니다.

다음으로는 각 상품의 상태에 따라서 할 일들을 미리 정해놓는 부분입니다. 

```typescript
this.store.validator = "https://billing.fovea.cc/";
this.store.when("my stuff")
    .approved((p: IAPProduct) => p.verify())
    .verified((p: IAPProduct) => p.finish());
```

ionic 문서에 다양한 케이스들이 있지만 그 부분들은 문서에서 충분히 알 수 있고,

이 부분이 제일 저를 속썩였던 부분이라 한번 짚고 넘어가도록 하겠습니다. 

먼저, 저런 문법은 사용이 불가합니다. ts버전이 다른걸까도 싶었지만 plugin 코드를 봐도 approved는 void를 return하기 때문에 성립할 수가 없는 문법입니다. 그냥 표기상의 편의를 위해 저렇게 작성해 놓은 듯 합니다. 그래서 아래처럼 나눠서 사용하면 문제 없습니다.

추가적으로, 검증에 실패했을 경우에는 unverified를 사용하면 됩니다.

```typescript
this.store.when('my_product_id').approved(p=>p.verify());
this.store.when('my_product_id').verified(p=>p.finish());
this.store.when('my_product_id').unverified(p=>p);
```

approved는 결제가 완료됬을 때 입니다. 인앱결제 프로세스가 끝나면 이 부분이 실행되고 

validator로 지정해 놓은 url을 통해 검증을 합니다.

저의 경우는 billing.fovea.cc에서 이를 알아서 처리하고 검증 결과에 따라 verified 혹은 unverified로 오게 됩니다. 

그런데 여기에서 제일 큰 의문이 들었습니다. approved 되었을 땐 이미 금액도 빠져나가고 결제가 된 상태인데 그 후에 검증하면 무슨 소용이 있을까 싶었습니다. 이를 바탕으로 billing fovea측에 문의 해본 결과 이렇게 답변이 왔습니다. 

> If payment is approved and verification failed, the payment is not acknowledged. The user will get refund (or not get charged) automatically by Google. You don't have to do anything.

결제는 승인 됬지만, acknowledged는 안됬고 그렇기에 자동으로 며칠 안으로 해당 유저는 환불 받게 될 것이라고 합니다. 여기에서 중요한 것이 p.finish() 입니다. 이를 해주면 acknowledged 상태가 되는 것이니 아무데서나 finish 함수를 사용하면 안되겠습니다. 

마지막으로 store.when의 세팅이 끝나면 아래와 같이 refresh를 해주고 원하는 곳에서 order 해주면 인앱결제가 완료됩니다.

```typescript
// Refresh the status of in-app products
this.store.refresh();

...

// To make a purchase
this.store.order("my_product_id");
```

여기에서 **주의해야 할 사항**이 하나 있습니다. 

**store.when의 세팅은 맨 처음 시작할때, 즉, app.component.ts에서 platform.ready() 직후에 해주는 것이 좋습니다.** 

처음에는 결제 부분이 있는 페이지에 코드를 작성했었습니다. 

그런데 그 페이지를 나갔다 들어올 때마다 세팅들이 중첩되기 시작했습니다. 예를들면, 

```typescript
this.store.when('my_product_id').cancelled(p=>{ alert('결제가 취소되었습니다') });
```

이러한 코드가 있을 때, 결제 도중에 취소해버리면 alert 창이 여러번(해당 페이지에 드나든 횟수만큼...) 뜨게 됩니다. 더욱 중요한 것은 approved도 여러번, 검증도 여러번 하게되어 DB에 중복 데이터들이 쌓이게 됬습니다. 이부분까지 주의해 주신다면 보다 수월하게 개발을 진행할 수 있습니다.. 이것 때문에 괜한 삽질을 너무 많이 했었던 기억이...

### **\[billing.fovea.cc\]**

일단, 이 서비스는 유료입니다.

한달 100건의 거래까지는 무료지만 5000건은 10달러, 10만건은 50달러 입니다.(작성일 기준). 

이 서비스의 문서에 각 플랫폼 별로 세팅하는 부분을 자세하게 설명해주고 있어서 그대로 따라하기만 하면 금방 세팅이 끝납니다. 결제가 완료되면 그 로그들도 직접 확인할 수도 있습니다. 

세팅이 완료되면 여기서 제공해주는 Validator URL을 위의 this.store.validator에 적용하면 되겠습니다. 

그런데 여기서 제일 삽질했던 부분은 iOS Subscription Status URL을 appstore에 작성하는 부분이었습니다. 

사실 제가 하는 개발에서는 구독 상품은 판매하지 않아서 필요 없었는데 혹시 몰라 미리 입력해 놓으려다가 시간을 많이 잡아먹었던 부분입니다. 

아래 IOS 파트에서도 언급하겠지만 유료상품 계약이 되야 구독 상품을 생성할 수 있고 구독 상품을 생성해야 iOS Subscription Status URL을 작성할 수 있습니다.

## **2\. 안드로이드**

### **\[상품등록\]**

상품등록 관련은 정보가 많아서 따로 기록하진 않겠습니다. 하지만, 주의(?) 해야 할 부분 하나만 짚고 넘어가겠습니다. 

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-1.png" />

위 사진을 보면, 분명 가격을 KRW 14900이라고 입력했지만, 아래 빨간 부분을 보면 지들 멋대로 1000원 단위로 끊어버립니다. 만원 이하는 900원이 제대로 되는 것 같은데, 그 이상은 안되는 듯 싶습니다.(가물가물..) 

그치만 큰 문제는 아니라 그냥 저 부분에도 14900원으로 입력해주시면 끝납니다.

### **\[sandbox\]**

안드로이드의 테스트 계정 등록은 간단합니다.  ','로 구분하여 이메일을 입력하면 됩니다. 

등록 후, 해당 play store 계정으로 구매버튼을 누르면 테스트 계정이라고 뜨며,

결제 성공, 취소, 조금 있다 성공, 조금 있다 취소 이렇게 4가지 종류로 바로 테스트 해볼 수 있도록 제공됩니다.(갓구글...)

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-2.png" />

## **3\. IOS**

### **\[상품등록\]**

IOS 상품 등록도 그리 어렵진 않지만 애플이 항상 그렇듯 귀찮은 부분이 있습니다. 

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-3.png" />

우선 위 사진의 앱 내 구입 옆에 + 버튼으로 상품을 추가해봅니다. 그러면 아래의 그림처럼 나옵니다.

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-4.png" />

하지만.. 사실 위 그림처럼 나오려면 조건이 필요합니다.. 아무것도 안한했다면 빨간 박스의 자동 갱신 구독이 없습니다. 

위에 billing.fovea.cc에서도 설명했듯이 아래 \[계약\] 부분을 완성해야 위 캡처처럼 보여, 구독 상품을 생성할 수 있고 구독 상품을 생성해야 iOS Subscription Status URL(아래 캡처의 구독 상태 URL)을 작성할 수 있습니다.

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-5.png" />

마지막으로, 상품 내용을 작성하면 되는데, **심사정보->스크린샷** 부분을 넣어주셔야 될 겁니다.(안 넣어도 된다는 말이 있지만 혹시나 해서...)

만약 저 부분을 비워놓으면 진행상태가 메타데이터 등록하라고 뜹니다. 

스크린샷 까지 넣고 나면, 진행 상태가 제출 준비됐다고 표시되고 여기까지만 진행하면 이제 끝입니다. 

제 캡처처럼 "승인됨"으로 뜨려면 앱 제출해서 심사까지 거쳐야 되는 것이고 제출 준비 상태까지만 되도 테스트하는데는 문제 없습니다.

### **\[sandbox\]**

사용자 및 액세스에 들어가면 왼쪽 제일 아래에 Sandbox가 있습니다. 

이곳에서 테스터 등록하면 되는데, 

저기에 들어갈 이메일은 **App store에 등록된 계정이 아니어야 됩니다!**

그리고 **실존하는 메일이 아니어도 됩니다.** 

그냥 아무렇게나 메일 작성하면 끝!! 

그리고 후에 테스트 할때는 **기존 로그인 되있는 계정은 로그아웃 하신 상태**에서 테스트 하시면 됩니다.

그러면 결제할 때 이 이메일과 비밀번호를 작성하면 정상적으로 Sandbox 결제가 진행됩니다.

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-6.png" />

### **\[계약\]**

<img src="https://steadev.github.io/assets/images/ionic/ionic-inapp-purchase-7.png" />

빨간 유료 앱 부분이 보이시나요?! 이 부분을 몰라서 엄청 이상한 곳에서 삽질을 오랫동안 했었습니다.. ㅠㅠ

유료 앱에는 주소, 세금 관련(...?) 등 정확한 내용은 기억이 안나지만 여러 정보들 입력하면 애플에서 또 심사를 합니다.. 그놈의 심사.. 

쨋든, **유료앱 부분이 활성 상태가 아니라면**, 암만 Ionic in app purchase를 고치고 고쳐도 product id를 알 수 없다는 피드백만 올 뿐입니다. 다른 걸 다 했더라도 이 부분이 안되면.. !!

이미 IOS 결제 개발 하셨던 분들이라면 당연히 알고 계실테지만, 처음 하시는 분들은 이게 조금이나마 도움이 됬으면 합니다. 

길었지만, 여기까지 따라 오셨다면 큰 문제 없이 구현이 될 것이라 생각합니다. 

질문이나 지적하실 부분이 있으시면 댓글 부탁드립니다 :)