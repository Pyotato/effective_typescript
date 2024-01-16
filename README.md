# 📝 ITEM 6 : 편집기를 사용하여 타입 시스템 탐색하기

타입스크립트를 설치하면, 다음 두 가지를 실행할 수 있습니다.

- 타입스크립트 컴파일러 (tsc)
- 단독으로 실행할 수 있는 타입스크립트 서버(tsserver)

보통은 타입스크립트 컴파일러를 실행하는 것이 주도니 목적이지만, 타입스크립트 서버 또한 '언어 서비스'를 제공한다는 점에서 중요합니다.
언어 서비스에는 코드 자동 완성, 명세 (사양, specification) 검사, 검색, 리펙터링이 포함됩니다.
보통은 편집기를 통해서 언어 서비스를 사용하는데, 타입스크립트 서버에서 언어 서비스를 제공하도록 설정하는 게 좋습니다. 
유용한 기능이니 꼭 사용하도록 합시다.
자동 완성 같은 서비스를 사용하면 타입스크립트 코드 작성이 간편해집니다. 이런 서비스들을 차치하더라도, 편집기는 코드를 빌드하고 타입 시스템을 익힐 수 있는 최고의 수단입니다.
그리고 편집기는 타입스크립트가 언제 타입 추론을 수행할 수 있는 지에 대한 개념을 잡게 해주는데, 이 개념을 확실히 잡아야 간결하고 읽기 쉬운 코드를 작성할 수 있습니다 ([아이템 9](https://github.com/Pyotato/effective_typescript/blob/item9/README.md)).

편집기마다 조금씩 다르지만 보통의 경우에는 심벌 위에 마우스 커서를 대면 타입스크립트가 그 타입을 어떻게 판단하고 있는 지 확인할 수 있습니다. 

<img width="162" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/da825b60-e3f1-4ecb-bafb-e6db5b067bbc"/>

num 변수의 타입을 number라고 직접 지정하지 않았지만, 타입스크립트는 10이라는 값을 보고 그 타입을 알아냅니다.

다음고 같이 함수의 타입도 추론할 수 있습니다.

<img width="376" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f0848ac5-00ec-4fec-a5b8-b811710a7bb1"/>

여기서 주목할 점은 추론된 함수의 반환 타입이 number라는 것입니다.

이 타입이 기대한 것과 다르다면 타입 선언을 직접 명시하고, 실제 문제가 발생하는 부분을 찾아 봐야 합니다.

특정 시점에서 타입스크립트가 값의 타입을 어떻게 이해하고 있는지 살펴보는 것은 타입 넓히기 ([아이템 21](https://github.com/Pyotato/effective_typescript/blob/item21/README.md))와 좁히기([아이템 22](https://github.com/Pyotato/effective_typescript/blob/item22/README.md))의 개념을 잡기 위해 꼭 필요한 과정입니다.
조건문의 분기에서 값의 타입이 어떻게 변하는지 살펴보는 것은 타입 시스템을 연마하는 매우 좋은 방법입니다.

<img width="314" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/a161606d-5408-4937-afe1-600371bad0a2"/>

객체에서는 개별 속성을 알아봄으로써 타입스크립트가 어떻게 각각의 속성을 추론하는지 살펴볼 수 있습니다.

```ts
const foo = {
  x: [1,2,3],
  bar: {
    name: 'Fred'
  }
}
```

<img width="197" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9bcc6989-60cc-4116-a804-2b77c3904a7f"/>

만약 x가 튜플 타입 `([number, number, number])`이어야 한다면, 타입 구문을 명시해야 합니다.

연산자 체인 (chain) 중간의 추론된 제너릭 타입을 아록 싶다면, 메서드 이름을 조사하면 됩니다.

<img width="676" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/b618719f-9d82-403e-99c0-4de192622a9f"/>

위의 `Array<string>`은 split 결과의 타입이 string이라고 추론되었음을 의미합니다. 
이번 예제는 간단하기 때문에 추론 정보가 없어도 되지만, 실제 코드에서 함수 호출이 길게 이어진다면 추론 정보는 디버깅하느 데 꼭 필요합니다. 

편집기상의 타입 오류를 살펴보는 것도 타입 시스템의 성향을 파악하는 데 좋은 방법입니다. 예를 들어, 다음은 id에 해당하거나 기본값인 HTMLElement를 반환하는 함수입니다.
타입스크립트는 두 곳에서 오류를 발생시킵니다.

```ts
function getElement(elOrId: string | HTMLElement | null): HTMLElement{
  if(typeof elOrId === 'object'){
    return elOrId;
  } else if(elOrId === null){
    return document.body;
  } else {
    const el = document.getElementById(elOrId);
    return el;
  }
}
```

<img width="1241" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/8da5a7a4-56c1-4b55-b6d4-4b96018c1cad"/>

첫번째 if분기문의 의도는 단지 HTMLElement라는 객체를 골라내느 것이었습니다. 그러나 자바스크립트에서 `typeof null`은 "object"이므로, elOrId는 여전히 분기문 내에서 null일 가능성이 있습니다.
그러므로 처음에 null 체크를 추가해서 바로 잢습니다.

두번째 오류는 document.getElementById가 null을 반환할 가능성이 있어서 발생했고, 첫번째 오류와 동일하게 null 체크를 추가하고 예외를 던져야 합니다.

언어 서비스는 라이브러리와 라이브러리의 타입 선언을 탐색할 때 도움이 됩니다.
코드 내에서 fetch함수가 호출되고, 이 함수를 더 알아보길 원한다고 가정해 보겠습니다.
편집기는 'Go to Definition(정의로 이동)' 옵션을 제공합니다. 다음과 같은 모습니다.

<img width="573" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/7fb4fc36-6276-43f0-a1fd-4821188ae355"/>

이 옵션을 선택하면 타입스크립트에 포함되어 있는 DOM 타입 선언인 <i>lib.dom.d.ts</i>로 이동합니다.

```ts
declare function fetch(
  input: RequestInfo, init?: RequestInit
): Promise<Response>;
```

fetch가 Promise를 반환하고 두 개의 매개변수를 받는 것을 볼 수 있습니다.
RequestInfo를 클릭하면 다음으로 이동합니다.

```ts
declare var Request {
  prototype: Request;
  new(input: RequestInfo, init?: RequestInit): Request;
};
```

여기서 Request 타입과 값은 분리되어 모델링되어 있습니다 ([아이템 8](https://github.com/Pyotato/effective_typescript/blob/item8/README.md)). RequestInfo는 이미 살펴보았고, RequestInit를 클릭하면 Request를 생성할 때 사용할 수 있는 모든 옵션이 나타납니다.

```ts
interface RequestInit{
  body?: BodyInit | null;
  cache?: RequestCache;
  credentials?: RequestCredentials;
  headers?: HeadersInit;
  // ....
}
```

<i>lib.dom.d.ts</i>에서 더 많은 타입을 탐색하다 보면, 다음과 같은 결론을 얻게 됩니다. 
타입 선언은 처음에는 이해하기 어렵지만 타입스크립트가 무엇을 하는지, 
어떻게 라이브러리가 모델링되었는지,
어떻게 오류를 찾아낼지 살펴볼 수 있는 훌륭한 수단이라는 것을 알 수 있습니다.
타입 선언에 관해서는 6장에서 더 깊게 다룹니다.
























  
