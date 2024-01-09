# ⛓️ ITEM 54 :객체를 순회하는 노하우

다음 예쩨는 정상적으로 실행되지만, 편집기에서는 오류가 발생합니다. `오류의 원인은 무엇일까요?`

```ts
const obj = {
  one: "uno",
  two: "dos",
  three: "tres",
};

for (const k in obj) {
  const v = obj[k]; // 오류!
}
```

코드를 수정해 가며 원인을 찾다보면 obj 객체를 순회하는 루프내의 상수k와 관련된 오류라는 것을 알 수 있습니다.

![image](https://github.com/Pyotato/effective_typescript/assets/102423086/939aa468-d26a-4d33-a347-75f1e6903533)

k의 탑인은 string인 반면, obj 객체에는 'one', 'two' , 'three' 세 개의 키만 존재합니다. **k와 obj 객체의 키 타입이 서로 다르게 추론**되어 오류가 발생한 것입니다.

```ts
const obj = {
  one: "uno",
  two: "dos",
  three: "tres",
};

let k: keyof typeof obj; // k의 타입을 더욱 구체적으로 명시해주면 오류가 사라집니다.

for (k in obj) {
  const v = obj[k];
}
```

`오류의 원인은 무엇일까요?`을 좀 더 구체적으로 바꿔 보겠습니다. 첫 번째 예쩨의 k타입이 `'one'| 'two' | 'three'`가 아닌 string으로 추론된 원인은 무엇일까요?

이해를 돕기 위해, 인터페이스와 함수가 가미된 다른 예제를 보겠습니다.

```ts
interface ABC {
  a: string;
  b: string;
  c: string;
}
function foo(abc: ABC) {
  for (const k in abc) {
    // const k:string;
    const v = abc[k]; // 오류!
  }
}
```

![image](https://github.com/Pyotato/effective_typescript/assets/102423086/40841f90-6229-4dbf-b820-7b50cdd38788)

첫 번째 예제와 동일한 오류입니다. 그러므로 (let k: keyof ABC)같은 선언으로 오류를 제거할 수 있습니다.
오류의 내용이 잘못된 거처럼 보이지만, 실제 오류가 맞고 또한 타입스크립트가 정확히 오류를 표시했습니다. 제대로 된 오류인 이유를 예로 들어 설명하겠습니다.

```ts
  const x = {a: 'a', b: 'b', c:2, d: new Date()};
  foo(x);
```

foo함수는 a,b,c속성 외에 d를 가지는 x객체로 호출이 가능합니다. foo함수는 ABC타입에 '할당 가능한' 어떤 값이든 매개변수로 허용하기 때문입니다 ([아이템 4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md)).
즉, ABC타입에 할당 가능한 객체에는 a,b,c 외에 다른 속성이 존재할 수 있기 때문에, 타입스크립트는 ABC타입의 키를 string 타입으로 선택해야합니다.

또한 keyof 키워드를 사용한 방법은 또 다른 문제점을 내포하고 있습니다. 

```ts
  function foo(abc:ABC){
    let k: keyof ABC;
    for(k in abc){
      const v = abc[k]; //string | number 타입
    }
  }

```

k가 `'a'| 'b' | 'c'`타입으로 한정되어 문제가 된 것처럼, v도 `string | number` 타입으로 한정되어 범위가 너무 좁아 문제가 됩니다. 
`d: new Date()`가 있는 이전 예제처럼, d 속성은 Date타입뿐만 아니라 어떠한 타입이든 될 수 있기 때문에 v가 `string | number` 타입으로 추론된 것은 잘못이며 런타임의 동작을 예상하기 어렵습니다.

골치 아픈 타입 문제 없이, 단지 객체의 키와 값을 순회하고 싶다면 어떻게 해야할까요? `Object.entries`를 사용하면 됩니다. (타입스크립트 3.8기준으로 표준함수가 아니며, tsconfig.json에 ES8설정을 추가하여 사용할 수 있습니다.)

```ts
  function foo(abc:ABC){
    for(const [k,v] of Object.entries(abc)){
      k // string 타입
      v // any 타입
    }
  }

```

Object.entries를 사용한 루프가 직관적이지는 않지만, 복잡한 기교없이 사용할 수 있습니다.

한편, 객체를 다룰 때에는 항상 **'프로토타입 오염'**의 가능성을 염두에 두어야합니다. `for-in`구문을 사용하면 객체의 정의에 없는 속성이 갑자기 등장할 수 있습니다.

![image](https://github.com/Pyotato/effective_typescript/assets/102423086/e9c1d119-302d-47db-bcc0-f1be19556d44)

실제 작업에서는 Object.prototype에 순회 가능한 속성을 절대로 추가하면 안됩니다. 
for-in 루프에서 k가 string 키를 가지게 된다면 프로토타입 오염의 가능성을 의심해 봐야합니다. 

객체를 순회하며 키와 값을 얻으려면, `(let k : keyof T)`같은  `keyof 선언`이나 `Object.entries`를 사용하면 됩니다. 
`keyof 선언`은 `상수이거나 추가적인 키 없이 정확한 타입`을 원하는 경우에 적절합니다.
`Object.entries`는 더욱 일반적으로 쓰이지만, 키와 값의 타입을 다루기 까다롭습니다.

