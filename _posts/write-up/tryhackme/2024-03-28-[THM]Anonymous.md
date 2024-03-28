---
layout: single
title: (THM) t - WriteUP
categories: THM
tag: [write up, ftp, smb, ,THM,TryHackMe, Pentest, Pentesting, cron, Privilege elevation]
toc: true
author_profile: false
---

해당 문제는 [https://tryhackme.com/r/room/anonymous](https://tryhackme.com/r/room/anonymous) 에서 확인할 수 있습니다.

***

# Anonymous 시스템 모의 침투
## 수행 내용
1. 정보 수집
2. ftp 서비스에 Anonymous 계정으로 접근가능
3. ftp 접속시 scripts 경로에 특정 기능을 cron 작업으로 수행하는 듯한 스크립트 파일 존재
4. 스크립트 파일 Other 사용자에게 쓰기 권한 부여 확인, Reverse Connection 쉘 코드 삽입 후 Reverse Connection 성공
5. 시스템 내에 env 명령어에 대한 잘못된 SUID 설정 확인 후 root 권한 획득 
## 정보수집
### Nmap 스캔 - 사용중인 포트 및 배너, 기본 정보 수집

```
nmap -p- --max-retries 1 -Pn -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./Anonymous {target_ip}
```

```
# Nmap 7.93 scan initiated Thu Mar 28 04:49:41 2024 as: nmap -p- --max-retries 1 -Pn -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./anonymous 10.10.138.55
Nmap scan report for 10.10.138.55
Host is up (0.41s latency).
Not shown: 47284 filtered tcp ports (no-response), 18247 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.4.47.45
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8bca21621c2b23fa6bc61fa813fe1c68 (RSA)
|   256 9589a412e2e6ab905d4519ff415f74ce (ECDSA)
|_  256 e12a96a4ea8f688fcc74b8f0287270cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled but not required
|_clock-skew: mean: 0s, deviation: 1s, median: -1s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)
| smb2-time: 
|   date: 2024-03-28T08:50:53
|_  start_date: N/A
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2024-03-28T08:50:53+00:00
```

- 21(FTP), 22(SSH), 130,445(SMB) 포트로 서비스 동작중 확인
- FTP 서비스 취약점 스크립트 점검 결과 익명 계정에 대한 로그인이 가능하며, scripts경로 발견

### smbmap을 통한 Share 목록 및 파일, 속성, 사용자등 수집

![그림 1-1](/assets/image/write-up/thm_anonymous/image-1.png)
- SMB 서비스 내에 icps 공유 디렉터리 존재

![그림 1-2](/assets/image/write-up/thm_anonymous/image.png)
- 유의미한 정보는 획득하지 못함

### ftp서비스 접근 및 탐색

![그림 1-3](/assets/image/write-up/thm_anonymous/image-2.png)
- FTP 서비스에 Anonymous 계정으로 로그인 후 scripts 경로로 접근
- 3개의 파일이 존재
- clean.sh 쉘 스크립트에 Other 사용자에 대한 모든 권한이 주어져 있음.

## 취약점 분석

![그림 1-4](/assets/image/write-up/thm_anonymous/image-3.png)
- 파일 삭제 및 로그 기록을 하는 쉘 스크립트로 보이며, 지속적으로 실행되어야 하는 스크립트이므로
- cron 서비스로 돌아갈 것이라 추측
- 해당 파일에는 Other 사용자에게 모든 권한이 있으니 ftp로 접속하여
- 해당 clean.sh 스크립트를 공격자 PC로 Reverse Connection 하는 악성 코드로 변조가 가능함.

### clean.sh 파일 내 Other 사용자에 대한 불필요한 권한이 설정 확인

![그림 1-5](/assets/image/write-up/thm_anonymous/image-4.png)
![그림 1-6](/assets/image/write-up/thm_anonymous/image-5.png)

## 내부침투

- 성공적으로 변조 하였으며, 공격자 PC에서 nc를 통해 포트 오픈 후 대기
![그림 1-7](/assets/image/write-up/thm_anonymous/image-8.png)
- 성공적으로 namelessone 사용자 쉘을 획득함.

## 관리자 권한 획득
### env 명령어에 대한 잘못된 SUID 설정 확인

![그림 1-8](/assets/image/write-up/thm_anonymous/image-6.png)
- 침투 후 linpeas.sh 실행 결과 SUID 설정이 되어있으면 안되는 env 명령어에 대한
- 잘못된 SUID 설정이 되어있는 것을 확인

![그림 1-9](/assets/image/write-up/thm_anonymous/image-7.png)
- 이를 통해 root 권한으로 권한 상승에 성공