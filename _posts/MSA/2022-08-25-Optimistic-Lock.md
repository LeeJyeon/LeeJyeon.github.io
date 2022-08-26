---
layout: post
title: Optimistic-Lock
categories:
  - MSA
tags:
- MSA
- Database
- Lock
date: 2022-08-25
toc: true
comment: true
---
다발적으로 접근하는 DB Lock을 Lazy하게 처리
{: .message}

## 설명

- 비관적 락(Pessimistic Lock)
1. 일반적으로 알고있는 Database Table Lock 획득
2. 다른 세션의 접근을 Lock 획득시 ~ 반환시점 까지 막게된다.
3. 한 세션이 오랜기단 동안 자원을 점유하면, 다른 세션은 Time-out limit 까지 기다릴 수 밖에 없다.

- 낙관적 락(Optimistic Lock)
1. Table에 version 정보라던지, 최근 접근자라던지, 하물며 timestamp 라던지의 정보로 where condition 추가 생성
2. 데이터 조회시점에 **1.**의 정보를 애플리케이션 단에서 보유하고 있는다.
3. CUD 시점의 **2.**의 정보를 같이 조합해서 DB에 요청함
4. 바로 Commit 해, 자원을 반환해야 의미가 있음!!

## Sequence Diagram
![Optimistic-Lock](/assets/img/Optimistic Lock.png)

## 관련 문서
- [Optimistic Locking with JPA](https://hackernoon.com/optimistic-and-pessimistic-locking-in-jpa)
- [JPA에서 Optimistic Lock과 Pessimistic Lock](https://skasha.tistory.com/49)

## 주의
Exception 발생시, 수정한 데이터에 대해서 **직접 원복** 처리가 필요하다

말이 Rollback이지, 데이터를 과거버전으로 다시 업데이트하는 느낌?!

Rollback 하려는데, 데이터의 version이 더 많이 수정되었다면 골치아프다..

- example
1.(name:first / version:1) → (name:second / version:2) 바꾸고
2.이후 로직을 진행하다가, Exception 발생
3.(name:second / version:2)를 찾아 (name:first / version:3) 원복하려는데
4.다른 곳에서 이미 (name:ninth / version:5)가 되있음

▶︎ 단순히 4~5 버전으로 다시 수정하면 되지않느냐?

다른 곳에서 정합하게 이름을 "아홉째" 로 바꿨는데....

내가 다시 바꿔줘도 되는지 알 수 없음!!