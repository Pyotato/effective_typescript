# ✨ ITEM 61 : 의존성 관계에 따라 모듈로 전환하기

[아이템 58](https://github.com/Pyotato/effective_typescript/blob/item58/README.md)에서 모던 자바스크립트로 전환하는 방법을 알아보았습니다.
그리고 [아이템 60](https://github.com/Pyotato/effective_typescript/blob/item60/README.md)에서 빌드 과정에 타입스크립트를 통합하고 테스트까지 수행하는 방법을 다루었습니다.
이제는 본격적으로 자바스크립트 코드를 타입스크립트로 전환하는 단계를 알아보겠습니다.

점진적 마이그레이션을 할 때는 모듈 단위로 각개격파하는 것이 이상적입니다.
그런데 한 모듈을 골라서 타입 정보를 추가하면, 해당 모듈이 의존 (임포트)하는 모듈에서 비롯되는 타입 오류가 발생하게 됩니다.
의존성과 관련된 오류 없이 작업하려면, 다른 모듈에 의존하지 않은 최하단 모듈부터 작업을 시작해서 의존성이 최상단에 있는 모듈을 마지막으로 완성해야 합니다.

프로젝트 내에 존재하는 모듈은 서드파티 라이브러리에 의존하지만 서드파티 라이브러리는 해당 모듈에 의존하지 않기 때문에, 서드파티 라이브러리 타입 정보를 가장 먼저 해결해야 합니다. 일반적으로 `@types` 모듈을 설치하면 됩니다.
예를 들어, lodash 라이브러리르 사용한다면 `@types/lodash`는 lodash의 타입 정보를 담고 있으며, 타입 정보는 lodash 라이브러리를 사용하는 모든 부분에 적용됩니다.

외부 API를 호출하는 경우도 있기 때문에 외부 API의 타입 정보도 추가해야 합니다.
서드 파티 라이브러리와 마찬가지로, 프로젝트 내의 모듈은 API에 의존하지만 API는 해당 모듈에 의존하지 않기 때문에 먼저 해결하는 것이 좋습니다.
외부 API의 타입 정보는 특별한 문맥이 없기 때문에 타입스크립트가 추론하기 어렵습니다.
그러므로 API에 대한 사양을 기반으로 타입 정보를 생성해야 합니다 ([아이템 35](https://github.com/Pyotato/effective_typescript/blob/item35/README.md)).

모듈 단위로 마이그레이션을 시작하기 전에, 모듈 간의 의존성 관계를 시각화해 보면 많은 도움이 됩니다.
예를 들어, 중간 규모인 dygraph라는 프로젝트에 [madge](https://github.com/pahen/madge)라는 도구를 적용해보면, 아래와 같은 의존성 관계도를 얻게 됩니다.

<img width="1043" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/dbce15ff-729f-4672-a1b2-d076353ac0b5"/>

의존성 과계도의 가장 아랴쪽을 보면 유틸리티 모듈인 <i>util.js</i>와 <i>tickers.js</i>가 순환 (circular) 의존성을 가지고 있습니다.
그리고 많은 모듈이 <i>util.js</i>에 의존하고 있음을 알 수 있습니다.
대부분의 프로젝트에서 의존성의 최하단에 유틸리티 종류의 모듈이 위치하는 패턴을 발견할 수 있으며 일반적인 현상입니다.

마이그레이션할 때는 타입 정보 추가만 하고, 리팩터링을 해서는 안 됩니다.
오래된 프로젝트일수록 개선이 필요한 부분을 자주 마주치겠지만, 당장의 목표는 코드 개선이 아니라 타입스크립트로 전환하는 것임을 명심해야 합니다.
개선이 필요한 부분을 찾게 된다면 나중에 리택터링 할 수 있도록 목록을 만들어 두면 됩니다.
[아이템 59](https://github.com/Pyotato/effective_typescript/blob/item59/README.md)에서 타입스크립트로 전환하며 발견하게 되는 일반적인 오류들을 다루었습니다.
여기서는 [아이템 59](https://github.com/Pyotato/effective_typescript/blob/item59/README.md)에서 다루지 못한 다른 오류들을 알아보겠습니다.

## 선언되지 않은 클래스 멤버

자바스크립트는 클래스 멤버 변수를 선언할 필요가 없지만, 타입스크립트에서는 명시적으로 선언해야 합니다.
멤버 변수를 선언하지 않은 클래스가 있는 <i>.js</i>파일을 <i>.ts</i>로 바꾸면, 참조하는 속성마다 오류가 발생합니다.

```ts
class Greeting{
 constructor(name){
  this.greeting = 'Hello';
  this.name = name;
 }
 greet(){
  return this.greeting + ' '+ this.name;
 }

}
```

<img width="476" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/22c2510e-83f9-4414-a0a6-bdf7e8437ea4"/>

아래와 같이 빠른 수정 (quick fix) 기능으로 간단히 해결할 수 있습니다.

<img width="421" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8e251c0c-59f0-4f61-b95c-a1e27ca315ad"/>

'누락된 모든 멤버 추가 (Add all missing members)'를 선택하면 타입을 추론하여 선언문이 추가됩니다.

```ts
class Greeting{
    greeting: string;
    name: any;
    constructor(name){
     this.greeting = 'Hello';
     this.name = name;
    }
    greet(){
     return this.greeting + ' '+ this.name;
    }
   }
```

greeting에 대한 타입은 string 타입으로 정확히 추론되었지만, name의 경우는 any로 채워졌습니다.
빠른 수정을 적용한 후에 속성을 훑어보고 any로 추론된 부분을 직접 수정해야 합니다.

타입스크립트로 전환하려는 클래스가 너무 많은 속성을 가지고 있어서 충격 받을 지도 모릅니다.
위에서 살펴본  <i>dygraph.js</i>의 디펜덴시 그래프의 메인 클래스에도 적어도 45개 이상의 멤버 변수가 있습니다.
자바스크립트 코드를 타입스크립트로 전화하다 보면, 잘몬쇤 설계를 발견하는 효과가 있습니다.
잘못된 설계를 발견했을 떄, 잘못된 설계 그대로 타입스크립트로 전환하는 것은 납득하기 어려운 일입니다.
그러나 다시 한번 언급하지만 리팩터링을 하면 안 됩니다.
개선할 부분을 기록해 두고, 리팩터링은 타입스크립트 전환 작업이 완료된 후에 생각해야 합니다.

## 타입이 바뀌는 값

다음 코드는 자바스크립트일 때는 문제가 없지만, 타입스크립트가 되는 순간 오류가 발생합니다.

<img width="745" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9cd80b89-c9ed-48ad-a1ad-174d61407825"/>

한꺼번에 객체를 생성하면 간단히 오류를 해결할 수 있습니다.
자세한 내용은 [아이템 23](https://github.com/Pyotato/effective_typescript/blob/item23/README.md)에서 다루었습니다.

```ts
const state = {
  name:'New York',
  capital :'Albany'
};
```

한꺼번에 생성하기 곤란한 경우에는 임시 방편으로 타입 단언문을 사용할 수도 있습니다.

```ts
interface State {
  name:string;
  capital: string
};

const state = {} as State;

state.name = 'New York';
state.capital = 'Albany';
```

당장은 마이그레이션이 중요하기 떄문에 타입 단언문을 사용한 것이며, 마이그레이션이 완료된 후에는 [아이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md)을 참고하여 문제를 제대로 해결해야 합니다.

한편, 자바스크립트 상태에서 JSDoc이나 `@ts-check`를 사용해 타입 정보를 추가한 상태라면 ([아이템 59](https://github.com/Pyotato/effective_typescript/blob/item59/README.md)), 타입스크립트로 전환하는 순간 타입 정보가 '무효화'된다는 것을 주의해야 합니다.
예를 들어, 다음 코드는 자바스크립트 상태에서 제대로 오류를 표시하고 있습니다.

<img width="778" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/c9465766-df31-43e2-a33c-a6b568609b7e"/>

그러나 타입스크립트로 전환하게 되면, `@ts-check`와 JSDoc은 작동하지 않습니다.
즉, double 함수의 매개변수 num의 타입은 any로 추론되고 오류가 사라집니다.

다행히 JSDoc 타입 정보를 타입스크립트 타입으로 전환해 주는 빠른 수정 기능이 있습니다.
아래와 같이 사용하면 됩니다.

<img width="509" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/d3d66b93-a7c7-4165-bf7c-5fac3a38b08b">

타입 정보가 생성된 후에는 불필요해진 JSDoc을 제거하면 됩니다 ([아이템 30](https://github.com/Pyotato/effective_typescript/blob/item30/README.md)).

<img width="794" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/b30ee417-16be-4893-bf85-6265af188359">

이제 정상적으로 오류가 표시됩니다.
해당 오류는 `noImplicitAny` 설정을 해서 잡을 수도ㅗ 있지만, 이미 존재하는 JSDoc의 타입 정보를 활용하느느 것이 더 낫습니다.

마지막 단계로, 테스트 코드를 타입스크립트로 전환하면 됩니다.
로직 코드가 테스트 코드에 의존하지 않기 때문에, 테스트 코드는 항상 의존성 관계도의 최상단에 위치하며 마이그레이션의 마지막 단계가 되는 것은 자연스러운 일입니다.
그리고 최하단의 모듈부터 타입스크립트로 전환하는 와중에도 테스트 코드는 변경되지 않았고, 테스트를 수행할 수 있었을 겁니다.
마이그레이션 기간 중에 테스트를 수행할 수 없다는 것은 엄청난 이점입니다.
