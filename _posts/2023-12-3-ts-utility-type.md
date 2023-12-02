---
layout: post
title: "Typescript Utility Types 훑어보기"
categories: [Dev, Frontend]
tags: [typescript]

---

# Utility Types?

Typescript에서 제공하는 타입이며 일반 타입을 편리(정말?)하게 활용 할 수 있게 해준다.

강타입 언어를 넘어 최강타입 언어정도가 되는게 목적이 아닐까?

굳이 언어 레벨에서 별도의 타입으로 제공할만한 내용들일까..? 라는 의문이 들지만 typescript 기반 라이브러리 내부 구현 파악에 필요하므로 한번 살펴보자.

정리하다보니 왠지 모르게 자바스크립트에 대한 호감도가 상승했다.

# Awaited<T>

T 가 Promise인 경우 Promise를 unwrap 해준다.

```tsx
type A = string | Promise<number>
A = string | number

type ShitTS = Promise<Promise<Promise<string>>>
type B = Awaited<ShitTS>
B = string
```

# Partial<T>

T의 프로퍼티의 부분 집합을 사용 할 수 있다.

```tsx
type User {
  name: string;
  age: number;
}

function updateUser(user: User, partial: Partial<User>) {
   return { ...user, ...partial }
}
```

# Required<T>

T의 모든 프로퍼티가 필수로 요구된다.

```tsx
type User {
  name?: string;
  age?: number;
}

const user: User = { name: "name" } // ok
const requiredUser: Required<User> = { name: "name" } // error!
```

# Readonly<T>

T의 프로퍼티들은 불변이다.

```tsx
type User {
   name: string
}

const user: Readonly<User> = { name: "name" }
user.name = "new name" // error!
```

# Record<K, T>

property = K, value = T

```tsx
interface CatInfo {
  age: number;
  breed: string;
}

type CatName = "miffy" | "boris" | "mordred";

const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
```

# Pick<T, K>

원하는 property를 선택 할 수 있다.

여러 sub type이 필요한 경우 사용.

```tsx
interface Main {
  a: string,
  b: string,
  c: string
}

type SubA = Pick<Main, "a">
type SubAB = Pick<Main, "a" | "b">
```

# Omit<T, K>

원하는 property들을 제거 마찬가지로 서브타입 정의에 활용

```tsx
interface Main {
  a: string,
  b: string,
  c: string
}

type SubWithoutA = Omit<Main, "a">
type SubWithoutAB = Omit<Main, "a" | "b">
```

# Exclude<UnionType, Members>

UnionType에서 특정 타입을 제거한다.

```tsx
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T3 = Exclude<Shape, { kind: "circle" }>
//T3 is square or triangle
```

# Extract<UnionType, Type>

UnionType에서 원하는 type을 추출한다.

```tsx
type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "square"; x: number }
  | { kind: "triangle"; x: number; y: number };

type T2 = Extract<Shape, { kind: "circle" }>
//T2 is circle
```

이 외에도 `NonNullable, Parameters, ConstructorParameters, ReturnType, InstanceType, ThisParameterType, OmitThisParameter, ThisType` 등 이 더 있다. 그만 알아보자..

---

당연한 이야기이지만, 강타입은 결국 static type check를 지원한다는 의미가 가장 크다. 이를 통해 동적 타입에서는 런타임에서만 식별되어지는 여러 휴먼 에러들을 사전에 방지할 수 있다.

즉, 우리가 짜는 코드가 더 “안전” 해지는 것이다. 공짜 점심은 없다는 말처럼 강타입 언어에서는 그에 따른 오버헤드가 반드시 따른다.

내가 바라보는 타입스크립트가 얘기하는 안전의 범위는 끝이 없어 보인다. 즉, 끝도 없이 안전해 질 수 있고 그에 비례하여 오버헤드도 함께 증가 할 수 있어 보인다. 좋은걸까? 나는 무조건 TS 기반의 프로젝트를 만들긴하겠지만 Utility Type의 3-4개 정도를 제외하면 실제 사용할 일은 없을듯하다. 효용성이 없어 보인다.

*JS는 너무 약하고, TS는 너무 강해지려한다. 오늘도 평화로운 웹 세상이다.*

# Reference

[Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
