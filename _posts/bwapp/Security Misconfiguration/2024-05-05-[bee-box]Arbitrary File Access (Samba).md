---
layout: single
title: (bWAPP)Arbitrary File Access (Samba)
categories: bwapp
tag: [Misconfiguration,samba, IDOR, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> samba는 리눅스 혹은 윈도우 시스템에서 파일 및 프린터 공유를 위한 프로그램이다. 이 때 samba 서비스에 대한 잘못된 보안설정으로 서버에 존재하는 파엘 대한 접근이 가능하다고 한다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image.png)
- samba 서비스가 동작중인 거 같으며, 임의의 파일에 대한 읽기,쓰기와 같은 Access 가 가능하다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-1.png)
- Kali Linux를 통해 희생자 서버에 대해 포트스캐닝을 한 결과 139,445 번 포트 Samba 서비스 사용시 사용되는 일반적인 포트가 오픈되어 있으며, Samba 서비스가 동작중인 걸 알 수 있다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-2.png)
- smbmap을 통해 Share 확인 결과 다양하게 존재하였다.
- 이 때 tmp의 권한이 read, write 즉 읽기 쓰기가 모두 가능하게 설정되어 있다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-3.png)
- 해당 Share로 접근하였다.

![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-4.png)
- put 명령을 통해 공격자 서버의 임의의 파일을 희생자 pc의 tmp 경로에 업로드가 가능했다.
- get을 통한 파일을 가져오기에는 실패 했으며, 특정 디렉터리들 간의 이동은 가능했다. (권한이 부족하여 목록을 보지는 못함)

![그림 1-6](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-5.png)
- enum4linux 를 통해 확인해보면 writing 기능이 N/A 거부 되어있는 걸 알 수 있다.

<br>

> msfconsole 검색결과 해당 설정 미흡중 traversal 결험으로 공유된 tmp 디렉터리에 rootfs라는 새로운 디렉터리 생성 및 루트 파일시스템에 연결하는 익스플로잇 코드가 존재한다.


![그림 1-7](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-6.png)
- amdin/smb/samba_symlink_traversal 모듈이며 필요 설정값을 채운 후 실행하게 되면
- rootfs 디렉터리가 생성되었다고 한다.

![그림 1-8](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-7.png)
- 확인해보면 실제로 rootfs 디렉터리가 생성된걸 확인할 수 있다.

![그림 1-9](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-8.png)
- 실제 rootfs 디렉터리는 최상위 경로인 /(root) 디렉터리와 심볼릭 링크되어 이동이 가능하며
- passwd 파일 또한 get을 통한 다운이 가능하다.

![그림 1-10](/assets/image/bwapp/Security%20Misconfiguration/Arbitrary%20File%20Access/image-9.png)
- 성공적으로 탈취 및 확인이 가능하다.