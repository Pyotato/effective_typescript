#  📄 ITEM 48 : API 주석에 TSDoc 사용하기

다음 코드는 인사말을 생성하는 타입스크립트 함수입니다.

```ts
// 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다.
function greet(name: string, title: string){
  return `Hello ${title} {name}`;
}
```

함수의 앞부분에 주석이 있어서 함수가 어떤 기능을 하는지 쉽게 알 수는 있습니다.
그러나 사용자를 위한 문서라면 JSDoc 스타일의 주석으로 만든는 것이 좋습니다.

```ts
/** 인사말을 생성합니다. 결과는 보기 좋게 꾸며집니다. */
function greetJSDoc(name: string, title: string){
  return `Hello ${title} ${name}`;
}
```

왜냐하면 대부분의 편집기는 함수가 호출되는 곳에서 함수에 붙어 있는 JSDoc 스타일의 주석을 툴팁으로 표시해 주기 때문입니다.

<img width="417" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/a08e2ef7-e6e6-4e4e-838c-9e4e3f45302a"/>

그러나 인라인(inline)주석은 편집기가 표시해주지 않습니다.

<img width="417" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/75022686-e55f-4096-9726-80f5c48ac099"/>

타입스크립트 언어 서비스가 JSDoc 스타일을 지원하기 때문에 적극적으로 활용하는 것이 좋습니다.
만약 공개 API에 주석을 붙인다면 JSDoc 형태로 작성해야 합니다.
JSDoc에는 `@param`과 `@returns`같은 일반적 규칙을 사용할 수 있습니다.
한편 타입스크립트 관점에서는 TSDoc이라고 부르기도 합니다.

```ts
/** 인사말을 생성합니다. 
 * @param name 인사할 사람의 이름
 * @param title 인사할 사람의 칭호
 * @returns 보기 좋은 형태의 인사말 */
function greetJSDoc(name: string, title: string){
  return `Hello ${title} ${name}`;
}
```

`@param`와 `@returns`를 추가하면 함수를 호출하는 부분에서 각 매개변수와 관련된 설명을 보여줍니다.

<img width="537" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6a0e4466-c104-45c0-8104-f4d0fa3b799b"/>

타입 정의에서 TSDoc을 사용할 수도 있습니다.

```ts
/** 특정 시간과 장소에 수행된 측정 */
interface Measurement{
    /** 어디에서 측정되었나? */
  position : Vector3D;
  /** 언제 측정되었나? epoch에서부터 초 단위로 */  
  time: number;
  /** 측정된 운동량 */
  momentum: Vector3D;
}
```

Measurement 객체의 각 필드에 마우스를 올려 보면 필드별로 설명을 볼 수 있습니다.

<img width="352" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/12a61934-7b01-480d-bd3f-57142cd5be27"/>

TSDoc 주석은 마크다운 (markdown) 형식으로 꾸며지므로 굵은 글씨, 기울임 글씨, 글머리 기호 목록을 사용할 수 있습니다.

```ts
/**
 * 이 _interface_는 **세 가지** 속성을 가집니다.
 *  1. x
 *  2. y
 *  3. z
 */
interface Vector3D{
  x: number;
  y: number;
  z: number;
}
```

<img width="368" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9f497bc5-4178-48ec-9f9c-0c691474cd01"/>

주석을 수필처럼 장황하게 쓰지 않도록 주의해야 합니다. 훌륭한 주석은 간단히 요점만 언급합니다.

JSDoc에는 타입 정보를 명시하는 규칙 (`@param {string} name ...`)이 있지만, 타입스크립트에서는 타입 정보가 코드에 있기 때문에 TSDoc에서는 타입 정보를 명시하면 안됩니다. ([아이템 30](https://github.com/Pyotato/effective_typescript/blob/item30/README.md))



























