---
layout: single
title: (bWAPP)Broken Auth. - CAPTCHA Bypassing
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/captcha-archive/image.png)
![그림 1-2](/assets/image/bwapp/Broken-Auth/captcha-archive/image-1.png)

- 위 두 사진을 보게되면 일반적인 로그인 기능에서 CAPTCHA 기능이 존재하는 걸 확인할 수 있다.
- 이 때 로그인 성공 및 실패 에 따라 CAPTCHA의 문자열은 랜덤한 문자열로 지속적으로 바뀌게 된다.
- 이럴 경우 BruteForce 공격이 불가능하다.
- 하지만 해당 CAPTCHA 문자열을 일정한 문자열로 고정시킨 후 BruteForce 공격을 수행할 수 있다.

![그림 1-3](/assets/image/bwapp/Broken-Auth/captcha-archive/image-2.png)
![그림 1-4](/assets/image/bwapp/Broken-Auth/captcha-archive/image-3.png)

- 우선 패스워드가 틀렸을 때 출력 되는 문자열과
- CAPTCHA가 틀렸을 경우 출력 되는 문자열이 서로 다르다는 걸 알 수 있다.
![그림 1-5](/assets/image/bwapp/Broken-Auth/captcha-archive/image-4.png)

- 다양한 방법이 존재하지만 BurpSuite 의 Intruder 기능을 사용하여 쉬운 조작이 가능하다.
![그림 1-6](/assets/image/bwapp/Broken-Auth/captcha-archive/image-5.png)
![그림 1-7](/assets/image/bwapp/Broken-Auth/captcha-archive/image-6.png)
- BruteForce 를 할 포지션을 두 구간으로 잡았으니
- Payload set1과 2를 각각 사전파일이 존재한다면 사전파일을 지정하고
- 해당 실습에서는 일부 단어들로 대체하여 실습을 진행하였다.

![그림 1-8](/assets/image/bwapp/Broken-Auth/captcha-archive/image-7.png)
- 그 후 패스워드를 잘못 입력시 출력 되는 문구를 Grep - Match에 추가하여 해당 문구가 출력될 때 grep되도록 설정 하였다.

> 이 때 CAPTCHA가 변경되지 않도록 Intruder 기능으로 보낼 때 프록시 intercept로 패킷을 잡아놓은 상태로 보내야 한다.

![그림 1-9](/assets/image/bwapp/Broken-Auth/captcha-archive/image-8.png)
- 그 후 공격을 수행하게 되면 Invalid credentials! Did you forgot your password? 문구가 존재하면 1이라는 체크값이 설정되지만
- 그렇지 않은 경우 즉, 로그인에 성공한 경우는 1로 설정된 값이 존재하지 않는 걸 알 수 있다.

![그림 1-10](/assets/image/bwapp/Broken-Auth/captcha-archive/image-9.png)

- 이와 같이 CAPTCHA 를 우회할 수 있다.
- 하지만 이는 단순한 문자열로 이루어진 쉬운 패스워드에만 가능하며,
- 대소문자 및 특수문자와 같이 복잡하게 설정되어있다면, 실실적으로 부르트포싱을 통한 패스워드 크랙은 불가능 하다.