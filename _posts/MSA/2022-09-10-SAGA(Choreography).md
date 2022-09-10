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

이에 따라 데이터 정합성 불일치 등의 문제가 발생할 수 있다!

이에 대한 **“보상트랜잭션"** 처리가 필요한데, 그 패턴 중 하나이다!

---

## Sequence Diagram

![SAGA(Choreography)](/assets/img/SAGA(Choreography).png)

---

## 주의

- temp