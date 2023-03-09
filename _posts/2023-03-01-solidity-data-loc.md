---
layout: post
title: "Solidity - storage, memory, calldata"
categories: [Dev, Blockchain]
tags: [blockchain, solidity]

---

### Storage

블록체인 네트워크에 영구적으로 저장되는 데이터

일반적으로 스마트 컨트랙트의 모든 상태 값(함수 스코프 외부에 있는 변수)들은 `storage` 영역에 저장된다.

struct 혹은 dynamic array와 같은 동적 변수를 제외하면 32 byte 단위로 패킹된다.

동적 타입(s의 경우 32 byte 사이즈로 패킹되며 실제 값들은 Keccak-256 계산된 별도의 슬롯에 저장된다.

상수값의 경우 `storage` 공간에 저장되는 것이 아닌 스마트 컨트랙트 바이트코드에 직접 주입된다.

### Memory

함수 스코프 내부에서 사용되어지며 스코프가 종료되면 휘발되는 임시 데이터

storage와 비교하여 비용이 저렴하므로 구현하는 컨트랙트에 적합하게 작성해야된다.

### Space storage vs memory

```solidity
uint4[4] arr;
```

위 처럼 `unit4` 타입의 array가 있는경우 `storage` 영역에서는 1 slot (32bytes)만 차지하는반면 `memory` 공간에서는 128 bytes (32bytes each elements) 를 차지한다.

### Calldata

`memory`와 라이프사이클은 동일하지만 읽기 전용

스마트 컨트랙트의 함수가 호출될때 읽기 전용으로 사용 할 파라미터에 사용하면 된다.

> *만약 `memory`를 사용하는 경우 파라미터를 `memory` 공간에 복사하는 과정이 필요하므로 추가 가스비가 발생한다*
>

다만, 읽기 전용이므로 수정이나 return 에 활용하는것은 불가능하므로 필요에 따라 적절한 키워드를 사용하면 된다.

# 참조

[https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_storage.html](https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_storage.html)

[https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_memory.html](https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_memory.html)

[https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_calldata.html](https://docs.soliditylang.org/en/v0.8.13/internals/layout_in_calldata.html)
