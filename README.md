# 🧚 ITEM 39 : any를 구체적으로 변형해서 사용하기

any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입입니다.
any 타입에는 모든 숫자, 문자열, 배열, 객체, 정규실, 함수, 클래스, DOM 엘리먼트는 물론 null과 undefined까지도 포함됩니다.
반대로 말하면, 일반적인 상황에서는 any보다 더 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높기 때문에 더 구체적인 타입을 찾아 타입 안정성르 높이도록 해야합니다.

예를 들어, any 타입의 값을 그대로 정규식이나 함수에 넣는 것을 권장하지 않습니다.

```ts
function getLengthBad(array: any){
  return array.length;
}

function getLength(array: any[]){
  return array.length;
}
```

앞의 예제에서 any를 사용하는 getLengthBad보다는 any[]를 사용하는 getLength가 더 좋은 함수 입니다. 
그 이유는 세 가지입니다.

- 함수 내의 array.length 타입이 체크됩니다.
- 함수의 반환 타입이 any 대신 number로 추론됩니다.
- 함수 호출될 때 매개변수가 배열인지 체크합니다.

배열이 아닌 값을 넣어 실행해 보면, getLength는 제대로 오류를 표시하지만 getLengthBad는 오류를 잡아내지 못하는 것을 볼 수 있습니다.

<img width="841" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4df403f5-2413-4e61-b8a3-4a6dcdff0407"/>

함수의 매개변수를 구체화 할 때, (요소의 타입에 관계없이) 배열의 배열 형태라면 `any[][]`처럼 선언하면 됩니다.
그리고 함수의 매개변수가 객체이긴 하지만 값을 알 수 없다면 `{[key: string]: any}`처럼 선언하면 됩니다.

```ts
function hasTwelveLetterkey(o: {[key: string]: any}){
  for(const key in o){
    if(key.length === 12){
      return true;
    }
  }
  return false;
}
```

위의 예제처럼 함수의 매개변수가 객체지만 값을 알수 없다면 `{[key: string]: any}` 대신 모든 비기본형(non-primitive) 타입을 포함하는 object타입을 사용할 수도 있습니다.
object 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서 `{[key: string]: any}`와 약간 다릅니다.  

```ts
function hasTwelveLetterkey(o: object){
  for(const key in o){
    if(key.length === 12){
      console.log(key. o[key]);
      return true;
    }
  }
  return false;
}
```

<img width="588" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0c664b79-7d2a-420c-a7bc-7e0f13a7161c">

객체지만 속성에 접근할 수 없어야 한다면 unknown 타입이 필요한 상황일 수 있ㅅ브니다.
unknown 타입은 [아이템 42](https://github.com/Pyotato/effective_typescript/blob/item42/README.md)에서 다루겠습니다.

함수의 타입에도 단순히 any를 사용해서는 안 됩니다.
최소한으로나마 구체화할 수 있는 세 가지 방법이 있습니다.

```ts
type Fn0 = () => any;                  // 매개변수 없이 호출 가능한 모든 함수
type Fn1 = (args: any) => any;         // 매개변수 1개
type Fn2 = (...args: any[]) => any;    // 모든 개수의 매개변수 "Function" 타입과 동일
```

위의 예제에 등장한 세 가지 함수 타입 모두 anyㅂㅎ더눈 구체적이빈다.
마지막 줄을 잘 살펴보면 `...args`의 타입을 any[]로 선언했습니다. 

> `...`문법은 ES6에 도입된 나머지 매개변수 (rest parameter)와 전개 연산자 (spread operator)입니다.
> 타입스크립트는 ES6문법을 포함하므로 사용 가능합니다.

any로 선언해도 동작하지만 any[]로 선언하면 배열 형태라는 것을 알 수 있어 더 구체적입니다.

```ts
const numArgsBad = (...args: any) -> args.length                 // any 반환
const numArgsGood = (...args: any[]) -> args.length              // number 반환 
```

이 예제가 any[] 타입을 사용하는 가장 일반적인 경우입니다.
