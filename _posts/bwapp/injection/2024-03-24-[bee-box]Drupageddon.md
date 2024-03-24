---
layout: single
title: (bWAPP)Drupal SQL Injection (Drupageddon)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/drupal-archive/image.png)

> CVE-2014-3794는 Dryupal 7에서 발견된 SQL 인젝션 취약점으로, Drupal 웹 사이트의 데이터베이스에 원격으로 액세스할 수 있다. 이는 views_query() 라는 데이터 조회 함수에서 발생했다. 이를 방어하기 위해서는 Drupal 7 이상의 버전으로 업데이트 해야한다.

![그림 1-2](/assets/image/bwapp/injection/drupal-archive/image2.png)
- Drupageddon 으로 만들어진 웹 페이지가 존재
- 일반적인 SQL Injection를 통한 인증 우회는 되지않음.

![그림 1-3](/assets/image/bwapp/injection/drupal-archive/image3.png)
- Metasploit에 CVE-2014-3794 취약점을 악용한 공격코드가 존재함.

![그림 1-4](/assets/image/bwapp/injection/drupal-archive/image4.png)
- show options 을 통해 공격에 필요한 입력값들을 확인
- 현재 옵션에서 넣어야할 값은 공격 대상 IP 인 RHOSTS값임

![그림 1-5](/assets/image/bwapp/injection/drupal-archive/image5.png)
- exploit시 meterpreter reverse shell 이 생성되며
- 해당 서버에 다한 원격 코드 실행이 가능하다.

![그림 1-6](/assets/image/bwapp/injection/drupal-archive/image6.png)
- searchsploit 에도 다양한 공격 코드들이 존재하며
- 해당 버전에 맞는 코드중 하나를 실습해보겠다. 34993 번 코드를 사용해보았다.

![그림 1-7](/assets/image/bwapp/injection/drupal-archive/image7.png)
- 해당 34993 소스코드를 보게되면 타겟 url 부분만 설정해주고 실행하면 된다.

![그림 1-8](/assets/image/bwapp/injection/drupal-archive/image8.png)
- 실행결과 id - admin, pw - admin 을 찾았다고 한다.

![그림 1-9](/assets/image/bwapp/injection/drupal-archive/image9.png)
- 성공적으로 admin 계정으로 로그인된 걸 알 수 있다.