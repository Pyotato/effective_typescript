# 🧑‍🤝‍🧑 ITEM 7 : 타입이 값들의 집합이라고 생각하기

런타임에 모든 변수는 자바스크립트 세상의 값으로부터 정해지는 각자의 고유한 값을 가집니다.<br/>
변수에는 다음처럼 다양한 종류의 값을 할당할 수 있습니다.

- 42
- null
- undefined
- 'Canada'
- {animal: 'Whale', weigh_lbs:40_000}
- /regex/
- new HTMLButtonElement
- (x, y) => x + y

그러나 코드가 실행되기 전, 즉 타입스크립트가 오류를 체크하는 순간에는 '타입'을 가지고 있습니다.<br/>
'할당 가능한 값들의 집합'이 타입이라고 생각하면 됩니다.
이 집합은 타입의 '범위'라고 부르기도 합니다.
예를 들어, 모든 숫자값의 집합을 number 타입이라고 생각할 수 있습니다.
42와 37.25는 number 타입에 해당되고, 'Canada'는 그렇지 않습니다.
null과 undefined는 strictNullChcecks 여부에 따라 number에 해당될 수도, 아닐 수도 있습니다.

가장 작은 집합은 아무 값도 포함하지 않는 공집합이며, 타입스크립트는 never 타입입니다. 
never 타입으로 선언된 변수의 범위는 공집합이기 때문에 아무런 값도 할당할 수 없습니다.

```ts
const x: never = 12;
```

<img width="1034" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4c996a82-8b40-4f9d-b93e-f565bf61de30"/>

그 다음으로 작은 집합은 한가지 값만 포함하는 타입입니다. 이들은 타입스크립트에서 유닛(unit)타입이라고도 불리는 리터럴(literal) 타입입니다.

```ts
type A = 'A';
type B = 'B';
type Twelve = 12;

```

두개 혹은 세개로 묶으려면 유니온(Union)타입을 사용합니다.

```ts
type AB = 'A' | 'B';
type AB12 = 'A' | 'B' | 12;

```

세 개 이상의 타입을 묶을 때도 동일하게 `|`로 이어주면 됩니다.<br/>
유니온 타입은 값 집합들의 합집합을 일컫습니다.

다양항 타입스크립트 오류에서 '할당 가능한'이라는 문구를 볼 수 있습니다.<br/>
이 문구는 집합의 관점에서 '~의 원소(값과 타입의 관계)' 또는 '~의 부분 집합(두 타입의 관계)'을 의미합니다.

```ts
const a : AB = 'A';
const c : AB = 'C';

```

<img width="1034" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/92a4ce9f-a1be-410b-bef1-693c99e569f5"/>

"C"는 유닛 타입입니다.<br/>
범위는 단일한 값 "C"로 구성되며 AB("A"와 "B"로 이루어진)의 부분 집합이 아니므로 오류입니다.
집합의 관점에서, 타입 체커의 주요 역할은 하나의 집합이 다른 집합인지 검사하는 것이라고 볼 수 있습니다.

```ts
const ab : AB = Math.random() < 0.5 ? 'A' : 'B';
const ab12 : AB = ab;

declare let twelve: AB12;
const back: AB = twelve;
```

<img width="1039" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/fc38d53f-db78-4cb2-9ea9-33f0a8fbfc4e"/>

앞의 인터페이스가 타입 범위 내의 값들에 대한 설명이라고 생각해 보겠습니다.<br/>
어떤 객체가 string으로 할당 가능한 id속성을 가지고 있다면 그 객체는 Identified입니다.

이 설명이 이번 아이템에서 말하고자 하는 '전부'입니다.<br/>
아이템 4에서 설명했듯이, 구조적 타이핑 규칙들은 어떠한 값이 다른 속성도 가질 수 있음을 의미합니다.
심지어 함수 호출의 매개변수에서도 다른 속성을 가질 수 있습니다.
이러한 사실은 특정 상황에서만 추가 속성을 허용하지 않는 잉여 속성 체크 (excess property checking)만 생각하다 보면 간과하기 쉽습니다. ([아이템 11](https://github.com/Pyotato/effective_typescript/blob/item11/README.md))

연산과 관련된 이해를 돕기 위해 값의 집합을 타입이라고 생각해 보겠습니다. 

```ts
interface Person {
  name: string;
}
interface LifeSpan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & LifeSpan;
```

`&` 연산자는 두 타입의 인터섹션(intersection, 교집합)을 계산합니다.<br/>
언뜻 보기에 Person과 LifeSpan 인터페이스는 공통으로 가지는 속성이 없기 때문에, PersonSpan 타입을 공직합(never 타입)으로 예상하기 쉽습니다.
그러나 추가적인 속성을 가지는 값도 여전히 그 타입에 속합니다.
그래서 Person과 LifeSpan을 둘다 가지는 값은 인터섹션 타입에 속하게 됩니다.

```ts
const ps: PersonSpan = {
  name: 'Alan Turing',
  birth: new Date('1912/06/23'),
  death: new Date('1954/06/07'), // 정상
};
```

당연히 앞의 세가지 보다 더 많은 속성을 가지는 값도 PersonSpan 타입에 속합니다. <br/>
인터섹션 타입의 값은 각 타입 내의 속성을 모두 포함하는 것이 일반적인 규칙입니다.

규칙이 속성에 대한 인터섹션에 관해서는 맞지만, 두 인터페이스의 유니온에서는 그렇지 않습니다.

```ts
type K = keyof (Person | LifeSpan); 
```

<img width="272" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/a9d906a4-1601-457b-99b3-aca6a1e340c1"/>

앞의 유니온 타입에 속하는 값은 어떠한 키도 없기 때문에, 유니온에 대한 `keyof`는 공집합(never)이어야만 합니다.<br/>

```ts
keyof (A&B) | (keyof B)
keyof (A|B) =  (keyof A) & (keyof B)

```

이 등식은 타입스크립트의 타입 시스템을 이해하는 데 큰 도움이 될 것입니다.

조금 더 일반적으로 PersonSpan 타입을 선언하는 방법은 `extends 키워드`를 쓰는 것입니다.

```ts
interface Person{
  name: string;
}
interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

타입의 집합이라는 관점에서 extends의 의미는 '~에 할당 가능한;과 비슷하게, '~의 부분 집합'이라는 의미로 받아들일 수 있습니다.<br/>
PersonSpan 타입의 모든 값은 문자열 name속성을 가져야 합니다. 그리고 birth 속성을 가져야 제대로 된 부분 집합이 됩니다.

'서브타입'이라는 용어를 들어 봤을 겁니다. <br/>
어떤 집합이 다른 집합의 부분집합이라는 의미입니다. 1차원, 2차원, 3차원 벡터의 관점에서 생각해 보면 다음과 같은 코드를 작성할 수 있습니다.

```ts
interface Vector1D{x: number;}
interface Vector2D extends Vector1D{y: number;}
interface Vector3D extends Vector2D{z: number;}
```

Vector3D는 Vector2D의 서브타입이고 Vector2D는 Vector1D의 서브타입입니다(클래스 관점에서는 '서브클래스'가 됩니다).
보통 이 관계는 상속 관계로 그려지지만, 집합의 관점에서는 벤 다이어그램을 그리는 게 더욱 적절합니다.

<img width="468" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/158eb1f3-0dc9-41e2-b38e-119382f29df8"/>

벤 다이어그램을 보고, extends 없이 인터페이스로 코드를 재작성해 보면 부분집합, 서브집합, 할당 가능성의 관계가 바뀌지 않는다는 걸 명확히 알 수 있습니다.

```ts
interface Vector1D{x: number;}
interface Vector2D {x: number; y: number;}
interface Vector3D {x: number; y: number; z: number;}
```

집합들은 바뀌지 않았고, 벤 다이어그램 역시 마찬가지입니다. <br/>
두 가지 스타일 모두 객체 타입에 대해서 잘 동작하지만, 리터럴 타입과 유니온 타입에 대해 생각해 본다면 집합 스타일이 훨신 직관적입니다.

`extends` 키워드는 제너릭 타입에서 한정자로도 쓰이며, 이문맥에서는 '~의 부분 집합'을 의미하기도 합니다 ([아이템 14](https://github.com/Pyotato/effective_typescript/blob/item14/README.md)) 

```ts
function getKey<K extends string>(val: any, key: K){
  // ...
}
```

string을 상속한다는 의미를 객체 상속의 관점으로 생각한다면 이해하기가 어렵습니다. 
상속의 관점에서는 객체 래퍼(wrapper) 타입 String([아이템 10](https://github.com/Pyotato/effective_typescript/blob/item10/README.md))의 서브클래스를 정의해야하지만, 그리 바랍직해 보이지 않습니다.

반면 string을 상속한다는 의미를 집합의 관점으로 생각해보면 쉽게 이해할 수 있습니다. string의 부분 집합 범위를 가지는 어떤 타입이 됩니다. 
이 타입은 string 리터럴 타입, string 리터럴 타입의 유니온, string 자신을 포함합니다.

```ts
getKey({}, 'x');
getKey({}, Math.random() < 0.5? 'a' : 'b');
getKey({}, document.title);
getKey({}, 12);
```

<img width="1236" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/70f65e24-8c59-468d-bc90-f516af50899c"/>

마지막 오류에서 "할당 될 수 없습니다"는 상속의 관점에서 "상속할 수 없습니다"로 바꿀 수 있고, 두 표현 모두 '~의 부분 집합'의 의미로 받아들인다면 문제가 없습니다.<br/>
이렇게 할당과 상속의 관점을 전환해보면, 객체의 키 타입을 반환한는 `keysof T`를 이해하기 수월합니다.

```ts
interface Point{
  x: number;
  y: number;
}

type PointKeys = keyof Point; // 타입은 "x" | "y"

function sortBy<K extends keyof T, T>(vals: T[], key: K){ /** */}
const pts : Point[] = [{x: 1, y: 1},{x:2, y: 0}];
sortBy(pts,'x');
sortBy(pts,'y');
sortBy(pts,Math.random()<0.5?'x':'y');
sortBy(pts,'z');
```

<img width="599" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/049e5e35-8830-424b-88b3-111e20247d8d"/>

타입들이 엄격한 상속 관께가 아닐 때는 집합 스타일이 더욱 바람직합니다.
예를 들어, string|number와 string|Date 사이의 인터섹션은 공집합이 아니며 (string입니다) 서로의 부분 집합도 아닙니다.
이 압들이 엄격한 상속 관계가 아니더라도 범위에 대한 관계는 명확합니다.

<img width="406" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/779acd25-e278-4cf5-81dd-e3cbd9c2a29f"/>

타입이 집합이라는 관점은 배열과 튜플의 관계 역시 명확하게 만듭니다. 예를 들어 보겠습니다.

```ts
const list = [1, 2]; // 타입은 number[]
const tuple: [number, number] = list;
```

<img width="1127" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/ac4b1e2c-8f51-439d-9529-5fa6281102f8"/>

이 코드에서 숫자 배열을 숫자들의 쌍(pair)이라고 할 수는 없습니다.<br/>
빈 리스트와 `[1]`이 그 반례입니다. `number[]`는 `[number, number]`의 부분 집합이 아니기 때문에 할당할 수 없습니다. (반면 그 반대로 할당하면 동작합니다.)

트리플(triple, 세 숫자를 가지는 타입)은 구조적 타이핑의 관점으로 생각하면 상으로 할당 가능할 것으로 생각됩니다.<br/>
그롷다면 쌍은 0번과 1번 키를 가지므로, 2번 같은 다른 키를 가질 수 잇을 지 확인해 보겠습니다.

```ts
const triple : [number,number,number] =[1,2,3];
const double: [number,number] = triple;
```

<img width="1286" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8b6d98e9-dd52-4453-ad98-0bce457ef997"/>

오류가 발생했는데, 그 이유가 매우 흥미롭습니다.<br/>
타입스크립트는 숫자의 쌍을 {0: number, 1: number}로 모델링하지 않고, {0: number, 1: number, length :2}로 모델링했습니다.
그래서 length의 값이 맞지 않기 떄문에 할당문에 오류가 발생했습니다.
쌍의 길이를 체크하는 것은 합리적이며, 이보다 나은 방법은 없습니다.

타입이 값의 집합이라는 건, 동일한 값의 집합을 가지는 두 타입은 같다는 의미가 됩니다. 
두 타입이 의미적으로 다르고 우연히 같은 범위를 가진다고 하더라도, 같은 타입을 두 번 정의할 이유는 없습니다.

한편 타입스크립트 타입이 되지 못하는 값의 집합들이 있다는 것을 기억해야합니다.<br/>
정수에 대한 타입, 또는 x와 y속성 외에 다른 속성이 없는 객체는 타입스크립트에 존재하지 않습니다.
가끔 Exclude를 사용해서 일부 타입을 제외할 수는 있지만, 그 결과가 적절한 타입스크립트 타입일 때만 유효합니다.

```ts
type T = Exclude < string|Date, string|number >;
type NonZeroNum = Exclude<number, 0>;
```

<img width="400" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0d095ea9-6046-4423-aca6-37be572b4b1e"/><img width="400" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/ed234273-7e1b-4c63-91af-2953fb767af4"/>




타입스크립트 용어와 집합 이론 용어 사이의 대응 관계는 다음과 같이 정리했습니다.

|타입스크립트 용어|집합 용어|
|:---|:------|
|`never`|∅(공집합)|
|`리터럴 타입`|원소가 한 개인 집합|
|`값이 T에 할당 가능`|값 ∈ T (값이 T의 원소)|
|`T1이 T2에 할당 가능`|T1 ⊆ T2 (T1이 T2의 부분집합)|
|`T1이 T2을 상속`|T1 ⊆ T2 (T1이 T2의 부분집합)|
|`T1 ❘ T2(T1과 T2의 유니온)` |T1 ⋃ T2 (T1와 T2의 합집합)|
|`T1 & T2(T1과 T2의 인터섹션)`|T1 ⋂ T2 (T1이 T2의 교집합)|
|`unknown`|전체 (universal) 집합|













