---
layout: single
title: (bWAPP)SQLiteManager Local File Inclusion
categories: bwapp
tag: [LFI, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> LFI는 응용 프로그램에서 File을 불러올 때 include를 사용하여 코드 내에 Bulit 하거나 동적으로 파일을 포함하는 로직이 구현되어 있을 경우 일반적으로 나타 나며, LFI는 로컬 시스템의 파일을 불러올 수 있는 취약점이다.

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/SQLiteManager%20Local%20File%20Inclusion/image.png)
- SQLiteManager 이 현재 LFI 취약점에 노출되어 있고 힌트로 cookie라는것이 주어졌다.

해당 취약점은 CVE-2007-1232로 명명되어 있다.
해당 취약점은 SQLiteManager 1.2.4 버전에서 main.php와 index.php에 취약점이 존재한다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/SQLiteManager%20Local%20File%20Inclusion/image-1.png)
- 해당 취약점의 내용을 보게되면, SQLiteManager_currentTheme 라는 쿠키의 값에 .. 와 같은 문자를 통해 임의의 파일에 접근이 가능하다고 한다.

```
Cookie: security_level=0; PHPSESSID=-; SQLiteManager_currentTheme=../../../../../../../../../../../etc/passwd%00
```

버프 스위트를 통해 쿠키값을 추가/변조를 통해 서버에 전송하여 일반적인 테마를 불러오는 것이 아닌 디렉터리 이동을 통해 passwd 파일을 읽어오도록 한다.

![그림 1-3](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/SQLiteManager%20Local%20File%20Inclusion/image-2.png)
- 성공적으로 불러오는 것을 확인할 수 있다.