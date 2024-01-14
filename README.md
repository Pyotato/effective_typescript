# 👎 ITEM 5 : any 타입 지양하기

타입스크립트의 타입 시스켐은 점진적(gradual)이고 선택적(optional)입니다.
코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입 체커를 해제할 수 있기 때문에 선택적입니다.
이 기능들의 핵심은 any타입 입니다.

```ts
let age: number;
age = '12';
age = '12' as any;
```

<img width="952" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/71e7ddd4-b37a-4afc-a32b-86838667c6ac"/>

타입 체커를 통해 앞의 코드에서 오류를 찾아냈습니다. 오류는 `as any`를 추가해 해결할 수 있습니다. 
여러분이 타입스크립트를 처음 접했다면 오류가 이해되지 않거나, 타입 체커가 틀렸다고 생각할 수도 있습니다.
타입 선언을 추가하는 데에 시간을 쏟고 싶지 않아서 `any 타입`이나 타입 단언문 `(as any)`을 사용하고 싶기도 할 겁니다.
그러나 일부 특별한 경우를 제외하고는 any를 사용하면 타입스크립트의 수많은 장점을 누릴 수 없게 됩니다.
부득이하게 any를 사용하더라도 그 위험성을 알고 있어야 합니다.

## any타입에는 타입 안정성이 없습니다

앞선 예제에서 age는 number 타입으로 선언되었습니다. 
그러나 as any를 사용하면 string타입을 할당할 수 있게 됩니다. 타입 체커는 선언에 따라 number 타입으로 판단할 것이고 혼돈은 걷잡을 수 없게 됩니다.

```ts
age += 1; // 런타임에 정상, age는 '121'
``

## any는 함수 시그니처를 무시해 버립니다.

함수를 작성할 때는 시그니처를 명시해야 합니다. 호출하는 쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입의 출력을 반환합니다.
그러나 any타입을 사용하면 이런 약속을 어길 수 있습니다.

> 원서에는 시그니처 대신 contract로 되어 있습니다.
contract를 직역하면, 계약, 약속 정도가 될 것입니다. 그러나 이 문맥에서 계약이란 단어는 어색하고 시그니처가 더 이해하기 좋습니다.

```ts
function calculateAge(birthDate: Date) : number {
// ...
}

let birthDate: any = '1990-01-19';
calculateAge(birthDate); // 정상
```

`birthDate` 매개변수는 string이 아닌 Date 타입이어야 합니다.
any타입을 사용하면 calculateAge의 시그니처를 무시하게 됩니다.
자바스크립트에서는 종종 암시적으로 타입이 변환되기 때문에 이런 경우 특지 문제가 될 수 있습니다. string 타입은 number 타입이 필요한 곳에서 오류 없이 실행될 때가 있고, 그럴 경우 다른 곳에서 문제를 일으키게 될 겁니다.

## any 타입에는 언어 서비스가 적용되지 않습니다

어떤 심벌에 타입이 이싿면 타입스크립트 언어 서비스는 자동완성 기능과 적절한 도움말을 제공합니다.

```ts
let person = {first: 'George', last: 'Washington'};
person.
```

<img width="490" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/d4974fcc-020e-44ab-9578-98ee098fe44d"/>

그러나 any타입인 심벌을 사용하면 아무런 도움을 받지 못합니다.

이름 변경 기능은 또 다른 언어 서비스입니다. 다음 코드는 Person 타입과 이름을 포매팅해 주는 함수입니다.

```ts
interface Person {
  first: string;
  last: string;
}

const formatName = (p: Person) => `${p.first} ${p.last}`;
const formatNameAny = (p:any) => `${p.first} ${p.last}`;
```

편집기에서 앞의 코드의 first를 선택하고, `Rename Symbol`을 클릭해, firstName으로 바꿀 수 있습니다.

```ts
interface Person {
  firstName: string;
  last: string;
}

const formatName = (p: Person) => `${p.firstName} ${p.last}`;
const formatNameAny = (p:any) => `${p.first} ${p.last}`;
```

타입스크립트의 모토는 '확장 가능한 자바스크립트'입니다.
'확장'의 중요한 부분은 바로 타입스크립트의 경험의 핵심 요소인 언어 서비스입니디. ([아이템 6](https://github.com/Pyotato/effective_typescript/blob/item6/README.md) 참고)
언어 서비스를 제대로 누려야 독자 여러분과 동료의 생산성이 향상됩니다.

## any 타입은 코드 리팩터링 때 버그를 감춥니다

어떤 아이템을 선택할 수 있는 웹 애플리케이션을 만든다고 가정해 보겠습니다.
애플리케이션에는 `onSelectItem` 콜백이 있는 컴포넌트가 있을 겁니다.
선택하려는 아이템의 타입이 무엇인지 알기 어려우니 `any`를 사용해 보겠습니다.

```ts
interface ComponentProps {
  onSelectItem: (item: any) => void;
}
```

다음과 같이 `onSelectItem` 콜백이 있는 컴포넌트를 사용하는 코드도 있을 겁니다.

```ts
function renderSelector(props: ComponentProps){
    /** */
}

let selectedId : number = 0;

function handleSelectItem(item: any){
  selectedId = item.id
}

renderSelector({onSelectItem: handleSelectItem});

```

`onSelectItem`에 아이템 객체를 필요한 부분만 전달하도록 컴포넌트를 개선해 보겠습니다.
여기서는 id만 필요합니다.
`ComponentProps`의 시그니처를 다음처럼 변경합니다.

```ts
interface ComponentProps {
  onSelectItem: (item: number) => void;
}
```

컴포넌트를 수정하고, 타입 체크를 모두 통과했습니다.

타입 체크를 통과했다고 끝난 것은 아닙니다.
`handleSelectItem`은 `any` 매개 변수를 받습니다. 따라서 id를 전달받아도 문제가 없다고 나옵니다.
id를 전달받으면, 타입 체커를 통과함에도 불구하고 런타임에는 오류가 발생합니다.
`any`가 아니라 구체적인 타입을 사용했다면, 타입 체커가 오류를 발견했을 겁니다.

## any 타입 설계를 감춰버립니다

애플리케이션 상태같은 객체를 정의하려면 꽤 복잡합니다.
상태 객체 안에 있는 수많은 속성의 타입을 일일이 작성해야 하는데, any 타입을 사용하면 간단히 끝내버릴 수 있습니다.

물론 이때도 `any`를 사용하면 안 됩니다.

이유는 앞에서 계쏙 설명해 왔습니다. 객체를 정의할 때 특히 문제가 되는데, 상태 객체의 설계를 감춰버리기 때문입니다.
4장에서 설명하겠지만, 깔끔하고 정확하고 명료한 코드 작성을 위해 제대로 된 타입 설계는 필수입니다.
`any`타입을 사용하면 타입 설계가 불분명해집니다.
설계가 잘 되었는지, 설계가 어떻게 되어 있는지 전혀 알 수 없습니다.
만약 동료가 코드를 검토해야 한다면, 동료는 애플리케이션의 상태를 어떻게 변경했는지 코드부터 재구성해 봐야 할 겁니다.
그러므로 설계가 명확히 보이도록 타입을 일일이 작성하는 것이 좋습니다.

## any는 타입시스템의 신뢰도를 떨어뜨립니다

사람은 항상 실수를 합니다. 보통은 타입 체커가 실수를 잡아주고 코드의 신뢰도가 높아집니다. 
그러나 런타임에 타입 오류를 발견하게 된다면 타입 체커를 신뢰할 수 없을 겁니다.
대규모 팀에 타입스크립트를 도입하려는 상황에서, 타입 체커를 신뢰할 수 없다면 큰 문제가 될 겁니다.
`any` ㅌ아ㅣㅂ을 쓰지 않으면 런타임에 발견될 오루를 미리 잡을 수 있고 신뢰도를 높일 수 있습니다.

타입스크립트는 개발자의 삶을 편하게 하는 데 목적이 있지만, 코드 내에 존재하는 수많은 `any` 타입으로 인해 자바스크립트보다 일을 더 어렵게 만들기도 합니다.
타입 오류를 고쳐야 하고, 여전히 머릿속에 실제 타입을 기억해야하기 때문입니다.
타입이 실제 값과 일치한다면 타입 정보를 기억해 둘 필요가 없습니다. 타입스크립트가 타입 정보를 기억해 주기 때문입니다.

어쩔 수 없이 `any`를 써야만 하는 상황도 있습니다. 이럴 때 고민해 볼 수 있는 좋은 방법과 좋지 못한 방법이 있습니다.
`any`의 단점을 어떻게 보완해야 하는지는 5장에서 자세히 다룹니다.

