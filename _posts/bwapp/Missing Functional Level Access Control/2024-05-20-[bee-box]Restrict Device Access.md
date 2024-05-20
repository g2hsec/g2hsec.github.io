---
layout: single
title: (bWAPP)Restrict Device Access
categories: bwapp
tag: [LFI, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> Restrict Device Access 취약점은 디지털 기기와 시스템에서 특정 디바이스나 네트워크 리소스에 대한 접근을 제한하는 데 사용되는 보안 메커니즘에 존재하는 취약점을 의미한다.

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Restrict%20Device%20Access/image.png)

- 일부 인증된 디바이스만 접근할 수 있으며, 현재 접근 매체는 스마트폰 혹은 컴퓨터가 아니라는 문구가 출력된다.

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
```

해당 페이지에 접근하는 요청 패킷을 가로채 보면, User-Agent 값이 Windows 인 것을 볼 수 있다.
<br>
해당 부분을 모바일 기기로 악의적인 수정을 통해 해당 페이지에 접근이 가능하다.

```
User-Agent: Mozilla/5.0 (Android NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36
```

- 위와 같이 Android로 변경 후 요청을 보내보았다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/Restrict%20Device%20Access/image-1.png)
- 성공적으로 해당 페이지에 접근이 가능했다.