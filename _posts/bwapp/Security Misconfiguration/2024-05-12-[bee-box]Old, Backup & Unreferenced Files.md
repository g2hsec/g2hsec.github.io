---
layout: single
title: (bWAPP)Old, Backup & Unreferenced Files
categories: bwapp
tag: [mitm, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---
**<u>주의</u>** 해당 글에서 소개하는 기법, 및 사이트등을 이용하여 특정 사이트에 대한 공격 행위는 불법, 즉 범죄 행위입니다.
{: .notice--primary}  

# 취약점 설명
> 오래된 백업 파일, 참조 파일, 디렉터리 등이 추측하기 쉬운 문자열로 되어 있을 경우, 부르트포싱을 통한 크랙이 가능하며, 디렉터리 리스팅이 이루어 질 수 있다. 또한 오래된 백업 파일의 경우 관리가 제대로 이루어지지 않을 경우 데이터 노출, 개인정보 보호 위반, 공격 표면 증가로 이어질 수 있게 된다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Old,%20Backup%20&%20Unreferenced%20Files/image.png)
- 해당 시나리오에서는 bWAPP 에서 오래된 백업 파일이 존재하지 않고 Old, Backup & Unreferenced Files 취약점에 대해 설명하는 듯 하다.

쉽게 구글 검색 필터를 사용하여 방치되고 있는 디렉터리, 파일 등의 검색이 가능하다.

```
intitle:"index.of" intext:(bak | backup| 백업 | dump | old | old | save | sav | copy | tmp | bkp | bac) | swp)
``` 

위와 같이 구글 검색 필터를 사용한 취약점 발견이 가능하다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Old,%20Backup%20&%20Unreferenced%20Files/image-1.png)
- 디렉터리 리스팅 취약점이 존재하며, 백업파일이 존재하는 사이트가 발견될 수 있다.


![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Old,%20Backup%20&%20Unreferenced%20Files/image-2.png)
- Exploit-DB 에서도 이러한 구글 검색필터를 통한 취약점을 도출할 수 있는 필터를 공유하고 있다.