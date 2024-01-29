# 🏗️ ITEM 34 : 부정확한 타입보다는 미완성 타입 사용하기

타입 선언을 작성하다 보면 코드의 동작을 더 구체적으로 또는 덜 구체적으로 모델링하게 되는 상호아을 맞닥뜨리게 됩니다.
일반적으로 타입이 구체적일 수록 버그를 더 많이 잡고 타입스크립트가 제공하는 도구를 활용할 수 있게 됩니다.
그러나 타입 선언의 정밀도를 높이는 일에는 주의를 기울여야 합니다.
실수가 발생하기 쉽고 잘못된 타입은 차라리 타입이 없는 것보다 못할 수 있기 때문입니다.

[아이템 29](https://github.com/Pyotato/effective_typescript/blob/item29/README.md)에서 보았던 GeoJSON 형식의 타입 선언을 작성한다고 가정해 보겠습니다.
GeoJSON 정보는 각각 다른 형태의 좌표 배열을 가지는 몇 가지 타입 중 하나가 될 수 있습니다.

```ts
interface Point {
  type: 'Point';
  coordinates: number[];
}
interface LineString {
  type: 'LineString';
  coordinates: number[][];
}
interface Polygon {
  type: 'Polygon';
  coordinates: number[][][];
}
type Geometry = Point | LineString | Polygon; // 다른 것들도 추가될 수 있습니다.
```

큰 문제는 없지만 좌표에 쓰이는 number[]가 약간 추상적입니다. 
여기서 number[]는 경도와 위도를 나타내므로 튜플 타입으로 선언하는 게 낫습니다.

```ts
type GeoPosition = [number, number];
interface Point {
  type: 'Point';
  coordinates: GeoPosition;
}
// ...
```

타입을 더 구체적으로 개선했기 때문에 더 나은 코드가 된 것 같습니다. 커뮤니티에 자랑하고 싶을 수 있습니다.
코드를 공개한 후에 '좋아요' 개수가 올라갈 것을 기대했겠지만, 안타깝게도 새로운 코드가 빌드를 깨뜨린다며 불평하는 사용자들의 모습만 보게 될 겁니다.
코드에는 위도와 경도만을 명시했지만, GeoJSON은 위치 정보에는 세 번쨰 요소인 고도가 있을 수 있고 또 다른 정보가 있을 수도 있습니다.
결과적으로 타입 선언을 세밀하게 만들고자 했지만 시도가 너무 과했고 오히려 타입이 부정확해졌습니다.
현재의 타입 선언을 그대로 사용하려면 사용자들은 타입 단언문을 도입하거나 `as any`를 추가해서 타입 체커를 완전히 무시해야 합니다.

이번에는 JSON으로 정의된 Lisp와 비슷한 언어의 타입 선언을 작성한다고 생각해 보겠습니다.

```lisp
12
"red"
["+",1,2]                           // 3
["/",20,2]                          // 10
["case",[">",20,10],"red","blue"]   // "red"
["rgb",255,0,127]                   // "#FF007F"
```

맵박스(Mapbox) 라이브러리는 이런 시스템을 사용하여 수많은 기기에서도 지도 기능의 형태를 결정합니다.
다음은 이런 동작을 모델링해 볼 수 있는 입력값의 전체 종류입니다.

1. 모두 허용
2. 문자열, 숫자, 배열 허용
3. 문자열, 숫자, 알려진 함수 이름으로 시작하는 배열 허용
4. 각 함수가 받는 매개뱐수의 개수가 정확한지 확인
5. 각 함수가 받는 매개변수의 타입이 정확한지 확인

처음의 두개 옵션은 가능합니다.

```ts
type Expression1 = any;
type Expression2 = number | string | any[];
```

또한 표현식의 유효성을 체크하는 테스트 세트를 도입해 보겠습니다.
타입을 구체적으로 만들수록 정밀도가 손상되는 것을 방지하는 데 도움이 됩니다 ([아이텝 52](https://github.com/Pyotato/effective_typescript/blob/item52/README.md)). 

```ts
const tests: Expression2[] = [
  10,
  'red',
  true,
  ['+',10,5],
  ["/",20,2]],
  ["case",[">",20,10],"red","blue","green"],   // 값이 너무 많습니다.
  ["**",2,31],                                 // "**"는 함수가 아니므로 오류가 발생해야 합니다.
  ["rgb",255,128,64],
  ["rgb",255,0,127,0]                           // 값이 너무 많습니다.
];
```

<img width="885" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f761ad11-e72b-4671-8b75-26edd85c53ea"/>

정밀도를 한 단계 더 끌어 올리기 위해서 튜플의 첫 번째 요소에 문자열 리터럴 타입의 유니온을 사용해 보겠습니다.

```ts
type FnName = '+' | '-' | '*' | '/' | '>' | '<' | 'case' | 'rgb';
type CallExpression = [FnName, ...any[]];
type Expression3 = number | string | CallExpression;
```

<img width="885" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/e168edfc-f050-4fa8-b0ea-d0f6c264f140"/>

정밀도를 유지하면서 오류를 하나 더 잡았습니다.

각 함수의 매개변수 개수가 정확한지 확인하기 위해 모든 함수 호출을 확인할 수도 있지만 재귀적으로 동작하기 때문에 좋은 방법은 아닙니다.
타입스크립트 3.6에서는 함수의 매개변수 개수를 알아내기 위해 최소한 하나의 인터페이스를 추가해야 합니다.
여러 인터페이스를 호출 표현식으로 한번에 묶을 수는 없기 때문에, 각 인터페이스를 나열해서 호출 표현식으로 작성합니다.
고정길이 배열은 튜플 타입으로 가장 간단히 표현할 수 있기 때문에, 어색해 보일 수는 있지만 다음 코드처럼 구현할 수 있습니다.

```ts

type Expression4 = number | string | CallExpression;
type CallExpression = MathCall | CaseCall | RGBCall;

interface MathCall {
  0:  '+' | '-' | '*' | '/' | '>' | '<' ;
  1: Expression4;
  2: Expression2;
  length: 3;
}

interface CaseCall {
  0:  'case' ;
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4 | 6 | 8 | 10 | 12 | 14 | 16 ; // 등등
 }

 interface RGBCall {
  0:  'rgb' ;
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4  ; // 등등
 }

const tests: Expression4[] = [
  10,
  'red',
  true,
  ['+',10,5],
  ["/",20,2],
  ["case",[">",20,10],"red","blue","green"],
  ["**",2,31], 
  ["rgb",255,128,64],
  ["rgb",255,0,127,0] 
];
```

<img width="1091" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1eed4b32-caa0-419b-a43e-b46896bd9ae1"/>

이제 무효한 표현식에서 전부 오류가 발생합니다.
이 코드에서는 타입스크립트 인터페이스를 사용해서 '짝수 길이의 배열'같은 것을 표현할 수 있습니다.

---

> 🌟 원본에서는 다음과 같은 오류들이 표시되기를 기대했지만 tsplayground에서의 실행 결과는 위와 같았습니다.

```ts
const tests: Expression4[] = [
  10,
  'red',
  true,
//---- : 'true' 형식은 'Expression4' 형식에 할당할 수 없습니다.
  ['+',10,5],
  ["/",20,2],
  ["case",[">",20,10],"red","blue","green"],
         // --------- :  형식은 'string' 형식에 할당할 수 없습니다.
  ["**",2,31],
//  -- :  'number' 형식은 'string' 형식에 할당할 수 없습니다.
  ["rgb",255,128,64],
  ["rgb",255,0,127,0]
      // --  - --- - :  'number' 형식은 'string' 형식에 할당할 수 없습니다.
];
```

그러나 오류가 나면 엉뚱한 메시지를 출력하며, `**`에 대한 오류는 오히려 이전 버전보다 메시지가 부정확해집니다.

----

타입 정보가 더 정밀해졌지만 결과적으로 이전 버전보다 선되었다고 보기는 어렵습니다.
잘못 사용된 코드에서 오류가 발생하기는 하지만 오류 메시지는 더 난해해졌습니다.
언어 서비스는 타입 체크 못지 않게 타입스크립트 경험에서 중요한 부분 ([아이템 6](https://github.com/Pyotato/effective_typescript/blob/item6/README.md))이므로, 타입 선언으로 인한 오류 메시지를 살펴보고 타입 선언이 동작하는 곳에는 자동 완성을 적용하는 것이 좋습니다. 
새 타입 선언은 더 구체적이지만 자동 완성을 방해하므로 타입스크립트 개발경험을 해치게 됩니다.

타입 선언의 복잡성으로 인해 버그가 발생할 가능성도 높아졌습니다.
예를 들어 Expression4는 모든 수학 연산자에 두 개의 매개변수가 필요하지만, 맵박스 표현식에서는 `+`와 `*`가 많은 매개변수를 받을 수 있습니다.
또한 입력을 음수로 바꿔 주는 `-`는 한 개의 매개변수만 필요합니다.
Expression4는 이러한 경우들에 잘못된 오류를 표시하게 됩니다.

```ts
const okExpressions: Expression4[] = [
  ['-', 12],
     // -- :  'number' 형식은 'string' 형식에 할당할 수 없습니다.
  ['+', 1, 2, 3],
     // -  -  - :  'number' 형식은 'string' 형식에 할당할 수 없습니다.
  ['*', 2, 3, 4],
     // -  -  -  :  'number' 형식은 'string' 형식에 할당할 수 없습니다.
];
```

코드를 더 정밀하게 만들려던 시도가 너무 과했고 그로 인해 코드가 오히려 더 부정확해졌습니다.
이렇게 부정확함을 바로잡는 방법을 쓰는 대신, 테스트 세트를 추가하여 놓친 부분이 없는지 확인해도 됩ㄴ다.
일번족으로 복잡한 코드는 더 많은 테스트가 필요하고 타입의 관점에서도 마찬가지입니다.

타입을 정제(refine)할때, '불쾌한 골짜기(uncanny valley)' 은유를 생각해보면 도움이 될 수 있습니다.
일반적으로 any같은 매우 추상적인 타입은 정제하는 것이 좋습니다.
그러나 타입이 구체적으로 정제된다고 해서 정확돡 무조건 올라가지는 않습니다.
타입에 의존하기 시작하면 부정확함으로 인해 발생하는 문제는 더 커질 겁니다.

> 불쾌한 골짜기란 로봇 공학과 인공 지능에서 많이 쓰이는 용어인데, 어설프게 인간과 비슷한 로봇에서 느끼는 불쾌함을 뜻합니다. <br/>
> 저자의 의도는 타입 선언에서 어설프게 완벽을 추구하려다가 오히려 역효과가 발생하는 것을 주의하자는 것입니다.
