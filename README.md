#  🪄 ITEM 41 : any의 진화를 이해하기

타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정됩니다.
그 후에 정제 될 수 있지만 (예를 들어 null인지 체크해서), 새로운 값이 추가되도록 확장할 수는 없습니다. 
그러나 any 타입과 관련해서 예외인 경우가 존재합니다.

자바스크립트에서 일정 범위의 숫자들을 생성하는 함수를 예로 들어 보겠습니다. 

```js
function range(start, limit){
  const out = [];
  for(let i=start; i<limit; i++){
    out.push(i);
  }
  return out;
}

```

이 코드를 타입스크립트로 변환하면 예상대로 정확히 동작합니다. 

```ts
function range(start: number, limit: number){
  const out = [];
  for(let i=start; i<limit; i++){
    out.push(i);
  }
  return out;  // 반환 타입이 number[]로 추론
}

```

그러나 자세히 살펴보면 한 가지 이상한 점을 발견할 수 있습니다.

out의 타입이 처음에는 any타입 배열인 `[]`로 초기화되었는데, 마지막에는 `number[]`로 추론되고 있습니다.

코드에 out이 등장하는 세 가지 위치를 조사해 보면 이유를 알 수 있습니다.

```ts
function range(start: number, limit: number){
  const out = [];                                  // 타입이 any[]
  for(let i=start; i<limit; i++){
    out.push(i);                                  // out의 타입이 any[]
  }  
  return out;                                    // 타입이 number[]
}

```

out의 타입은 `any[]`로 선언되었지만 number 타입의 값을 넣는 순간부터 타입은 number[]로 진화(evolve)합니다.

타입의 진화는 타입 좁히기 ([아이템 22](https://github.com/Pyotato/effective_typescript/blob/item22/README.md))와 다릅니다.
배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화합니다.

```ts
const result = [];       // 타입이 any[]
result.push('a');
result                  // 타입이 string[]
result.push(1);
result;                 // 타입이 (string | number)[]
```

또한 조건문에서는 분기에 따라 타입이 변할 수도 있습니다.
다음 코드에서는 배열이 아닌 단순 값으로 예를 들어 보겠습니다.

```ts
let val ;                   // 타입이 any
if(Math.random() < 0.5){
  val = /hello/;
  val                      // 타입이 RegExp
} else {
  val = 12;  
  val                       // 타입이 number
]
val                         // 타입이 number | RegExp
```

변수의 초깃값이 null인 경우도 any의 진화가 일어납니다.
보통은 try/catch블록 안에서 변수를 할당하는 경우에 나타납니다. 

```ts
let val ;                 // 타입이 any    
try {
  somethingDangerous();
  val = 12;
  val                     // 타입이 number
} catch(e) {
  console.warn('alas');                      
]
val                         // 타입이 number | null
```

any 타입의 진화는 noImplicitAny가 설정된 상태에서 변수의 타입이 암시적인 any인 경우에만 일어납니다.
그러나 다음처럼 명시적으로 any를 선언하면 타입이 그대로 유지됩니다. 

```ts
let val: any;                 // 타입이 any    
if(Math.random() < 0.5){
  val = /hello/;
  val                      // 타입이 any
} else {
  val = 12;  
  val                       // 타입이 any
]
val                         // 타입이 any
```

> ✅ 타입의 진화는 값을 할당하거나 배열에 요소를 넣은 후에만 일어나기 때문에, 편집기에서는 이상하게 보일 수 있습니다.
> 할당이 일어난 줄의 타입을 조사해봐도 여전히 any 또는 any[]로 보일 겁니다.

다음 코드처럼, 암시적 any상태인 변수에 어떠한 할당도 하지 않고 사용하려고 하면 암시적 any 오류가 발생하게 됩니다. 

```ts
function range(start: number, limit: number){
  const out = [];
  if(start === limit) return out;
  for(let i = start; i< limit; i++){
    out.push(i);
  }
  return out;
}
```

<img width="1532" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/26ce17dd-fa98-4765-8b83-c1727893822f"/>

any타입의 진화는 암시적 any 타입에 어떤 값을 할당할 때만 발생합니다.
그리고 어떤 변수가 암시적 any 상태일 때 값을 읽으려고 하면 오류가 발생합니다.

암시적 any 타입은 함수 호출을 거쳐도 진화하지 않습니다.
다음 코드에서 forEach안의 화살표 함수는 추론에 영향을 미치지 않습니다. 

```ts
function makeSquares(start: number, limit: number){
  const out = [];
  //    ~~~ 'out' 변수는 일부 위치에서 암시적으로 'any[]' 형식입니다.
  range(startmlimit.forEach(i=>{
    out.push(i*i);
  ))
  return out;
   //    ~~~ 'out' 변수는 일부 위치에서 암시적으로 'any[]' 형식이 포합됩니다.
}
```

앞의 코드와 같은 경우라면 루프로 순회하는 대신, 배열의 map과 filter 메서드를 통해 단일 구문으로 배열을 생성하여 any 전체를 진화시키는 방법을 새악해 볼 수 있습니다. 
[아이템 23](https://github.com/Pyotato/effective_typescript/blob/item23/README.md)과 [27](https://github.com/Pyotato/effective_typescript/blob/item27/README.md)을 참고하길 바랍니다.


any가 진화하는 방식은 일반적인 변수가 추론되는 원리와 동일합니다.
예를 들어, 진화한 배열의 타입이 `(string|number)[]`라면, 원래 `number[]`타입이어야 하지만 실수로 string이 섞여서 잘못 진화한 것일 수 있습니다.
타입을 안전하게 지키기 위해서는 암시적으로 any를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 더 좋은 설계입니다.







