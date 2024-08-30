---
layout: single
title: Frame Pointer Overflow [FPO], SFP Overflow, SFPO
categories: system-vuln
tag: [FPO, SFPO, Frame Pointer Overflow, systemhacking, SFP Overflow]
toc: true
author_profile: false
---

# FPO? SFPO? 

> 시스템 해킹을 하다보면 보통 ret영역의 값을 악의적으로 변형시켜 system함수를 실행시키는 등 해당 프로그램을 제어하지만, 입력버퍼의 제한으로 ret영역까지 접근을 하지 못 할 수 있다. 하지만 SFP영역의 단 하위1바이트만으로 ret영역 제어와 비슷한 흐름을 이끌어 낼 수 있다. 그것이 바로 FPO기법이다.
<hr>

<div class="notice">
  <h4>해당 기법을 이해하기 위해서는 스택 에필로그 과정과 프롤로그 과정에 대한 이해도가 필요하다.</h4>
</div>

해당 기법을 사용하기 위해서는 2가지의 조건이 충족 되어야 한다.<br>
1. 서브 함수가 존재하여야 한다.
2. Buffer Overflow로 서브함수의 SFP의 최소 하위 1Byte를 덮을 수 있어야 한다.

대체 어떻게 ret를 건들지 않고 원하는 주소로 이동이 가능한 것일까?<br>
이는 함수 에필로그 과정의 eip를 이용하기 때문에 가능하다.<br><br>

우선 에필로그 부분을 보게되면.<br><br>

<span style="font-weight: bold; color: red;">"leave"</span><br>
<span style="font-weight: bold; color: red;">mov esp, ebp</span><br>
<span style="font-weight: bold; color: red;">pop ebp</span><br><br>

<span style="font-weight: bold; color: red;">"ret"</span><br>
<span style="font-weight: bold; color: red;">pop eip</span><br>
<span style="font-weight: bold; color: red;">jmp eip</span><br><br>

이렇게 leave와 ret로 되어있다. 이 두 부분을 악의적으로 사용하는 것이다.<br>
스택에서 main함수가 존재하고 새로운 함수가 호출되면 main함수 위에 쌓이게 되며,<br>
<span style="font-weight: bold; color: red;">이때 main함수의 ebp는 서브 함수의 SFP에 저장되게 된다.</span><br>

![그림 1-1](/assets/image/vuln/system-vuln/FPO_SFPO/image.png)<br>

위 진처럼 이루어지게 된다. 그럼 이제 본격적으로 에필로그 과정을 악용하여 사용하는 FPO기법에대해알아보자.<br>
아래 사진은 서브함수까지 호출 된 상황에서 에필로그 과정을 실행하기 이전의 모습이다.<br>

![그림 1-2](/assets/image/vuln/system-vuln/FPO_SFPO/image-1.png)<br>
그 후 1Byte 만큼 Overflow 시켜 하위 1바이트를 다른 값으로 변경시켰다.<br>
그 후 에필로그 과정의 첫 번째 "mov esp ebp" 와 "pop ebp"를 실행시키면<br>

![그림 1-3](/assets/image/vuln/system-vuln/FPO_SFPO/image-2.png)<br>
ebp와 esp의 위치가 같은 상태가 됩니다 해당 상태에서는<br>
<span style="font-weight: bold; color: red;">스택의 맨 위에는 sfp가 존재하게 되며,</span><br>
두 번째 pop ebp를 통해 sfp를 pop 하고 <span style="font-weight: bold; color: red;">ebp에 sfp의 값을 저장하게 된다.</span><br>
그럼 기존 main함수의 주소를 담고 있어야할 sfp가 조작된 0xffbf12d4라는 주소를 담고있기에<br>
ebp는 main의 ebp가아닌 저 0xffbf12d4의 위치로 이동하게 됩니다. 그후 esp는 ret를 가리키고 있다.<br><br>
 

그 후 pop eip를 통해 ret에 있던 기존값은 eip에 들어가게 됩니다. 그 후 jmp eip를 통해<br>
main함수에서 서브함수를 실행시킨 이후 실행된 명령어의 위치로 들어가게 된다.<br>
(eip = 다음 실행할 명령어의 위치를 담고있는 레지스터)<br>

<hr>
아래는 jmp eip를 통해 main으로 복귀후 main함수가 전부 종료되고, main함수의
에필로그 과정이다.

![그림 1-4](/assets/image/vuln/system-vuln/FPO_SFPO/image-3.png)<br>
기존 ebp는 서브함수의 에필로그 과정에서 조작한 다른 위치로 이동된 상태에서 main함수의 mov esp ebp를 통해<br>
ebp의 값이 esp로 복사 되게 되며, esp가 ebp의 위치로 이동하게 된다.

![그림 1-5](/assets/image/vuln/system-vuln/FPO_SFPO/image-4.png)<br>
<span style="font-weight: bold; color: red;">그 상태에서 pop ebp를 하게될 경우 기존에 가리키고있던 0xffbf12d4의 값이 pop 되며 ebp에 들어가게 되며
esp는 -4 위치인 0xffbf12d8을 가리키게 된다.</span><br>
**이때 0xffbf12d8 위치에 쉘코드 or 취약한 함수 의 주소라면??**<br>
해당 위치에서 "ret" 단계의 pop eip -> jmp eip를 통해 해당 부분으로 이동한 뒤 실행되게 된다.<br><br>


그럼 여기서 기억을 되짚어보면, 서브함수에서 1바이틀 악의적으로 변조했는대,<br>
이때 우리가 줘야하는 입력값은<br>
우리가 이동하고자 하는 함수 주소의 -4 한 값을 입력해줘야한다. <br>
(처음 부터 -4값을 입력해줘야합니다. 라고 설명했을 혼동으로인해 이해가 안될 수 있어 뒤에 설명 했다.)<br>
<hr>

# 결론 
1. 서브 함수에서 sfp의 값을 변조시킬 때 우리가 이동하고자 하는 주소의 -4의 값에 맞게 값을 변조시킨다.
2. mov esp, ebp -> pop ebp를 통해 ebp는 우리가 원하는 주소의 -4 위치로 이동하게 된다.
3. 메인 함수의 에필로그에서 동일하게 mov esp, ebp를 통해 main함수의 esp위치를 악의적으로 이동시킨 ebp위치와 동일한 위치로 이동시킨다.
4. pop ebp를 통해 원하는 주소의 -4한 주소를 pop ebp를 통해 ebp에 넣고 ebp는 그 위치로 이동한다
5. 그럼 esp는 기존 ebp의 +4된 주소 즉 원하고자 했던 위치를 가리키게 된다.
6. pop eip 를 통해 eip에 우리가 원하고자 하는 주소를 저장한다.
7. jmp eip를 통해 원하고자 하는 주소로 이동하게되며, 실행되게 된다.

<div class='notic'>
해당 기법은 함수의 스택, 에필로그에 대해 자세히 이해하고 있어야하며, ebp, sfp, esp, eip 등등의 역할등에<br>
대해 자세히 인지하고 있어야하고, 상당히 헷갈리는 기법이다. <br>
그만큼 ret까지 접근하지 않아도 된다는 큰 메리트를 가진 기법일 수 있다.
</div>

# Referenece
- https://dokhakdubini.tistory.com/228?category=809542