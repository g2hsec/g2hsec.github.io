---
layout: single
title: (bWAPP)SQL Injection - Blind - Time-Based (SQLMap)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low
![그림 1-1](/assets/image/bwapp/injection/sql-blind-time-archive/image.png)
- 사용자의 입력값을 받아 영화 제목과 매칭되는 기능인 듯 하다.
- 그에 따른 결과는 이메일로 전송된다는 문구가 함께 출력되어 있다.
- 여러 입력값 및 ‘(싱글쿼터)등을 삽입하여도 어떠한 변화도 일어나지 않는다.


```sql
' or 1=2 and sleep(1)-- 
```

![그림 1-2](/assets/image/bwapp/injection/sql-blind-time-archive/image-1.png)
- 거짓일 경우 sleep(1) 을 실행하도록 하였을 때 거의 변화 없이 9mm 지연정도 밖에 일어나지 않는다.

```sql
' or 1=1 and sleep(1)-- 
```

![그림 1-3](/assets/image/bwapp/injection/sql-blind-time-archive/image-2.png)
- 하지만 쿼리의 결과 값이 참일 경우 slepp(1)을 실행하도록 하였을 때 그림 1-3을 보게되면 10.11s 의 지연이 발생한 것을 볼 수 있다.
- 이를 통해 Time Based SQLI 취약점이 존재한다는 것을 알 수 있다.

> 본 문제에서는 SQLMap 이라 불리는 SQL Injection에 대해 자동으로 감지 및 공격하는 툴을 사용하여 실습 해 보았다.

```SQLMap 의 경우 서버에 많은 부하가 일어날 수 있으므로, 실제 사용에 있어서 주의가 필요하다.
```

- SQLMap 의 옵션은 아래와 같다.
    1. -u : 공격 대상의 URL
    2. —cookie : 해당 유저에 대한 cookie 값
    3. —dbs : Dataebase 목록을 추출

![그림 1-4](/assets/image/bwapp/injection/sql-blind-time-archive/image-3.png)
- title 이라는 파라미터에서 time-based sqli 취약점이 발견되었고, DBMS 종류는 Mysql ≥ 5.0.12  로 확인되었다고 한다.
![그림 1-5](/assets/image/bwapp/injection/sql-blind-time-archive/image-4.png)
- 데이터베이스도 출력되고 있다.

![그림 1-6](/assets/image/bwapp/injection/sql-blind-time-archive/image-5.png)
- -D 옵션을 통해 원하는 Database 를 지정
- —tables 를 통해 설정한 데이터베이스 내에 존재하는 테이블을 추출 한다.

![그림 1-7](/assets/image/bwapp/injection/sql-blind-time-archive/image-6.png)
- bWAPP 데이터베이스내에 5가지의 테이블이 존재하며
- users 테이블에 대해 공격을 진행해보도록 했다.

![그림 1-8](/assets/image/bwapp/injection/sql-blind-time-archive/image-7.png)
- 추가적인 옵션을 주었다.
    1. -p : 공격 파라미터 지정
    2. -T : 테이블 지정
    3. —batch : SQLMap 실행시 질문에 대한 요청값을 기본값으로 설정
    4. —level —risk : 최고 레벨 지정
    5. —threads : 스레드 지정

![그림 1-9](/assets/image/bwapp/injection/sql-blind-time-archive/image-8.png)
- users 테이블에는 총 9개의 컬럼이 존재한다.
- 그림 1-10에서 존재하는 모든 컬럼들을 확인할 수 있다.

![그림 1-10](/assets/image/bwapp/injection/sql-blind-time-archive/image-9.png)
- -C 옵션을 추가하여 데이터를 추출할 컬럼들을 설정하고
- —dump 를 통해 최종적으로 데이터를 추출할 수 있다.

![그림 1-11](/assets/image/bwapp/injection/sql-blind-time-archive/image-10.png)
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

