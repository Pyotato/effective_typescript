# 🏡 ITEM 12 : 함수 표현식에 타입 적용하기

자바스크립트 (그리고 타입스크립트)에서는 함수 '문장(statement)'과 '함수 표현식 (expression)'을 다르게 인식합니다.

```ts
function rollDice1(sides: number): number{ / * ... * / }; // 문장
const rollDice2 = function(sides:number):number { / * ... * / }; // 표현식
const rollDice3 = (sides: number): number => { / * ... * / }; // 표현식

```

타입스크립트에서는 함수 표현식을 사용하는 것이 좋습니다. 
함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문입니다.

```ts
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => { /* ... */ };
```

편집기에서 sides에 마우스를 올려 보면, 타입스크립트에서는 이미 sides의 타입을 number로 인식하고 있다는 걸 알 수 있습니다.
예시가 간단해서 함수 타입을 선언할 수 있다는 것이 장점으로 와닿지 않을 수 있습니다.
함수 타입 선언의 장점을 좀 더 알아보겠습니다.

함수 타입의 선언은 불필요한 코드의 반복을 줄입니다. 사칙 연산을 하는 함수 네 개는 다음과 같이 작성할 수 있습니다.

```ts
function add(a: number, b: number) {return a + b;}
function sub(a: number, b: number) {return a - b;}
function mul(a: number, b: number) {return a * b;}
function div(a: number, b: number) {return a / b;}
```

반복되는 함수 시그니처를 하나의 함수 타입으로 통합할 수도 있습니다.

```ts
type BinaryFn = (a: number, b: number) => number;
const add : BinaryFn = (a, b) => a + b;
const sub : BinaryFn = (a, b) => a - b;
const mul : BinaryFn = (a, b) => a * b;
const div : BinaryFn = (a, b) => a / b;
```

이 예제는 함수 타입 선언을 이용했단 예제보다 타입 구문이 적습니다. 함수 구현부도 분리 되어 있어 로직이 보다 분명해집니다.
모든 함수 표현식의 반환 타입까지 number로 선언한 셈입니다.

라이브러리는 공통 함수 시그니처를 타입으로 제공하기도 합니다.
예를 들어, 리액트는 함수의 매개변수에 명시하는 MouseEvent 타입 대신에 함수 전체에 적용할 수 있는 MouseEventHandler 타입을 제공합니다.
만약 여러분이 라이브러리를 직접 만들고 있다면, 공통 콜백 함수를 위한 타입 선언을 제공하는 것이 좋습니다.

시그니처가 일치하는 다른 함수가 있을 때도 함수 표현식에 타입을 적용해볼 만합니다.
예를 들어, 웹브라우저에서 fetch함수는 특정 리소스에 HTTP 요청을 보냅니다.

```ts
const responseP = fetch('/quote?by=Mark+Twain'); // 타입이 Promise<Response>
```

그리고 `response.json()` 또는 `response.text()`를 사용해 응답의 데이터를 추출합니다.

```ts
async function getQuote(){
  const response = fetch('/quote?by=Mark+Twain');
  const quote = waite response.json(); 
  return quote;
}

// {
//     "quote":"If you tell me the truth, you don't have to remember anything.",
//     "source":"notebook",
//     "date":"1894",
// }
```

(Promise와 async/await에 대해서는 [아이템 25](https://github.com/Pyotato/effective_typescript/blob/item25/README.md)에서 다룹니다.)

여기에 버그가 있습니다. `/quote`가 존재하지 않는 API라면, '404 Not Found'가 포함된 내용을 응답합니다.
응답은 JSON 형식이 아닐 수 있습니다.
`response.json()`은 JSON 형식이 아니라는 새로운 오류 메시지를 담아 거절된 (rejected) 프로미스르 반환합니다.
호출한 곳에서는 새로운 오류 메시지가 전달되어 실제 오류인 404가 감추어집니다.

또한 fetch가 실패하면 거절된 프로미스를 응답하지는 않는다는 걸 간과하기 쉽습니다.
그러니 상태 체크를 수행해줄 `checkedFetch`함수를 작성해서 작성해 보겠습니다.
fetch의 타입 선언은 <i>lib.dom.ts</i>에 있으며 다음과 같습니다.

```ts
declare function fetch(
  input: RequestInfo, init?: RequestInit
): Promise <Response>;
```

`checkedFetch`는 다음처럼 작성할 수 있습니다.

```ts
async function checkedFetch(input: RequestInfo, init?: RequestInit){
  const response = await fetch(input, init);
  if(!response.ok){
    // 비동기 함수 내에서는 거절된 프로미스로 변환합니다.
   throw new Error('Request Failed:' + response.status); 
  }
  return response;
}
```

이 코드도 잘 동작하지만, 다음과 같이 더 간결하게 작성할 수 있습니다.

```ts
const checkedFetch : typeof fetch = async (input,init) =>{
  const response = await fetch(input, init);
  if(!response.ok){
    throw new Error('Request Failed:' + response.status); 
  }
  return response;
}
```

함수 문장을 함수 표현식으로 바꿨고 함수 전체에 타입 (typeof fetch)을 적용했습니다.
이는 타입스크립트가 input과 init의 타입을 추론할 수 있게 해 줍니다.

타입 구문은 또한 checkedFetch의 반환 타입을 보장하며, fetch와 동일합니다.
예를 들어 throw 대신 return을 사용했다면, 타입스크립트는 그 실수를 잡아냅니다.

<img width="1658" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8a336a74-0583-4419-a422-a4467d747235"/>

checkedFetch를 함수 문장으로 잦ㄱ성한 예제에서도 throw가 아니라 return을 사용할 경우 오류가 발생합니다.
그러나 오류는 첫 번째 예제와 달리 checkedFetch 구현체가 아닌, 함수를 호출한 위치에서 발생합니다.

함수의 매개변수에 타입 선언을 하는 것보다 함수 표현식 전체 타입을 정의하는 것이 코드도 간결하고 안전합니다.
다른 함수의 시그니처와 동일한 타입을 가지는 새 함수를 작성하거나, 동일한 타입 시그니처를 가지는 여러 개의 함수를 작성할 때는 매개변수의 타입과 반환 타입을 반복해서 작성하지 말고 함수 전체의 타입 선언을 적용해야 합니다.
