---
layout: single
title: (bWAPP)SQL Injection
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/sqli-get/image.png)
- 검색 기능이 존재하며, GET 방식으로 전송된다.
- ‘ (싱글쿼터) 입력을 통해 Error 유/무를 체크해보았다.

![그림 1-2](/assets/image/bwapp/sqli-get/image2.png)
- Error 가 발생되며 DBMS Error 가 페이지상에 노출되고 있다.
- MySQL 로 확인된다.

![그림 1-3](/assets/image/bwapp/sqli-get/image3.png)
- 연결 연산자를 통해 추가 확인을 했다.
- DBMS Error 가 노출되고 있으니 Error Based 공격을 수행했다.

```sql
' and updatexml(null,concat(0x3a,(select database())),null) and '%%'='%
```

> Bee-Box 에서는 updatexml 함수는 사용이 되지 않는다. (다른 환경에선는 정상 작동 함)

- 해당 Bee Box 에서는 Error Based 가 불가능 한 것으로 판단
- Union Based 공격을 수행

