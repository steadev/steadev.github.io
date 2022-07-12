---
title: "[Android] Notification - 알림 띄우기"
author: steadev
date: 2021-07-04 16:24:00 +0900
categories: [Android]
tags: [Android, Notification]
---


[https://developer.android.com/training/notify-user/build-notification?hl=ko](https://developer.android.com/training/notify-user/build-notification?hl=ko) 

 [알림 만들기  |  Android 개발자  |  Android Developers

알림은 사용 중이 아닌 앱의 이벤트에 관한 짧고 시기적절한 정보를 제공합니다. 이 페이지에서는 Android 4.0(API 레벨 14) 이상의 다양한 기능을 사용하여 알림을 만드는 방법을 설명합니다. Android

developer.android.com](https://developer.android.com/training/notify-user/build-notification?hl=ko)

위의 안드로이드 공식 문서에 알림 띄우는 부분이 잘 정리가 되어 있어서

구체적인 방법에 대한 블로깅은 생략하고자 합니다.

여기서는 알림을 띄울 때 삽질 했던 부분에 대해서 남기고자 합니다.

제가 겪었던 문제는 2가지가 있습니다.

1\. 알림들이 여러개 왔을 때, 무엇을 클릭하든 제일 처음 온 알림으로만 인식하는 문제

2\. 상태창에만 알림이 뜨고 head up Notification이 나오지 않는 문제

**1\. 알림들이 여러개 왔을 때, 무엇을 클릭하든 제일 처음 온 알림으로만 인식하는 문제** 

알림 1, 2, 3이 왔을 때 무조건 1로만 가버리는 문제가 있었는데요..ㅂㄷㅂㄷ

PendingIntent의 개념?에 대해 잘 이해 못해서 생긴 문제였는데, 아래 코드로 바꾸면서 해결이 되었습니다. 

```kotlin
// from
PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT)
// to
PendingIntent.getActivity(context, System.currentTimeMillis().toInt(), intent, PendingIntent.FLAG_UPDATE_CURRENT)
```

getActivity는 다음과 같이 구성되어 있는데요

```kotlin
public static PendingIntent getActivity(Context context, int requestCode, Intent intent, int flags) {
	throw new RuntimeException("Stub!");
}
```

context와 requestCode가 같으면, 같은 pendingIntent로 취급해서 flag에 영향을 받게 됩니다.

flag가 FLAG\_UPDATE\_CURRENT라면 최신 알림으로 override 되어버립니다.

flag에 여러 종류가 있는데 여기서는 스킵하도록 하겠습니다.

**2\. 상태창에만 알림이 뜨고 head up Notification이 나오지 않는 문제**

Ios에서는 기본적으로 헤드업알림으로 뜨는데, Android는 아니더라구요

결론부터 말씀드리자면, 알림 우선순위를 높게 설정해주면 해결되는 문제였습니다. (+ vibrate 설정해줘야 된다고 하더라구요..!)

Channel과 Notification builder 두 군데의 우선순위를 변경해줘야 합니다.

```kotlin
// channel
val channel = NotificationChannel(
	context.getString(R.string.channel_name),
	context.getString(R.string.channel_name),
	NotificationManager.IMPORTANCE_HIGH // IMPORTANCE_HIGH로 세팅!!
)

// notification builder
NotificationCompat.Builder(context, context.getString(R.string.channel_name))
	.setPriority(NotificationCompat.PRIORITY_HIGH) // PRIORITY_HIGH로 세팅!!
	.setDefaults(NotificationCompat.DEFAULT_SOUND or NotificationCompat.DEFAULT_VIBRATE)
	...
```