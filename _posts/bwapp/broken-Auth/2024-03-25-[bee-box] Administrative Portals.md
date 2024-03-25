---
layout: single
title: (bWAPP)Session Mgmt. - Administrative Portals
categories: bwapp
tag: [broken auth, auth, session mgmt, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

## Level - Low

![그림 1-1](/assets/image/bwapp/Broken-Auth/%20Administrative%20Portals-archive/image.png)
- 해당 페이지에 대한 접근이 거부되고 있다.
- 주어지는 힌트로는 URL을 검사하라고 한다.
- 해당 시나리오는 일반 사용자가 관리자 페이지에 접근할 수 있는 취약점으로 확인된다.

```
https://192.168.146.133/bWAPP/smgmt_admin_portal.php?admin=0
```

- 접근 URL 을 확인하면 GET 형식으로 admin 파라미터에 0이라는 값이 설정되어 있다.
- 즉 관리자임을 확인하는 파라미터가 노출되며 GET형식으로 전달되고 있다.

```
https://192.168.146.133/bWAPP/smgmt_admin_portal.php?admin=1
```

![그림 1-2](/assets/image/bwapp/Broken-Auth/%20Administrative%20Portals-archive/image-1.png)

- 해당 admin 파라미터의 값을 1로 수정시 해당 페이지에 접근할 수 있다.

## Level - Medium & High

```
https://192.168.146.133/bWAPP/smgmt_admin_portal.php
```

- medium Level 의 경우 파라미터가 함께 넘어가지 않는다.

```
Cookie: PHPSESSID=ebe02e2802cd17b103c8f0ecd54eb740; security_level=1; admin=0
```

- 버프스위트를 통해 요청 패킷을 잡아보면
- Cookie 를 통해 admin 값이 넘어가며, 이를 1로 수정하여 전송시
- 해당 페이지에 접근이 가능하다.

> high 레벨은 dba에 존재하는 계정이 존재해야하며, bee 계정으로 접근시 해제된다.