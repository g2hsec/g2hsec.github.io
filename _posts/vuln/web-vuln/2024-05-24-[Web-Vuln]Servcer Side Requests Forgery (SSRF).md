---
layout: single
title: Servcer Side Requests Forgery (SSRF)
categories: Web-vuln
tag: [Servcer Side Requests Forgery, SSRF, web, web vulnerability, injection]
toc: true
author_profile: false
---

# SSRF 취약점이란?

> SSRF는 클라이언트측의 입력값을 위조시켜 위조된 HTTP 요청을 보내, 일반적으로 외부에서 접근이 불가능한 **<u style="color:red;">서버 내부망에 접근(Access)</u>** 하여, 데이터 유출 및 서버의 기밀성, 가용성, 무결성을 파괴한다. SSRF 의 경우 CSRF 공격과 유사하지만, 공격이 이루어지는 부분이 클라이언트측이냐 서버측이냐의 차이점이 존재한다. 