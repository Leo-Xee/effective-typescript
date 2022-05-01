# 아이템 6. 편집기를 사용하여 타입 시스템 탐색하기

IDE에서 타입스크립트를 사용하기 위해서는 타입스크립트를 아래와 같이 전역으로 설치하면

```bash
$ npm install -g typescript
# or
$ yarn global add typescript
```

다음과 같이 크게 2가지 기능을 실행할 수 있다.

- 타입스크립트 컴파일러(tsc)
- 타입스크립트 서버(tsserver)

일반적으로 타입스크립트 컴파일러를 실행하는 것이 주요 목적이지만 타입스크립트 서버 또한 '언어 서비스'를 제공한다는 점에서 유용하다. 대부분의 IDE도 비슷하겠지만 [VSC 기준](https://code.visualstudio.com/docs/typescript/typescript-compiling)으로 언어 서비스는 코드 자동완성, 명세 검사, 검색, 리팩토링을 제공한다.

## IDE에서 타입스크립트 서버를 활용하는 방법

- 심벌 위에 마우스 커서를 대면 타입스크립트가 해당 타입을 어떻게 판단하고 있는 지 확인할 수 있다.
- 라이브러리를 사용할 때 해당 함수나 클래스에 대한 타입 선언을 알아보기 원한다면 `Go to Definition` 옵션을 사용해서 `*.d.ts`으로 이동해서 자세한 내용을 확인할 수 있다.

## 정리

- IDE의 언어 서비스 기능을 활용해서 타입스크립트가 현재 어떻게 어디까지 타입을 잘 추론하고 있는지 파악하자.
- 특정 라이브러리의 함수나 클래스의 타입 선언 파일을 유연하게 확인하는 방법을 익히자.

<br>

# 아이템 7. 타입이 값들의 집합이라고 생각하기

타입스크립트에서 타입들을 집합 혹은 범위라고 생각하면 이해하기가 수월해진다. 타입을 할당할 때 항상 넓은 범위의 집합에서 좁은 범위의 집합으로는 할당이 절대 불가능하다고 생각하자! 마치 넓은 집에서 살다가 좁은 집으로 이사가면 답답해서 살 수 없는 것처럼 말이다.

![typescript-diagram](./images/typescript-diagram.png)

## never 타입

never 타입은 타입스크립트에서 가장 작은 집합으로 아무것도 포함하지 않는 공집합이다.

```ts
const leo: never = "Leo"; // error: "Leo" 형식은 "never" 형식에 할당할 수 없습니다.
```

## 리터럴 타입

리터럴(Literal) 타입은 유닛(Unit) 타입이라고도 불리는 타입스크립트에서 두 번째로 작은 집합으로 한 가지 값만 포함하는 타입이다.

```ts
type Leo = "Leo";
type Ryan = "Ryan";
```

## 유니온 타입

유니온(Union) 타입은 2개 이상의 타입을 묶을 때 `|`로 이어서 사용하며 값 집합의 합집합을 일컫는다.

```ts
type LeoAndRyan = Leo | Ryan;
const friendLikeLion: LeoAndRyan = "Leo"; // Good!!
const friendLikeLion: LeoAndRyan = "Apeach"; // error: "Apeach"는 "LeoAndRyan" 형식에 할당할 수 없습니다.
```

## 인터섹션 타입

인터섹션(Intersection) 타입은 2개 이상의 타입의 교집합을 일컫고 `&`로 이어서 사용한다.

아래의 예시에서 교집합이라는 단어만 생각하면 `Person`과 `FrontEnd` 타입의 교집합은 공집합이라고 잘못 이해할 수 있지만 값의 **집합**으로 생각해보면 두 타입의 속성을 모두 가지는 타입이 인터섹션 타입이다. 이 점을 주의하자!!

```ts
type Person = {
  name: string;
  birth: Date;
};

type FrontEnd = {
  skills: string[];
};

type FE_Developer = Person & FrontEnd;

const Ryan: FE_Developer = {
  name: "Ryan",
  birth: new Date("2014/10/01"),
  skills: ["C++", "JS", "TS"],
};
```

하지만 위와 같이 선언하는 것보다 아래와 같이 인터페이스의 상속의 사용하는 방법이 일반적이다.

```ts
interface Person = {
  name: string;
  birth: Date;
};

interface FE_Developer extends Person = {
  skills: string[];
};


const Ryan: FE_Developer = {
  name: "Ryan",
  birth: new Date("2014/10/01"),
  skills: ["C++", "JS", "TS"],
};
```

## 타입과 집합의 용어 정리

| 타입스크립트 용어   | 집합 용어           |
| ------------------- | ------------------- |
| never               | 공집합              |
| 리터럴 타입         | 원소가 1개인 집합   |
| 값이 T에 할당 가능  | 값이 T의 원소       |
| T1이 T2에 할당 가능 | T1이 T2의 부분 집합 |
| T1이 T2를 상속      | T1이 T2의 부분 집합 |
| T1 \| T2            | T1과 T2의 합집합    |
| T1 & T2             | T1과 T2의 교집합    |

## 정리

- 타입을 값들의 집합으로써 생각하면 이해하기 쉬워진다. 항상 집합 및 범위 관점에서 타입의 할당을 바라보도록 하자.

<br>

# 아이템 8. 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌(Symbol)은 타입 공간이나 값 공간 중의 한 곳에 존재한다. 타입 공간은 타입스크립트 컴파일러에 의해 컴파일된 이후에 사라지는 코드를 의미하며 남아 있는 코드는 값 공간을 의미한다.

```ts
type Leo = "Leo"; // 컴파일 이후에 사라질 타입 공간
const Ryan = "Ryan"; // 컴파일 이후에 남아있을 값 공간
```

## 타입 공간과 값 공간에서 혼용되는 키워드들

`typeof`, `this` 이외에도 많은 다른 연산자들과 키워드들이 타입 공간과 값 공간에서 다른 목적으로 사용된다.

### typeof

타입 공간에서 `typeof`는 값을 읽어서 타입스크립트의 타입을 반환하고 값 공간에서는 자바스크립트 런타임의 연산자가 된다.

```ts
type Person: {
  name: string;
}

const p: Person = {
  name: "Leo",
}

type NewPerson = typeof p; // NewPerson의 타입은 Person

const v = typeof p; // v는 "object"
```

### this

타입 공간에서 `this`는 `this`의 타입스크립트 타입이고 값 공간에서는 자바스크립트의 `this` 키워드가 된다.

### const와 as const

`const`는 새 변수를 선언하지만 `as const`는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꾼다.

### extends

`extends`는 서브클래스(`class A extends B`), 서브타입(`interface A extends B`), 제너릭 타입의 한정자(`Generic<T extends number>`)를 정의할 수 있다.

### in

`in`은 루프(`for (key in object)`) 또는 매핑된(mapped) 타입에 등장한다.

## 정리

- 타입스크립트 코드를 읽을 때 타입 공간과 값 공간을 구분하는 방법을 익히도록 하자.
- `class`와 `enum` 같은 키워드는 타입과 값으로 모두 사용될 수 있다는 것에 유의하자.
- `"foo"`는 문자열 리터럴이거나 문자열 리터럴 타입일 수 있으므로 잘 구별하도록 하자.

# 참조

- https://www.oreilly.com/library/view/programming-typescript/9781492037644/ch03.html
