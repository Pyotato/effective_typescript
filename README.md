#  🙅‍♀️🙆‍♀️ ITEM 13 : 타입과 인터페이스의 차이점 알기

타입스크립트에서 명명된 타입(named type)을 정의하는 방법은 두 가지가 있습니다.
다음처럼 타입을 사용할 수 있습니다.

```ts
type TState = {
  name: string;
  capital: string;
}
```

또는 인터페이스를 사용해도 됩니다.

```ts
interface IState = {
  name: string;
  capital: string;
}
```

> ⚠️ 이 아이템 내의 예제에는 타입을 I(인터페이스) 또는 T(타입)로 시작해 어떤 형태로 정의 되었는지 나타냈습니다. <br/>
> 실제 코드에서는 이렇게 하면 안 됩니다. 인터페이스 접두사로 I를 붙이는 것 C#에서 비롯된 관례입니다. <br/>
> 이 영향을 받아 타입스크립트 초기에는 종종 사용하였으나 현재는 지양해야 할 스타일로 여겨집니다. <br/>
> 표준 라이브러리에서도 일관성있게 도입되지 않았기 때문에 유용하지도 않습니다. <br/>


(명명된 타입을 정의할 때 인터페이스 대신 클래스를 사용할 수도 있지만, 클래스는 갑승로도 쓰일 수 이쓴 자바스크립트 런타임의 개념입니다. 이 내용은 [아이템 8](https://github.com/Pyotato/effective_typescript/blob/item8/README.md)에서 다룹니다.)

대부분의 경우에는 타입을 사용해도 되고 인터페이스를 사용해도 됩니다.

그러나 타입과 인터페이스 사이에 존재하는 차이를 분명하게 알고, 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야 합니다.
그러려면 하나의 타입에 대해 두 가지 방법을 모두 사용해서 정의할 줄 알아야 합니다.

먼저, 인터페이스 선언과 타입 선언의 비슷한 점에 대해 알아보겠습니다. 
명명된 타입은 인터페이스로 정의하든 타입으로 정의하든 상태에는 차이가 없습니다.
만약 IState과 TState를 추가 속성과 함께 할당한다면 동일한 오류가 발생합니다.

```ts
const wyoming: TState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000
}
```

<img width="638" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/5ed389a7-31bd-464c-815f-b63514ef743f"/>

인덱스 시그니처는 인터페이스와 타입에서 모두 사용할 수 있습니다.

```ts
type TDict = {[key: string]: string};
interface IDict {
  [key: string]: string;
}
```

또한 함수 타입에서도 인터페이스나 타입으로 정의할 수 있습니다.

```ts
type TFn = (x: number) => string;
interface IFn {
  (x: number) => string;
}

const toStrT: TFn = x => '' + x;  //정상
const toStrI: IFn = x => '' + x; //정상
```

이런 단순한 함수 타입에는 타입 별칭(alias)이 더 나은 선택이겠지만, 함수는 타입에 추가적인 속성이 있다면 타입이나 인터페이스 어떤 것을 선택하든 차이가 없습니다.

```ts
type TFnWithProperties = {
  (x: number) => number;
  prop: string;
}

type IFnWithProperties = {
  (x: number) => number;
  prop: string;
}
```

문법이 생소할 수도 있지만 자바스크립트에서 함수는 호출 가능한 객체라는 것을 떠올려 보면 납득할 수 있는 코드입니다.

타입 별칭과 인터페이스는 모두 제너릭이 가능합니다.

```ts
type TPair<T> = {
  first: T;
  second: T;
}

interface IPair<T> = {
  first: T;
  second: T;
}
```

인터페이스는 타입을 확장할 수 있으며 (주의사항이 몇 가지 있는데, 뒤에서 설명합니다), 타입은 인터페이스를 확장할 수 있습니다.

```ts
interface IStateWithPop extends TState{
  population: number;
};

type TStateWithPop = IState & {population: number;};
```

IStateWithPop과 TStateWithPop은 동일합니다.
여기서 주의할 점은 인터페이스는 유니온 타입같은 복잡한 타입을 확장하지는 못한다는 것입니다.
복잡한 타입을 확장하고 싶다면 타입과 &룰 사용해야 합니다.

한편 클래스를 구현(implements)할 때는, 타입(TState)과 인터페이스(IState) 둘 다 사용할 수 있습니다.

```ts
class StateT implements TState{
  name: string = '';
  capital: string = '';
};

class StateI implements IState{
  name: string = '';
  capital: string = '';
};
```

지금까지 타입과 인터페이스의 비슷한 점들을 살펴보았습니다.
이제부터는 타입와 인터페이스의 다른 점들은 알아보겠습니다.

이미 언급한 대로, 유니온 타입은 있지만 유니온 인터페이스라는 개념은 없습니다.

```ts
type AorB = 'a' | 'b';
```

인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없습니다. 
그런데 유니온 타입을 확장하는 게 필요할 때가 있습니다.
다음 코드를 예로 들어 보겠습니다.
Input과 Output은 별도의 타입이며 이 둘을 하나의 변수명으로 매핑하는 VariableMap 인터페이스를 만들 수 있습니다. 

```ts
type Input = { /*...*/ };
type Output = { /*...*/ };
interface VariableMap{
  [name:string]: Input | Output;
}
```

또는 유니온 타입에 name 속성을 붙인 타입을 만들 수도 있습니다. 
다음과 같습니다.

```ts
type NamedVariable = (Input | Output) & {name: string};
```

이 타입은 인터페이스로 표현할 수 없습니다.
type 키워드는 일반적으로 interface보다 쓰임새가 많습니다.
type 키워드는 유니온이 될 수도 있고, 매핑된 타입 또는 조건부 타입 같은 고급 기능에 활용되기도 합니다.

튜플과 배열 타입도 type 키워드를 이용해 더 간결하게 표현할 수 있습니다.

```ts
type Pair = [number,number];
type StringList = string[];
type NamedNums = [string. ...number[]];
```

인터페이스로도 튜플과 비슷하게 구현할 수 있기는 합니다.

```ts
interface Tuple {
  0: number;
  1: number;
  length : 2;
}

const t: Tuple = [10, 20]; // 정상 
```

그러나 인터페이스로 튜풀과 구현하면 튜플에서 사용할 수 있는 concat같은 매서드들을 사용할 수 없습니다.
그러므로 튜플은 type 키워드로 구현하는 것이 낫습니다.
숫자 인덱스와 관련된 문제점은 [아이템 16](https://github.com/Pyotato/effective_typescript/blob/item16/README.md)에서 살펴 봅니다.

반면 인터페이스는 타입에 없는 몇 가지 기능이 있습니다.

그중 하나는 바로 '보강(augment)'이 가능하다는 것입니다.
이번 아이템 처음에 등장했던 State 예제에 population 필드를 추가할 때 보강 기법을 사용할 수 있습니다.

```ts
interface IState = {
  name: string;
  capital: string;
}
interface IState = {
  population: number;
}
const wyoming: TState = {
  name: 'Wyoming',
  capital: 'Cheyenne',
  population: 500_000 
} //  정상
```

이 예제처럼 속성을 확장하는 갓을 '선언 병합(declaration merging)'이라고 합니다.
선언 병합을 본 적이 없다면 매우 생소하게 느껴질 것입니다.
선언 병합은 주로 타입 선언 파일 (6장)에서 사용됩니다.
따라서 타입 선언 파일을 작성할 때는 선언 병합을 지원하기 위해 반드시 인터페이스르 사용해야 하며 표준을 따라야 합니다.
타입 선언에는 사용자가 채워야 하는 빈틈이 있을 수 있는데, 이 선언 병합이 그렇습니다.

타입스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 여러 타입을 모아 병합합니다.
예를 들어, Array 인터페이스는 <i>lib.es5.d.ts</i>에 정의되어 있고 기본적으로 <i>lib.es5.d.ts</i>에 선언된 인터페이스가 사용됩니다.
그러나 <i>tsconfig.json</i>의 lib 목록에 ES2015를 추가하면 타입스크립트는 <i>lib.es2015.d.ts</i>에 선언된 인터페이스를 병합합니다.
여기에는 ES2015에 추가된 또 다른 Arrayt선언의 find같은 메서드가 포함됩니다. 이들은 병합을 토애 다른 Array 인터페이스에 추가됩니다.
결과적으로 각 선언이 병합되어 전체 메서드를 가지는 하나의 Array 타입을 얻게 됩니다.

병합은 선언처럼 일반적인 코드라서 언제든지 가능하다는 것을 알고 있어야 합니다.
그러므로 프로퍼티가 추가되는 것을 원하지 않는다면 인터페이스 대신 타입을 사용해야 합니다.

이번 아이템의 처음 질문으로 돌아가 타입과 인터페이스 중 어느 것을 사용해야 할지 결론을 내려 보겠습니다.
복잡한 타입이라면 고민할 것도 없이 타입 별칭을 사용하면 됩니다.
그러나 타입과 인터페이스, 두 가지 방법으로도 모두 표현할 수 있는 간단한 객체 타입이라면 일관성과 보강의 관점에서 고려해 봐야 합니다.
일관되게 인터페이스를 사용하는 코드베이스에서 작업하고 있다면 인터페이스를 사용하고, 일관되게 타입을 사용하는 코드베이스에서 작업하고 있다면 타입을 사용하면 됩니다.

아직 스타일이 확립되지 않은 프로젝트라면, 향후에 보강의 가능성이 있을 지 생각해 봐야 합니다.
어떤 API에 대한 타입 선언을 작성해야 한다면 인터페이스를 사용하는 게 좋습니다.
API가 변경될 때 사용자가 인터페이스를 통해 새로운 필드를 병합할 수 있어 유용하기 때문입니다.
그러나 프로젝트 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계입니다.
따라서 이럴 때는 타입을 사용해야 합니다.
