# 🎒 ITEM 59: 타입스크립트 도입 전에 @ts-check의 JSDoc으로 시험해 보기

본격적으로 타입스크립트로 전환하기([아이템 60](https://github.com/Pyotato/effective_typescript/blob/item60/README.md))에 앞서, `@ts-check`지시자를 사용하면 타입스크립트 전환시에 어떤 문제가 발생하는지 미리 시험해 볼 수 있습니다.
`@ts-check` 지시자를 사용하여 타입 체커가 파일을 분석하고, 발견된 오류를 보고하도록 지시합니다.
그러나 `@ts-check` 지시자는 매우 느슨한 수준으로 타입 체크를 수행하는데, 심지어 `noImplicitAny` 설정을 해제한 것보다 헐거운 체크를 수행한다는 점을 주의해야 합니다 ([아이템 2](https://github.com/Pyotato/effective_typescript/blob/item2/README.md)). 

```js
// @ts-check
const person = {first: 'Grace', last:'Hopper'};
2*person.first;
```

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9bb0b3db-15b0-4fb6-b6c0-aeba2c540eb2"/>

`person.first`의 타입은 string으로 추론되었고, `2*person.first`는 타입 불일치 오류가 되었습니다.
`@ts-check` 지시자를 사용하면 타입 불일치나 함수의 매개변수 개수 불일치같은 간단한 오류 외에도, 다음에 소개하는 몇 가지 의미 있는 오류들을 찾아낼 수 있습니다.

## 선언되지 않은 전역 변수 

변수를 선언할 때 보통 `let`이나 `const`를 사용합니다.
그러나 어딘가에 '숨어 있는' 변수 (예를 들면, HTML파일내의 `<script>`태그)라먄. 변수를 제대로 인식할 수 있게 별도로 타입 선언 파일을 만들어야 합니다.

```js
// @ts-check
console.log(user.firstName);
```

<img width="540" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6411c5ca-e9ec-4d4e-a08a-feaafd5d2416"/>

user 선언을 위해 <i>types.d.ts</i> 파일을 만들어 보겠습니다.

```ts
interface UserData{
  firstName: string;
  lastName: string;
}
declare let user : UserData;
```

타입 선언 파일을 만들면 오류가 해결됩니다.
선언 파일을 찾지 못하는 경우는 트리플 슬래시 `///` 참조를 사용하여 명시적으로 임포트를 할 수 있습니다.

```js
// @ts-check
/// <reference path='./types.d.ts'/>
console.log(user.firstName);  // 정상
```

앞에서 작성한 <i>types.d.ts</i> 파일은 프로젝트 타입 선언의 초석이 되는 중요한 파일입니다.

## 알 수 없는 라이브러리

서드파티 라이브러리를 사용하는 경우, 서드파티 라이브러리의 타입 정보를 알아야 합니다.
예를 들어, 제이쿼리를 사용하여 HTML 엘리먼트의 크기를 설정하는 코드에서 `@ts-check` 지시자를 사용하면 제이쿼리를 사용한 부분에서 오류가 발생합니다.

```js
// @ts-check
$('#graph').style({'width':'100px','height':'100px'});
```

<img width="808" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/bb2460f4-980e-4727-b1f6-830687577f9f"/>

제이쿼리 타입 선언을 설치하면 제이쿼리의 사양 정보를 참조하게 됩니다.

>  $ npm i --save-dev @types/jquery

이제 오류가 제이쿼리의 사용과 관련된 내용으로 바뀝니다.

```js
// @ts-check
$('#graph').style({'width':'100px','height':'100px'});
         // ------ : 'JQuery<HTMLElement> 형식에는 'style' 속성이 없습니다.'
```

마지막으로 `.style` 메서드를 `.css`으로 바꾸면 오류가 사라집니다.

앞의 제이쿼리 예제처럼 `@ts-check`룰 사용하면 타입스크립트로 마이그레이션하기 전에 서드파티 라이브러리들의 타입 선언을 활용하여 타입 체크를 시험해 복 수 있습니다. 

## DOM 문제

웹브라우저에서 동작하는 코드라면, 타입스크립트는 DOM 엘리먼트 관련 부분에 수많은 오류를 표시하게 됩니다.

```js
// @ts-check
const ageEl = document.getElementById('age');
ageEl.value = '12';
```

<img width="808" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0007cd4e-9a31-4bf0-a6a8-16216a446b16"/>

`HTMLIputElement` 타입에는 value 속성이 있지만, `document.getElementById`은 더 상위 개념인 `HTMLElement` 타입을 반환하는 것이 오류의 원인입니다 ([아이템 55](https://github.com/Pyotato/effective_typescript/blob/item55/README.md)).
그러나 앞의 코드는 자바스크립트 파일이므로 타입스크릡트 단언문 `as HTMLInputElement`을 쓸 수 없습니다.
대신 JSDoc을 사용하여 타입 단언을 대체할 수 있습니다.

```js
// @ts-check
const ageEl = /** @type {HTMLInputElement} */ (document.getElementById('age'));
ageEl.value = '12'; // 정상
```

한편 `@ts-check`를 활성화하면 이미 존재하던 JSDoc에서 부작용이 발생하기도 합니다.
바로 이어서 다루게 될 '부정확한 JSDoc'입니다.

## 부정확한 JSDoc

프로젝트에서 이미 JSDoc 스타일의 주석을 사죵 중이었다면, `@ts-check` 지시자를 설정하는 순간부터 기존 주석에 타입 체크가 동작하게 되고 갑자기 수많은 오류가 발생하게 될 겁니다.
이때는 당황하지 말고 타입 정보를 차근차근 추가해 나가면 됩니다.

다음 예제에서는 `@ts-check` 지시자를 설정하는 순간 두 개의 오류가 발생합니다.

```js
// @ts-check
/**
 * 앨리먼트의 크기 (힉셀 단위)를 가져옵니다.
 * @param {Node} el 해당 엘리먼트
 * @return {{w: number, h:number}} 크기
 */
function getSize(el){
    const bounds = el.getBoundingClientRect();
    return {width: bounds.width,nheight: bounds.height };
}

```

<img width="893" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/da57f91d-e8df-47ba-bcbf-a41a8cd8c324"/>

첫 번째 오류는 DOM 타입 불일치로 발생했습니다.
`getBoundingClientRect`는 Node가 아니라 Element에 정의되어 있기 때문에 `@param`태그를 Node에서 Element로 수정해야 합니다.
두 번째 오류는 `@return` 태그에 명시된 타입과 실제 반환 타입이 맞지 않아서 발생했습니다.
코드만 봐서는 `@return` 태그와 반환 타입 중 어디가 잘못된 것인지 확신할 수 없지만, 프로젝트 전체를 예상해보면 `width`와 `height`가 일반적으로 사용되는 이름이기 때문에 `@return` 태그를 수정하는 것이 올바른 방향일 것입니다.

JSDoc 관련된 오류를 모두 수정햇다면, 다음 차례로 JSDoc을 활용하여 타입 정보를 점진적으로 추가할 수 있습니다.
타입스크립트 언어 서비스는 타입을 추론해서 JSDoc을 자동으로 생성해 줍니다.
다음 코드와 그림의 예시에서 확인할 수 있습니다.

```js
// @ts-check

function double(val){
  return val*2;
}
```

<img width="504" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1d263da4-7504-4bdb-9eb0-c1a0d021bbc2"/>

빠른 수정을 실행하면 다음처럼 타입 정보가 JSDoc 주석으로 생성됩니다.

<img width="509" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0b8e3404-4b9f-4ba9-bf73-1796c78f65a5"/>

<img width="213" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/9b5132a2-ecda-404d-8dfb-eaa709fbfa75"/>

자동 생성 기능은 타입 정보를 빠르게 추가할 수 있기 때문에 유용하지만, 잘 동작하지 않은 경우도 있습니다.
다음 코드를 보겠습니다.

```js
// @ts-check

function loadData(data){
  data.files.forEach(async (file) => {
    // ...
  });
}
```

data 타입 정보를 자동 생성하면 좋지 않은 결과를 얻게 됩니다.

<img width="342" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/b43939bd-4e3f-4651-95d0-2238afc6ffcb"/>

----------
> ⚠️ 원문에서는 다음과 같은 JSDoc이 생성되었지만 tsplayground에서 실행은 위와 같았습니다.

```js

// @ts-check

/**
* @param files:{forEach: (arg0: (file.any) => Promise<void>) => void;} data
*/
function loadData(data){
  data.files.forEach(async (file) => {
    // ...
  });
}

```

구조적 타이핑이 엉뚱하게 적용되었습니다 ([아이템 4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md)).
`files.forEach` 메서드를 가지는 어떠한 객체라도 동작이 가능하겠지만, 원래 loadData 함수가 의도한 시그니처는 `{ files: string[] }`였을 겁니다.

----------

자바스크립트 환경에서도 `@ts-check` 지시자와 JSDoc 주석이라면 타입스크립트와 비슷한 경험으로 작업이 가능합니다.
특별한 작업이 필요 없기 때문에 점진적 마이그레이션 과정 중에 유용하지만, `@ts-check` 지시자와 JSDoc 주석을 너무 장기간 사용하는 것은 좋지 않습니다.
주석이 코드 분량을 늘려서 로직을 해석하는 데 방해가 될 수 있기 때문입니다.
타입스크립트는 <i>.ts</i>파일에서 가장 잘 동작하며, 마이그레이션의 궁극적인 목표는 자바스크립트에 JSDoc 주석이 있는 형태가 아니라 모든 코드가 타입스크립트 기반으로 전환되는 것임을 잊지 말야야 합니다. 
이미 JSDoc 주석으로 타입 정보가 많이 담겨 있는 프로젝트라면 `@ts-check` 지시자만 간단히 추가함으로써 기존 코드에 타입 체커를 실험해 볼 수 있고 초기 오류를 빠르게 잡아낼 수 있다는 점을 기억해야 합니다.
