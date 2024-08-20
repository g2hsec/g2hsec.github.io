---
layout: single
title: HA Joker CTF
categories: THM
tag: [THM,TryHackMe, Pentest, Pentesting, HA Joker CTF]
toc: true
author_profile: false
---

해당 문제는 [https://tryhackme.com/r/room/jokerctf](https://tryhackme.com/r/room/jokerctf) 에서 확인할 수 있습니다.

***

# Dreaming 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. 디렉터리 부르트 포싱으로 /app 경로 확보
3. pluck CMS 4.7.13 의 CVE가 존재
4. 웹쉘 업로드를 통한 쉘 획득

## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
sudo nmap -p- --open -sV -T 4 -n -o result  {target_ip}
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-08-07 08:58 EDT
Nmap scan report for 10.10.195.254
Host is up (0.43s latency).
Not shown: 65497 closed tcp ports (reset), 35 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8080/tcp open  http    Apache httpd 2.4.29
Service Info: Host: localhost; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 2 IP addresses (1 host up) scanned in 127.00 seconds
```

- 22(ssh), 80(http), 8080(http) 서비스 동작중
- 두 개의 포트로 웹 서비스가 구동중인 것을 확인 했다.
- 우선적으로 웹 서비스부터 탐색을 시도했다.

![그림 1-1](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image.png)
- 조커 그림이 존재하는 웹 페이지가 존재했으며, 유의미한 정보획득 및 특정 기능이 존재하지는 않았다.
- 추가적인 경로가 존재하는지 확인하기 위해 디렉터리 부르트포싱을 시도했다.

![그림 1-2](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-1.png)
- 디렉터리 부르트포싱 결과 secret.txt 파일을 발견했다.

![그림 1-3](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-2.png)
- joker, batman 두 명의 인물의 대화 내용이 담긴 텍스트 파일이었으며, 이를 통해 두 사용자명을 유추할 수 있다.
- 해당 80포트로 동작중인 서비스에서는 추가적으로 얻을 수 있는것은 없어 8080 포트 웹을 살펴보았다.

![그림 1-4](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-3.png)
- 해당 포트의 웹 서비스로 접속시 username과 password를 검증하도록 되어있다.
- 기본적인 default 값으로 시도해본 결과 성공하지 못하였다.

![그림 1-5](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-4.png)
- 이전에 획득한 joker 사용자 명을 토대로 hydra 를 통한 부르트포싱을 시도하였으며,.
- 성공적으로 hannah 라는 패스워드를 찾을 수 있었다.

![그림 1-6](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-5.png)
- 평범한 웹 페이지가 출력되었으며 ,추가적인 정보 획득을 위해 nikto를 사용했다.

![그림 1-7](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-6.png)
- nikto 결과 robots.txt에서 다양한 경로가 노출되고 있는듯 하다.

![그림 1-8](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-7.png)
- 다양한 경로가 노출되며, admin 페이지인 administrator페이지가 존재하는 걸 확인할 수 있다.
- 하지만 계정정보를 획득할 수 없어 우선 당장 접근은 불가하다.

![그림 1-9](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-8.png)
- nikto에서 추가적인 정보를 획득할 수 있었으며, backup.zip 파일이 존재했다.

![그림 1-10](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-9.png)
- 해당 백업 파일에 패스워드가 존재했으며,
- 이 또한 부르트포싱으로 해결했다.

![그림 1-11](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-10.png)
<br>

![그림 1-12](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-11.png)
- 해당 패스워드는  hannah로 설정되어있는 것을 확인했다.
- 성공적으로 압축해제 후 sql 파일을 획득했다.

![그림 1-13](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-12.png)
- 그 후 해당 파일에서 admin 문자열이 포함된 부분만 출력했을 경우 admin 계정과 그에 해당하는 해시화된 패스워드를 확인할 수 있다.

![그림 1-14](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-13.png)
- john 을 통해 크랙한 결과 admin 계정의 패스워드를 획득할 수 있었다.

![그림 1-15](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-14.png)
- admin 계정으로 Control Panel에 접속할 수 있었으며,
- 해당 패널을 살펴보던 중 템플릿이 존재했으며, 이 때 소스코드를 악의적으로 변경하여 외부에서 호출하여 Reverse Connection이 가능했다.

![그림 1-16](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-15.png)
- component.php 를 악용했으며, php Reverse connection 을 수행했다.

![그림 1-17](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-16.png)
- 성공적으로 shell을 획득할 수 있었다.
- 여기서 주의하여 볼 수 있는 정보가 있었다.
- 115(lxd) 그룹에 포함되어 있는 걸 확인할 수 있는데 lxd 그룹에 속해 있을 경우, 별도의 자격증명 없이 로컬시스템에서 권한 상승이 가능한 취약점이 존재한다.

![그림 1-18](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-17.png)
- 공격을 수행하기 위해 공격자 pc에서 몇가지의 준비가 필요하다.
- 각각의 단계를 통해 tar 파일을 획득할 수 있다.

![그림 1-19](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-18.png)<br>
![그림 1-20](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-19.png)
- 해당 tar파일을 희생자 pc에서 다운받아야 하므로, 웹 서버를 구동시켜 준다.

![그림 1-21](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-20.png)
- 성공적으로 tar 파일을 가져올 수 있다.

![그림 1-22](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-21.png)
- 성공적으로 이미지를 가져오고 리스트를 확인해보면 추가된걸 볼 수 있다.

![그림 1-23](/assets/image/write-up/thm/thm_HA%20Joker%20CTF/image-22.png)
- 그 후 루트디렉터리에 마운트하여 root 권한의 shell을 획득할 수 있다.
