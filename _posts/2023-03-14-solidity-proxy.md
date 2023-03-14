---
layout: post
title: "Solidity: Proxy 정리"
categories: [Dev, Blockchain]
tags: [blockchain, solidity]

---

# 왜 필요한가?

> *스마트 컨트랙트를 통해 트랙잭션이 생성되며 이에 대한 검증과정을 거쳐 네트워크상에 추가된다. 이때 추가되는 기록과 배포된 스마트 컨트랙트 불변이다.*
>

여기서, 만약 배포된 스마트 컨트랙트에 취약점 또는 버그가 발견되었거나 기능을 추가하고 싶다면 어떻게 해야될까?

스마트 컨트랙트는 불변이므로 직접 배포된 컨트랙트를 수정하거나 하는 일은 불가능하다.

이 때 사용 할 수 있는것이 Proxy pattern이며 UUPS, Transparent, Beacon 등 여러 구현 방식이 있다.

본 글에서는 Solidity에서 proxy pattern 이해하기위해 필요한 내용들과 구현 방식들 에 대한 설명 및 장단점에 대해 정리한다.

# Delegate call

Contract에서 다른 주소의 Contract code를 동적으로 실행

Caller Contract Storage의 주소와 상태 값은 의 내용을 유지하되 코드만 호출하는 방식

# How to work?

![[https://eips.ethereum.org/EIPS/eip-1822](https://eips.ethereum.org/EIPS/eip-1822)](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0ff5eca0-4c0b-4580-bd0c-f5efd21140aa/Untitled.png)

[https://eips.ethereum.org/EIPS/eip-1822](https://eips.ethereum.org/EIPS/eip-1822)

Proxy contract에서 delegatecall 을 통해 Logic contract의 code만 호출하는 방법.

delegate call을 통하므로 Storage 영역은 Proxy contract(caller)의 값을 그대로 사용한다.

# Transparent Proxy

> *현재 해당 Standard는 Deprecated 상태이며 이를 개선한 ERC-2535: Diamonds, Multi-Facet Proxy를 사용할 것은 권장한다.*
>

[proxy selector clashing](https://medium.com/nomic-foundation-blog/malicious-backdoors-in-ethereum-proxies-62629adf3357) 문제를 해결하기위 제안된 proxy 구현

- Contract의 모든 함수는 4byte의 함수 식별자로 분류된다. 단일 Contract 내부에서는 함수 식별자가 충돌 할 문제는 없지만 Proxy Contract와 Logic Contract 간에는 충돌할 수 있는 위험이있다.

![Transparent Proxy](/assets/img/230314_1_transparent.png)

### 동작 방식

- msg.sender에 따라 위임 여부를 결정하는 방식으로 구현한다.
- 만약 sender가 Admin이면 위임 delegateCall을 비활성화 한다
  - upgradeTo 함수만 호출가능하다.
- 그 외의 경우 모든 호출을 delegateCall하다.
  - Proxy Contract 자체의 함수는 호출하지 않는다.

# UUPS Proxy pattern

![Transparent Proxy](/assets/img/230314_1_uups.png)

*Universal Upgradeable Proxy Standard*, Proxy contract의 모든 호출은 delegateCall을 통해 Logic Contract로 위임된다. Transparent proxy의 Upgrade의 책임이 Proxy Contract에 있는 반면 UUPS Proxy의 경우 Logic Contract에 있다

# Beacon Proxy pattern

![Transparent Proxy](/assets/img/230314_1_beacon.png)

기존 Proxy에서 Logic Contract를 사용하는 Proxy Contract가 N개인 경우 Logic Contract가 새롭게 배포되면 새로운 주소를 N번 업데이트 해야되는 문제가 있었다. 이를 해결하기위해 Beacon Contract를 중간에 두고 Proxy Contract에서 Beacon Contract를 통해 Logic Contract의 주소를 획득한 뒤 호출하는 Proxy pattern.

# 결론

Proxy Contract들의 기본적인 구현 종류에 대해 정리해봤으며 해당 Proxy들을 개선한 버전의 Proxy (Minimal, Diamond 등)이 존재한다.

어떤 Proxy를 사용하더라도 결국은 추가적인 트랜잭션 비용이 발생 할 수 밖에 없다.

각 프록시 별 Motivation과 Feature를 잘 이해하고 구현하는 Contract의 기능에 적합한 Proxy를 선택하는 것이 중요하고 만약 필요하지않다면 Proxy 패턴을 적용하지 않는것이 좋다.

# 참조

[https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#](https://docs.soliditylang.org/en/v0.8.17/internals/layout_in_storage.html#)

[https://eips.ethereum.org/EIPS/eip-1167](https://eips.ethereum.org/EIPS/eip-1167)

[https://eips.ethereum.org/EIPS/eip-1538](https://eips.ethereum.org/EIPS/eip-1538)

[https://eips.ethereum.org/EIPS/eip-1967](https://eips.ethereum.org/EIPS/eip-1967)

[https://eips.ethereum.org/EIPS/eip-2535](https://eips.ethereum.org/EIPS/eip-2535)
