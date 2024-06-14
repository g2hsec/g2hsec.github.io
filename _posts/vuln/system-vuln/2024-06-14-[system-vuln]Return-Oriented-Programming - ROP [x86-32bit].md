---
layout: single
title: Open Return-Oriented-Programming - ROP [x86-32bit] 
categories: system-vuln
tag: [rop, pwn, pwnable, systemhacking, Return Oriented Programming]
toc: true
author_profile: false
---
# 해당 취약점은 무엇인가?
> ROP기법이란 Return-Oreinted-Programming의 약자로 반환 지향형 프로그래밍 이다. ret영역에 가젯을 이용하여 연속적으로 함수를 호출하며 공격자가 원하는 흐름대로 프로그래밍 하듯 공격한다는 뜻의 기법이다.

## 사전 지식
ROP는<br>
ASLR, DEP/NX등의 메모리 보호기법을 우회하여 공격할 수 있는 기법이며,<br>
해당 기법을 사용하기위해서는 몇가지의 사전지식이 필요하다.<br>
<span style="color: red; text-decoration: underline;">우선 RTL, RTL-Chaining, PLT,GOT, GOT_Overwrite, Gadget등이 필요하다.</span><br>

**<u>해당 기법은 <br>
Stage0과 Stage1 로 나뉘어서 진행되게 된다.</u>** 
{: .notice--primary} 

<span style="font-weight: bold; color: red;">첫번 째 Stage1 에서는 공격을 하기위해 필요한 요소들을 구하는 과정이고<br>
두번 째 Stage0 에서는 Stage0에서 구한 요소들을 가지고 Exploit 하는 과정이다.
 </span>

즉 Libc_base주와 함수의 실제주소, 가젯 등 필요 요소를 Stage1에서 구하고<br>
이를 통해 Stage0 에서 Payload를 작성하요 Exploit을 하게되는 것이다.<br><br>

x86-32bit 환경에서는 가젯을 구하기 어렵지 않지만 64bit 환경에서는 필요 레지스터가 존재한다<br>
우선 32bit부터 살펴보도록 하겠다.<br><br>

기본적인 원리부터 알아보자.

<img src="/assets/image/vuln/system-vuln/rop-32/image.png" alt="description" width="600"/><br>
위와 같은 상황에서 Buffer Overwrite가 터져서 buf의 64바이트보다 더 많은 값을 입력받을 수 있을때<br><br>

우리는 RET영역까지 접근이 가능하다. 여기서 RTL_Chaining와 같이 가젯을 사용하여, 함수를 호출하고, 인자를 정리하고 또 호출하고를<br> 반복적으로 수행 할 수 있다. <br><br>

# Stage 1
이경우 보통 Stage1에서 해야할 작업으로, 출력함수를 이용하여 특정 함수의 got주소를 출력시킨 후, <br> 
출력된 함수의 got에서 그 함수의 offset값을 빼 libc_base주소를 구하게 된다.<br> 
이렇게 구한 libc_base주소를 가지고 우리가 원하는 보통 system()함수의 offset을 더해 system()함수의<br> 
실제 주소를 획득 할 수 있다. <br> <br> 

그 후 "/bin/sh" 문자열이 저장되어있는 주소를 찾거나, 쓰기권한이 있는 .bss영역등에 해당 문자열을 저장하여 해당 <br> 
/bin/sh 문자열을 쓸 수 있다.<br>

# Stage 0
got_overwrite를 통해 특정 함수의 got에 Stage1 단계에서 구한 system()함수의 주소를 덮어씌워<br> 
특정함수를 실행 시키게 되면 system()함수를 실행시키는 효과를 볼 수 있다.<br> <br> 
그 후 Stage0에서 Exploit을 진행하게 된다.<br> 
Stage0의 단계를 보면 아래의 그림과 같이 이루어진다.<br> 

<img src="/assets/image/vuln/system-vuln/rop-32/image-1.png" alt="description" width="600"/><br>

<div class="notice">

1. puts() 함수를 이용하여 puts()함수의 got주소를 출력시킨다. 출력시킨 Payload에서 특정 변수에 담아둔다.
그 후 puts_got - puts_offset를 통해 libc_base주소를 획득하고
libc_base - system_offset을 통해 system()함수의 실제 주소를 획득할 수 있다. 
2. read()함수를 통해 .bss영역에 system()함수의 인자로 필요한 /bin/sh문자열을 저장하게 된다.
3. read()함수를 통해 puts()함수의 got에 앞서 얻은 system()함수의 실제주소를 저장하여 got_overwrite를 한다.
이로서 puts_plt를 호출하면 got를 참조하여 system()함수가 실행되게 된다.
</div>
그 후 모든 요소들을 payload로 작성하여 Stage1의 Exploit를 수행하면 된다.

<img src="/assets/image/vuln/system-vuln/rop-32/image-2.png" alt="description" width="600"/><br>
위 사진과 같이 puts_plt를 실행시키면 내부적으로 system()함수가 실행되고 인자로 <br>
.bss영역의 주소를 주면 .bss영역에는 /bin/sh 문자열이 저장되어있기에<br>
system(/bin/sh)이 실행되며 해당 명령 실행 후 더이상 작업이 없기에 AAAA로 끝나게 된다.<br><br>   

🌝 **<u>(참고로 got_overwrite를 사용하지않고 Stage1 단계에서 구한 system()함수의 주소를 바로 써도 된다.)</u>** 
{: .notice--primary}    

## Payload
해당 ROP기법은 알아야 할 사전지식이 많으며, 그만큼 강력한 기법이다
마지막으로 write()함수와 read()함수를 가지고 rop기법을 이용한 전체 Payload를 참고하자.
<img src="/assets/image/vuln/system-vuln/rop-32/image-3.png" alt="description" width="600"/><br>

# 보안대책
1. NX(No-eXecute) 비트 설정
2. ASLR(Address Space Layout Randomization)
3. Control Flow Integrity (CFI)
4. Stack Canary:
5. 사용자 입력의 검증과 제한