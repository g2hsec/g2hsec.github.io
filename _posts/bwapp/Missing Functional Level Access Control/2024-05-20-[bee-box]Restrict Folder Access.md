---
layout: single
title: (bWAPP)Restrict Folder Access
categories: bwapp
tag: [LFI, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> Restrict Folder Access 취약점은 특정 폴더나 디렉터리에 대한 접근 권한을 제한하는 보안 메커니즘에서 발생할 수 있는 취약점을 의미한다.

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Restrict%20Folder%20Access/image.png)
- 다양한 파일들이 존재하며, 권한이 있어야만 문서 폴더에 액세스 할 수 있다고 한다.

즉 /bWAPP/documents/Terminator_Salvation.pdf 와 같이 외부에서 바로 접근이 가능하거나, 
<br>
google 검색 필터를 이용 intitle:"index of" intext":문서 와 같이 외부에서 접근하면 안되지만 설정 미흡으로 인해 접근이 허용되는 경우 취약한 상황인 시나리오이다.