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
nmap -p- --max-retries 1 -Pn -n --open --min-rate 5000 -T4 -sV -sC -A -oA ./Archangel 10.10.182.2
```