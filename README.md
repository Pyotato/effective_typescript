# ⚜️ ITEM 8 : 타입 공간과 값 공간의 심벌 구분하기

타입스킓트의 심벌(symbol)은 타입 공간이나 값 공간 중 한 곳에는 존재합니다.

심벌은 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있기 때문에 혼란스러울 수 있습니다.

```ts
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({radius, height});
```

`interface Cylinder`에서 Cylinder눈 타입으로 쓰입니다.
`const Cylinder`에서 `Cylinder`와 이름은 같지만 값으로 쓰이며, 서로 아무런 관련도 없습니다.
상황에 따라서 `Cylinder`는 타입으로 쓰일 수도 있고, 값으로 쓰일 수도 있습니다.
이런 점이 가끔 오류를 야기합니다. 

```ts
function calculateVolume(shape: unknown){
  if(shape instanceof Cylinder){
    shape.radius
  }
}
```

<img width="526" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4bab8045-c14c-4f9d-b0f2-0f05929e562f"/>

오류를 살펴보겠습니다. 이미도 `instanceof`를 이용해 shape가 `Cylinder` 타입인지 체크하려고 했을 겁니다.
그러나 `instanceof`는 자바스크립트 런타임 연산자이고, 값에 대해서 연산을 합니다.
그래서 `instanceof Cylinder`는 타입이 아니라 함수를 참조합니다.

한 심벌이 타입인지 값인지는 언뜻 봐서 알 수 없습니다.
어떤 형태로 쓰이는지 문맥을 살펴 알아내야 합니다.
많은 타입 코드가 값 코드와 비슷해 보이기 떄문에 더더욱 혼란스럽습니다.

```ts
type T1 = 'string literal';
type T2 = 123;
const v1 = 'string  literal';
const v2 = 123;
```

일반적으로 type이나 interface 다음에 나오는 심벌은 타입인 반면, const나 let 선언에 쓰이는 것은 값입니다.

두 공간에 대한 개념을 잡으려면 [타입스크립트 플레이그라운드](https://www.typescriptlang.org/play)를 활용하면 됩니다.
타입 선언(:) 또는 단언은 단언문(as) 다음에 나오는 심벌은 타입인 반면, `=` 다음에 나오는 모든 것은 값입니다.
예를 들어, 다음 코드를 보겠습니다.

```ts
interface Person{
  first: string;
  last: string;
}

const p: Person = {first: 'Jane', last: 'Jacobs'};
//    -            ------------------------------    : 값
//       ------                                      : 타입 
```

일부 함수에서는 타입과 값이 반복적으로 번갈아 가며 나올 수 있습니다.

```ts
function email(p: Person, subject: string, body: string): Response { /**..*/};
//       ----  -          -------          ----                                 : 값
//                ------           ------        ------   --------              : 타입 
```

class와 enum은 상황에 따라 타입과 값 두 가지 모두 가능한 예약어입니다. 
다음 예제에서 `Cylinder`클래스는 타입으로 쓰였습니다.

```ts
class Cylinder {
  radius = 1;
  height = 1;
}

function calculateVolume(shape: unknown){
  if(shape instanceof Cylinder){
    shape
    shape.radius
  }
}
```

<img width="315" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/93903324-4f58-4463-88e8-d35a0223aba7"/>
<img width="348" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f2fccd33-bd9f-44c1-9a84-d37b4aec0ac9"/>

클래스가 타입으로 쓰일 때는 형태(속성과 메서드)가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용됩니다.
한편, 연산자 중에서도 타입에서 쓰일 때와 값에서 쓰일 때 다른 기능을 하는 것들이 있습니다.
그 예 중 하나로 `typeof`를 들 수 있습니다.

```ts
type T1 = typeof p;                  // 타입은 Person
type T2 = typeof email;              // 타입은 (p: Person, subject: string, body: string)

const v1 = typeof p;                 // 값은 'object'
const v2 = typeof email;             // 값은 'function'
```

타입의 관점에서 typeof 는 값을 읽어서 타입스크립트 타입을 반환합니다.
타입 공간의 typeof는 보다 큰 타입의 일부분으로 사용할 수 있고, type구문으로 이름을 붙이는 용도로 사용할 수 있습니다.

값의 관점에서 `typeof`는 자바스크립트 런타임의 `typeof` 연산자가 됩니다.
값 공간의 `typeof`는 대상 심벌의 런타임 타입을 가리키는 문자열을 반환하며, 타입스크립트 타입과는 다릅니다.
자바스크립트의 런타임 타입 시스템은 타입 스크립트의 정적 타입 시스템보다 훨씬 간단합니다.
타입스크립트 타입의 종류가 무수히 많은 반면, 자바스크립트에는 과거부터 지금까지 단 6개 (string, number, boolean, undefined, object, function)의 런타임 타입만이 존재합니다.

`Cylinder` 예제에서 본 것처럼 class 키워드는 값과 타입 두 가지로 모두 사용됩니다.
따라서 클래스에 대한 `typeof`는 상황에 따라 다르게 동잡합니다.

```ts
const v = typeof Cylinder;             // 값은 'function'
type T = typeof Cylinder;             // 타입이 typeof Cylinder
```

클래스가 자바스크립트에서는 실제 함수로 구현되기 때문에 첫 번째 줄의 값은 `"function"`이 됩니다.
두 번째 줄의 타입은 무슨 의미인지 감이 오지 않을 겁니다. 
여기서 중요한 것은 Cylinder가 인스턴스의 타입이 아니라는 점입니다.
실제로는 new키워드를 사용할 때 볼 수 있는 생성자 함수입니다.

```ts
declare let fn:T;
const c = new fn(); // 타입이 Cylinder
```

다음 코드처럼 InstanceType 제너릭을 사용해 생성자 타입과 인스턴스 타입을 전환할 수 있습니다.

```ts
type C = InstanceType<typeof Cylinder>
```

속성 접근자인 []는 타입으로 쓰일 때에도 동일하게 동작합니다.
그러나 `obj['field']`와 `obj.field`는 값이 동일하더라도 타입은 다를 수 있습니다.
따라서 타입의 속성을 얻을 때에는 반드시 첫 번째 방법(`obj['field']`)을 사용해야 합니다.

```ts
const first: Person['first'] = p['first'];
//    -----                    ----------   : 값
//           --------------                 : 타입
```

`Person['first']`는 여기서 타입 맥락(: 뒤에) 쓰였기 때문에 타입입니다.
인덱스 위치에는 유니온 타입과 기본형 타입을 포함한 어떠한 타입이든 사용할 수 있습니다.

```ts
type personEl = Person['first' | 'last'];   // 타입은 string
type Tuple = [string, number, Date];
type TupleEl = Tyuple[number ]              // 타입은 string | number | Date
```

속성 접근자에 대한 내용은 [아이템 14](https://github.com/Pyotato/effective_typescript/blob/item14/README.md)에서 더 자세히 다룹니다.

두 공간 사이에서 다른 의미를 가지는 코드 패턴들이 있습니다.

- 값으로 쓰이는 this는 자바스크립트의 this 키워드입니다 ([아이템 49](https://github.com/Pyotato/effective_typescript/blob/item49/README.md)). 타입으로 쓰이는 this는, 일명 '다형성(poltmorphic) this'의 타입스크립트 타입입니다. 서브클래스의 메서드 체인을 구현할 때 유용합니다.
- 값에서 & 와 |는 AND와 OR 비트 연산입니다. 타입에서는 인터섹션과 유니온입니다.
- const는 새 변수를 선언하지만, as const는 리터럴 또는 리터럴 표현식의 추론된 타입을 바꿉니다 ([아이템 21](https://github.com/Pyotato/effective_typescript/blob/item21/README.md)).
- extends는 서브클래스(`class A extends B`) 또는 서브타입(`interface A extends B`)또는 제너릭 타입의 한정자 (`Generic<T extends number>`)를 정의할 수 있습니다.
- in은 루프 (for(key in object)) 또는 매핑된 (mapped) 타입에 등장합니다.

타입스크립트 코드가 잘 동작하지 않는다면 타입 공간과 값 공간을 혼동해서 잘못 작성했을 가능성이 큽니다.
예를 들어, 단일 객체 매개변수를 받도록 email 함수를 변경했다고 생각해 보겠습니다.

```ts
function email(options: {person: Person, subject: string, body:string}){/** */}
```

자바스크립트에서는 객체 내의 각 속성을 로컬 변수로 만들어 주는 구조 분해 (destructuring) 할당을 사용할 수 있습니다.

```ts
function email({person, subject, body}){/** */}
```

그런데 타입스크립트 구조 분해 할당을 하면, 이상한 오류가 발생합니다.

```ts
funciton email({person: Person, subject: string, body:string})
```

<img width="1199" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/e64de597-e5e8-4f19-8a03-82ddadb8c5a1"/>


값의 관점에서 Person과 string이 해석되었기 때문에 오류가 발생했습니다.
Person이라는 변수명과 string이라는 이름을 가지는 두개의 변수를 생성하려 한 것입니다.
문제를 해결하려면 타입과 값을 구분해야 합니다.

```ts
funciton email({person, subject, body}: {person: Person, subject: string, body:string}){/** */}
```

이 코드는 장황하긴 하지만, 매개변수에 명명된 타입을 사용하거나 문맥에서 추론되도록 잘 동작합니다 ([아이템 26](https://github.com/Pyotato/effective_typescript/blob/item26/README.md)).
타입과 값을 비슷한 방식으로 쓰는 점이 처음에는 혼란스러울 수 있지만, 요령을 터득하기만 한다면 마치 연상기호(mnemonic)처럼 무의식적으로 쓸 수 있습니다.
