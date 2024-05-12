---
layout: single
title: (bWAPP)Insecure SNMP Configuration
categories: bwapp
tag: [SNMP, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> SNMP란 네트워크 장비나 시스템을 관리하고 감시하기 위한 프로토콜이며, 주로 네트워크 장비나 서버등과 같은 IT시스템에서 사용, SNMP를 통해 장비의 상태정보를 수집 및 설정을 변경할 수 있다. 또한 SNMP는 실제 장비나 시스템에서 실행되는 "에이전트" 데몬과 관리자 컴퓨터에 실행되는 "메니저" 가 서로 통신을 하게된다. 매니저는 에이전트를 통해 상태를 확인하게된다. 이러한 SNMP 데몬 구성이 올바르게 이루어지지 않을 경우, 메니저와 에이전트간 인증정보롤 사용되는 "커뮤니티 문자열"이 평문으로 전송되어 노출되거나, 구 버전 사용시 알려진 취약점 혹은 명령주입과 같은 보안 취약점으로 이어질 수 있다.

![그림 1-1](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image.png)
- SNMP 데몬 구성이 올바르게 이루어지지 않고 있다고 한다.

![그림 1-2](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-1.png)
- 포트스캔을 통해확인한 결과 SNMP가 동작중이며, 버전은 SNMPv1을 사용중이다.
- SNMPv1의 경우 보안 기능이 없으며, 커뮤니티 문자열만 일치할 경우 통신이 가능하므로 매우 취약하다.

![그림 1-3](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-2.png)
- 우선 MSF 를 통해 NMSP 관련 모듈중 snmp_login 이라는 모듈을 사용했다.
- 해당 모듈은 snmp의 커뮤니티 문자열을 부르트포싱 하여 찾아주는 모듈이다.

![그림 1-4](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-3.png)
- 스캔이 완료되고 2개의 커뮤니티 문자열을 찾아냈다.

![그림 1-5](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-4.png)
- 와이어샤크를 통한 패킷을 보면 수많은 request로 부르트포싱을 통해
- 2개의 응답을 확인할 수 있다.

![그림 1-6](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-5.png)
- "public" 커뮤니티 문자열

![그림 1-7](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-6.png)
- "private" 커뮤니티 문자열

![그림 1-8](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-7.png)
- snmp의 상세 정보 확인을 위해 snmp_enum 모듈을 사용했다.
- 커뮤니티 문자열이 public로 설정되어 있어 공격 대상 호스트 주소만 입력하면 된다. 

![그림 1-9](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-8.png)
- 공격 대상 서버의 정보를 확인할 수 있다.

![그림 1-10](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-9.png)
- 또한 udp 플러딩을 통한 공격도 가능하다.

![그림 1-11](/assets/image/bwapp/Security%20Misconfiguration/Insecure%20SNMP%20Configuration/image-10.png)
- 공격 대상 서버의 CPU 정보를 보면 과도한 사용량을 보이는 걸 알 수 있다.