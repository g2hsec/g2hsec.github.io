---
layout: single
title: (bWAPP)XML/XPath Injection (Search)
categories: bwapp
tag: [xml injection, xpath injection, sql, injection, bwapp, bee box, GET, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 검증 로직
> Low Level 에서는 보안 대책이 적용되어 있지 않다.

## Level - Low

![그림 1-1](/assets/image/bwapp/injection/xml-xpath%20injection%20search-archive/image.png)
- 검색 목록이 존재하며, 목록을 선택하여 검색하면 관련도딘 값들이 순차별로 나오는 것을 알 수 있다.

![그림 1-2](/assets/image/bwapp/injection/xml-xpath%20injection%20search-archive/image-1.png)
- 또한 데이터 전송 방식은 GET 형식으로 전송되는 것을 알 수 있다.

![그림 1-3](/assets/image/bwapp/injection/xml-xpath%20injection%20search-archive/image-2.png)
- ‘(싱글쿼터) 입력시 Xpath 문법 오류가 발생하는 것을 확인할 수 있다.
- 이를 통해 XPath Injection이 가능하다는 걸 확인할 수 있다.

```sql
$result = $xml->xpath("//hero[contains(genre, '$genre')]/movie");
```

- 해당 검색 기능에서 적용되는 xpath 쿼리문은 위 코드와 같이 설정되어 있다.
> 이 때 contains() 함수는 피연산자1, 피연산자2 에서 동일한 문자가 하나라도 포함되어 있다면 참을 반환 한다. 즉 contains(abc, azz) 일 경우 a가 동일하므로 참을 반환한다.

- 사용자의 전달값이 &genre 에 포함되므로 해당 구문에서 참을 만들기 위해서는

```sql
') or 1=1][('1
```

- 위 코드와 같이 설정할 수 있다.

![그림 1-4](/assets/image/bwapp/injection/xml-xpath%20injection%20search-archive/image-3.png)
- 참을 반환하여 Action 을 검색한 것과 동일한 결과를 반환한다.

```sql
') or 1=1] | //* | exploit[('1
```

![그림 1-5](/assets/image/bwapp/injection/xml-xpath%20injection%20search-archive/image-4.png)
- 위 코드를 사용하여 현재 노드로부터 모든 노드를 조회할 수 있다.
- ‘) or 1=1] 을 통해 참을 만들고 |(파이프라인)을 통해 다음 명령어 실행이 가능하다.
- 이 떄 //*을 통해 모든 노드를 조회하고 
- 구문완성을 위해 임의의 값으로 exploit[(’1 을 주어서 구문을 완성시킨다.
- //* 자리에 원하는 XPath 문법을 통해 데이터를 조회할 수 있다.