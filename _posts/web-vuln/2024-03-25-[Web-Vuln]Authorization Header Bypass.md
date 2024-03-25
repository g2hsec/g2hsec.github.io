---
layout: single
title: (Web-Vuln) Authorization Header Bypass
categories: Web-vuln
tag: [Header bypass, Authorization, web, web vulnerability, Authorization Header Bypass]
toc: true
author_profile: false
---
# 해당 취약점은 무엇인가?
> HTTP Authorization 요청 헤더는 클라이언트가 서버의 자격 증명을 제공하는데 사용된다. 즉 웹 어플리케이션에서의 리로스에 대한 인증을 하는데 주로 사용된다.
>> 서버에서 401 Unauthorized 상태를 WWW-Authenticate 헤더로 알려준 후 발생한다.


![그림 1-1](/assets/image/web-vuln/Authorization%20Header%20Bypass/image.png)

## Syntax

```
Authorization: <type> <credentials>
```

- <type> : 자격 증명 유형
- <credentials> : 자극 증명 자체

> 이때 credentials 의 형식은 Base64-Encoded(A:B)로 이루어져 있다.

- Basic 인증 : 클라이언트는 사용자 이름과 암호를 Authorization 헤더에 포함 이때 Base64인코딩 방식을 사용한다.
- Bearer 토큰 : 클라이언트는 서버에서 발급한 토큰을 Authorization 헤더에 포함 서버는 해당 토큰을 확인해서 인증
- OAtuh : 클라이언트는 타사 서비스에서 엑세스 토큰을 요청하고Authorization 헤더에 포함 서버는 엑세스 토큰을 확인하고 타사 서비스에 대한 엑세스 여부를 결정

## 아래의 3가지 유형에서 주로 사용된다.

1. 웹 어플리케이션의 API에 엑세스 하는경우
2. 클라이언트에서 서버에 파일을 업로드 및 다운로드 하는경우
3. 웹 애플리케이션이 로그인한 사용자만 특정 리소스에 ACCESS 할 수 있도록 하는경우

> 해당 Authorization 헤더는 편리함으로 인해 최근에도 활발하게 사용되고 있다.

***<span style="color:red"> 이러한 Authorization 헤더를 잘못 사용할 경우 인증페이지에 대한 자격증명을 우회할 수 있는 취약점이 존재할 수 있다. </span>*** 

***

# Authorization Header Bypass

- **Authorization Header 를 제거하고 요청을 보내는 방법**
- 허위 인증정보를 사용 및 중간자 공격으로 인한 인증값 탈취
- **`GET, POST를 제외한 요청헤더 사용 ex) PUT, OPTIONS…등`**

> PUT과 같은 헤더는 Authorization 헤더를 허용하지 않기 때문이다. (PUT 메서드는 인증이 필요하지 않기 때문

![그림 1-2](/assets/image/web-vuln/Authorization%20Header%20Bypass/image2.png)

# 대응책

- **Authorization 헤더를 보호하기 위해 적절한 인증 및 권한 부여 프로토콜을 사용**
- **Authorization 헤더를 캐시하지 않도록 설정**
- **Authorization 헤더를 암호화**
- **Authorization 헤더를 정기적으로 검토하고 업데이트**