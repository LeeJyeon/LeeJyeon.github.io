---

layout: post
title: Parallel-Process(Async)
categories:

- MSA
tags:
- MSA
- Transaction
- Parallel
date: 2022-08-28
toc: true
comment: true

---

비동기 요청/응답 기반의 병렬처리
{: .message}

## 설명

Kafka 와 같은 비동기 Message Queue 기반의 서비스간 이벤트 발행을 응용한 사례이다.

### 1️⃣ 단건 요청

일반적으로 1건에 대해서는 아래와 같다.

1. A Service → B Service (Request Topic)을 통해 요청한다. [토픽을 따로 두지않고, 메시지 내에 구분자로 구분해도 된다]
2. B Service → A Service (Response Topic)을 통해 응답한다.

A와 B 서비스는 비동기 메시지로 데이터를 주고 받는다

프로세스 처리 중 대기시간이 최소화 된다. (점유된 자원이 대기하는 시간이 최소화된다.)

### *️⃣ 다건 요청

A Service 에서 B, C Service 로의 요청 및 응답이 필요하다.

필요한 요청 내역을 Kafka 를 통해 이벤트 발행 후, 다른 이벤트가 처리한 내역을 기다리면 된다.

- 기존
    - A → B 처리 요청 및 응답
    - A → C 처리 요청 및 응답
- 다건 병렬요청
    - A → B , C 에 요청
    - B , C 각자 처리 후 응답

---

## Sequence Diagram

![Parallel-Process(Async)](/assets/img/Parallel Process(Async).png)

---

## 단점?

- 구조가 너무 복잡하다…
    - 토픽을 통해 요청/응답 받는 구조 차제가 매우 복잡하다.
    - 관리해야할 토픽의 개수가 많아지거나, 메시지의 구조가 복잡해지기 마련이다.
- Thread Safe
    - 응답을 받는 서비스(최초에 다른 서비스들에게 요청을 뿌렸던 메인 서비스)가 Thread Safe 하게 응답들을 처리하기 위한 공유 자원 사용이 필요하다.
    - Redis Lock 을 통해 구현해보았지만, 아무래도 그냥 Table Lock 잡고 사용하는게 여러모로 편하다..
- 보상 트랜잭션
    - 더군다나 실패할 경우, 보상 트랜잭션을 발생시키는 정책? 룰? 에 대한 추가 정의도 필요하다.
    - 전체 보상 메시지를 발행하고, Consumer 들이 알아서 처리or무시
    - 실패한 요청에만 보상을 던질지… 등

## 대안?

관련해서 **Java Stream Parallel** 처리 등의 대안이 충분히 있어보인다.

하지만, Thread pool 관리, Time out 관리 등의 부가적으로 고려해야할 부분이 있어서 고민해보자!