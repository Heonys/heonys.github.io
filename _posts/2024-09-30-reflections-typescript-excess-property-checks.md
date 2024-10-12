---
created: 2024-09-30
title: "[TS] 잉여 속성 검사에 대한 고찰"
description: 타입 호환성을 위한 Freshness 개념과 잉여 속성 검사, 그리고 그 특이점에 대해 더 깊이 알아보자
categories: [Typescript]
tags: [Typescript, Freshness, Excess-Property-Checking]
layout: post
image:
  path: /assets/images/2024-09-30-reflections-typescript-excess-property-checks/thumbnail.jpg
---


## 💡 시작에 앞서 

최근 타입스크립트로 개발하면서, 기존에 잘 알고 있다고 생각했던 부분에서 예상치 못한 타입 시스템의 동작에 당황한 경험이 있었습니다. 이러한 경험을 통해, 타입스크립트의 구조적 타이핑 방식과 타입 호환성의 관계를 정리해 보고 타입스크립트 컴파일러를 분석하면서 타입 호환성을 위한 `Freshness` 개념과 잉여 속성 검사, 그리고 그 특이점에 대해 더 깊이 탐구하려 합니다. 


---
## 💡 구조적 타이핑 

구조적 타이핑이란 명시적인 상속 관계나 타입 선언과 상관없이, 오직 객체의 구조를 기반으로 타입 호환성을 허용하는 방식으로 타입의 이름이나 상속관계가 명확한 경우에만 타입 호환을 허용하는 명목적 타이핑과 반대되는 방식입니다.

```ts
type A = {
  name: string;
}

type B = {
  name: string;
  age: number;
}
```
즉, `B`타입은 `A`타입에 존재하는 모든 속성을 갖고있기 때문에 `B`타입은 구조적 타이핑 관점에서 `A`타입으로 볼 수 있습니다. <br />
따라서 `B`타입을 `A`타입으로 할당이 가능하지만 반대의 경우는 `age` 속성이 없을 수도 있기 때문에 할당이 불가능합니다.

> 이러한 구조적 타이핑은 타입 호환성을 판단하는 아주 중요한 특성으로 타입간의 위계관계를 설명합니다
{: .prompt-info }


### 🌟 간단한 예시 

아래의 코드에서 `Object.keys` 메소드의 결과를 쉽게 예측할 수 있습니다. 
```ts
const person = {
  firstName: "Jiheon",
  lastName: "Kim",
  email: "jiheon.kim@example.com",
};

Object.keys(person) // [ 'firstName', 'lastName', 'email' ]
```
객체의 키 값은 정해져 있으니까 `Object.keys`으로 반환되는 이 배열의 타입은 `string[]`보다 더 구체적일 수 있지 않을까? 라는 생각을 했던적이 있습니다. 가령 해당 키들의 리터럴 배열 혹은 튜플이 되거나 유사배열 형태로 제공될 수도 있겠다고 생각해 봤는데 실제로는 그렇지 않고 항상 `string[]` 타입을 반환됩니다.

```ts
// Object.keys(person)의 예상 타입
type T1 = ("firstName" | "lastName" | "email")[];
type T2 = ["firstName", "lastName", "email"];
type T3 = {
    0: "firstName";
    1: "lastName";
    2: "email";
}
```
`T1`은 속성의 중복을 허용하고, `T2`는 순서를 강제하며, `T3`는 애초에 배열을 객체타입으로 정의하는 게 말이 안됩니다. 실제로 `Object.keys` 타입을 확인해 보면 제네릭 타입도 지원하지 않고 `string[]`으로 고정되는데 생각해 보면 이를 표현할 수 있는 방법도 마땅치 않을 뿐더러 타입스크립트의 구조적 타이핑에 의해서 모든 객체는 어떠한 속성도 가질 수 있기 때문에 타입스크립트에서 `Object.keys`의 반환 타입을 `string[]`으로 고정합니다.


###  🦆 덕 타이핑(Duck typing) 


덕 타이핑과 구조적 타이핑은 모두 이름으로 타입을 구분하는 명목적 타이핑과 달리 구조와 속성 기반으로 타입을 판단하는 공통점이 있지만, 구조적 타이핑은 타입 시스템에 기반하여 컴파일 시점 혹은 타입체커에 의해 타입을 체크하는 반면, 덕 타이핑은 런타임에 타입을 체크하며 객체가 해당 메소드나 속성이 존재하는 경우 타입에 관계없이 해당 동작을 수행할 수 있는지의 초점을 맞춘 방식입니다.


>자바스크립트의 덕 타이핑 특성을 고려하여, 타입스크립트는 더욱 자연스럽고 유연한 사용성을 제공하기 위해 구조적 타이핑 방식을 채택하여 설계되었습니다.
{: .prompt-info }

### 🏷️ 명목적 타이핑 구현하기 


타입스크립트는 명목적 타이핑 방식을 지원하지 않지만, 이를 사용할 수 있는 트릭이 있는데 바로 `private` 멤버를 만드는 것입니다. 구조적 타이핑에서는 두 객체가 동일한 구조를 가지고 있다면 서로 호환된다고 판단하지만, 속성이나 메소드에 `private` 접근 제어자를 추가하면 명목적으로 취급됩니다.

```ts
class A {
  private name = "A";
}
class B {
  name = "B";
}
class C {
  private name = "C";
}
class D extends A {}

const a1: A = new B(); // ❌
const a2: A = new C(); // ❌
const a3: A = new D(); // ✅
```

즉, 두 객체가 같은 속성과 메소드를 가지고 설령 똑같이 `private` 접근제어자가 있을지언정, 클래스가 다르면 서로 할당되지 않는데 이런 방법을 사용하면 명목적 타이핑을 통해 엄격한 타입검사를 할 수 있습니다. 


---

> 여기서부터는 이 글에서 말하고 싶은 본격적인 이야기를 해보려 합니다
{: .prompt-warning }


## 💡 잉여 속성 검사

구조적 타이핑 방식은 타입 호환성의 유연함을 제공하지만 실제 다루는 타입보다 더 많은 데이터를 받아들인다는 오해를 불러일으킬 수 있으며, 추가된 속성에 대해서는 타입검사를 하지 않는다는 문제가 있습니다.
```ts
type User = { name: string }
function registerUser(user: User){}
 
const user= { 
    name: "Ezreal", 
    location: "Piltover",
    position: "ad",
}
registerUser(user) // ✅
```
따라서 타입스크립트는 객체 리터럴을 사용한 경우에 한해서 타입 호환의 예외가 발생하도록 하고 있는데 **잉여 속성 검사** (`excess property checking`)란 객체를 리터털 방식으로 정의하여 할당할 땐 타입 검사를 더 엄격하게 하여 초과된 속성이 있으면 에러가 나도록 하는 검사를 의미합니다. (초과 속성 검사라고도 부름)

```ts
type User = { name: string }
function registerUser(user: User){}
 
// Error: Object literal may only specify known properties
registerUser({ 
    name: "Ezreal", 
    location: "Piltover",
    position: "ad"
})
```
위의 코드에서 매개변수로 전달하는 객체는 구조적 타이핑 관점에서 보면 충분히 `User` 타입의 구조를 만족하기에 할당 가능하지만 잉여 속성 검사라는 특별한 메커니즘 때문에 `name`을 제외한 추가 속성을 갖고있어서 에러를 냅니다.

이는 변수를 사용하는 것과 다르게 리터럴 방식으로 정의한 객체는 한 번만 사용되고 재사용될 수 없기에 굳이 구조적 타이핑을 지원할 필요 없이 엄격한 타입검사를 통해 안전성을 확보하는 것입니다. 임시 변수를 통해서 이러한 검사를 우회할 수 있지만 객체 리터럴 방식에선 구조적 타이핑이 적용되지 않고 객체의 구조가 동일하지 않으면 개발자의 실수로 간주하는 메커니즘입니다.


### 🍃Freshness 이해하기 

타입스크립트는 내부적으로 `Freshness`(신선도) 라는 개념을 사용하여 잉여 속성 검사의 적용 여부를 확인하는데 리터럴로 만들어진 객체는 `FreshLiteral`이라는 플래그를 갖고, 초기에 `fresh`한 상태를 유지하게 됩니다.

```ts
// src/compiler/types.ts 
const enum ObjectFlags {
    None             = 0,
    Class            = 1 << 0, // 1
    Interface        = 1 << 1, // 2
    Reference        = 1 << 2, // 4
    Tuple            = 1 << 3, // 8
	// ...
    JSLiteral        = 1 << 12, // 4096
    FreshLiteral     = 1 << 13, // 8192 
  	//...
}
```
```ts
getObjectFlags(source) & ObjectFlags.FreshLiteral
```
타입스크립트의 `src/compiler/types.ts`을 보면 내부적으로 비트 플래그를 사용해서 해당 객체의 여러 플래그들 중에서 `FreshLiteral`플래그가 있는지 확인하는 방식으로 해당 객체가 `Fresh` 한지를 확인하고 있습니다. 즉, 해당 객체의 모든 플래그 중에서 찾고자 하는 비트 플래그와 `and`연산을 해서 0이 아니라는 건 객체가 해당 플래그를 갖고있다는 뜻이고 즉, `fresh`한 객체라는 것을 의미합니다. 


이러한 `FreshLiteral` 플래그가 초기에 어떻게 적용되는지 궁금해서 분석해 보면 아래와 같은 흐름으로 적용됩니다. 

```ts
// shsrc/compiler/checker.ts
checkExpression() -> checkExpressionWorker() -> checkObjectLiteral()
```

`checkExpression` 함수에서 표현식을 확인하고 `Worker`에 의해서 분기처리 되며, 객체 리터럴 형태의 표현식이라면 `checkObjectLiteral`함수를 실행합니다. 여기서 최종적으로 리터럴 객체에 `FreshLiteral`플래그를 설정하고 있습니다.
 
```ts
// src/compiler/checker.ts
function checkObjectLiteral(node: ObjectLiteralExpression) {
  // 객체 리터럴의 문법을 검증하여 객체의 형식이나 구조가 올바른지 확인
  checkGrammarObjectLiteralExpression(node)

  // 객체 플래그 설정
  let objectFlags: ObjectFlags = ObjectFlags.FreshLiteral;
  
  // 조건에 따라서 다양한 플래그들이 설정되고 
  // 함수 내부에서 객체에 objectFlags 속성을 적용하여 반환 
  return createObjectLiteralType() 
}
```

이렇게 `FreshLiteral` 플래그가 설정된 객체는 이후에 타입이 확장되거나 단언하면 `fresh` 상태가 사라지게 됩니다.

```ts
type Point = { x: number; y: number };
const p1 = { x: 1, y: 2, z: 3 }; // Fresh object
const p2: Point = p1; // ✅
/* 
  객체 리터럴로 정의되는 그 순간은 Fresh object 이지만 
  p1으로 할당되는 순간 더 일반적인 형태로 타입이 확장되어 fresh 상태가 사라짐 
*/
const p3: Point = { x: 1, y: 2, z: 3 } as Point // ✅
// 마찬가지로 타입 단언을 통해 fresh 상태가 사라짐 
```

### ⚙️ 잉여 속성 검사의 동작 원리 

`checker.ts` 파일에서 `hasExcessProperties` 라는 함수를 통해 검사의 구체적인 로직을 확인할 수 있습니다. 실제로는 훨씬 복잡하지만 잉여 속성 검사가 동작하는 원리를 간소화해서 순차적으로 나열해 보면 아래와 같습니다. 

```ts
// src/compiler/checker.ts
function hasExcessProperties(
    source: FreshObjectLiteralType, // FreshLiteral 플래그를 갖는 Fresh객체
    target: Type, // 비교할 타입
    reportErrors: boolean
): boolean
```
1. ***함수 실행***
`source`가 객체 리터럴이면서 `FreshLiteral` 플래그를 갖고 있으면 함수 호출  

2. ***검사 대상 여부 확인 :*** 
`target`이 객체 타입과 같은 잉여 속성 검사 대상인지를 확인하고 `JSX` 속성 및 인덱스 시그니처가 있는 경우와 같은 잉여속성 검사가 필요하지 않은 특정 상황 여부 확인

3. ***타입 비교 시작 :*** 
`source`의 타입과 `target`의 타입을 비교 시작

4. ***초과 속성 검사 :*** 
`source`에 있는 속성들을 하나씩 검사하면서, `target`에 없는 속성이 있는지 확인하고 초과속성이 발견되면 에러 보고

5. ***속성 타입 검사 :*** 
속성 이름만이 아니라, 속성의 타입도 일치하는지 검사하며, 속성의 타입이 일치하지 않으면 에러 보고

---
## 💡 잉여 속성 검사의 특이점 

이러한 잉여 속성 검사는 타입스크립트를 처음 배울 때 누구나 쉽게 접하고 단순한 메커니즘 이지만, 유니온 타입에서는 꽤나 혼란스러운 상황이 발생하게 됩니다. 

```ts
type A = { a: number; b: number; c: number };
type B = { a: number };

const ab: A | B = { a: 1, b: 2 }; // ✅
```
위의 코드는 잉여 속성 검사의 특징을 알고 있는 시점에서 볼 때 약간의 특이점이 있습니다 우선 `ab`라는 변수에 할당 되고있는 리터럴 객체 `{ a: 1, b: 2 }`는 구조상 `c` 속성이 없기에 `A`타입이 절대 될 수 없습니다. 따라서 `ab`객체는 현재 `B`타입으로 평가되고 있고 `{ b: 2 }`라는 추가속성이 적용되어 있는 상태 입니다. 하지만 여기서 평가되는 타입보다 속성이 더 많고 리터럴 타입임에도 불구하고 잉여 속성 검사가 정상적으로 되고 있지 않는 것을 볼 수 있습니다. 


### ❓ 객체간 유니온 타입의 의미 

`A | B`는 합집합을 뜻하기에 `A`타입, `B`타입 각각이 될 수 있을 뿐더러 둘 다 될 수 있는 상태의 값도 포함하고, 유니언의 구성 요소 중 어느 속성이든 포함될 수 있습니다. 

```ts
type Point = { x: number; y: number };
type Label = { name: string };

// 어느 하나의 타입이 아닌 둘다 될 수 있는 상태 
const thing: Point | Label = {
  x: 0,
  name: "true",
}; 
// 모든 속성을 가진 상태 -> 이것도 결국 둘 다 될 수 있는 상태 
const thing2: Point | Label = {
  x: 0,
  y: 0,
  name: "true",
}; 
```
`thing` 변수에 할당 하려는 리터럴 객체는 `Point`타입에 `name`속성이 추가된 것 일 수도있고 `Label`타입에 `x`속성이 추가된 것일 수도 있기에 두 가지 상태의 가능성이 모두 공존합니다. 위의 코드에서도 잉여 속성 검사가 이루어지지 않는 것을 볼 수 있는데 이러한 유니온 타입의 특징으로 볼 때 객체가 현재 유니온 타입의 두 가지 상황에 모두 포함되는 상태로 확정되지 않는 상태일 때는 전체 속성들에 대해서는 잉여 속성 검사가 일어나지 않습니다. 

```ts
const thing3: Point | Label = {
  x: 0,
  name: "true",
  z: 1 // ❌ Object literal may only specify known properties
}; 

const thing4: Point | Label = {
  x: 0, // ❌ 'Property 'y' is missing in type '{ x: number }'
};

```
하지만 `thing3`와 같이 아직 정해지지 않은 두 타입에 포함되지 않는 `z` 속성에 대해서는 정상적으로 잉여 속성 검사가 이루어지는 것을 확인할 수 있고, 또한 `thing4`은 `x` 속성이 있기에 `Point`타입으로 평가되지만 `y`속성이 없어서 타입 에러가 납니다. 

```ts
type A = { name: string };
type B = {
  name?: never;
  timeout: number;
};

const ab: A | B = {
  name: "ab",
  timeout: 100, // ❌ Object literal may only specify known properties
};
```
위의 예시에선 `ab`가 유니온 타입이지만, 할당하려는 리터럴 객체가 `name` 속성이 포함됬다면 이는 `B`타입이 절대 될 수 없기에 `A`타입으로 확정되어 `timeout` 속성이 초과 속성이 되어 에러가 나게 됩니다. 

#### 🔍 정리하자면
객체의 형태가 만약 유니온 타입의 한 가지로 딱 정해지는 상황에서는 그 타입에 맞게 잉여 속성 검사가 일어납니다. 하지만 객체가 현재 유니온 타입의 두 가지 상황에 모두 포함되는 상태일 때는 유니온 타입에 정의된 속성들에 대해서는 검사가 발생하지 않고, 유니온 타입의 멤버에 정의되지 않은 추가 속성에 대해서만 검사가 이루어집니다.

>[TypeScript 3.5 release notes](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-5.html#improved-excess-property-checks-in-union-types)를 보면 유니온 타입에서 초과 속성 검사의 동작에 대해서 이야기하며, 이전 버전에 발생한 버그를 수정하고 개선했다는 이야기를 하고있습니다. 
{: .prompt-info }



### ❓엄격한 유니온 타입 만들기

```ts
type A = { a: number; b: number };
type B = { c: number };
type C = A | B;

const c1: C = { a: 1, c: 3 }; // ✅
const c2: C = { a: 1, b: 2, c: 3 }; // ✅
```
`C`타입은 현재 유니온 타입의 특성 때문에 `c1`, `c2` 처럼 다양한 구조의 객체가 할당 가능하지만, 정확히 `A`타입 또는 `B`타입으로만 할당할 수 있게 하고 싶은 경우도 있을 수 있습니다. 

```ts
type A = { a: number; b: number; c?: never };
type B = { a?: never; b?: never; c: number };
type C = A | B;

const c1: C = { a: 1, c: 3 }; // ❌
const c2: C = { a: 1, b: 2, c: 3 }; // ❌
const c3: C = { a: 1, b: 2 }; // ✅
const c4: C = { c: 3 }; // ✅
```
이를 위해선 `c1`, `c2`와 같은 할당을 객체 리터럴이 호환되지 않도록 `never` 타입의 속성을 옵셔널로 추가하면 됩니다. 이를 일반화하면 아래와 같은 유틸리티 타입을 만들 수 있습니다. 

```ts
type UnionKeys<T> = T extends T ? keyof T : never;
type StrictUnionHelper<T, TAll> = T extends any
  ? T & Partial<Record<Exclude<UnionKeys<TAll>, keyof T>, never>>
  : never;
type StrictUnion<T> = StrictUnionHelper<T, T>;
```
---

## 💡 마무리

기존에 잉여 속성 검사에 대해 잘 알고 있다고 생각해서 유니온 타입에서의 일반적이지 않은 동작을 정리하고자 시작한 글이었습니다. 그러나 컴파일러를 직접 분석하면서 `Freshness`를 비롯하여 타입스크립트에 대해 더욱 깊이 이해하게 되어 흥미로웠습니다. 또한 구조적 타이핑이 명시적으로 타입을 지정해야 하는 수고를 덜어주고 엄청 편리하다고만 생각했었는데 유니온 타입과 같은 경우에선 오히려 혼란을 줄 수 있고 복잡해지는 느낌이 들었습니다. 



## 📕 참고 문서
>[typescript handbook - release notes 3.5](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-5.html#improved-excess-property-checks-in-union-types) <br />
[stackoverflow - Type union not checking for excess properties](https://stackoverflow.com/questions/65805600/type-union-not-checking-for-excess-properties) <br />
[reddit - Why this is valid](https://www.reddit.com/r/typescript/comments/18rylyg/why_this_is_valid/?rdt=39878) <br />
[toss tech - TypeScript 타입 시스템 뜯어보기: 타입 호환성 ](https://toss.tech/article/typescript-type-compatibility) <br />
