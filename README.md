# 💓 ITEM 22 : 타입 좁히기

타입 넓히기의 반대는 타입 좁히기입니다. 타입 좁히기는 타입스크립트가 넣은 타입으로부터 좁은 타입으로 진행하는 과정을 말합니다.
아마도 가장 일반적인 예시는 null 체크일 겁니다.

```ts
const el = document.getElementById('foo');
if(el){
  el.innerHTML = 'Party Time'.blink();
} else {
  alert('No element #foo');
}
```

<img width="310" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/e018cf27-0051-4187-9520-fa8c6134df85"/>
<img width="310" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1643de70-1bef-44e0-af43-f5f37b98ebd0"/>
<img width="310" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4f87e1c6-a0d4-4df3-b69f-852f9e8f5b0a"/>

만약 el이 null이라면, 분기문의 첫 번째 블록이 실행되지 않습니다.
즉, 첫 번째 블록에서 `HTMLElement | null` 타입의 null을 제외하므로, 더 좁은 타입이 되어 작업이 훨씬 쉬워집니다.
타입 체커는 일반적으로 이러한 조건문에서 타입 좁히기를 잘 해내지만, 타입 별칭이 존재한다면 그러지 못할 수도 있습니다. 타입 별칭에 대한 내용은 [아이템 24](https://github.com/Pyotato/effective_typescript/blob/item24/README.md)에서 다루겠습니다.

분기문에서 예외를 던지거나 함수를 반환하여 블록의 나머지 부분에서 변수의 타입을 좁힐 수도 있습니다. 예를 들어 보겠습니다.

```ts
const el = document.getElementById('foo');
if(!el) throw new Error('Unable to find #foo');
el;
el.innerHTML = 'Party Time'.blink();
```

<img width="354" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/e34a8de9-a7f9-4e30-a79a-ebfca1debe26"/>

이 외에도 타입을 좁히는 방법은 많이 있습니다. 다음은 `instanceof`를 사용해서 타입을 좁히는 예제입니다.

```ts
function contains(text: string, search: string | RegExp){
  if(search instanceof RegExp){
    search // 타입이 RegExp
    return !!search.exec(text); 
 }
  search // 타입이 string
  return text.includes(search);
}
```

속성 체크로도 타입을 좁힐 수 있습니다. 

```ts
interface A {a: number}
interface B {b: number}
function pickAB(ab: A | B){
  if('a' in ab){
    ab // 타입이 A
 } else {
    ab // 타입이 B
  }
  ab // 타입이 A | B
}
```

`Array.isArray` 같은 일부 내장 함수로도 타입을 좁힐 수 있습니다.

```ts
function contains(text: string, terms: string | string[]){
  const termList = Arra.isArray(terms)? terms: [terms];
  termList // 타입이 string[]
}
```

타입스크립트는 일반적으로 조건문에서 타입을 좁히는 데 매우 능숙합니다.
그러나 타입을 섣불리 판단하는 실수를 저지르기 쉬우믈 다시 한번 꼼꼼히 따져 봐야 합니다.
예를 들어, 다음 예제는 유니온 타입에서 null을 제외하기 위해 잘못된 방법을 사용했습니다.

```ts
const el = document.getElementById('foo'); // 타입이 HTMLElement | null
if(typeof el === 'object'){
  el; // 타입이 HTMLElement | null
}
```

자바스크립트에서 typeof null이 "object"이기 때문에, if 구문에서 null이 제외되지 않습니다.
또한 기본형 값이 잘못되어도 비슷한 사례가 발생합니다.

```ts
function foo(x?: number | string | null){
  if(!x){
    x; // 타입이 number | string | null | undefined
  }
}
```

빈 문자열과 ''과 0 모두 false가 되기 때문에, 타입은 전혀 좁혀지지 않았고 x는 여전히 블록 내에서 string 또는 number가 됩니다.

타입을 좁히는 또 다른 일반적인 방법은 명시적 '태그'를 붙이는 것입니다.

```ts
interface UploadEvent {type: 'upload'; filename: string; contents: string}
interface DownloadEvenet {type: 'download'; filename: string;}
type AppEvent = UploadEvent | DownloadEvent;
funciton handleEvent(e: AppEvent){
  switch(e.type){
    case 'download':
      e // xkdlqdl DownloadEvent
    case 'upload':
      e; // xkdlqdl UploadEvent
      break;
  }
}
```

이 패턴은 '태그된 유니온(tagged union)' 또는 '구별된 유니온(discriminated union)'이라고 불리며, 타입스크립트 어디에서나 찾아볼 수 있습니다.

만약 타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기 위해 커스텀 함수를 도입할 수 있습니다.

```ts
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement){
  if(isInputElement(el)){
    el; // 타입이 HTMLInputElement
    return el.value;
  }
  el; // 타입이 HTMLElement
  return el.textContent; 
}

```

이러한 기법을 '사용자 정의 타입 가드'라고 합니다. 반환 타입의 el is HTMLInputElement는 함수의 반환이 true인 경우, 타입 체커에게 매개변수의 타입을 좁힐 수 있다고 알려줍니다.

어떤 함수들은 타입 가드를 사용하여 배열과 객체의 타입 좁히기를 할 수 있습니다.
예를 들어, 배열에서 어떤 탐색을 수행할 때 undefined가 될 수 있는 타입을 사용할 수 있스빈다. 

```ts
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
);   // 타입이 (string | undefined)[]
```

filter함수를 사용해 undefined를 걸러 내려고 해도 잘 동작하지 않을 겁니다.

```ts
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
).filter(who => who !== undefined);   // 타입이 (string | undefined)[]
```

이럴 때 타입 가드를 사용하면 타입을 좁힐 수 있습니다. 

```ts
function isDefined<T>(x:T | undefined): x is T{
  return x !== undefined;
}
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
).filter(isDefined); // 타입이 string[];
```

<img width="510" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/a0eebac6-9f29-409c-ad0d-bd3af609a639"/>

편집기에서 타입을 조사하는 습관을 가지면 타입 좁히기가 어떻게 동작하는지 자연스레 익힐 수 있습니다.
타입스크립트에서 타입이 어ㄸ허게 좁혀지는지 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알수 있으며, 
타입 체커를 더 효율적으로 이용할 수 있습니다.










