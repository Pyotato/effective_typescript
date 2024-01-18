#  ☝️ ITEM 23 : 한꺼번에 객체 생성하기

[아이템 20](https://github.com/Pyotato/effective_typescript/blob/item20/README.md)에서 설명했듯이 변수의 값은 변경될 수 있지만, 타입스크립트의 타입은 일반적으로 변경되지 않습니다. 
이러한 특성 덕분에 일부 자바스크립트 패턴을 타입스크립트로 모델링하는 게 쉬워집니다.
즉, 객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입 추론에 유리합니다.

다음은 자바스크립트에서 2차원 점을 표현하는 객체를 생성하는 방법입니다.

```js
const pt = {};
pt.x = 3;
pt.y = 4;
```

타입스크립트에서는 각 할당문에 오류가 발생합니다.

<img width="519" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/089b366f-7401-4c47-a880-ebf3394b6e2e"/>

왜냐하면 첫 번째 줄의 pt 타입은 {} 값을 기준으로 추론되기 때문입니다.
존재하지 않는 속성을 추가할 수는 없습니다.

만약 Point 인터페이스를 정의한다면 오류가 다음처럼 바뀝니다.

```ts
interface Point{ x: number; y: number; }
const pt: Point = {};
pt.x = 3;
pt.y = 4;
```

<img width="570" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f9ed7a5e-5848-4ac9-965d-ebc7623e56b7"/>

이 문제들은 객체를 한번에 정의하면 해결할 수 있습니다.

```ts
const pt: Point = {x = 3, y = 4};
```

객체를 반드시 제각각 나눠서 만들어야 한다면, 타입 단언문(as)을 사용해 타입 체커를 통과하게 할 수 있습니다.

```ts
const pt = {} as Point;
pt.x = 3;
pt.y = 4;
```

물론 이 경우에도 선언할 때 객체를 한꺼번에 만드는 게 낫습니다. ([아이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md))

작은 객체들을 조합해서 큰 객체를 만들어야 하는 경우에도 여러 단계를 거치는 것은 좋지 않는 생각입니다.

```ts
interface Point{ x: number; y: number; }
const pt: Point = {x : 3, y : 4};
const id = {name: 'Pythagoras'};
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
```

<img width="475" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/610ae259-5370-4020-b7f5-50adf25ae080"/>

다음과 같이 '객체 전개 연산자' `...`를 사용하면 큰 객체를 한꺼번에 만들어 낼 수 있습니다.

```ts
const namedPoint = {...pt, ...id};
namedPoint.name; //  정상, 타입이 string
```

객체 전개 연산자를 사용하면 타입 걱정 없이 필드 단위로 객체를 생성할 수도 있습니다.
이때 모든 업데이트마다 새 변수를 사용하여 각각 새로운 타입을 얻도록 하는게 중요합니다.

```ts
const pt0 = {};
const pt1 = {...pt0, x:3};
const pt: Point = {...pt1, y:4}; // 정
```

이 방법은 간단한 객체를 만들기 위해 우회하기는 했지만, 객체에 속성을 추가하고 타입스크립트가 새로운 타입을 추론할 수 있게 해 유용합니다.

타입에 안전한 방식으로 조건부 속성을 추가하려면, 속성을 추가하지 않는 null 또는 {}으로 객체 전개를 사용하면 됩니다.

```ts
declare let hasMiddle: boolean;
const firstLast = {first:'Harry', last:'Truman'};
const president = {...firstLast, ...(hasMiddle?{middle:'S'}:{})};
```

<img width="475" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/67cfed7e-49c2-4f3b-af7a-1b3a6bfe6fef"/>

편집기에서 president 심벌에 마우스를 올려 보면, 타입이 선택적 속성을 가진 것으로 추론된다는 것을 확인할 수 있습니다.

전개 연산자로 한꺼번에 여러 속성을 추가할 수도 있습니다.

```ts
declare let hasDates: boolean;
const nameTitle = {name:'Khufu', title:'Pharaoh'};
const pharaoh = {...nameTitle, ...(hasDates? { start:-2589, end: -2566 }: {})};
```

<img width="537" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/5ce98355-bd8f-4294-a0a4-bd19ae9a8b69"/>

------

🌟 이 부분은 이펙티브 타입스크립트 원문의 내용입니다. tsplayground에서 실행 결과는 위와 같았습니다.

편잡기애서 pharaoh 심벌에 마우스를 올려 보면, 이제는 타입이 유니온으로 추론됩니다.

```ts
const pharaoh = {
  start: number;
  end: number;
  name: string;
  title: string;
} | {
  name: string;
  title: string;
};
```

start와 end가 선택적 필드이기를 원했다면 이런 결과가 당황스러울 수 있습니다. 
이 타입에서는 start를 읽을 수 없습니다.

```ts
pharaoh.start
      // ~~~~ `{name:string; title: string}`형식에 'start'속성이 없습니다.
```

이 경우는 start와 end가 항상 함께 정의됩니다. 

------


이 점을 고려하면 유니온을 사용하는게 가능한 값의 집합을 더 정확히 표현할 수 있습니다 ([아이템 32](https://github.com/Pyotato/effective_typescript/blob/item32/README.md)).
그런데 유니온보다는 선택적 필드가 다루기에는 더 쉬울 수 있습니다. 선택적 필드 방식으로 표현하려면 다음처럼 헬퍼 함수를 사용하면 됩니다.


```ts
function addOptional<T extends object, U extends object>(
  a: T, b: U | null
): T & Partial<U>{
  return {...a,...b};
}

const pharaoh  = addOptional(name:Title, hasDates? ({ start:-2589, end: -2566 }: null));
pharaoh.start 
```

<img width="620" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/50fe7957-4421-45b5-a0d9-729388667027"/>

<img width="352" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/db77af13-ce74-4d44-96a9-d674272cd6b6"/>

가끔 객체나 배열을 변환해서 새로운 객체나 배열을 생성하고 싶을 수도 있습니다
이런 경우 루프대신 내장된 함수형 기법 또는 로대시(Lodash)같은 유틸리티 라이브러리를 사용하는 것이 '한꺼번에 객체 생성하기'관점에서 보면 옳습니다.
[아이템 27](https://github.com/Pyotato/effective_typescript/blob/item27/README.md)에서 더 자세히 다룹니다.















