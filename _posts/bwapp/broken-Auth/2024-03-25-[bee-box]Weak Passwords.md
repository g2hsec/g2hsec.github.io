---
layout: single
title: (bWAPP)Broken Auth. - Weak Passwords
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low
![그림 1-1](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image.png)
- 해당 시나리오는 약한 패스워드 정책으로 인한 문제를 다루는 것으로 추정된다.

```php
$login = "test";
switch($_COOKIE["security_level"])
     28 {
     29 
     30     case "0" :
     31 
     32         $password = "test";
     33         break;
     34 
     35     case "1" :
     36 
     37         $password = "test123";
     38         break;
     39 
     40     case "2" :
     41 
     42         $password = "Test123";
     43         break;
     44 
     45     default :
     46 
     47         $password = "test";
     48         break;
     49
```

- 해당 소스코드를 보면, Security Level 에 따라 패스워드가 지정되어 있는 것을 확인할 수 있다.
- Low, Medium, Hihg 모두 추측가능한 약한 패스워드인 걸 알 수 있다.
- 계정 정보 또한 test로 설정되어 있다.
- 이럴 경우 사전대입 부르트포싱을 통해 쉽게 깨질 수 있는 계정 정보이다.

![그림 1-2](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image-1.png)
- 계정 정보가 잘못된 경우 “Invaild credentials!” 문구가 출력된다.

![그림 1-3](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image-2.png)
- 버프스위를 통해 요청 패킷을 가로채 Intruder기능을 사용할 수 있다.
- login 파라미터와 password 파라미터에 페이로드를 설정한다.

![그림 1-4](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image-3.png)
![그림 1-5](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image-4.png)
![그림 1-6](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image-5.png)
- 사전 파일을 설정해도 되며, 위 사진들과 같이 Payload 1,2 를 따로 문자열로 지정해도 된다.
- 그 후 매칭되는 문자열을 계정정보 입력 실패시 출력되는 문자로 설정 후
- 공격을 실행한다.

![그림 1-7](/assets/image/bwapp/Broken-Auth/Weak%20Passwords-archive/image-6.png)
- 실행 결과를 보면 Payload1 이 test 이며, Payload2가 test인 계정 정보에 대한 로그인에 성공하여
- “Invalid credentials!” 문구가 출력되지 않은 걸 확인할 수 있다.
- Medium Level 과 High Level 의 경우도 추측가능한 약한 패스워드로 설정되어 있어,
- 동일한 방법으로 계정탈취가 가능하다.

> 강한 패스워드 정책을 사용하여 패스워드를 설정하는 것을 권장한다.