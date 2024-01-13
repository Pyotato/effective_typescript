# 🙅 ITEM 3 : 코드 생성과 타입이 관계없음을 이해하기

큰 그림에서 보면, 타입스크립트 컴파일러는 두 가지 역할을 수행합니다.

- 최신 타입스크립트/자바스크립트 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일(transfile)합니다.
- 코드의 타입 오류를 체크합니다.

>   번역(translate)과 컴파일(compile)이 합쳐져 트랜스파일이라는 신조어가 탄생했습니다.
>
> 소스코드를 동일한 동작을 하는 다른 형태의 소스코드 (다른 버전, 다른 언어 등)로 변환하는 행위를 의미합니다.
> 결과물이 여전히 컴파일되어야하는 소스코드이기 때문에 컴파일과는 구분해서 부릅니다.

여기서 놀라운 점은 이 두 가지가 서로 완벽히 독립적이라는 것입니다. 다시 말해서, 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 주지 않습니다.
또한 그 자바스크립트의 실행 시점에도 타입은 영향을 미치지 않습니다.

타입스크립트 컴파일러가 수행하는 두 가지 역할을 되짚어 보면, 타입스크립트가 할 수 있는 일과 할 수 없는 일을 짐작할 수 있습니다.

## 타입 오류가 있는 코드도 컴파일이 가능합니다

컴파일은 타입 체크과 독립적으로 동작하기 떄문에, 타입 오류가 있는 코드도 컴파일이 가능합니다.

```ts

let x = 'hello';
x = 1234;

```

<img width="1001" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/527b0538-ce7e-4b39-ab62-a715063d546b"/><img width="484" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/fb5373a6-2a43-4aa9-83db-cd99ebf43c9a"/>


타입 체크와 컴파일이 동시에 이루어지는 C나 자바같은 언어를 사영하던 사람이라면 이러한 상황이 매우 황당하게 느껴질 겁니다. 
타입스크립트 오류는 C나 자바같은 언어들의 경고(warning)와 비슷합니다. 문제가 될 만한 부분을 알려주지만, 그렇다고 빌드를 멈추지는 않습니다.

> 💡 컴파일과 타입 체크
>
> 코드에 오류가 있을 때 '컴파일에 문제가 있다'고 말하는 경우를 보았을 겁니다. 그러나 이는 기술적으로 틀린 말입니다.
> 엄밀히 말하면 오직 코드 생성만이 '컴파일'이라고 할 수 있기 때문입니다.
> 작성한 타입스크립트가 유효한 자바스크립트라면 타입스크립트 컴파일러는 컴파일을 해냅니다.
> 그러므로 코드에 오류가 있을 때 '타입 체크에 문제가 있다'고 말하는 것이 더 정확한 표현입니다.

타입 오류가 있는 데도 컴파일된다는 사실 때문에 타입스크립트가 엉성한 언어처럼 보일 수 있습니다.
그러나 코드에 오류가 있더라도 컼ㅁ파일된 산출물이 나오는 것이 실제로 도움이 됩니다.
웹 애플리케이션을 만들면서 어떤 부분에 문제가 발생했다고 가정해 보겠습니다.
타입스크립트는 여전히 컴파일된 산출물을 생성하기 때문에, 문제가 된 오류를 수정하지 않더라도 애플리케이션의 다른 부분을 테스트할 수 있습니다.
만약 오류가 있을 때 컴파일하지 않으려면, <i>tsconfig.json</i>에 `noEmitOnError`를 설정하거나 빌드 도구에 동일하게 적용하면 됩니다.

## 런타임에는 타입체크가 불가능합니다

다음처럼 코드르 작성해 볼 수 있습니다.

```ts
interface Square{
  width: number;
}
interface Rectangle extends Square{
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape:Shape){
  if(shape instanceof Rectangle){
    return shape.width + shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

<img width="1105" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0b7d7209-d118-4088-952d-526625aa99a5"/>

instanceof 체크는 런타임에 일어나지만, Rectangle은 타입이기 때문에 런타임 시점에는 아무런 역할을 할 수 없습니다.
타입스크립트의 타입은 '제거 가능(erasable)'합니다.
실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입구문은 그냥 제거 되어 버립니다.

앞의 코드에서 다루고 있는 shape 타입을 명확하게 하려면, 런타임에 타입 정보를 유지하는 방법이 필요합니다.
하나의 방법은 height속성이 있는 지 체크해 보는 것입니다.
다음 코드를 통해 알아보겠습니다.

```ts
function calculateArea(shape: Shape){
  if('height' in shape){
    return shape.width + shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

<img width="338" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/2c042390-a725-48db-a651-cb7a211fd1f3"/><img width="338" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f8cc985f-8c3b-42df-bb4e-feb124a5134f"/>

속성 체크는 런타임에 접근 간으한 값에만 관련되지만, 타입 체커 역시도 shape의 타입을 Rectangle로 보정해 주기 때문에 오류가 사라집니다.

타입 정보를 유지하는 또 다른 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 '태그'기법이 있습니다.

```ts
interface Square{
  kind: 'square';
  width: number;
}
interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape){
  if(shape.kind === 'rectangle'){
    return shape.width + shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

<img width="335" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/c897bc2e-4e2c-40a2-bcf2-d5d1572dfb40"/><img width="335" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f8ffd392-40f1-4bac-9ef3-9097f235ae1b"/>

여기서 Shape 타입은 `태그된 유니온(tagged union)`의 한 예입니다.
이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있습니다.

타입(런타임 접근 불가)과 값(런타임 접근 가능)을 둘 다 사용하는 기법도 잇습니다. 타입을 클래스로 만들면 됩니다.
Square와 Rectangle을 클래스로 만들면 오류를 해결할 수 있습니다.

```ts
class Square{
  constructor(public width: number){}
}
class Rectangle extends Square {
  constructor(public width: number, public height : number){
    super(width);
  }
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape){
  if(shape instanceof Rectangle){
    return shape.width + shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

안터패아스는 타입으로만 사용 가능하지만, Rectangle을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없습니다.


`type Shape = Square | Rectangle`  부분에서 Rectangle은 타입으로 참조되지만, `shape instanceof Rectangle` 부분에서는 값으로 참조됩니다.
어떻게 참조되는지 구분하는 건 매우 중요한데, 이는 [아이템 8](https://github.com/Pyotato/effective_typescript/blob/item8/README.md)에서 다룹니다.

## 타입 연산은 런타임에 영향을 주지 않습니다

string 또는 number 타입인 값을 항상 number로 정제하는 경우를 가정해 보겠습니다. 다음 코드는 타입 체커를 통과하지만 잘못된 방법을 썼습니다.

```ts
function asNumber(val: number| string): number {
  return val as number;
}
```

변환된 자바스크립트 코드를 보면 이 함수가 실제로 어떻게 동작하는지 알 수 있습니다.

<img width="747" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0e7323f5-6d6f-42b4-b61c-0db8a2143ac8"/>

코드에 아무런 정제 과정이 없습니다.
`as number`는 타입 연산이고 란타임 동작에는 아무런 영향을 미치지 않습니다. 
값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 자바스크립트 연산을 통해 변환을 수행해야합니다.

```ts
function asNumber(val: number| string): number {
  return typeof(val) ==='string'? Number(val) : val;
}
```

`as number`는 **타입 단언문**입니다.
이런 문법이 언제 적절히 사용될 수 있는지는 [아이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md)에서 다룹니다.

## 런타임 타입은 선언된 타입과 다를 수 있습니다

다음 함수를 보고 마지막의 `console.log`까지 실행될 수 있을 지 생각해 보기 바랍니다.

```ts
function setLightSwitch(value : boolean){
  switch(value){
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log(`실행되지 않을까봐 걱정됩니다.`);
  }
}
```

타입스크립트는 일반적으로 실행되지 못하는 죽은 (dead) 코드를 찾아내지만, 여기서는 strict를 설정하더라도 찾아내지 못합니다.
그러면 마지막 부분을 실행할 수 있는 경우는 무엇일까요?

`: boolean`이 타입 선언문이라는 것에 주목하기 바랍니다.
타입스크립티이기 때문에 `: boolean`은 런타임에 제거됩니다. 자바스크립트였다면 실수로 `setLightSwitch`를 "ON"으로 호출할 수도 있었을 겁니다.

순수 타입스크립트에서도 마지막 코드를 실행하는 방법이 존재합니다. 
예를 들어, 네트워크 호출로부터 받아오은 값으로 함수를 실행하는 경우가 있습니다.

```ts
interface LightApiResponse{
  lightSwitchValue: boolean;
}

async function setLight(){
  const response = await fetch('/light');
  const result : LightApiResponse = await response.json();
  setLightSwitch(result.lightSwitchValue);
}
```

`/light`를 요청하면 그 결과로 `LightApiResponse`를 반환하라고 선언했지만, 실제로 그렇게 되리라는 보장은 없습니다.
API를 잘못 파악해서 lightSwitchValue가 실제로는 문자열이었다면, 런타임에는 setLightSwitch 함수까지 전달될 것입니다.
또는 배포된 후에 API가 변경되어 lightSwitchValue가 문자열이 되는 경우도 있을 겁니다.

타입스크립트에서는 런타임 타입과 타입이 맞지 않을 수 있습니다.
타입이 달라지는 혼란스러운 상황을 가능한 한 피해야 합니다. 선언된 타입이 언제든지 달라질 수 있다는 것을 명심해야 합니다.

## 타입스크립트 타입으로는 함수를 오버로드할 수 없습니다

C++과 같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용합니다.
이를 '함수 오버로딩'이라고 합니다.

그러나 타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에, 함수 오버로딩은 불가능합니다.

```ts
function add(a: number, b: number){return a+b;}
function add(a: string, b: string){return a+b;}
```

<img width="822" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/dbf048ec-d00d-482e-9753-72ef790248a6"/>

타입스크립트가 함수 오버로딩 기능을 지원하기는 하지만, 온전히 타입 수준에서만 동작합니다.
하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체 (implementation)는 오직 하나뿐입니다.

```ts
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a,b){
    return a+b;
}

const three = add(1,2);
const twelve = add('1','2');
```

<img width="932" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6648153a-490a-4773-862a-94345614ea24"/>

`add`에 대한 처음 두 개의 선언문은 타입 정보를 제공할 뿐입니다. 
이 두 선언문은 타입스크립트가 자바스크립트로 변환되면서 제거되며, 구현체만 남게 됩니다.
(이런 스타일의 오버로딩을 사용하용하려면, [아이템 50](https://github.com/Pyotato/effective_typescript/blob/item50/README.md)을 먼저 살표보기 바랍니다. 꼭 알아아햘 몇 가지 주의 사항이 있습니다.)

## 타입스크립트 타입은 런타임 성능에 영향을 주지 않습니다.

타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에, 런타임의 성능에 아무런 영향을 주지 않습니다.
타입스크립틩 정적 타입은 실제로 비용이 전혀 들지 않습니다.
타입스크립트를 쓰는 대신 런타임 오버헤드를 감수하며 타입 체크를 해본다면, 타입스크립트 팀이 다음 주의 사항들을 얼마나 테스트해 왔는지 몸소 느끼게 될 것입니다.

- '런타임' 오버헤드가 없는 대신, 타입스크립트 컴파일러는 '빌드타임' 오버헤드가 있습니다. 타입스크립트 팀은 컴파일러 성능을 매우 중요하게 생각합니다. 
따라서 컴파일은 일반적으로 상당히 빠른 편이며 특히 증분(incremental) 빌드 시에 더욱 체감됩니다. 오버헤드가 커지면, 빌드 도구에서 '트랜스파일만(transpile only)'을 설정하여 타입 체크를 건너뛸 수 있습니다.
- 타입스클비트가 컴파일하는 코드는 오래된 런타임 환경을 지원하기 위해 호환성을 높이고 성능 오버헤드를 감안할지, 호환성을 포기하고 성능 중심의 네이티브 구현체를 선택할지의 문제에 맞닥뜨릴 수도 있습니다.
예를 들어 제너레이터 함수가 ES5 타겟으로 컴파일되려면, 타입스크립트 컴파일러는 호환성을 위한 특정 헬퍼 코드를 추가할 것입니다. 이런 경우가 제너레이터의 호환성을 위한 오버헤드 또는 성능을 위한 네이티브 구현체 선택의 문제입니다.
어떤 경우든지 호환성과 성능 사이의 선택은 컴파일 타겟과 언어 레벨의 문제이며 여전히 타입과는 무관합니다.


