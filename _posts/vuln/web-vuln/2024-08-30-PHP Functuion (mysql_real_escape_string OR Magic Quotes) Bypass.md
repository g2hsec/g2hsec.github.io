---
layout: single
title: PHP Functuion (mysql_real_escape_string OR Magic Quotes) Bypass
categories: Web-vuln
tag: [php, magic quotes, function, web Vulnability]
toc: true
author_profile: false
---

# 두 함수는 무슨 역할을 함수인가?

> 두 함수의 공톰점은 PHP에서 사용되는 함수이며, 주로 SQL Injection 을 방지하기 위해 사용되는 함수이다. 쉽게 설명해서 입력값에 ‘(싱글쿼터), “(더블쿼터), \(역슬래쉬), 등등 악의적인 쿼리로 이어질 수 있는 입력값이 들어오면 이스케이프 문자인 \(역슬래쉬)를 자동으로 붙여 공격을 방지한다.

```
mysql_real_escape_string 함수의 경우 PHP 7.0.0 버전 이전에 사용되었으며, 이후 버전에서는 사용되지 않으며, 7.5.0 이후의 버전부터는 완전히 삭제되었다. 또한 Magic Quotes 함수 또한 PHP 5.3.0 버전부터 비활성화 되어 5.4.0 버전 이후에 완전 제거 되었다.
```

# Bypass

입력값 앞에 자동으로 \가 생성되지만, 이때 \앞에 %a1 ~ %fe 의 값을 입력하게되면, %a1\ 이 하나의 문자로 인식되게된다. 즉 이스케이프 처리가 되지 않게 된다는 것이다.

```
%a1\’ or 1=1 —
```