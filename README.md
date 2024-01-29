# ⚠️ ITEM 52 : 테스팅 타입의 함정에 주의하기

프로젝트를 공개하려면 테스트 코드를 작성하는 것은 필수이며, 타입 선언도 테스트를 거쳐야 합니다.
그러나 타입 선언을 테스트하기는 매우 어렵습니다.
그래서 타입 선언에 대한 테스트 코드를 작성할 때 타잆스크립트가 제공하는 도구를 사용하여 단언문으로 때우기 십상이지만, 이런 방법에는 몇 가지 문제가 있습니다.
궁극적으로 `dtslint` 또는 타입 시스템 외부에서 타입을 검사하는 유사한 도구를 사용하는 것이 더 안전하고 간단합니다.

유틸리티 라이브러리에서 제공하느 map함수의 타입 선언을 작성한다고 가정해 보곘습니다 (매우 유명한 유틸리티 라이브러리인 로대시와 언더스코어는 map 함수를 제공합니다).

```ts
declare function map<U, V>(array: U[], fn: (u:U) => V):V[];
```

타입 선언이 예상한 타입으로 결과를 내는지 체크할 수 있는 한 가지 방법은 함수를 호출하는 테스트 파일을 작성하는 것입니다 (구현체를 위한 별도의 테스트는 있다고 가정).

```ts
map(['2017','2018','2019'], v=> Number(v));
```

이 코드는 오류 체크를 수행하지만 허점이 존재합니다.
예를 들어 map의 첫번째 매개변수에 배열이 아닌 단일 값이 있었다면 매개변수의 타입에 대한 오류는 잡을 수 있습니다.
그러나 반환값에 대한 체크가 누락되어 있기 때문에 완전한 테스트랄고 할 수 없습니다.
이해를 돕기 위해 다른 예를 들어 보겠습니다.

앞의 코드와 동일한 스타일로 square라는 함수의 런타임 동작을 테스트한다면 다음과 같은 테스트 코드가 됩니다.

```ts
test('square a number',()=>{
  square(1);
  square(2);
});
```

이 테스트 코드는 square 함수의 '실행'에서 오류가 발생하지 않는지만 체크합니다.
그런데 반환값에 대해서는 체크하지 않기 때문에, 실제로는 실행의 겨로가에 대한 테스트는 하지 않은 게 됩니다.
따라서 square의 구현이 잘못되어 있더라도 이 테스트를 통과하게 됩니다.

타입 선언 파일을 테스팅할 때는 이 테스트 코드처럼 단순히 함수를 실행만 하는 방식을 일반적으로 적용하게 되는데, 그 이유는 라이브러리 구현체의 기존 테스트 코드를 복사하면 간단히 만들 수 있기 때문입니다.
함수를 실행만 하는 테스트 코드가 의미 없는 것은 아니지만, 실제로 반환 타입을 체크하는 것이 훨씬 좋은 테스트 코드입니다.

반환값을 특정 타이브이 변수에 할당하여 간단히 반환 타입을 체크할 수 있는 방법을 알아봅시다.

```ts
const lengths : number[] = map(['john','paul'],name=> name.length);
```

이 코드는 일반적으로 불필요한 타입 선언([아이템 19](https://github.com/Pyotato/effective_typescript/blob/item19/README.md))에 해당합니다. 
그러나 테스트 코드 관점에서는 중요한 역할을 하고 있습니다.
`number[]` 타입 선언은 map 함수의 반환 타입이 `number[]`임을 보장합니다.
실제로 `DefinitelyTyped`를 살펴보면, 테스팅을 위해 정확히 동일한 방식을 사용한 수 많은 타칩 선언을 볼 수 있습니다.
그러나 테스팅을 위해 할당을 사용하는 방법에는 두 가지 근본적인 문제가 있습니다.

> 👎 첫 번쨰, 불필요한 변수를 만들어야 합니다.

반환값을 할당하는 변수는 샘플코드처럼 쓰일 수도 있지만, 일부 린팅 규칙(미사용 변수 경고)을 비활성해야 합니다.

> 👍 일반적인 해결책은 변수를 도입하는 대신 헬퍼 함수를 정의하는 것입니다.

```ts
function assertType<T>(x: T){}
assertType<number[]>(map(['john','paul'],name=> name.length))
```

이 코드는 불필요한 변수 문제를 해결하지만, 또 다른 문제점이 남아 있습니다.

> 👎 두 번쨰, 두 타입이 동일한지 체크하는 것이 아니라 할당 가능성을 체크하고 있습니다.

다음 예제처럼 잘 동작하는 경우도 있습니다.

```ts
const n = 12;
assertType<number>(n);
```

n 심벌을 조사해 보면, 타입이 실제로 숫자 리터럴 타입인 12임을 볼 수 있습니다.
12는 number의 서브타입이고 할당 가능성 체크를 통과합니다.

그러나 객체의 타입을 체크하는 경우를 살펴보면 문제를 발견하게 될 겁니다.

```ts
const beatles = ['john', 'paul', 'george', 'ringo'];
assertType<{name:string}[]>(
  map(beatles, name => ({
    name, 
    inYellowSubmarine: name=== 'ringo'
  }))
);    // 정상
```

map은 `{name: string, inYellowSubmarine: boolean}` 객체의 배열을 반환합니다.
반환된 배열은 `{name:string}[]`에 할당 가능하지만, `inYellowSubmarine` 속성에 대한 부분이 체크되지 않았습니다.
상황에 따라 타입이 정확한지 체크할 수도 있고, 할당이 가능한지 체크할 수도 있습니다.

게다가 assertType에 함수를 넣어 보면 이상한 결과가 나옵니다.

```ts
const add = (a: number, b: number) => a+b;
assertType<(a:number,b:number) => number>(add); // 정상

const double = (x: number) => 2* x;
assertType<(a:number,b:number) => number>(double); // 😱 정상
```

double 함수의 체크가 성공하는 이유는, 타입스크립트의 함수는 매개변수가 더 적은 함수 타입에 할당 가능하기 때문입니다.
이해를 돕기 위해 또 다른 예를 들어 보곘습니다.

```ts
const g: (x: string) => any = () => 12; //  정상
```

앞의 코드는 선언된 것보다 적은 매개변수를 가진 함수를 할당하는 거이 아무런 문제가 없다는 것을 보여줍니다.
이러한 사례는 콜백 함수에서 흔히 볼 수 있기 때문에, 타입스크립트에서는 이러한 (선언보다 많은 수의 매개변수) 동작을 모델링하도록 설계되어 있습니다.
예를 들어, 로대시의 map 함수의 콜백은 세 가지 매개변수를 받습니다.

```ts
map(array, (name, index, array) => { /** */};
```

콜백 함수는 세 가지 변수 name,index,array 중에서 한두 개만 보통 사용합니다.
매개변수 세개를 모두 사용하는 경우는 매우 드뭅니다.
만약 매개변수의 개수가 맞지 않는 경우를 타입 체크에서 허용하지 않으면, 매우 많은 곳의 자바스크립트 코드에서 콜백 함수의 타입과 관련된 오류들이 발생하게 될 겁니다.

다시 assertType 문제로 돌아와 보겠습니다.
제대로 된 assertType 사용 방법은 무엇일까요?
다음 코드처럼 Parameters와 ReturnTpe 제너릭 타입을 이용해 함수의 매개변수 타입과 반환 타입만 분리하여 테스트할 수 있습니다.

```ts
const double = (x: number) => 2*x;
let p: Parameters<typeof double> = null!;
assertType<[number, number]>(p);

let r : ReturnType<typeof double> = null!;
assertType<number>(r);
```

<img width="990" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/45d73bd8-636b-4379-ba9f-d83144d77ea4"/>

한편, this가 등장하는 콜백 함수의 경우는 또 다른 문제가 있습니다.
map은 콜백 함수에서 this의 값을 사용할 때가 있으며 타입스크립트는 이러한 동작을 모델링할 수 있으므로 ([아이템 49](https://github.com/Pyotato/effective_typescript/blob/item49/README.md)), 타입 선언에 반영해야 하며 테스트도 해야 합니다.

앞서 이번 장에 등장했더 map에 대한 테스트는 모두 블랙박스 스타일이었습니다.
map의 매개변수로 배열을 넣어 함수를 실행하고 반환 타입을 테스트했지만, 중간 단계의 세부 사항은 테스트하지 않았스빈다.
세부 사항을 테스트하기 위해서 콜백 함수 내부에서 매개변수들의 타입과 this를 직접 체크해 보겠습니다.

```ts
const beatles = ['john', 'paul', 'george', 'ringo'];

assertType<number[]>(map(
  beatles,
  function(name,i,array){
    assertType<string>(name);
    assertType<number>(i);
    assertType<string[]>(array);
    assertType<string[]>(this);
  }
));  
```

<img width="1169" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6a5f3b26-b201-42fb-92a7-9936070bd2c1"/>

이 코드는 map의 콜백 함수에서 몇가지 문제가 발생했습니다.
한편 이번 예제의 콜백 함수는 화살표 함수가 아니기 때문에 this의 타입을 테스트할 수 있음을 주의하기 바랍니다.

다음 코드의 선언을 사용하면 타입 체크를 통과합니다.

```ts
declare function map<U, V>(
  array: U[], 
  fn: (this: U[], u:U ,i: number, array: U[]) => V
  ):V[];
```

> tsplayground에서 실행한 결과는 통과 X
> <img width="550" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/cd2a940b-120c-4c26-88d2-3a1e0dddb5d2"/>

그러나 여전히 중요한 마지막 문제가 남아 있습니다.
다음 모듈 선언은 까다로운 테스트를 통과할 수 있는 완전한 타입 선언 파일이지만, 결과적으로 좋지 않은 설계가 됩니다.

```ts
declare module 'overbar';
```

이 선언은 전체 모듈에 any 타입을 할당합니다.
따라서 테스트는 전부 통과하겠지만, 모든 타입 안전성을 포기하게 됩니다.
더 나쁜 점은, 해당 모듈에 속하는 모든 함수의 호출마다 암시적으로 any 타입을 반환하기 때문에 코드 전반에 걸쳐 타입 안전성을 지속적으로 무너뜨리게 된다는 것입니다.
`noImplicitAny`를 설정하더라도 타입 선언을 통해 여전히 any 타입이 생겨나게 됩니다.

타입시스템 내에서 암묵적 any 타입을 발견해 내는 것은 매우 어렵습니다.
이러한 어려움 떄문에 타입 체커와 독립적으로 동작하는 도구를 사용해서 타입 선언을 테스트하는 방법이 권장됩니다.

DefinitelyTyped의 타입 선언을 위한 도구는 `dtslint`입니다.
`dtslint`는 특별한 형태의 주석을 통해 동작합니다.
`dtslint`을 사용하면 beatles 관련 예제의 테스트를 다음처럼 작성할 수 있습니다.

```ts
const beatles = ['john', 'paul', 'george', 'ringo'];
assertType<{name:string}[]>(
  map(beatles, function (
    name,                     // $ExpectType string
    i,                        // $ExpectType number
    array                     // $ExpectType string[]
  ){
  this                       // $ExpectType string[]
  return name.length;       // $ExpectType number[]
});
```

`dtslint`는 할당 가능성을 체크하는 대신 각 심벌의 타입을 추출하여 글자 자체가 같은지 비교합니다.
이 비교 과정은 편집기에서 타입 선언을 눈으로 보고 확인하는 것과 같은데 `dtslint`는 이러한 과정을 자동화합니다.
그러나 글자 자체가 같은지 비교하는 방식에는 단점이 있습니다.
`number | string`과 `string | number`는 같은 타입이지만 글자 자체로 보면 다르기 때문에 다른 카입으로 인식됩니다.
string과 any를 비교할 대도 마찬가지인데, 두 타입은 서로 간에 할당이 가능하지만 글자 자체는 다르기 때문에 다른 타입으로 인식됩니다.

타입 선언을 테스트한다는 것은 어렵지만 반드시 해야 하는 작업입니다.
앞에서 소개한 몇 가지 일반적인 기법의 문제점을 인지하고, 문제점을 방지하기위해 `dtslint`같은 도구를 사용하도록 합시다.















