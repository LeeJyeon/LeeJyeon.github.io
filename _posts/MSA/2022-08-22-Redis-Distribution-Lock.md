---
layout: post
title: Redis-Distribution-Lock
categories:
  - MSA
tags:
- MSA
- Redis
date: 2022-08-22
toc: true
comment: true
---
분산 환경에서 다발적으로 인입하는 Transaction 제어
{: .message}

## Redis-Distribution-Lock
기본적인 Sequence Diagram 은 아래와 같다.

![Redis-Lock](/assets/img/Transaction Lock-Redis.png)

## 관련 문서
[Distributed Locks with Redis | Redis](https://redis.io/docs/reference/patterns/distributed-locks/)
[RLock - redisson](https://www.javadoc.io/doc/org.redisson/redisson/2.8.2/org/redisson/api/RLock.html)

## 설명

1. DB 를 통한 Locking 까지 도달하기 전에, Controller (API, Kafka Listener) 단계에서 Trasaction 마다 Locking 한다.
2. 물론, Transaction Lock 을 위한 Key 는 비즈니스 로직에서 정의 해야한다. (User_id, 획득한 Session 등…)
3. 두 가지 Time Limit 이 설정 필요함
- Wait Time : 새로 인입한 트랜잭션의 Lock이 선점되있을 경우, 기다리는 시간
- Release Time : 획득한 Lock 을 유효시간 동안만 유지하고 해제하게 됨

## 주의
Release Time 을 **직접 제어** 하기 때문에 RDB 만큼 절대적으로 제어할 수 없음

**동일 트랜잭션이 밀려들어오는 상황을 선제적으로 제어** 하는 정도로만 이해하고 사용하자!