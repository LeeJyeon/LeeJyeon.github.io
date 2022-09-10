---

layout: post
title: SAGA(Choreography)
categories:
  - MSA
tags:
- MSA
- Transaction
- Compensation
date: 2022-09-10
toc: true
comment: true

---

MSA 서비스간 보상 패턴 (Choreography)
{: .message}

## 설명

MSA 환경에서는 서비스간 별도의 DB를 운영해 환경을 격리시킨다….

[SAGA(Orchestartion)](https://leejyeon.github.io/msa/2022/09/03/SAGA(Orchestration)/)

이에 따라 데이터 정합성 불일치 등의 문제가 발생할 수 있다! (문제가 되는 케이스는 위 포스트에 있다!)
이에 대한 **“보상트랜잭션"** 처리가 필요한데, 그 패턴 중 하나이다!

---

## Sequence Diagram

![SAGA(Choreography)](/assets/img/SAGA(Choreography).png)

---

## 주의

- 위의 다이어그램에서 보다시피, 서비스간 호출이 연쇄되어 깊이가 깊어지면 확인이 불가능한 수준에 이르를 수 있다…
- 중간에 보상이 실패한 경우라도 발생하면 어찌할 수가 없다…
- 따라서 매우 **국소적인 상황**에만 적용해야하지 않을까 싶다!

---

## SAGA Pattern?

*패턴 자체에 대한 고찰*

Rest Request, Event Broker 를 통한 데이터 처리 요청(Create/Update/Delete) 모두 보상 트랜잭션이 필요한 경우가 발생한다.

여기서 보상트랜잭션은 기본적으로 Rollback 이 목적이나,

- 실제로는 트랜잭션이 이미 완료된 타 애플리케이션의 Rollback은 진행될 수 없기 때문에
- Rollback 과 유사한 처리가 진행되도록 **또다른 트랜잭션**을 요청하는 것이다.

이러한 트랜잭션의 요청을 Orchestator 가 처리하느냐 (Orchestration SAGA Pattern) , 각자 수행하느냐 (Choreography SAGA Pattern) 의 차이가 있다. 사실 **어떻게든 처리**가 되기만 하면된다. ~~누가 요청하는지 알게 무어냐..~~

다만, 애플리케이션 아키텍쳐의 복합도가 증가할 수록 관리, 운영, 모니터링 등의 이슈에서 체계적인 보상 트랜잭션 관리는 필요하다. 이에 대한 포스트는 추후 작성토록 해야겠다!