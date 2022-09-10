---

layout: post
title: SAGA(Orchestration)
categories:
- MSA
tags:
- MSA
- Transaction
- Compensation
date: 2022-09-03
toc: true
comment: true

---

MSA 서비스간 보상 패턴 (Orchestration)
{: .message}

## 설명

MSA 환경에서는 서비스간 별도의 DB를 운영해 환경을 격리시킨다.

이럴 경우 명확한 서비스 분리, DB 수준에서 발생가능 한 장애 격리 등의 장점이 있는데…

반대로 서비스간 보관하는 데이터의 정합성이 불일치 할 수 있다!

### 예를 들면?

아래 예시를 보자!

{% highlight text %}

✉️ Subscribe Service (구독서비스 가입 관리가 목적인 서비스)
💵 Pay Service (실시간 결제가 목적인 서비스)
🗄 Balance Service (통장 잔고 관리가 목적인 서비스)

⚠️ 구독서비스 가입시 → 실시간 결제가 필요하며 → 통장 잔고의 차감이 필요하다!!
{% endhighlight %}

위의 예시에서 데이터의 정합성이 보장 안되는 경우는 다양하게 발생할 수 있다.

그 중 심각한 케이스는 다음과 같다.

{% highlight text %}
🧐 결제까지 완료해, 통장 잔고는 줄었으나….구독서비스 DB 장애로 가입이 실패했다!
😡 사용자 입장에서는 서비스 가입도 안됐고, 돈만 줄어든 어처구니 없는 상황일 뿐이다!
{% endhighlight %}

- Pay 서비스는 기존 결제내역이 취소되었다는 처리가! **반드시 필요하다!**
- Balance 서비스는 잔고의 환불이! **반드시 필요하다!**

**반드시 필요한 처리***(보상트랜잭션이라고 부른다)* 를 누가 책임감을 갖고 완수하느냐

👆 이게 Orchestration / Choreography 를 나누는 기준이라는데…

~~뭐 효율적으로만 처리되면 되지않을까?~~ 여튼 **보상트랜잭션을 잘 만들어주는**게 관점이다!

---

## Sequence Diagram

![/assets/img/SAGA(Orchestration).png](/assets/img/SAGA(Orchestration).png)

---

## 주의

- 위의 다이어그램은 Kafka 기반으로 요청을 주고 받는 경우를 예시로 했다.
    - 이벤트를 수신했는데, 뭔가 처리를 못해서 그 내용을 전파해줘야 하는 경우도 있을 수 있으니까!
- 물론 Main Service 의 최초 API 요청시(main thread)에,
    - 타 서비스에 처리 요청이 필요한 경우
    - (main thread) 내에서 책임지고 보상트랜잭션까지 발생시키는
    - 설계가 일반적이고 옳은 방법이지 않을까?(더 보편적인 방법이겠다!)
- 보상트랜잭션도 처리를 못한다? → retry & alert 가 필요하겠다!
- 조회의 경우는 보상트랜잭션이 필요없다!! (데이터 CUD 만 Rollback 하면 되니까!)