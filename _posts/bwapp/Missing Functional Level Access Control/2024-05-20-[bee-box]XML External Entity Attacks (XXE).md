---
layout: single
title: (bWAPP)XML External Entity Attacks (XXE)
categories: bwapp
tag: [LFI, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 웹 어플리케이션에서 브라우저와 서버간에 데이터 전달시 XML을 사용할 경우 XML 파서의 잘못된 파싱방식으로 인해 외부 엔터티를 참조하게될 경우 발생할 수 있는 취약점이다. 해당 취약점은 XML 파서가 위치한곳으로 부터 공격이 이루어지기 때문에 SSRF, RCE등의 취약점으로 이어질 수 있다.

![그림 1-1](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/XML%20External%20Entity%20Attacks%20(XXE)/image.png)
- secret 값을 리셋 시켜주며, 해당 버튼 클릭시 xml 형식으로 데이터가 전송된다.

```
<reset><login>bee</login><secret>Any bugs?</secret></reset>
```

- 기존 xml을 이용한 전송은 위와 같이 이루어진다.
- 이를 악의적으로 변경하여 내부 시스템의 파일을 읽어들일 수 있다.

![그림 1-2](/assets/image/bwapp/Missing%20Functional%20Level%20Access%20Control/XML%20External%20Entity%20Attacks%20(XXE)/image-1.png)
- 이와 같이 해당 경로에 존재하는 파일들을 읽을 수 있다.