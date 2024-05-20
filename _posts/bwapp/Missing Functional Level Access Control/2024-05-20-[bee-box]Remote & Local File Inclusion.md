---
layout: single
title: (bWAPP)SQLiteManager Local File Inclusion
categories: bwapp
tag: [LFI, bwapp, bee box, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---

# 취약점 설명
> 해당 취약점은 응용 프로그램에서 File을 불러 올 때 include를 사용하여 코드내에 Built 하거나 동적으로 파일을 포함하는 로직이 구현되어있을경우 일반적으로 나타나며, 다른 Path Traversal 과 같은 타 취약저믈과 연계될 수 있다. 이러한 File Inclusion은 Local File Inclusion(LFI)와 Remote File Inclusion(RFI)로 나뉜다.