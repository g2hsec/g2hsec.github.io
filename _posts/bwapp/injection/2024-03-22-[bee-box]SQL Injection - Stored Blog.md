---
layout: single
title: (bWAPP)SQL Injection (SQLite)
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image.png)
- 사용자의 입력값이 DB 에 저장된 후 불러와 다시금 출력하는 듯 하다.

![그림 1-2](/assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image2.png)
- 사용자의 입력값이 Entry에 입력되는 걸 확인 할 수 있다.
- 이럴 경우 보통 XSS 공격만 테스트 하고 넘어가는 경우가 많다.
- 하지만 DB에 저장된 값을 SQL 구문을 통해 불러오거나 저장하는 경우 SQL Injection도 충분히 일어날 수 있다.

![그림 1-3](/assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image3.png)
- ‘(싱글쿼터) 입력시 DBMS Error가 발생하는 것을 볼 수 있다.

```sql
$sql = "INSERT INTO blog (date, entry, owner) VALUES (now(),'" . $entry . "','" . $owner . "')";
```

> INSERT 절을 통해 입력 되는 SQL 구문은 위와 같다.

```sql
test',select(select 1))--
```

![그림 1-4](/assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image4.png)
- select 1 을 사용하여 테스트 해본 결과 1이 정상적으로 출력되고 있다.