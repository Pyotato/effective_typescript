# 💭 ITEM 50 : 오버로딩 타입보다는 조건부 타입 사용하기

다음 예제의 double 함수에 타입 정보를 추가해 보겠습니다.

```ts
function double(x) {
  return x+x;
}
```

double 함수에는 string 또는 number 타입의 매개변수가 들어올 수 있습니다.
그러므로 유니온 타입을 추가했습니다. (참고로 다음 예제는 타입스크립트의 함수 오버로딩 개념을 사용했습니다. 기억나지 않는다면 [아이템 3](https://github.com/Pyotato/effective_typescript/blob/item3/README.md)을 다시 살펴보기 바랍니다.)

```ts
function double(x: number| string): number| string;
function double(x: any) {
  return x+x;
}
```

선인이 틀린 것은 아니지만, 모호한 부분이 있습니다.
예를 들어 보겠습니다.

```ts
const num = double(12);    // number| string
const str = double('x');   // number| string
```

double에 number타입을 매개변수로 넣으면 number타입을 반환합니다. 
그리고 string 타입을 매개 변수로 넣은면 string타입을 반환합니다.
그러나 선언문에는 number 타입을 매개변수로 넣고 string 타입을 반환하는 경우도 포함되어 있습니다.

제너릭을 사용하면 이러한 동작을 모델링할 수 있습니다.

```ts
function double <T extends number | string>(x: T): T;
function double(x: any){ return x+x; }

const number = double(12);    // 타입이 12
const str = double('x');       // 타입이 "x"
```

타입을 구체적으로 만들어 보려는 시도는 좋았지만 너무 과했습니다.
이제는 string 타입이 너무 과하게 구체적입니다.
string 타입을 매개변수로 넘기면 string 타입이 반환되어야 합니다.
그러나 리터럴 문자열 'x'를 매개변수로 넘긴다고 해서 동일한 리터럴 문자열 'x' 타입이 반환되어야 하는 것은 아닙니다.
'x'의 두배는 'x'가 아니라 'xx'입니다.

또 다른 방법은 여러 가지 타입 선언으로 분리하는 것입니다.
타입스크립트에서 함수의 구현체는 아니지만, 타입 선언은 몇 개든지 만들 수 있습니다.
이를 활용해 double의 타입을 개선할 수 있습니다.

```ts
function double(x: number): number;
function double(x: string): string;
function double(x: any){return x+x;}

const number = double(12);    // 타입이 number
const str = double('x');       // 타입이 string
```

함수 타입이 조금 명확해졌지만 여전히 버그는 남아 있습니다. 
string이나 number 타입의 값으로는 잘 동작하지만, 유니온 타입 관련해서 문제가 발생합니다.

```ts
function f(x: number| string) {
  return double(x);
}
```

<img width="1482" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/a1324019-14f1-434d-9c1e-eee2d50af624"/>

이 예제에서 double 함수의 호출은 정상적이며 string|number 타입이 반환되기를 기대합니다.
한편 타입스크립트는 오버로딩 타입 중에서 일치하는 타입을 찾을 때까지 순차적으로 검색합니다.
그래서 오버로딩 타입의 마지막 선언 (string 버전)까지 검색했을 때, string|number 타입은 string에 할당할 수 없기 때문에 오류가 발생합니다.

새 번째 오버로딩(string|number 타입)을 추가하여 문제를 해결할 수도 있지만, 가장 좋은 해결책은 조건부 타입(conditional type)을 사용하는 것입니다.
조건부 타입은 타입 공간의 if구문과 같습니다.

```ts
function double<T extends number| string>(x: T) : T extends string? string: number;
function double(x: any){ return x+x; }
```

이 코드는 제너릭을 사용했던 예제와 유사하지만, 반환 타입이 더 정교합니다.
조건부 타입은 자바스크립트의 삼항 연산자(?:)처럼 사용하면 됩니다.

- T가 string의 부분 집합이면 (string, 또는 문자열 리터럴, 또는 문자열 리터럴의 유니온), 반환 타입이 string입니다.
- 그 외의 경우는 반환 타입이 number 입니다.

조건부 타입이라면 앞선 모든 예제가 동작합니다.

<img width="1513" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/639da974-952d-4f6c-a44f-5a317f843937"/>

유니온에 조건부 타입을 적용하면, 조건부 타입의 유니온으로 분리되기 때문에 number| string의 경우에도 동작합니다. 예를 들어 T가 number|string이라면, 타입스크립트는 조건부 타입을 다음 단계로 해석합니다. 

> (number | string) extends string? string: number -> (number extends string? string: number) | (string extends string? string: number) -> number | string


오버로딩 타입이 작성하기는 쉽지만, 조건부 타입은 개별 타입의 유니온으로 일반화하기 때문에 타입이 더 정확해집니다.
타입 오버로딩이 필요한 경우에 가끔 조건부 타입이 필요한 상황이 발생합니다.
각각의 오버로딩 타입이 독립적으로 처리되는 반면, 조건부 타입은 타입 체커가 단일 표현식으로 받아들이기 때문에 유니온 문제를 해결할 수 있습니다.
오버로딩 타입을 작성 중이라면 조건부 타입을 사용해서 개선할 수 있을지 검토해 보는 것이 좋습니다. 
