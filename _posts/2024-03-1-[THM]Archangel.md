---
layout: single
title: (THM) Archangel - WriteUP
categories: THM
tag: [log injection, RCE, LFI, log poisoning,THM,TryHackMe, Pentest, Pentesting]
toc: true
author_profile: false
sidebar:
    nav: "counts"
---

해당 문제는 https://tryhackme.com/room/archangel 에서 확인할 수 있습니다.

***

# Archangel 시스템 모의 침투
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집
```
nmap -p- --max-retries 1 -Pn -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./Archangel {target_ip}
```
```
Starting Nmap 7.93 ( https://nmap.org ) at 2024-03-18 01:27 EDT
Nmap scan report for {target_ip}
Host is up (0.42s latency).
Not shown: 49099 filtered tcp ports (no-response), 16434 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9f1d2c9d6ca40e4640506fedcf1cf38c (RSA)
|   256 637327c76104256a08707a36b2f2840d (ECDSA)
|_  256 b64ed29c3785d67653e8c4e0481cae6c (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Wavefire
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.49 seconds
```
- 22(SSH), 80(http) 포트로 서비스 동작중을 확인함.
- SSH의 경우 Username 열거에 관한 취약점 제외 유의미한 취약점은 없는듯 함.
- 80번 포트로 HTTP 웹 서비스가 서비스 동작중이므로, 해당 서비스를 중심으로 공격 방향을 잡음

![그림1-1](/assets/image/thm_archangel//image.png)
- 해당 주소 접속시 아무런 동작을 수행하지 않는 사이트가 존재함.
- 소스코드에서 유의미한 정보가 노출되어있지 않음
### Gobuster 을 통한 디렉터리 부르트포싱
```
gobuster dir -u {target_ip} -w /usr/share/wordlists/dirb/common.txt -f
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://{target_ip}
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess/           (Status: 403) [Size: 276]
/.hta/                (Status: 403) [Size: 276]
/.htpasswd/           (Status: 403) [Size: 276]
Progress: 598 / 4615 (12.96%)obuster dir -u 10.10.182.2 -w /usr/share/wordlists/dirb/common.txt -f
/flags/               (Status: 200) [Size: 934]
/icons/               (Status: 403) [Size: 276]
/images/              (Status: 200) [Size: 0]
/layout/              (Status: 200) [Size: 0]
/pages/               (Status: 200) [Size: 0]
/server-status/       (Status: 403) [Size: 276]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```
- 몇몇의 숨겨진 경로를 발견하였으나, 유의미한 경로는 아님
- 유의미한 디렉터리 및 파일은 발견되지 않았음, 
- 공격 대상 IP에 대해 가상 호스트 사용 가능성을 두고 웹 사이트 내 도메인을 찾아봄
- 해당 웹 사이트 우측 상단에 관리자의 이메일이 존재하였으며, 해당 도메인을 /etc/hosts에 타겟 시스템IP와 매칭시켜 접속을 시도해봄
![그림1-2](/assets/image/thm_archangel/image2.png)
- 플래그를 획득 할 수 있었음
### 추가 획득한 도메인에 대한 2차 디렉터리 부르트 포싱
```
gobuster dir -u http://mafialive.thm/ -w /usr/share/wordlists/dirb/common.txt -f -x php, txt
```
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://mafialive.thm/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/./                   (Status: 200) [Size: 59]
/.php/                (Status: 403) [Size: 278]
/.hta/                (Status: 403) [Size: 278]
/.hta.php/            (Status: 403) [Size: 278]
/.hta./               (Status: 403) [Size: 278]
/.htaccess/           (Status: 403) [Size: 278]
/.htpasswd/           (Status: 403) [Size: 278]
/.htaccess./          (Status: 403) [Size: 278]
/.htaccess.php/       (Status: 403) [Size: 278]
/.htpasswd.php/       (Status: 403) [Size: 278]
/.htpasswd./          (Status: 403) [Size: 278]
/icons/               (Status: 403) [Size: 278]
/server-status/       (Status: 403) [Size: 278]
/test.php/            (Status: 200) [Size: 286]

```
- test.php 경로 발견
![그림1-3](/assets/image/thm_archangel/image3.png)
- URI 확인시 view 파라미터를 통해 특정 경로의 파일을 include하고 있음.
- 이를 통해 LFI 취약점이 존재할 것이라고 판단함.


