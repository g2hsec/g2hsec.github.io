---
layout: single
title: (bWAPP)Insecure WebDAV Configuration
categories: bwapp
tag: [WebDAV, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> WevDAV란 HTTP 프로토콜을 기반으로 하는 웹 서버와 클라이언트간의 파일 공유와 협업을 위한 프로토콜 및 기술이다. 이러한 WebDAB를 사용하면 웹 서버에서 파일을 read/write 및 폴더를 생성 삭제가 가능하다. 이러한 작업을 웹 브라우저나 WebDAV 지원 클라이언트를 통해 가능하다. 파일 공유 및 협업에 유용하지만, 제대로 구성하지 않을 경우 보안적 취약점을 발생시킨다. 대표적으로 디렉터리 리스팅, 잘못된 헤더 설정으로 인한 임의 파일 업로드 등이 존재한다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image.png)
- 아파치 웹 서버가 동작 중이며, WebDAV가 안전하지 않게 구성되어 있다고 한다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-1.png)
- 실제 WebDAV 경로가 존재하며, 접근이 가능하다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-2.png)
- 또한 존재하는 파일 및 디렉터리들로 접근이 가능하여 디렉터리 리스팅에도 취약하다.
- phpinfo 파일을 열거하여 서버의 php정보를 확인할 수도 있다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-3.png)
- 해당 WebDAV 페이지에 대한 허용된 메서드를 확인해본 결과
- 여러 메서드가 허용되지만 그 중 PUT또한 허용되고 있다.
- PUT 가 허용될 경우 WebDAV경로에 사용자가 임의의 파일을 업로드 할 수 있게 된다.

윈도우에서 웹쉘을 버프스윗을 통해 업로드하거나, 리눅스 상에서 WebDAV전용 cadaver이라는 명령을 통해 파일 업로드, 삭제, 이동등의 작업을 수행할 수 있다.

![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-4.png)
- 임의의 파일을 업로드 해보았다.
![그림 1-6](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-5.png)
- 성공적으로 업로드 된 걸 볼 수 있다.

![그림 1-7](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-6.png)
- 웹쉘 또한 성공적으로 업로드가 된다.

![그림 1-8](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20WebDAV%20Configuration/image-7.png)
- 웹쉘 실행이 가능하며,
- 서버 장악이 가능하다.