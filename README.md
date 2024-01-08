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
오류의 내용이 잘못된 거처럼 보이지만, 실제 오류가 맞고 또한 타입스크립트가 정확히 오류를 표시했스빈다. 제대로 된 오류인 이유를 예로 들어 설명하겠습니다.
