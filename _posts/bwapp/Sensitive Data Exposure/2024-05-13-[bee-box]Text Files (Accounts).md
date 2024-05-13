---
layout: single
title: (bWAPP)Text Files (Accounts)
categories: bwapp
tag: [Text Files, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 시나리오의 취약점은 평문 및 sha-1과 같은 약한 해시 알고리즘을 통한 데이터 전송시 이를 크랙하여 전송 데이터 유출이 가능하게 된다.

## Low-Level

![그림 1-1](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image.png)
- 텍스트 파일에 계정을 입력하는 듯한 기능이 존재한다.

![그림 1-2](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-1.png)
- 계정 정보 입력시 파일에 저장되고 다운로드 할 수 있다.

![그림 1-3](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-2.png)
- 해당 파일은 정적파일저장 형식으로 디렉터리에 저장되어 디렉터리 리스팅 취약점과 연계될 경우 계정정보 탈취가 가능하게 된다.
- 또한 파일 내에 데이터가 평문으로 저장되어 있는걸 확인할 수 있다.

## Medium-Level

![그림 1-4](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-3.png)
- Medium 에서는 평문이 아닌 문자열로 되어 있다.

![그림 1-5](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-4.png)
- SHA1 으로 해시화 된 걸로 추정되며, 이는 약한 해시알고리즘으로 크랙이 가능할 수 있다.

![그림 1-6](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-5.png)
- 레인보우 테이블을 이용한 크랙이 가능했다.

## High-Level

![그림 1-7](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-6.png)
- High 에서는 salt값이 추가되어 해시를 시도하는 듯 하다.

![그림 1-8](/assets/image/bwapp/sensitive%20data%20exposure/Text%20Files%20(Accounts)/image-7.png)
- SHA-256 을 통한 해시화가 된 걸로 보인다. 이는 안전한 해시 알고리즘으로 사실상 해독이 거의 불가능 하다.