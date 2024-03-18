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

![](/assets/image/image.png)
```
gobuster dir -u 10.10.182.2 -w /usr/share/wordlists/dirb/common.txt -f
```