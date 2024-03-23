---
layout: single
title: (tools) keepass2john
categories: Tools
tag: [wordlist, attacks, tool, tools, pentest, pentest tool,password,keepass2john]
toc: true
author_profile: false
---

# keepass2john란 무엇인가?

> keePass 데이터베이스 파일(.kdbx)을 John the Ripper가 사용할 수 있는 형식으로 변환해준다. 이를 통해 Keepass 데이터베이스에서 추출한 해시를 John the Ripper 로 사용하영 하여 Password Cracking이 가능하다.

## 기본적인 사용 방법

```shell
keepass2john -i /opt/security_txt.kdbx > output_file
```

## 주요 옵션

- **-i, --keepass2john-input**: KeePass 데이터베이스 파일의 경로를 지정.

> Password Crack에 성공 했다면 해당 패스워드를 keepassxc tool을 사용하여 .kdbx 파일을 OPEN 할 수 있다.