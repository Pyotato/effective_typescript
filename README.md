# effective_typescript

## Contents

### 1장 : 타입스크립트 알아보기

> 1장에서는 타입스크립트의 큰 그림을 이해하는 데 도움이 될 내용을 다룹니다. 본격적인 내용에 앞서 타입스크립트란 무엇이고, 타입스크립트를 어떻게 여겨야 하는지, 자바스크립트와는 어떤 관계인지, 타입스크립틔 타입들은 null이 가능한 지, any 타입에서는 어떨지, 덕 타이핑 (duck-typing)이 가능한지 등을 알아봅니다. <br/>
> 타입스크립트는 사용 방식 면에서 조금은 독특한 언어입니다. 인터프리터(파이썬이나 루비 같은)로 실행되는 것도 아니고, 저수준 언어로 컴파일(자바나 C같은)되는 것도 아닙니다.
> 또 다른 고수준 언어인 자바스크립트로 컴파일되면, 실행 역시 타입스크립트가 아닌 자바스크립트로 이루어집니다.
> 그래서 타입스크립트와 자바스크립트의 관계는 필연적이며, 이 밀접한 관계때문에 혼란스러운 일이 벌어지기도 합니다.
> 이러한 타입스크립와 자바스크립트의 관계를 잘 이해한다면 타입스크립트 개발자로서 한 단계 성장할 수 있습니다.<br/>
> 타입스크립트의 타입 시스템도 조금 독특한 특징을 가지고 있는데, 이 특징들은 자세히 알아둬야 합니다.
> 타입 시스템에 관래서는 뒤에서 자세히 다룰 텐데, 여기서는 몇 가지 주목할 만한 점을 미리 짚고 넘어가겠습니다.


| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  1  | [타입스크립트와 자바스크립트의 관계 이해하기](https://github.com/Pyotato/effective_typescript/blob/item1/README.md) | 💡 타입스크립트는 자바스크립트의 상위집입니다. 다시 말해서, 모든 자바스크립트 프로그램은 이미 타입스크립트 프로그램입니다. 반대로, 타입스크립트는 별도의 문법을 가지고 있기 떄문에 일반적으로는 유효한 자바스크립트 프로그램이 아닙니다. <br/> 💡 타입스크립트는 자바스크립트 런타임 동작을 모델링하는 타입 시스템을 가지고 있기 떄문에 런타임 오류를 발생시키는 코드를 찾아내려고 합니다. 그러나 모든 오류를 찾아내리라 기대하면 안됩니다. 타입체커를 통과하면서도 런타임 오류를 발생시키는 코드는 충분히 존재할 수 있습니다.  <br/> 💡 타입스크립트 타입 시스템은 전반적으로 자바스크립트 동작을 모델링합니다. 그러나 잘못된 매개변수 개수로 함수를 호출하는 경우처럼, 자바스크립트에서는 허용되지만 타입스크립트에서는 문제가 되는 경우도 있습니다. 이러한 문법의 엄격함은 온전히 취향의 차이이며 우열을 가릴 수 없는 문제입니다.                                                                                       |
|  2  | [타입스크립트 설정 이해하기](https://github.com/Pyotato/effective_typescript/blob/item2/README.md) | 💡 타입스크립트 컴파일러는 언어의 핵심 요소에 영향을 미치는 몇 가지 설정을 포함하고 있습니다. <br/> 💡 타입스크립트 설정은 커맨드 라인을 이용하기 보다는 <i>tsconfig.json</i>을 사용하는 것이 좋습니다. <br/> 💡 자바스크립트 프로젝트를 타입스크립트로 전환하는 게 아니라면 `noImplicitAny`를 설정하는 것이 좋습니다. <br/> 💡 "undefined는 객체가 아닙니다"같은 런타임 오류를 방지하기 위해 `strictNullChecks`를 설정하는 것이 좋습니다.  <br/> 💡 타입스크립트에서 엄격한 체크를 하고 싶다면 strict 설정을 고려해야 합니다.  |
|  3  | [코드 생성과 타입이 관계없음을 이해하기](https://github.com/Pyotato/effective_typescript/blob/item3/README.md) | 💡 코드 생성은 타입 시스템과 무관합니다. 타입스크립트 타입은 런타임 동작이나 성능에 영향을 주지 않습니다. <br/> 💡 타입  오류가 존재하더라도 코드 생성(컴파일)은 가능합니다. <br/> 💡 타입스크립트 타입은 런타임에 사용할 수 없습니다. 런타임에 타입을 지정하려면, 타입 정보를 위한 별도의 방법이 필요합니다. 일반적으로는 태그된 유니온과 속성 체크 방법을 사용합니다. 또는 클래스 같이 타입스크립트 타입과 런타임 값, 둘 다 제공하는 방법이 있습니다.                                                                        |
|  4  | [구조적 타이핑에 익숙해지기](https://github.com/Pyotato/effective_typescript/blob/item4/README.md) | 💡 자바스크립트가 덕 타이핑(duck typing) 기반이고 타입스크립트가 이를 모델링하기 위해 구조적 타이핑을 사용함을 이해해야합니다. 어떤 인터페이스에 할당 가능한 값이라면 타입 선언에 명시적으로 나열된 속성들을 가지고 있을 겁니다. 타입은 '봉인'되어 있지 않습니다. <br/> 💡 클래스 역시 구조적 타이핑 규칙을 따른다는 것을 명심해야 합니다. 클래스의 인스턴스가 예상과 다를 수 있습니다.  <br/> 💡구조적 타이핑을 사용하면 유닛 테스팅을 손쉽게 할 수 있습니다.  |
|  5  | [any 타입 지양하기](https://github.com/Pyotato/effective_typescript/blob/item5/README.md) | 💡 any 타입을 사용하면 타입 체커와 타입스크립트 언어 서비스를 무력화시켜 버립니다. any 타입은 진짜 문제점을 감추며, 개발 경험을 나쁘게 하고, 타입 시스템의 신뢰도를 떨어뜨립니다. 최대한 사용을 피하도록 합시다. | 


### 2장 : 타입스크립트의 타입 시스템

> 타입스크립트는 코드를 자바스크립트로 변환하는 역할도 하지만 [(아이템 3)](https://github.com/Pyotato/effective_typescript/blob/item3/README.md), 가장 중요한 역할은 타입 시스템에 있습니다.<br/>
> 타입스크립트를 사용하는 진정한 이유이기도 합니다.<br/>
> 2장에서는 타입 시스템의 기초부터 살펴봅니다. 타입 시스템이란 무엇인지, 어떻게 사용해야 하는지, 무엇을 결정해야 하는지, 가급적 사용하지 말아야할 기능은 무엇인지 알아봅니다.<br/>
> 타입스크립트의 타입 시스템은 매우 강력하며, 생각보다 더 많은 것을 할 수 있습니다. 2장의 아이템들은 타입스크립트 코드를 작성할 때, 그리고 이 책의 나머지를 읽을 때 필요한 개념의 견고한 토대를 마랸해 줄 겁니다.<br/>

| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  6  | [편집기를 사용하여 타입 시스템 탐색하기](https://github.com/Pyotato/effective_typescript/blob/item6/README.md) | 💡 편집기에서 타입스크립트 언어 서비스를 적극 활용해야 합니다. <br/> 💡 편잡기를 사용하면 어떻게 타입 시스템이 동작하는지, 그리고 타입스크립트가 어떻게 타입을 추론하는지 개념을 잡을 수 있습니다. <br/> 💡 타입스크립트가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일을 찾아보는 방법을 터득해야 합니다. |
|  7  | [타입이 값들의 집합이라고 생각하기](https://github.com/Pyotato/effective_typescript/blob/item7/README.md) | 💡 타입을 값의 집합으로 생각하면 이해하기 편합니다(타입의 '범위'). 이 집합은 유한(boolean 또는 리터럴 타입)하거나 무한(number 또는 string)합니다. <br/> 💡 타입스트립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합 (벤 다이어그램)으로 표현 됩니다. 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있습니다. <br/> 💡 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있습니다. <br/> 💡 타입스트립트 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합 (벤 다이어그램)으로 표현 됩니다. 두 타입은 서로 서브타입이 아니면서도 겹쳐질 수 있습니다. <br/> 💡 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있습니다. <br/> 💡 타입 연산은 집합의 범위에 적용됩니다, A와 B의 인터섹션은 A의 범위와 B의 범위의 인터섹션입니다. 객체 타입에서는 `A & B`인 값이 A와 B의 속성을 모두 가짐을 의미합니다.. <br/> 💡 `A는B를 상속` === `A는 B에 할당 가능` === `A는B의 서브타입` === `A는B의 부분 집합`, 모두 같은 의미입니다.  |
|  8  | [타입 공간과 값 공간의 심벌 구분하기](https://github.com/Pyotato/effective_typescript/blob/item8/README.md) | 💡 타입스크립트 코드를 읽을 때는 타입인지 값인지 구분하는 방법을 터득해야 합니다. 타입스크립트 플레이그라운드를 활용해 개념을 잡는 것이 좋습니다.<br/> 💡 모든 값은 타입을 가지지만, 타입은 값을 가지지 않습니다. `type`과 `interface`같은 키워드는 타입 공간에만 존재합니다. <br/> 💡 class나 enum같은 키워드는 타입과 값 두 가지로 사용될 수 있습니다. <br/> 💡 `"foo"`는 문자열 리터럴이거나, 문자열 리터럴 타입일 수 있습니다. 차이점을 알고 구별하는 방법을 터득해야 합니다. <br/> 💡   `typeof`, `this` 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있습니다.  |
|  9  | [타입 단언보다는 타입 선언을 사용하기](https://github.com/Pyotato/effective_typescript/blob/item9/README.md) | 💡 타입 단언 `(as Type)`보다는 타입 선언 `(: Type)`를 사용해야 합니다. <br/> 💡 화살표 함수의 반환 타입을 명시하는 방법을 터득해야 합니다. <br/> 💡 타입스크립트보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문보다는 null 아님 단언문을 사용하면 됩니다.  |
|  10  | [객체 래퍼 타입 피하기](https://github.com/Pyotato/effective_typescript/blob/item10/README.md) | 💡 기본형 값에 매서드를 제공하기 위해 객체 래퍼 타입이 어떻게 쓰이는지 이해해야 합니다. 직접 사용하거나 인스턴스르 생성하는 것은 피해야합니다. <br/> 💡 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야합니다. String 대신 `string`, Number 대신 `number`, Boolean 대신 `boolean`, Symbol 대신 `symbol` BigInt 대신 `bigint`를 사용해야 합니다.  |
|  11  | [잉여 속성 체크의 한계 인지하기](https://github.com/Pyotato/effective_typescript/blob/item11/README.md) | 💡 객체 리터럴을 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행됩니다. <br/> 💡 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다릅니다. 할당의 개념을 정확히 알아야 잉여 속성 체크와 일반적인 구조적 할당 가능성 체크를 구분할 수 있습니다. <br/> 💡 잉여 속성 체크에는 한계가 있습니다. 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다는 점을 기억해야 합니다.  |
|  12  | [함수 표현식에 타입 적용하기](https://github.com/Pyotato/effective_typescript/blob/item12/README.md) | 💡 매개변수나 반환 값에 타입을 명시하기보다는 함수 표현식 전체에 타입 구문을 적용하는 것이 좋습니다. <br/> 💡 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾아보도록 합니다. 라이브러리를 직접 만든다면 공통 콜백에 타입을 제공해야 합니다. <br/> 💡 다른 함수의 시그니처를 참조하려면 `typeof fn`을 사용하면 됩니다.  |
|  13  | [타입과 인터페이스의 차이점 알기](https://github.com/Pyotato/effective_typescript/blob/item13/README.md) | 💡 타입과 인터페이스의 차이점과 비슷한 점을 이해해야 합니다. <br/> 💡 한 타입을 type과 interface 두 가지 문법을 사용해서 작성하는 방법을 터득해야 합니다. <br/> 💡 프로젝트에서 어떤 문법을 사용할 지 결정 할 때 한 가지 일관된 스타일을 확립하고, 보강 기법이 필요한지 고려해야 합니다.  |
|  14  | [타입 연산과 제너릭 사용으로 반복 줄이기](https://github.com/Pyotato/effective_typescript/blob/item14/README.md) | 💡 DRY(Don't Repeat Yourself) 원칙을 타입에도 최대한 적용해야 합니다. <br/> 💡 타입에 이름을 붙여서 반복을 피해야 합니다. extends를 사용해서 인터페이스 필드의 반복을 피해야 합니다. <br/> 💡 타입들 간의 매핑을 위해 타입스크립트가 제공한 도구들을 공부하면 좋습니다. 여기에는 `keyof`,`typeof`,인덱싱, 매핑된 타입들이 포함됩니다. <br/> 💡 제너릭 타입은 타입을 위한 함수와 같습니다. 타입을 반복하는 대신 제너릭 타입을 사용하며 타입들 간에 매핑을 하는 것이 좋습니다. 제너릭 타입을 제한하려면 extends를 사용하면 됩니다. <br/> 💡 표준 라이브러리에 정의된 `Pick, Partial, ReturnType`같은 제너릭 타입에 익숙해져야 합니다. |
|  15  | [동적 데이터에 인덱스 시그니처 사용하기](https://github.com/Pyotato/effective_typescript/blob/item15/README.md) | 💡 런타임 때까지 객체의 속성을 알 수 없을 경우에만 (예를 들어 CSV 파일에서 로드하는 경우) 인덱스 시그니처를 사용하도록 합니다. <br/> 💡 안전한 접근을 위해 인덱스 시그니처의 값 타입에 `undefined`를 추가하는 것을 고려해야 합니다. <br/> 💡 가능하다면 인터페이스, Record, 매핑괸 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋습니다. |
|  16  | [number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기](https://github.com/Pyotato/effective_typescript/blob/item16/README.md) | 💡 배열은 객체이므로 키는 숫자가 아니라 문자열입니다.인덱스 시그니처로 사용된 number 타입은 버그를 잡기 위한 순수 타입스크립트 코드입니다. <br/> 💡 인덱스 시그니처에 number를 사용하기 보다 Array나 튜플, 또는 ArrayLike타입을 사용하는 것이 좋습니다. |
|  17  | [변경 관련된 오류 방지를 위해 readonly 사용하기](https://github.com/Pyotato/effective_typescript/blob/item17/README.md) | 💡 만약 함수가 매개변수를 수정하지 않는다면, `readonly`로 선언하는 것이 좋습니다. `readonly` 매개변수는 인터페이스를 명확하게 하며, 매개변수가 변경되는 것을 방지합니다. <br/> 💡 `readonly`를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고, 변경ㅇ이 발생하는 코드도 쉽게 찾을 수 있습니다.<br/> 💡 const와 `readonly`의 차이를 이해해야 합니다. <br/> 💡 `readonly`는 얕게 동작한다는 것을 명심해야 합니다.|
|  18  | [매핑된 타입을 사용하여 값을 동기화하기](https://github.com/Pyotato/effective_typescript/blob/item18/README.md) | 💡 매핑된 타입을 사용해서 관련된 값과 타입을 동기화하도록 합니다. <br/> 💡 인터페이스에 새로운 속성을 추가할 떄, 선택을 강제하도록 매핑된 타입을 고려해야 합니다.|

### 3장 : 타입 추론

> 산업계에서 사용되는 프로그래밍 언어들에서는 '정적 타입'과 '명시적 타입'이 전통적으로 같은 의미로 쓰였습니다. 그래서 C, C++, 자바에서는 타입을 직접 명시합니다.<br/>
> 그러나 학술계의 언어에서는 이 두가지 타입은 결코 혼동해서 쓰지 않습니다. 학술계로 분류되는 ML과 하스켈 같은 언어는 오래전부터 정교한 타입 추론 시스템을 가지고 있었습니다. <br/>
> 학술계 언어의 발전에 대응하여 10년 전부터는 기존 산업계의 언얻에도 타입 추론 기능이 추가되기 시작했습니다.
> C++는 auto를 추가했고, 자바는 var를 추가했습니다.<br/><br/>
>
> 타입스크립트는 타입 추론을 적극적으로 수행합니다. 타입 추론은 수동적으로 명시해야 하는 타입 구문의 수를 엄청나게 줄여 주기 때문에, 코드의 전체적인 안정성이 향상됩니다.
> 타입스크립트 초보자와 숙련자는 타입 구문의 수에서 차이가 납니다.
> 숙련된 타입스크립트 개발자는 비교적 적은 수의 구문 (그러나 중요한 부분에는 사용)을 사용합니다. 반면, 초보자의 코드는 불필요한 타입 구문으로 도배되어 있을 겁니다.]
>
> 3장에서는 타입 추론에서 발생할 수 있는 몇 가지 문제와 그 해법을 안내합니다.
> 3장을 읽은 후에는 타입스크립트가 어떻게 타입을 추론하는지, 언제 타입을 선언 해야하는지, 타입 추론이 가능하더라도 명시적으로 타입 선언을 작성하는 것이 필요한 상황이 언제인지 잘 이해할 수 있을 겁니다.


| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  19  | [공식 명칭에는 상표를 붙이기](https://github.com/Pyotato/effective_typescript/blob/item19/README.md) | 💡 타입스크립트가 타입을 추론할 수 있다면 타입 구문을 작성하지 않는게 좋습니다.<br/>  💡 이상적인 경우 함수/메서드의 시그니처에는 타입 구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없습니다. <br/> 💡 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려해야합니다. 이는 내부 구현의 오류가 사용자 코드 위치에 나타나는 것을 방지해 줍니다.                                                                  |
|  20  | [다른 타입에는 다른 변수 사용하기](https://github.com/Pyotato/effective_typescript/blob/item20/README.md) | 💡 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않습니다. <br/>  💡 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 합니다.                       |
|  21  | [타입 넓히기](https://github.com/Pyotato/effective_typescript/blob/item21/README.md) | 💡 타입스크립트가 넗히기를 통해 상수의 타입을 추론하는 법을 이해해야 합니다. <br/>  💡 동작에 영향을 줄 수 있는 방법인 const, 타입 구문, 문맥, as const에 익숙해져야 합니다.                           |
|  22  | [타입 좁히기](https://github.com/Pyotato/effective_typescript/blob/item22/README.md) | 💡 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야 합니다.<br/>  💡 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원활하게 만들 수 있습니다.                           | 
|  23  | [한꺼번에 객체 생성하기](https://github.com/Pyotato/effective_typescript/blob/item23/README.md) | 💡 속성을 제각각 추가하지 말고 한꺼번에 객체로 만들어야 합니다. 안전한 타입으로 속성을 추가하려면 객체 전개 `{...a, ...b}`를 사용하면 됩니다. <br/>  💡  객체에 조건부로 속성을 추가하는 방법을 익히도록 합니다.                           | 
|  24  | [일관성있는 별칭 사용하기](https://github.com/Pyotato/effective_typescript/blob/item24/README.md) | 💡 별칭은 타입스크립트가 타입을 좁히는 것을 방해합니다. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야 합니다. <br/> 💡 비구조화 문법을 사용해서 일관된 이름을 사용하는 것이 좋습니다. <br/> 💡 함수 호출이 객체 속성의 타입 정체를 무효화할 수 있다는 점을 주의해야합니다. 속성보다 지역 변수를 사용하면 타입 정제를 믿을 수 있습니다.             | 
|  25  | [비동기 코드에는 콜백대신 async 함수 사용하기](https://github.com/Pyotato/effective_typescript/blob/item25/README.md) | 💡 콜백보다는 프로미스를 사용하는 게 코드 작성과 타입 추론 면에서 유리합니다.<br/> 💡 가능하면 프로미스를 생성하기보다는 async와 await를 사용하는 것이 좋습니다. 간결하고 직관적인 코드를 작성할 수 있고 모든 종류의 오류를 제거할 수 있습니다. <br/> 💡 어떤 함수가 프로미스를 반환한다면 async로 선언하는 것이 좋습니다.             | 
|  26  | [타입 추론에 문맥이 어떻게 사용되는지 이해하기](https://github.com/Pyotato/effective_typescript/blob/item26/README.md) | 💡 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴봐야 합니다. <br/> 💡 변수를 뽑아서 별도로 선언할 때 오류가 발생한다면 타입 선언을 추가해야 합니다.  <br/> 💡 변수가 정말로 상수 단어(as const)을 사용해야 합니다. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야 합니다.    | 
|  27  | [함수형 기법과 라이브러리로 타입 흐름 유지하기](https://github.com/Pyotato/effective_typescript/blob/item27/README.md) | 💡 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해 직접 구현하기보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋습니다.    | 


### 4장 : 타입 설계

> 누가 순서도를 보여주면서 테이블을 감추면 나는 여전히 갸우뚱할 것이다.<br/>
> 하지만 테이블을 보여준다면 순서도는 별로 필요하지 않다. 보지 않더라도 명백할 것이기 때문이다.
> - <<맨먼스 미신(The Mythical Man Month)>>(인사이트, 2015)

> <<맨먼스 미신>>의 저자 프레드 브룩스의 이 말은 오래 되었지만, 지금도 유효합니다.<br/>
> 연산이 이루어지는 데이터나 데이터 타입을 알 수 없다면 코드를 이해하기 어렵습니다.<br/>
> 타이 시스템의 큰 장점 중 하나는 데이터 타입을 명확히 알 수 있어 코드를 이해하기 쉽다는 것입니다. <br/>

> 다른 장들에서는 타입스크립트 타입의 실질적인 사항을 다루고 있습니다. <br/>
> 타입을 사용하고, 추론하고, 선언문을 작성하는 것들입니다.<br/>
> 그런데 4장에서는 타입 자체의 설계에 대해 다룹니다.<br/>
> 참고로 4장의 예제들은 모두 타입스크립트를 염두에 두고 작성했지만, 대부분 다른 언어에도 적용될 수 있는 아이디어입니다.<br/>
>
> 4장을 이해하고 타입을 제대로 작성한다면, 인용문에서 비유한 것처럼 테이블(코드의 타입)뿐만 아니라 순서도(코드의 로직) 역시 쉽게 이해할 수 있을 겁니다.


| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  28  | [유효한 상태만 표현하는 타입 지정하기](https://github.com/Pyotato/effective_typescript/blob/item28/README.md) | 💡 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란은 초래하기 쉽고 오류를 유발하게 됩니다. <br/> 💡 유효한 상태만 표현하는 타입을 지향해야 합니다. 코드가 길어지거나 표현하기 어렵지만 결국은 시간을 절약하고 고통을 굴일 수 있습니다.  | 
|  29 | [유효한 상태만 표현하는 타입 지정하기](https://github.com/Pyotato/effective_typescript/blob/item29/README.md) | 💡 보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있습니다. 선텍적 속성과 유니온 타입은 반환 타입보다 매개변수 타입에 더 일반적입니다. <br/> 💡 매개변수와 반환 타입의 재사용을 위해서 기본형태 (반환 타입)와 느슨한 형태 (매개변수 타입)를 도입하는 것이 좋습니다. | 
|  30 | [문서에 타입 정보를 쓰지 않기](https://github.com/Pyotato/effective_typescript/blob/item30/README.md) | 💡 주석과 변수명에 타입 정보를 적는 것은 피해야 합니다. 타입 선언이 중복되는 것으로 끝나면 다행이지만 최아그이 경우는 타입 정보에 모순이 발생하게 됩니다. <br/> 💡 타입이 명확하지 않은 경우는 변수명에 단위 정보를 포함하는 것을 고려하는 것이 좋습니다 (예를 들어 timeMs 또는 temperatureC).| 
|  31  | [타입 주변에 null 값 배치하기](https://github.com/Pyotato/effective_typescript/blob/item31/README.md) | 💡 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안 됩니다. <br/> 💡 API작성 시에는 반환 타입을 큰 객체로 만들고 반환 타입 전체가 null이거나 null이 아니게 만들어야 합니다. 사람과 타입 체커 모두에게 명료한 코드가 될 것입니다. <br/> 💡 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 nulldl 존재하지 않도록 하는 것이 좋습니다.  <br/> 💡 `strictNullChecks`를 설정하면 코드에 많은 오류가 표시되겠지만, null값과 관련된 문제점을 찾아낼 수 있기 때문에 반드시 필요합니다.                                                                                             |
|  32  | [유니온의 인터페이스보다는 인터페이스의 유니온 사용하기](https://github.com/Pyotato/effective_typescript/blob/item32/README.md) | 💡 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하므로 주의해야 합니다. <br/> 💡 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋습니다. <br/> 💡 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 합니다. 태그된 유니온은 타입스크립트와 매우 잘 맞기 때문에 자주 볼 수 있는 패턴입니다.                                                                                                   |
|  33  | [string 타입보다 더 구체적인 타입 사용하기](https://github.com/Pyotato/effective_typescript/blob/item33/README.md) | 💡 | 
|  34  | [](https://github.com/Pyotato/effective_typescript/blob/item34/README.md) | 💡 | 
|  36  | [해당 분야의 용어로 타입 이름 짓기](https://github.com/Pyotato/effective_typescript/blob/item36/README.md) | 💡 가독성을 높이고, 추상화 수준을 올리기 위해서 해당 분야의 용어를 사용해야 합니다.<br/> 💡 같은 의미에 다른 이름을 붙이면 안됩니다. 특별한 의미가 있을 때만 용어를 구분해야 합니다. |
|  37  | [공식 명칭에는 상표를 붙이기](https://github.com/Pyotato/effective_typescript/blob/item37/README.md) | 💡 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에, 값을 세밀하게 구분하지 목하는 경우가 있습니다. 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려해야합니다. <br/> 💡 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있습니다.<br/> |


### 5장 : any 다루기

> 전통적으로 프로그래밍 언어들의 타입시스템은 완전히 정적이거나 완전히 동적으로 확실히 구분되어 있었습니다.<br/>
> 그러나 타입스크립트의 타입 시스템은 선택적(optional)이고 점진적(gradual)이기 때문에 정적이면서도 동적인 특정을 동시에 가집니다.<br/>
> 따라서 타입스크립트는 프로그램의 일부분에만 타입시스템을 적용할 수 있습니다.<br/>
>
> 프로그램의 일부분에만 타입 시스템을 적용할 수 있다는 특성 덕분에 점진적인 마이그레이션 (자바스크립트 코드를 타입스크립트로 전환)이 가능합니다 ([아이템 8](https://github.com/Pyotato/effective_typescript/blob/item8/README.md)).<br/>
> 마이그레이션을 할 때 코드의 일부분에 타입 체크를 비활성화시켜주는 any 타입이 중요한 역할을 합니다.<br/>
> 또한 any를 현명하게 사용하는 방법을 익혀야만 효과적인 타입스크립트 코드를 작성할 수 있습니다.<br/>
> any가 매우 강력한 힘을 가지므로 남용하게 될 소지가 높기 때문입니다.<br/>
> 5장에서는 any의 장점은 살리면서 단점을 줄이는 방법들을 살펴보겠습니다.<br/>


| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  38  | [any 타입은 가능한 한 좁은 범위에서만 사용하기](https://github.com/Pyotato/effective_typescript/blob/item38/README.md) | 💡 의도치 않은 타입 안전성의 손실을 피하기 위해서 `any`의 사용 범위를 최소한으로 좁혀야 합니다.<br/> 💡 함수의 반환 타입이 `any`인 경우 타입 안전성이 나빠집니다. 따라서 `any`타입을 반환하면 절대 안됩니다.<br/> 💡 강제로 타입 오류를 제거하려면 any대신 `@ts-ignore`를 사용하는 것이 좋습니다. |
|  39  | [ any를 구체적으로 변형해서 사용하기](https://github.com/Pyotato/effective_typescript/blob/item39/README.md) | 💡 any를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면멸히 검토해야 합니다.  <br/> 💡 any보다 더 정확하게 모델링 할 수 있도록 `any[]` 또는 `{[id: string]: any }` 또는  `() => any` 처럼 구체적인 형태를 사용해야 합니다.     |
|  40  | [함수 안으로 타입 단언문 감추기](https://github.com/Pyotato/effective_typescript/blob/item40/README.md) | 💡 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 합니다. 불가피하게 사용해야한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 합니다.                                                                                                       |
|  41  | [any의 진화를 이해하기](https://github.com/Pyotato/effective_typescript/blob/item41/README.md) | 💡 일반적으로 타입들은 정제되기만 하는 반면, 암시적 any와 `any[]`타입은 진화할 수 있습니다. 이러한 동작이 발생하는 코드를 인지하고 이해할 수 있어야 합니다. <br/> 💡 any를 진화시키는 방식보다는 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법입니다.                                                                                                       |
|  42  | [모르는 타입의 값에는 any대신 unknown을 사용하기](https://github.com/Pyotato/effective_typescript/blob/item42/README.md) | 💡 `unknown`은 `any`대신 사용할 수 있는 안전한 타입입니다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우라면 `unknown`을 사용하면 됩니다.💡 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 `unknown`을 사용하면 됩니다. <br/>💡 `{}`,`object`, `unknown`의 차이점을 이해해야 합니다. |



### 6장 : 타입 선언과 @types

> 타입스크립트를 포함한 모든 언어들에서 라이브러리 의존성 관리는 어려운 일입니다. <br/>
> 6장에선는 타입스크립트에서 의존성이 어떻게 동작하는지 설명하여 의존성에 대한 개념을 잡을 수 있게 합니다. <br/>
> 또한 의존성 관리를 하다가 맞닥뜨릴 수 잇는 몇 가지 문제를 보여 주고 해결하는 방법을 찾아봅니다.<br/>
> 이런 것들이 프로젝트를 공개하기 전에 타입 선언 파일을 작성하는데 도움이 될 겁니다.<br/>
> 제대로 된 타입 선언문을 작성하여 공개하는 것은 프로젝트뿐만 아니라 타입스크립트 전체 커뮤니티에 기여하는 일이기도 합니다.

| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  47  |     [공개 API에 등장하는 모든 타입을 익스포트하기](https://github.com/Pyotato/effective_typescript/blob/item47/README.md)     | 💡 공개 메서드에 등장한 어떤 형태의 타입이든 익스포트합시다. 어차피 라이브러리 사용자가 추출할 수 있으므로, 익스포트하기 쉽 만드는 것이 좋습니다.  |
|  48  |     [API 주석에 TSDoc 사용하기](https://github.com/Pyotato/effective_typescript/blob/item48/README.md)     | 💡 익스포트된 함수, 클래스, 타입에 주석을 달 때는 JSDoc/TSDoc 형태를 사용합시다. JSDoc/TSDoc 형태의 주석을 달면 편집기가 주석 정보를 표시해 줍니다.  <br/> 💡 `@param`,`@returns` 구문과 문서 서식을 위해 마크다운을 사용할 수 있습니다. <br/> 💡 주석에 타입 정보를 포함하면 안됩니다. ([아이템 30](https://github.com/Pyotato/effective_typescript/blob/item30/README.md))  |
|  49  |     [콜백에서 this에 대한 타입 제공하기](https://github.com/Pyotato/effective_typescript/blob/item49/README.md)     | 💡 this 바인딩이 동작하는 원리를 이해해야 합니다. <br/> 💡 콜백 함수에서 this를 사용해야 한다면, 타입 정보를 명시해야 합니다.  |
|  50  |     [오버로딩 타입보다 조건부 타입 사용하기](https://github.com/Pyotato/effective_typescript/blob/item50/README.md)     | 💡 오버로딩 타입보다 조건부 타입을 사용하는 것이 좋습니다. 조건부 타입은 추가적인 오버로딩 없이 유니온 타입을 지원할 수 있습니다.  |



### 7장 : 코드를 작성하고 실행하기

| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  53  |     [타입스크립트 기능보다는 ECMAScript 기능을 사용하기](https://github.com/Pyotato/effective_typescript/blob/item53/README.md)     | 💡 일반적으로 타입스크립트 코드에서 모든 타입 정보를 제거하면 자바스크립트가 되지만, 열거형, 매개변수 속성, 트리플 슬래시 임포트, 데코레이터는 타입 정보를 제거한다고 자바스크립트가 되지 않습니다. <br/> 💡 타입스크립트의 역할을 명확하게 하려면, 열거형, 매개변수 속성, 트리플 슬래시 임포트, 데코레이터는 사용하지 않는 것이 좋습니다. |
|  54  |     [객체를 순회하는 노하우](https://github.com/Pyotato/effective_typescript/blob/item54/README.md)     | 💡 객체를 순회할 때, 키가 어떤 타입인지 정확히 파악하고 있다면 `let k : keysof T`와 `for-in` 루프를 사용합시다. 함수의 매개변수로 쓰이는 객체에는 추가적인 키가 존재할 수 있다는 점을 명심합시다. <br/> 💡 객체를 순회하며 키와 값을 얻는 가장 일반적인 방법은 `Object.entries` 를 사용하는 것입니다. |

### 8장 : 타입스크립트로 마이그레이션하기


> 타입스크립트는 자바스크립트보다 개선된 언어입니다.
> 그러므로 프로젝트를 새로 시작한다면 처음부터 타입스크립트를 사용하면 됩니다.
> 그런데 규모가 크고 오래된 자바스크립트 프로젝트가 이미 존재한다면, 그 프로젝트를 타입스크립트로 전환하기에 시간이 많이 듭니다.
> 그렇다고 그대로 자바스크립트로 유지보수하는 것도 매우 고통스러울 겁니다.
>
> 8장에서는 덩치 크고 낡은 자바스크립트 프로젝트라고 할지라도, 꾸준하게 타입스크립트로 마이그레이션할 수 있게 해 주는 몇 가지 방법을 소개합니다.
>
> 큰 프로젝트를 타입스크립트로 마이그레이션하는 작업은 어렵지만, 프로젝트 품질을 크게 개선할 수 있는 가능성을 열어 줍니다.
> 2017년 한 조사에 따르면 깃헙에 있는 자바스크립트 프로젝트에서 발견된 버그의 15%는, 타입스크립트를 사용했다면 컴파일 시점에서 미리 방지했을 수 있었을 거라고 합니다.
> 또한 에어비앤비에서 진행된 프로젝트들의 사후 분석 6개월치를 조사해 보니, 발견된 버그의 38%가 역시 타입스크립트에서 방지할 수 있었던 것들이라고 합니다.
> 팀 단위 프로젝트에서 타입스크립트를 도입한다면 깃헙과 에어비앤비의 사례를 들어 팀원들을 설득할 수 있을 겁니다.
> 프로젝트에 타입스크립트를 사용하기로 결정했다면, 본격적인 작업에 앞서 점진적 마이그레이션을 실험해 보고 테스트해야 합니다.
> [아이템 59](https://github.com/Pyotato/effective_typescript/blob/item59/README.md)에서 타입스크립트로 실험하는 방법을 설명합니다.
>
> 힌까반에 많은 코드를 타입스크립트로 전환할 수 없기 때문에, 대규모 프로젝트를 마이그레이션할 때는 점진적으로 전환해야 합니다.
> [아이템 60](https://github.com/Pyotato/effective_typescript/blob/item60/README.md)에서 이러한 점진적 마이그레이션을 다룹니다.
>
> 마이그레이션 작업은 오랜 시간이 필요하기 때문에, 진행 상황을 모니터링하고 추적해서 중복된 작업을 방지할 수 있어야 합니다.
> 마이그레이션이 얼마나 진행되었고, 현재 상황이 어떤지 수치화하여 눈으로 볼 수 있게 하면 팀원들에게 동기 부여도 됩니다.
> [아이템 61](https://github.com/Pyotato/effective_typescript/blob/item61/README.md)에서 마이그레이션 진행 상황을 수치화하는 방법을 다룹니다.
>
> 참고로 8장은 대부분의 예제가 순수 자바스크립트여서 타입스크립트 컴파일러에서는 오류가 발생할 수 있습니다.
> 타입 체크 설정을 해제 (예를 들어 `noImplicitAny`룰 off로)하여 타입 체크가 동작하지 않도록 하기 바랍니다. 


| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  62  |     [마이그레이션의 완성을 위해 noImplicitAny 설정하기](https://github.com/Pyotato/effective_typescript/blob/item62/README.md)     | 💡 `noImplicitAny`설정을 활성화하여 마이그레이션의 마지막 단계를 진행해야 합니다. `noImplicitAny` 설정이 없다면 타입 선언과 관련된 실제 오류가 드러나지 않습니다. <br/> 💡 `noImplicitAny`를 전면 적용하기 전에 로컬에서부터 타입 오류룰 점진적으로 수정해야 합니다. 엄격한 타입 체크를 적용하기 전에 팀원들이 타입스크립트에 익숙해질 수 있도록 시간을 줍시다. |
