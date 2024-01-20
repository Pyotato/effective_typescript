#  😌 ITEM 14 : 타입 연산과 제너릭 사용으로 반복 줄이기

다음은 원기둥 (cylinder)의 반지름과 높이, 표면적, 부피를 출력하는 코드입니다.

```ts
console.log('Cylinder = 1 x 1', 'Surface area: ', 6.283185 * 1 * 1 + 6.283185 * 1 * 1, 'Volume: ', 3.1459 * 1 * 1 * 1);
console.log('Cylinder = 1 x 2', 'Surface area: ', 6.283185 * 1 * 1 + 6.283185 * 2 * 1, 'Volume: ', 3.1459 * 1 * 2 * 1);
console.log('Cylinder = 2 x 1', 'Surface area: ', 6.283185 * 2 * 1 + 6.283185 * 2 * 1, 'Volume: ', 3.1459 * 2 * 2 * 1);
```

비슷한 코드가 반복되어 있어 보기 불편합니다.
값과 상수가 반복되는 바람에 드러나지 않은 오류까지 가지고 있습니다.
이 코드에서 함수, 상수, 루프의 반복을 제거해 코드를 개선해 보겠습니다.

```ts
const surfaceArea = (r, h) => 2 * Math.PI * r * (r+h);
const volume = (r, h) => Math.PI * r * r * h;
for(const [r, h] of [[1,1], [1,2], [2,1]]){
  console.log(`Cylinder = ${r} x ${h}`,
              `Surface area: ${surfaceArea(r, h)}`,
              `Volume: ${volume(r, h)}`)
}
```

이게 바로 같은 코드를 반복하지 말라는 DRY(Don't Repeat Yourself) 원칙입니다.
소프트웨어 개발자라면 어느 분야에서든 들어 봤을 조언입니다.
그런데 반복된 코드를 열심히 제거하며 DRY 원칙을 지켜왔던 개발자라도 타입에 대해서는 간과했을지 모릅니다.

```ts
interface Person{
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate {
  firstName: string;
  lastName: string;
  birth: Date;
}
```

타입 중복은 코드 중복만틈 많은 문제를 발생시킵니다.
예를 들어 선택적 필드인 middleName을 Person에 추가한다고 가정해 보겠습니다.
그러면 Person과 PersonWithBirthDate는 다른 타입이 됩니다.

타입에 중복이 더 흔한 이유 중 하나는 공유된 패턴을 제거하는 매커니즘이 기존에서 하던 것과 비교해 덜 익숙하기 때문입니다.
헬퍼 함수 중복 제거와 동일한 활동이 타입 시스템에서는 어떤 것에 해당할지 상상이 잘 되지 않습니다.
그러나 타입 간에 매핑하는 방법을 익히면, 타입 정의에서도 DRY의 장점을 적용할 수 있습니다.

반복을 줄이는 가장 간단한 방법은 타입에 이름을 붙이는 것입니다.
다음 예제의 거리 계산 함수에는 타입이 반복적으로 등장합니다.

```ts
function distance(a: {x: number, y: number}, b: {x: number, y: number}){
  return Math.sqrt(Math.pow(a.x-b.x,2) + Math.pow(a.y-b.y,2));
}
```

코드를 수정해 타입에 이름을 붙여 보겠습니다.

```ts
interface Point2D{
  x: number;
  y: number;
}

function distance(a:Point2D, b: Point2D){ /*....*/ };
```

이 코드는 상수를 사용해서 반복을 줄이는 기법을 동일하게 타입 시스템에 적용한 것입니다.
중복된 타입 찾기가 항상 쉬운 일은 아닙니다.
중복된 타입은 종종 문법에 의해서 가려지기도 합니다.
예를 들어, 몇몇 함수가 같은 타입 시그니처를 공유하고 있다고 해 보겠습니다.

```ts
function get(url: string, opts: Options) : Promise<Response>{ /*....*/ };
function post(url: string, opts: Options) : Promise<Response>{ /*....*/ };
```

그러면 해당 시그니처를 명명된 타입으로 분리해 낼 수 있습니다.

```ts
type HTTPFunction = {url: string, opts: Options} =>  Promise<Response>;
function get: HTTPFunction (url, opts) { /*....*/ };
function post: HTTPFunction (url, opts) { /*....*/ };
```

(명명된 타입과 관련해서 [아이템 12](https://github.com/Pyotato/effective_typescript/blob/item12/README.md)에서 자세히 다룹니다.)

Person/PersonWithBirthDate 에제에서는 한 인터페이스가 다른 인터페이스를 확장하게 해서 반복을 제거할 수 있습니다.

```ts
interface Person{
  firstName: string;
  lastName: string;
}

interface PersonWithBirthDate extends Person{
  birth: Date;
}
```

이제 추가적인 필드만 작성하면 됩니다.
만약 두 인터페이스가 필드의 부분 집합을 공유한다면, 공통 필드만 골라서 기반 클래스로 분리해 낼 수 있습니다.
코드 중복의 경우와 비교하면, 3.14159과 6.283185 대신 PI와 PI*2로 작성하는 것과 비슷한 이치입니다.

이미 존재하는 타입을 확장하는 경우에, 일반적이지는 않지만 인터섹션 연산자(&)를 쓸 수도 있습니다.

```ts
interface Person{
  firstName: string;
  lastName: string;
}

type PersonWithBirthDate =  Person & {birth: Date };
```

이런 기법은 유니온 타입(확장할 수 없는)에 속성을 추가하려고 할 때 특히 유용합니다. 유니온 타입은 [아이템 13](https://github.com/Pyotato/effective_typescript/blob/item13/README.md)에서 자세히 다룹니다.

이제 다른 측면을 생각해 보겠습니다.
전체 애플리케이션의 상태를 표현하는 State 타입과 단지 부분만 표현하는 TopNavState가 있는 경우를 살펴보겠습니다.

```ts
interface State{
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

interface TopNavState{
  userId: string;
  pageTitle: string;
  recentFiles: string[];
}
```

TopNavState를 확장하여 State를 구성하기보다, State의 부분 집합으로 TopNavState를 정의하는 것이 바람직해 보입니다.
이 방법이 전체 앱의 상태를 하나의 인터페이스로 유지할 수 있게 해 줍니다.

State를 인덱싱하여 속성의 타입에서 중복을 제거할 수 있습니다.

```ts
interface TopNavState{
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}
```

중복 제거가 아직 끝나지 않았습니다. State내의 pageTitle의 타입이 바뀌면 TopNavState에도 반영됩니다.
그러나 여전히 반복되는 코드가 존재합니다.
이 때 '매핑된 타입'을 사용하면 좀 더 나아집니다. 

```ts
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles'] : State[k]
};
```

TopNavState에 마우스를 올리면 정의가 표시되는데, 이정의는 앞의 예제와 완전히 동일합니다.

<img width="445" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0f8eaf52-606a-4b1b-aab8-9ac3a06b370f"/>

매핑된 타입은 배열의 필드를 루프 도는 것가 같은 방식입니다.
이 패턴은 표준 라이브러리에서도 일반적으로 찾을 수 있으며, Pick이라고 합니다.

```ts
type Pick<T, K> = {[k in K]: T[k]};
```

정의가 완전하지는 않지만 다음과 같이 사용할 수 있습니다.

```ts
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

여기서 Pick은 제너릭 타입입니다.
중복된 코드를 없앤다는 관점으로, Pick을 사용하는 것은 함수를 호출하는 것에 비유할 수 있습니다.
마치 함수에서 두 개의 매개변수 값을 받아서 결괏값을 반호나하는 것처럼, Pick은 T와 K 두 가지 타입을 받아서 결과 타입을 반환합니다.

태그된 유니온에서도 다른 형태의 중복이 발생할 수 있습니다.
그런데 단순히 태그를 붙이기 위해서 타입을 사용한다면 어떨지 생각해 보겠습니다.

```ts
interface SaveAction{
  type: 'save';
  // ...
}

interface LoadAction{
  type : 'load';
  // ...
}

type Action= SaveAction | LoadAction;
type ActionType = 'save' | 'load'; // 타입이 반복
```

Action 유니온을 인덱싱하면 타입 반복 없이 ActionType를 정의할 수 있습니다.

```ts
type ActionType = Action['type']; // 타입은 'save' | 'load'
```

Action 유니온에 타입을 더 추가하면 ActionType은 자동적으로 그 타입을 포함합니다.
ActionType은 Pick을 사용하여 얻게 되는, type 속성을 가지는 인터페이스와는 다릅니다.

```ts
type ActionRec = Pick<Action, 'type'>; // { type: 'save' | 'load' }
```

한편 생성하고 난 다음에 업데이트가 되는 클래스를 정의한다면, update 매서드 매개변수의 타입은 생성자와 동일한 매개변수이면서, 타입 대부분이 선택적 필드가 됩니다.

```ts
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}

interface OptionsUpdate {
  width?: number;
  height?: number;
  color?: string;
  label?: string;
}

class UIWidget{
  constructor(init: Options){/*...*/}
  update(options: OptionsUpdate)
}
```

매핑된 타입과 keyof를 사용하면 Options으로부터 OptionsUpdate를 만들 수 있습니다.

```ts
type OptionsUpdate = {[k in keyof Options]?: Options[k]};
```

keysof는 타입을 받아서 속성 타입의 유니온을 반환합니다.

```ts
type OptionsKeys = keyof Options;
// 타입이 "width" | "height" |  "color" | "label"
```

매핑된 타입(`[k in keyof Options]`)은 순회하면서 Options 내의 k값에 해당하는 속성이 있는지 찾습니다.
?는 각 속성을 선택적으로 만듭니다.
이 패턴 역시 아주 일반적이며 표준 라이브러리에 Partial이라는 이름으로 포함되어 있습니다.

```ts
class UIWidget(){
  constructor(init: Options){/*...*/}
  update(options: Partial<Options>){/*...*/}
}
```

값의 형태에 해당하는 타입을 정의하고 싶을 때도 있습니다.

```ts
const INIT_OPTIONS = {
  width: 640,
  height: 480,
  color: '#00FF00',
  label: 'VGA';
};
interface Options {
  width: number;
  height: number;
  color: string;
  label: string;
}
```

이런 경우 typeof를 사용하면 됩니다.

```ts
type Options = typeof INIT_OPTIONS;
```

이 코드는 자바스크립트의 런타임 연산자 typeof를 사용한 것처럼 보이지만, 실제로는 타입스크립트 단계에서 연산되며 훨씬 더 정확하게 타입을 표현합니다.
(typeof에 대한 자세한 내용은 [아이템 8](https://github.com/Pyotato/effective_typescript/blob/item8/README.md)에서 다룹니다.)
그런데 값으로부터 타입을 만들어 낼 때는 선언의 순서에 주의해야 합니다.
타입 정의를 먼저하고 값이 그 타입에 할당 가능하다고 선언하는 것이 좋습니다.
그렇게 해야 타입이 더 명확해지고, 예상하기 어려운 타입 변동을 방지할 수 있습니다. ([아이템 21](https://github.com/Pyotato/effective_typescript/blob/item21/README.md))

함수나 메서드의 반환 값에 명명된 타입을 만들고 싶을 수도 있습니다.

```ts
function getUserInfo(userId: string){
  // ...
  return {
    userId,
    name,
    age,
    height,
    weight,
    favoriteColor,
  }
}

// 이 때 타입은 { userId: string, name: string, age: number, ... }
```

이 때는 조건부 타입([아이템 50](https://github.com/Pyotato/effective_typescript/blob/item50/README.md))이 필요합니다.
그러나 앞에서 살펴본 것처럼, 표준 라이브러리에는 이러한 일반적 패턴의 제너릭 타입이 정의되어 있습니다.
이런 경우 ReturnType 제너릭이 정확히 들어맞습니다.

```ts
type UserInfo =  ReturnType<typeof getUserInfo>;
```

ReturnType은 함수의 '값'인 getUserInfo가 아니라 함수의 '타입'인 typeof getUserInfo에 적용되었습니다.
`typeof`와 마찬가지로 이런 기법은 신중하게 사용해야 합니다.
적용 대상이 값인지 타입인지 정확히 알고, 구분해서 처리해야 합니다.

제너릭 타입은 타입을 위한 함수와 같습니다.
그리고 함수는 코드에 대한 DRY 원칙을 지킬 때 유용하게 사용됩니다.
따라서 타입에 대한 DRY 원칙이 핵심이 제너릭이라는 것은 어쩌면 당연해 보이는데, 간과한 부분이 있습니다.
함수에서 매개변수로 매핑할 수 있는 값을 제한할 수 있는 방법이 필요합니다.

제너릭 타입에서 매개변수를 제한할 수 있는 방법은 `extends`를 사용하는 것입니다.
`extends`를 이용하면 제너릭 매개변수가 특정 타입을 확장한다고 선언할 수 있습니다.
예를 들어 보겠습니다.

```ts
interface Name {
  first: string;
  last: string;
}

type DancingDuo<T extends Name>=  [T,T];

const couple1 : DancingDuo<Name>  = [
  {first: 'Fred', last: 'Astaire'},
  {first: 'Ginger', last: 'Rogers'},
]; // OK

const couple2: DancingDuo<{first: string}>  = [
  {first: 'Sonny'},
  {first: 'Cher'},
]; 

```

<img width="816" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/eea19207-1788-4a74-9c3f-40a5ccfb0ada"/>

> ✅ 현재의 타입스크립트에서는 선언부에 항상 제너릭 매개변수를 작성하도록 되어 있습니다. <br/>
> DancingDuo<Name> 대신 DancingDuo를 쓰면 동작하지 않습니다.<br/>
> 타입스크립트가 제너릭 매개변수의 타입을 추론하게 하기 위해, 함수를 작성할 때는 신중하게 타입을 고려해야합니다. <br/>
> ```ts
> const dancingDuo = <T extends Name> (x: DancingDuo<T>) => x;
> const couple1 = dancingDuo([
>   {first: 'Fred', last: 'Astaire'},
>   {first: 'Ginger', last: 'Rogers'},
> ]);
> const couple2 = dancingDuo([
>   {first: 'Bono'},
>   {first: 'Prince'},
> ]);
> ```
>
> <img width="1188" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6d724e3c-47cd-4d75-bb7f-1b58c3af33e6"/>
> 

앞에 나온 Pick의 정의는 extends를 사용해서 완성할 수 있습니다.
타입 체커를 통해 기존 예제를 실행해 보면 오류가 발생합니다.

```ts
type Pick<T, K> = {
  [k in K] : T[k]
}
```

<img width="1188" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/704558a8-9c8c-4e1f-ae2b-66eecf6fb298"/>

K는 T타입과 무관하고 범위가 너무 넓습니다. K는 인덱스로 사용될 수 있습니다. `string | number | symbol`이 되어야 하며 실제로는 범위를 조금 더 좁힐 수 있습니다.
K는 실제로 T의 키의 부분 집합, 즉 `keyof T`가 되어야 합니다.

```ts
type Pick<T, K extends keyof T> = {
  [k in K] : T[k]
}
```

타입([아이템 7](https://github.com/Pyotato/effective_typescript/blob/item7/README.md))이 값의 집합이라는 관점에서 생각하면 extends를 '확장'이 아니라 '부분 집합'이라는 걸 이해하는데 도움이 될 겁니다.

점점 더 추상적인 타입을 다루고 있지만, 원래의 목표를 잊으면 안 됩니다.

원래의 목표는 유효한 프로그램은 통과시키고 무효한 프로그램에는 오류를 발생시키는 것입니다.
이번 경우의 목표는 바로 Pick에 잘못된 키를 넣으면 오류가 발생해야 한다는 것입니다.

```ts
type FirstLast = Pick<Name, 'first'|'last'>; // 정상
type FirstMiddle = Pick<Name, 'first' | 'middle'>;
```

<img width="751" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/5384a359-1797-4fb8-b441-b5730bf314f2"/>

값 공간에서와 마찬가지로 반복적인 코드는 타입 공간에서도 좋지 않습니다.

타입 공간에서 반복을 줄이려는 작업들은 프로그램 로직에서 하던 작업만큼 익숙하지는 않겠지만, 배울 만한 가치가 있습니다.
반복하지 않도록 주의해야 합니다.


