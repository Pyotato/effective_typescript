# 🛤️ ITEM 44 : 타입 커버리지를 추적하여 타입 안전성 유지하기

`noImplicitAny`를 설정하고 모든 암시적 `any` 대신 명시적 타입 구문을 추가해도 `any`타입과 관련된 문제들로부터 안전하다고 할 수 없습니다.
`any`타입이 여전히 프로그램 내에 존재할 수 있는 두 가지 경우가 있습니다.

- 명시적 `any`타입
  [아이템 38](https://github.com/Pyotato/effective_typescript/blob/item38/README.md)과 [아이템 39](https://github.com/Pyotato/effective_typescript/blob/item39/README.md)의 내용에 따라 `any`타입의 범위를 좁히고 구체적으로 만들어도 여전히 `any`타입입니다. 특히 `any[]`와 `{[key:string]: any}` 같은 타입은 타입 인덱스를 생성하면 단순 `any`가 되고 코드 전반에 영향을 미칩니다.

- 서드파티 타입 선언
  이 경우는 `@types` 선언 파일로부터 `any`타입이 전파되기 때문에 특별히 조심해야 합니다.
  `noImplicitAny`를 설정하고 절대 `any`를 사용하지 않았다 하더라도 여전히 `any`타입은 코드 전반에 영향을 미칩니다.

`any`타입은 타입 안전성과 생산성에 부정적 영향을 미칠 수 있으므로 ([아이템 5](https://github.com/Pyotato/effective_typescript/blob/item5/README.md)), 프로젝트에서 `any`의 개수를 추적하는 것이 좋습니다.
npm의 type-coverage 패키지를 활용하여 `any`를 추적할 수 있는 몇 가지 방법이 있습니다.

> $ npx type-coverage <br/>
> 9985/10117 98.69%
  
![image](https://github.com/Pyotato/effective_typescript/assets/102423086/614f4d0e-28b5-4040-b763-5f88f4935959)

결과를 통해 이 프로젝트의 10,117심벌 중 9,985ro (98.69%)가 `any`가 아니거나 `any`의 별칭이 아닌 타입을 가지고 있음을 알 수 있습니다.
실수로 `any`타입이 추가된다면 백분율(%)이 감소하게 됩니다.

앞 결과의 백분율은 5장의 다른 아이템에서 소개한 조언들을 얼마나 잘 따랐는지에 대한 점수라고 할 수 있습니다.
그러므로 점수를 추적함으로써 시간이 지남에 따라 코드의 품질을 높일 수 있습니다.

타입 커버리지 정보를 수집해 보는 것도 유용할 수 있습니다.
`type-coverage`를 실행할 때, `--detail` 플래그를 붙이면, `any` 타입이 있는 곳을 모두 출력해 줍니다.

> $ npx type-coverage --detail <br/>
> path/to/code.ts 1:10 getColumnInfo <br/>
> path/to/module.ts 7:1 pt2
  
이것을 조사해 보면 미처 발견하지 못한 `any`의 근원지를 찾을 수도 있습니다.

코드에 `any`가 남아 있는 이유는 다양합니다.
오류를 간단히 해결하기 위해 종종 명시적으로 `any`를 선언했을 수 있습니다.
타입 오류가 발생했지만 해결하는 데 시간을 쏟고 싶지 않았을 수도 있습니다.
또는 아직까지 타입을 제대로 작성하지 못했을 수도 있습니다.
아니면 급하게 작업하느라 `any`인 채로 놔두었을 수도 있습니다.

`any`가 등장하는 몇 가지 문제와 그 해결책을 살펴보겠습니다.
표 현태의 데이터에서 어떤 종류의 열(column) 정보를 만들어 내는 함수를 만든다고 가정해 봅시다.

```ts
function getColumnInfo(name: string): any{
  return utils.buildColumnInfo(appstate.dataSchema, name); // any 반환
}
```

`util.buildColumnInfo` 호출은 `any`를 반환합니다.
그래서 `getColumnInfo` 함수의 반환에는 주석과 함께 명시적으로 `: any` 구문을 추가했습니다.

이후에 타입 정보를 추가하기 위해 `ColumnInfo` 타입을 정의하고 `util.buildColumnInfo`가 `any`대신 `ColumnInfo`를 반환하도록 개선해도 `getColumnInfo`함수의 반환문에 있는 `any`타입이 모든 타입 정보를 날려 버리게 됩니다.
`getColumnInfo`에 남아 있는 `any`까지 제거해야 문제가 해결됩니다.

서드파티 라이브러리로부터 비롯되는 `any`타입은 몇 가지 형태로 등장할 수 있지만 가장 극단적인 예는 전체 모듈에 `any`타입을 부여하는 것입니다.

```ts
declare module 'my-module';
```

앞의 선언으로 인해 `my-module`에서 어떤 것이든 오류 없이 임포트 할 수 있습니다.
임포트한 모든 심벌은 `any`타입이고, 임포트한 값이 사용되는 곳마다 `any` 타입을 양산하게 됩니다.

```ts
import {someMethod, someSymbol} from 'my-module';   //  정상
const pt1 = {                                       // {x: number, y: number} 타입 
  x:1,
  y:2,
}  
const pt2 = someMethod(pt1, someSymbol);             // 타입이 any
```

일반적인 모듈의 사용법과 동일하기 때문에, 타입 정보가 모두 제거됐다는 것을 간과할 수 있습니다.
또는 동료가 모든 타입 정보를 날려 버렸지만, 알아채지 못하는 경우일 수도 있습니다.
그렇기 때문에 가끔 해당 모듈을 점검해야 합니다.
어느 순간 모듈에 대한 공식 타입 선언이 릴리스되었을지도 모릅니다.
또는 모듈을 충분히 이해한 후에 직접 타입 선언을 작성해서 커뮤니티에 공개할 수도 있습니다.

서드파티 라이브러리로부터 비롯되는 `any`의 또 다른 형태는 타입에 버그가 있는 경우입니다.
예를 들어 [아이템 29](https://github.com/Pyotato/effective_typescript/blob/item29/README.md)의 조언 (값을 생성할 때는 엄격하게 타입 적용)을 무시한 채로, 함수가 유니온 타입을 반환하도록 선언하고 실제로는 유니온 타입보다 훨씬 더 특정된 값을 반환하는 경우입니다.

선언된 타입과 실제 반환된 타입이 맞지 않는다면 어쩔 수 없이 `any` 단언문을 사용해야 합니다.
그러나 나중에 라이브러리가 업데이트되어 함수의 선언문이 제대로 수정된다면 `any`를 제거해야 합니다.
또는 직접 라이브러리의 선언문을 수정하고 커뮤니티에 공개할 수도 있습니다.

`any`타입이 사용되는 코드가 실제로는 더 이상 실행되지 않는 코드일 수 있습니다.
또는 어쩔 수 없이 `any`를 사용했던 부분이 개선되어 제대로 된 타입으로 바뀌었다면 `any`가 더 이상 필요 없을 수도 있습니다.
버그가 있는 타입 선언문이 업데이트되어 제대로 타입 정보를 가질 수도 있습니다.
타입 커버리지를 추적하면 이러한 부분들을 쉽게 발견할 수 있기 때문에 코드를 꾸준히 점검할 수 있게 해 줍니다.
