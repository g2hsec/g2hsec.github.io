---
layout: single
title: Flawed Broadcast Receivers
categories: Android
tag: [android, mobile, drozer,insecurebankv2, app, application]
toc: true
author_profile: false
---

# Flawed Broadcast Receivers

## Broadcast Receivers란 무엇인가?

<div class="notice--primary" markdown="1">
애플리케이션의 4대 구성 단위중 하나로서 안드로이드 앱에서 발생하는 시스템 브로드캐스트 메시지를 수신하여 처리하는 구성 요소이다. 브로드캐스트 리시버는 안드로이드 앱의 다른 구성 요소들이나 외부 시스템에서 발생한 이벤트에 반응하여 특정 작업을 수행할 수 있도록 해준다.
    ```
브로드캐스트 리시버는 안드로이드의 매니페스트 파일에 등록되어야 하며, 등록된 브로드캐스트 이벤트가 발생했을 경우 시스템에 의해 호출된다.
    ``` 
</div>
