# 🧑‍⚖️ ITEM 9 : 타입 단언보다는 타입 선언을 사용하기

타입스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지입니다.

```ts
interface Person {name : string};

const alice : Person = {name: 'Alice'}; // 타입은 Person
const bob = {name: 'bob'} as Person; // 타입은 Person

```

이 두가지 방법은 결과가 같아 보이지만 그렇지 않습니다.

첫번째 `alice : Person`은 변수에 `'타입 선언'`을 붙여서 그 값이 선언된 타입임을 명시합니다.
두번째 `as Person`은 `'타입 단언'`을 수행합니다.
그러면 타입스크립트가 추론한 타입이 있더라도 Person 타입으로 간주합니다.

타입 단언보다 탸입 선언을 사용하는 게 낫습니다.
그 이유는 다음 코드에서 확인 할 수 있습니다.

```ts
const alice : Person = {name: 'Alice'};
const bob = {name: 'bob'} as Person;
```

<img width="1111" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/2ae92e41-e903-443e-819b-0f6b61d4b79a"/>

타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사합니다.
앞의 예제에서는 그러지 못했기 때문에 타입스크립트가 오류를 표시했습니다.
타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것입니다.

타입 선언과 단언의 차이는 속성을 추가할 때도 마찬가지입니다.

```ts
const alice : Person = {name: 'Alice', occupation:'Typescript developer'};
const bob = {name: 'bob', occupation:'Typescript developer'} as Person;
```

<img width="1432" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1c579cad-2636-4ea2-8ff8-08e9e8ef7195"/>

타입 선언문에서는 잉여 속성 체크 ([아이템 11](https://github.com/Pyotato/effective_typescript/blob/item11/README.md)) 동작하지만, 단언문에서는 적용되지 않습니다.

타입 단언이 꼭 필요한 경우가 아닌라면, 안전성 체크도 되는 타입 선언을 사용하는 것이 좋습니다.

> ✅ 여러분은 아마 `const bob = <Person>{}`같은 코드르 본 적이 있을 겁니다.
이 코드는 단언문의 원래 문법이며 `{} as Person`과 동일합니다.
> 그러나 `const bob = <Person>{}`같은 코드는 `<Person>`이 _.tsx_파일(타입스크립트 + 리액트)에서 컴포넌트 태그로 인식되기 때문에 현재는 잘 쓰이지 않습니다.

화살표 함수의 타입 선언은 추론된 타입이 모호할 때가 있습니다. 
예를 들어, 다음 코드에서 `Person` 인터페이스르 사용하고 싶다고 가정해 보겠습니다.

```ts
const people = ['alice', 'bob', 'jan'].map(name => ({name}));
```

<img width="584" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9cdd0695-e8a6-46ea-a128-d6b138f3b1fc"/>

`{name}`에 타입 단언을 쓰면 문제가 해결되는 것처럼 보입니다.

```ts
const people = ['alice', 'bob', 'jan'].map(name => ({} as Person));
```

단언문을 쓰지 않고, 다음과 같이 화살표 함수 안에서 타입과 함께 변수를 선언하는 것이 가장 직관적입니다.

```ts
const people = ['alice', 'bob', 'jan'].map(name => {
  const person : Person = {name};
  return person;
});
```

그러나 원래 코드에 비해 꽤나 번잡하게 보입니다. 
코드를 좀 더 간결하게 보이기 위해 변수 대신 화살표 함수의 반환 타입을 선언해 보겠습니다.

```ts
const people = ['alice', 'bob', 'jan'].map((name):Person => ({name}));
```

이 코드는 바로 앞의 번잡한 버젼과 동일한 체크를 수행합니다. 여기서 소괄호는 매우 중요한 의미를 지닙니다. 

`(name):Person`은 name의 타입이 없고, 반환 타입이 Person임을 명시합니다.
그러나 `(name):Person`은 name의 타입이 Person임을 명시하고 반환 타입은 없기 때문에 오류가 발생합니다.

다음 코드는 최종적으로 원하는 타입을 직접 명시하거, 타입스크립트가 할당문의 유효성을 검사하게 됩니다.

```ts
const people : Person[] = ['alice', 'bob', 'jan'].map((name):Person => ({name}));
```

그러나 함수 호출 체이닝이 연속되는 곳에서는 체이닝 시작에서부터 명명된 타입을 가져야합니다.
그래야 정확한 곳에 오류가 표시됩니다.

다음으로 타입 단언이 꼭 필요한 경우를 살펴보겠습니다.
타입 단언은 타입 체커가 추론한 타입보다 여러분이 판단하는 타입이 더 정확할 때 의미가 있습니다.
예를 들어, DOM엘리먼트에 대해서는 타입스크립트보다 여러분이 더 정확히 알고 있을 겁니다.

```ts
document.querySelector('#myButton').addEventListener('click',e =>{
  // e.currentTarget // 타입은 EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button  // 타입은 HTMLButtonElement
});
```

타입스크립트는 DOM에 접근할 수 없기 떄문에 #myButton이 버튼 엘리먼트인 지 알지 못합니다.
그리고 이벤트의 currentTarget이 같은 버튼이어야 하는 것도 알지 못합니다.
우리는 타입스크립트가 알지 못하는 정보를 가지고 있기 때문에 여기서는 타입 단언문을 쓰는 것이 타당합니다.
DOM타입에 대해서는 [아이템 55](https://github.com/Pyotato/effective_typescript/blob/item55/README.md)에서 더 자세히 다룹니다.
또한 자주 쓰이는 특별한 문법(!)을 사용해서 null이 아님을 단언하는 경우도 있습니다.

```ts
const elNull = document.getElementById('foo');
const el = document.getElementById('foo')!;
```

<img width="399" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/2365ceaf-4ac5-47d8-a0af-a45b1a735af4"/>
<img width="399" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/44d4bd0b-9697-49b7-9975-39cea58b935d"/>

변수의 접두사로 쓰인 `!`는 boolean의 부정문입니다.
그러나 접미사로 쓰인 `!`는 그 값이 null이 아니라는 단언문으로 해석됩니다.
우리는 `!`를 일반적인 단언문처럼 생각해야 합니다.
단언문은 컴파일 과정 중에 제거되므로, 타입 체커는 알지 못하지만 그 값이 null이 아니라고 확신할 수 있을 때 사용해야합니다.
만약 그렇지 않다면, null인 경우를 체크하는 조건문을 사용해야 합니다.

타입 단언문으로 임의의 타입 간에 변환을 할 수는 없습니다.
A가 B의 부분 집합인 경우에 타입 단언문을 사용해 변환을 할 수는 없습니다.
`HTMLElement`는 `HTMLElement | null`의 서브타입이기 때문에 이러한 타입 단언은 동작합니다. `HTMLButtonElement`는 `EventTarget`의 서브 타입이기 때문에 역시 동작합니다.

그러나 Person과 HTMLElement는 서로의 서브타입이 아니기 때문에 변환이 불가능합니다.

```ts
interface Person {name: string;};
const body = document.body;
const el = body as Person;
```

<img width="1638" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f65a7538-64ef-4eac-97af-cecadf0ab775"/>

이 오류를 해결하려면 'unknown'타입 ([아이템 42](https://github.com/Pyotato/effective_typescript/blob/item42/README.md)을 사용해야 합니다. 
모든 타입은 unknown의 서브타입이기 때문에 unknown이 포함된 단언문은 항상 동잡합니다. 
unknown 단언은 임의의 타입 간의 변환을 가능케 하지만, unknown을 사용한 이상 적어도 무언가 위험한 동작을 하고 있다는 걸 알 수 있습니다.

```ts
const el = document.body as unknown as Person;
```



