---
layout: single
title: unshadow
categories: Tools
tag: [unshadow, attacks, tool, tools, pentest, pentest tool,password, shadow]
toc: true
author_profile: false
---

# unshadow 란 무엇인가?

> 일반적인 리눅스 시스템 상에서 passwd 파일과 shadow파일을 합쳐준다. 이를 통해 john과 같은 Password Cracking Tool을 통한 Hash Cracking이 가능하다.

## 기본적인 사용 방법

```shell
unshadow /etc/passwd /etc/shadow > password_file
```
