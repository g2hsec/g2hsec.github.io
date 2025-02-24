---
layout: single
title: amass
categories: Tools
tag: [amass, attacks, tool, tools, pentest, pentest tool,recon]
toc: true
author_profile: false
---

# amass 란 무엇인가? 

> OWASP 에서 제작한 Amass는 오픈소스 수집과 능동적 정찰을 사용하여 공격 표면의 네트워크 매핑 및 자산을 검색한다. 서브도메인 인텔리전스와 패시브 및 액티브 정보 수집, 연결 관계 분석, 다양한 API를 지원하는 강력한 도구이다.

# Information Gathering Techniques

| **Technique**    | **Data Sources** |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **APIs**        | 360PassiveDNS, Ahrefs, AnubisDB, BeVigil, BinaryEdge, BufferOver, BuiltWith, C99, Chaos, CIRCL, DNSDB, DNSRepo, Deepinfo, Detectify, FOFA, FullHunt, GitHub, GitLab, GrepApp, Greynoise, HackerTarget, Hunter, IntelX, LeakIX, Maltiverse, Mnemonic, Netlas, Pastebin, PassiveTotal, PentestTools, Pulsedive, Quake, SOCRadar, Searchcode, Shodan, Spamhaus, Sublist3rAPI, SubdomainCenter, ThreatBook, ThreatMiner, URLScan, VirusTotal, Yandex, ZETAlytics, ZoomEye |
| **Certificates** | Active pulls (optional), Censys, CertCentral, CertSpotter, Crtsh, Digitorus, FacebookCT |
| **DNS**         | Brute forcing, Reverse DNS sweeping, NSEC zone walking, Zone transfers, FQDN alterations/permutations, FQDN Similarity-based Guessing |
| **Routing**     | ASNLookup, BGPTools, BGPView, BigDataCloud, IPdata, IPinfo, RADb, Robtex, ShadowServer, TeamCymru |
| **Scraping**    | AbuseIPDB, Ask, Baidu, Bing, CSP Header, DNSDumpster, DNSHistory, DNSSpy, DuckDuckGo, Gists, Google, HackerOne, HyperStat, PKey, RapidDNS, Riddler, Searx, SiteDossier, Yahoo |
| **Web Archives** | Arquivo, CommonCrawl, HAW, PublicWWW, UKWebArchive, Wayback |
| **WHOIS**       | AlienVault, AskDNS, DNSlytics, ONYPHE, SecurityTrails, SpyOnWeb, WhoisXMLAPI |


## 설치 방법

***다양한 설치 방법과 도커를 활용한 방법이 존재하나, 해당 포스팅에서는 Kali Linux를 기반으로 설명합니다.***

```
apt-get update
apt-get install amass
```

## 기본적인 사용 방법 (서브 도메인 스캔)

```
amass enum -d example.com
```

<br>
이 외에도 다양한 옵션과 함께 사용이 가능하다.
<br>

1. 패시브 정보 수집 모드

    ```
    amass enum -passive -d example.com
    ```

2. 액티브 정보 수집 모드드

    ```
    amass enum -active -d example.com
    ```

3. 외부 데이터 소스 활용

    ```
    amass enum -d example.com -src
    ```

4. 서브도메인 브루트 포싱

    ```
    amass enum -d example.com -brute
    ```

5. 네트워크 그래프 생성

    ```
    amass enum -d example.com -json results.json
    ```

<br>

# 특징
1. 정서브 도메인 인텔리전스 활용 및 탐색색
2. 패시브 및 액티브 모드 
3. 도메인과 IP간의 관계를 시각화하는 연결 관계 분석