---
layout: single
title: (bWAPP)Insecure FTP Configuration
categories: bwapp
tag: [FTP, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> FTP 서버의 구성이 안전하지 않다는 시나리오이며, FTP 는 파일 전송 프로토콜로 서버와 클라이언트 간에 전송하는데 사용되는 네트워크 프로토콜이다. 이러한 FTP 프로토콜의 대표적인 보안적 이슈는 "암호화 되지 않은 평문 전송으로 인한 중간자 공격, Anonymous 유저 허용으로 인한 보안 이슈, 알려진 취약점(CVE)" 등이 존재한다. 이로 인해 ProFTPD와 같은 취약한 프로토콜 대신 sftp, ftps와 같은 보안 전송 프로토콜 사용이 권장된다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20FTP%20Configuration/image.png)
- 동작중인 FTP 서버에 구성설정이 안전하지 않다고 한다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20FTP%20Configuration/image-1.png)
- 우선 FTP 서비스에 대한 대표적인 보안 취약점중 하나인 Anonymous 계정의 접근 여부를 확인 했다.
- 역시 Anonymous 계정으로의 접근이 가능했으며, 이럴 경우, 서버 내에 임의 파일을 업로드/다운로드가 가능할 수 있는 위협이 존재한다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20FTP%20Configuration/image-2.png)
- 존재하는 파일들 또한 확인이 가능했으며, php로 동작중인 서버의 phpinfo 정보가 담긴 파일 또한 공격자 pc로 다운로드가 가능했다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20FTP%20Configuration/image-3.png)
- 또한 FTP의 대표적인 취약점인 암호화를 하지 않은 평문 데이터 전송을 확인할 수 있다.
- 만약 동일 네트워크에서 누군가 FTP 서버에 접속하게 될 경우 악의적인 사용자는
- 이를 스니핑 할 수 있게 된다. 즉, 계정 정보 및 어떤 파일을 다운로드 하는지 어떤 작업을 수행하는지 확인이 가능하다.

![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20FTP%20Configuration/image-4.png)
- 또한 해당 서버는 ProFTPD 1.3.1 이라는 오래된 버전을 사용하고 있다.
- 이렇게 오래된 버전을 사용할 경우 보안 패치가 되지 않아, 알려진 취약점에 노출되게 된다.