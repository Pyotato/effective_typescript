# 🅴🅲🅼🅰6️⃣ ITEM 53 : 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

타입스크립트가 태동하던 2010년경, 자바스크립트는 결함이 많고 개선해야할 부분이 많은 언어였습니다.
그리고 클래스, 데코레이터, 모듈 시스템 같은 기능이 없어서 프레임워크나 트랜파일러로 보완하는 것이 일반적인 모습이었습니다. 
그렇기 때문에 타입스크립트도 초기 버전에는 독립저긍로 개발한 클래스, 열거형(enum), 모듈 시스템을 포함시킬 수 밖에 없었습니다. 

시간이 흐르며 TC39(자바스크립트를 관장하는 표준 기구)는 부족했던 점을 대부분 내장 기능으로 추가했습니다.
그러나 자바스크립트에 새로 추가된 기능은 타입스트립트 초기 버전에서 독립적으로 개발했던 기능과 호환성 문제를 발생시켰습니다.
그렇기에 타입스크립트 진영에서는 다음 전략 중 하나를 선택해야 했습니다.
한 가지 전략은 타입스크립트 초기 버전의 형태를 유지하기 위해 타입스크립트 신규 기능을 변형해서 끼워 맞추는 것입니다.
또 다른 전략은 자바스크립트의 신규 기능을 그대로 채택하고 타입스크립트 초기 버전과 호환성을 포기하는 것입니다.

타입스크립트 팀은 대부분 두 번째 전략을 선택했습니다. 결국 TC39는 런타임 기능을 발전시키고, 타입스크립트 팀은 타입 기능만 발전시킨다는 명확한 원칙을 세우고 현재까지 지켜오고 있습니다.

그런데 이 원칙이 세워지기 전에, 이미 사용되고 있던 몇 가지 기능이 있습니다.
이 기능들은 타입 공간(타입스크립트)과 공간(자바스크립트)의 경계를 혼란스럷게 만드릭 때문에 사용하지 않는 것이 좋습니다. 여기서는 피해야 하는 기능을 몇가지 살펴봅니다.
그리고 불가피하게 이 기능을 사용하게 될 경우 어떤 점에 유의해야 호환성 문제를 일으키지 않는지 알아봅니다.

## 열거형(enum)

많은 언어에서 몇몇 값의 모음을 나타내기 위해 열거형을 사용합니다. 타입스크립트에서도 열거형을 사용할 수 있습니다.

```ts
enum Flavor{
  VANILLA = 0,
  CHOCOLATE = 1,
  STRAWBERRY = 2,
}

let flavor = Flavor.CHOCOLATE;

Flavor. // 자동완성 추천 : VANILLA, CHOCOLATE, STRAWBERRY
Flavor[0] //  값이 "VANILLA"

```

<img width="521" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/46291a7c-49fb-4dca-8c83-6bcaa419f38c"/>

단순히 값을 나열하는 것보다 실수가 적고 명확하기 때문에 일반적으로 열거형을 사용하는 것이 좋습니다.
그러나 티입스크립트의 열거형은 몇 가지 문제가 있습니다. 
타입스크립트의 열거형은 다음 목록처럼 상황에 따라 다르게 동작합니다.

- 숫자 열거형 (앞 예제의 `Flavor`)에서 0,1,2외의 다른 숫자가 할당되면 매우 위험합니다.(이 방법은 원래 비트 플래그 구조를 표현하기 위해 설계되었습니다.)
- 상수 열거형은 보통의 열거형과 달리 런타임에 완전히 제거됩니다. 앞의 예제를 `const enum Flavor`로 바꾸면, 컴파일러는 Flavor.CHOCOLATTE을 0으로 바꿔 버립니다. 이런 결과는 기대하지 않은 것이며, 문자열 열거형과 숫자 열거형과 전혀 다른 동작입니다.
- `preserveConstEnums`플래그를 설정한 상태의 상수 열거형은 보통의 열거형처럼 런타임 코드에 상수 열거형 정보를 유지합니다.
- 문자열 열거형은 런타임의 타입 안정성과 투명성을 제공합니다.그러나 타입스크립트의 다른 타입과 달리 구조적 타이핑이 아닌 명목적 타이핑을 사용합니다 (바로 이어서 설명합니다).

타입스크립트의 일반적인 타입들이 할당 가능성을 체크하기 위해서 구조적 타이핑([아이템4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md))를 사용하는 반면, 문자열 열거형은 명목적 타이핑(`nominally typing`)을 사용합니다.

> 구조적 타이핑은 구조가 같으면 할당이 허용되는 반면, 명목적 타이핑은 타입의 이름이 같아야 할달이 허용됩니다.

```ts
enum Flavor{
  VANILLA = 'vanilla',
  CHOCOLATE = 'chocolate',
  STRAWBERRY = 'strawnerry',
}

let flavor = Flavor.CHOCOLATE;
flavor = 'strawberry';

```

<img width="1603" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1e2b1b12-0f9f-4f09-918c-50f101c04c76"/>

명목적 타이핑은 라이브러릴르 공개할 때 필요합니다.
Flavor를 매개변수로 받는 함수를 가정해 봅시다.

`function scoop(flavor: Flavor){/*....*/}`

Flavor는 런타임 시점에는 문자열이기 때문에, 자바스크립트에서 다음처럼 호출할 수 있습니다.

`scoop('vanilla')`

<img width="1478" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/53419b7b-7c3a-43d7-83f2-10d85231b09e"/>

그러나 타입스크립트에서는 열거형을 임포트하고 문자열 대신 사용해야 합니다.

```ts
import {Flavor} from 'ice-cream';
scoop(Flavor.VANILLA);
```

이처럼 자바스크립트와 타입스크립트에서 동작이 다르기 때문에 문자열 열거형은 사용하지 않는 것이 좋습니다.
열거형 대신 `리터럴 타입의 유니온`을 사용하면 됩니다.

```ts
type Flavor = 'vanilla' | 'chocolate' | 'strawberry' ;
let flavor : Flavor = 'chocolate';
flavor = 'mint chip'; 

```

<img width="1330" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8cc47da4-5e1d-4fee-810a-fb439a7ec10f">

리터럴 타입의 유니온은 열거형만큼 안전하며 자바스크립트와 호환되는 장점이 있습니다.
그리고 편집기에서 열거형처럼 자동완성 기능을 사용할 수 있습니다. (아래의 예시에서 자동완성이 'vanilla'를 추천합니다.

```ts
function scoop(flavor: Flavor){
  if(flavor === 'v
}
```

<img width="593" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/c174beca-618b-48b2-b879-de351b631d50">


자세한 내용은 [아이템 33](https://github.com/Pyotato/effective_typescript/blob/item33/README.md)

## 매개변수 속성

읿반적으로 클래스를 초기화할 때 속성을 할당하기 위해 생생자의 매개변수를 사용합니다.

```ts
class Person{
  name: string;
  constructor(name: string){
    this.name = name;
  }
}
```

타입스크립트는 더 간결한 문법을 제공합니다.

```ts
class Person{
  constructor(public name: string){}
}
```

예제의 public name은 '매개변소 속성'이라고 불리며, 
멤버 변수로 name을 선언한 이전 예제와 동일하게 동작합니다.
그러나 매개변수 속서오가 관련된 몇 가지 문제점이 존재합니다.

- 일반적으로 타입스크립트 컴파일은 타입 제거가 이루어지므로 코드가 줄어들지만, 매개변수 속성은 코드가 늘어나는 문법입니다.
- 매개변수 속성이 런타임에는 실제로 사용되지만, 타입스크립트 관점에서는 사용되지 않는 것처럼 보입니다.
- 매개변수 속성과 일반속성을 섞어서 사용하면 클래스의 설계가 혼란스러워집니다.

문제점들에 대한 예를 들어 보겠습니다.

```ts
class Person{
  first : string;
  last : string;
  constructor(public name: string){
    [this.first, this.last] = name.split(' ');
  }
}
```

Person클래스에는 세 가지 속성(first,last,name)이 있지만, first와 last만 속성에 나열되어 있고 name은 매개변수 속성에 있어서 일관성이 없습니다.

클래스에 매개변수 속성만 존재한다면 클래스 대신 인터페이스로 만들고 객체 리터럴을 사용하는 것이 좋습니다. 
구조적 타이핑 특성 때문에 다음 예제처럼 할당할 수 있다는 것을 주의해야합니다. ([아이템4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md))

```ts
class Person{
  constructor(public name: string){}
}
const p : Person = {name: 'Jed Bartlet'};
```

매개변수 속성을 사용하는 것이 좋은지에 대해서는 찬반 논란이 있습니다. 
저는 매개변수 속성을 선호하지 않지만, 어떤 이들은 코드양이 줄어들어서 좋아하기도 합니다.
매개변수 속성은 타입스크립트의 다른 패턴들과 이질적이고, 초급자에게 생소한 문법이라는 것을 기억해야 합니다.
또한 매개변수 속성과 일반 속성을 같이 사용하면 설계가 혼란스러워지기 때문에 한가지만 사용하는 것이 좋습니다.

## 네임스페이스와 트리플 슬래시 임포트

ECMAScript 2015 이전에는 자바스크립트에 공식적인 모듈 시스템이 없었습니다.
그래서 각 환경마다 자신만의 방식으로 모듈 시스템을 마련했습니다.
Node.js는 require와 module.exports를 사용하는 반면, AMD는 define 함수와 콜백을 사용합니다.

타입스크립트 역시 자체적으로 모듈 시스템을 구축했고, module 키워드와 '트리플 슬래시' 임프토를 사용했습니다.
ECMAScript 2015가 공식적으로 모듈시스템을 도입한 이후, 타입스크립트는 충돌을 피하기 위해 module과 같은 기능을 하는 namespace 키워드를 추가했습니다.

```ts
namespace foo {
  function bar(){}
}
/// <reference path="other.ts"/>
foo.bar();
```

트리플 슬래시 임포트와 module 키워드는 호환성을 위해 남아 있을 뿐이며, 이제는 ECMAScript 2015 스타일의 모듈 (import와 export)을 사용해야 합니다.
[아이템 58](https://github.com/Pyotato/effective_typescript/blob/item58/README.md)을 참고하시길 바랍니다.

## 데코레이터

데코레이터는 클래스, 메서드, 속성에 애너테이션(annotation)을 붙이거나 기능을 추가하는 데 사용할 수 있습니다.
예를 들어, 클래스의 메서드가 호출될 때마다 로그르 남기려면 logged 애너테이션을 정의할 수 있습니다.

```ts
class Greeter{
  greeting : string;
  constructor(message: string){
    this.greeting = message;
  }
  @logged
  greet(){
    return "Hello," + this.greeting;
  }
}

function logged(target:any, name:string, descriptor: PropertyDescriptor){
  const fn = target[name];
  descriptor.value = function(){
    console.log(`Calling ${name}`);
    return fn.apply(this, arguments);
  };
}

console.log(new Greeter('Dave').greet());

```

<img width="1118" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/58846fba-7ac2-403f-881c-8d5df092dc58">

데코레이터는 처음에 앵귤러 프레임워크를 지원하기 위해 추가되었으며 _tsconfig.json_에 exprerimentalDecorators 속성을 설정하고 사용해야 합니다.
현재까지도 표준화가 완료되지 않았기 때문에, 사용 중인 데코레이터가 비표준으로 바뀌거나 호환성이 깨질 가능성이 있습니다.
앵귤러를 사용하거나 애너테이션이 필요한 프레임워크를 사용하고 있는게 아니라면, 데코레이터가 표준이 되기 전에는 타입스크립트에서 데코레이터를 사용하지 않는게 좋습니다.


