---
layout: single
title: (bWAPP)Insecure DOR - Insecure DOR (Order Tickets)
categories: bwapp
tag: [Insecure DOR, Access Contorol, IDOR, bwapp, bee box, session, OWASP TOP 10, OWASP]
toc: true
author_profile: false
---


![그림 1-1](/assets/image/bwapp/idor/order-tickets-archive/image.png)
- 어떠한 티켓을 판매하는 기능이 구현되어 있다.
- 티켓 구매에 대한 개수 제한은 없으며, 한장당 15유료에 판매되고 있다.

![그림 1-2](/assets/image/bwapp/idor/order-tickets-archive/image-1.png)
- 구매시 즉시 구매가 되며, 15 유료가 자동으로 빠졌다고 되어 있다.

```
ticket_quantity=1&ticket_price=15&action=order
```

- request 값을 보게되면, ticket_quantity 파라미터로 티켓의 개수 그리고 ticket_price 파라미터로 값이 지정되어 요청되는 걸 알 수 있다.

```
ticket_quantity=10&ticket_price=0&action=order
```

- 이 때 개수를 10개, 가격을 0으로 임의 조작하여 요청값을 전송 해보았다.

![그림 1-3](/assets/image/bwapp/idor/order-tickets-archive/image-2.png)
- 공격자의 의도에 맞게 10개의 티켓을 0유료로 구입이 가능하다.
