# ✨ ITEM 58: 모던 자바스크립트로 작성하기

타입스크립트는 타입 체크 기능 외에, 타입스크립트 코드르 특정 버전의 자바스크립트로 컴파일하는 기능도 가지고 있습니다.
심지어 1999년에 나온 ES3버전의 자바스크립트 코드로 컴파일할 수도 있습니다.
즉, 타입스크립트 컴파일러를 자바스크립트 '트랜스파일러(transpiler)'로 사용할 수 있습니다.
타입스크립트는 자바스크리브의 상위집합이기 때문에, 최신 버전의 자바스크립트 코드를 예전의 버전의 자바스크립트 코드로 변환할 수 있습니다.

옛날 버전의 자바스크립트 코드를 타입스크립트 컴파일러에서 동작하게 만들면 이후로는 최신 버전의 자바스크립트 기능을 코드에 추가해도 문제가 없습니다.
옛날 버전의 자바스크립트 코드를 최신 버전의 자바스크립트로 바꾸는 작업은 타입스크립트로 전환하는 작업의 일부로 볼 수 있습니다.
따라서 마이그레이션은 어디서부터 시작해야 할지 몰라 막막하다면 옛날 버전의 자바스크립트 코드를 최신의 버전의 자바스크립트로 바꾸는 작업부터 시작해 보기 바랍니다.
타입스크립트는 자바스크립트의 상위집합이기 때문에, 코드를 최신 버전으로 바꾸다 보면 타입스크립트의 일부를 저절로 익힐 수 있게 됩니다.

이번 아이템은 모던 자바스크립트의 주요 기능 몇 가지를 간력히 다룹니다.

> 모던 자바스크립트는 최신 버전의 자바스크립트를 의미합니다. <br/>
> 2021년 기준으로 ES2015(ES6) 버전부터 모던 자바스크립트라고 부르고 있습니다. <br/>
> 시간이 흘러 5년, 10년 후에는 모던 자바스크립트의 기준이 달라질 수 있습니다. <br/>

현재 아이템에서 소개하는 기능들은 모두 ES2015(일면 ES6) 버전부터 도입된 것이며, 다른 책들과 인터넷에서 매우 자세히 다루고 있기 때문에 잘 모르는 부분이 있다면 쉽게 찾아 볼 수 있습니다. 
현제 아이템에서 다루는 것은 자바스크립트지만, 타입스크립트 컴파일러를 사용하면 `async/await` 같은 새로운 기능을 배울 떄 매우 유용합니다.
최신 자바스크립트 기능도 모두 체크되기 때문에 코드 작성 시에 도움을 받을 수 있고 제대로 된 사용법을 배울 수 있습니다.
모던 자바스크립트의 새로운 기능들은 모두 배울 만한 가치가 있지만, 타입스크립트를 도입할 때 가장 중요한 기능은 ECMAScript 모듈과 ES2015 클래스입니다.

## ECMAScript 모듈로 사용하기

ES2015 이전에는 코드를 개별 모듈로 분할하는 표준 방법이 없었지만, 지금은 개별 모듈로 분할하는 방법이 많아졌습니다.
여러 개의 `<script>` 태그를 사용하기, 직접 갖다 붙이 (manual cooncatenation), Makefile 기법, NodeJS 스타일의 require 구문, AMD 스타일의 `defined` 콜백까지 매우 다양합니다.
심지어 타입스크립트에도 자체적인 모듈시스템 ([아이템 53](https://github.com/Pyotato/effective_typescript/blob/item53/README.md))이 있습니다.

ES2015부터는 임포트 (import)와 익스포트 (export)를 사용하는 ECMAScript 모듈 (또는 ES 모듈)이 표준이 되었습니다.
만약 마이그레이션 대상인 자바스크립트 코드가 단일 파일이거나 비표준 모듈 시스탬을 사용 중이라면 ES 모듈로 전환하는 것이 좋습니다.
그리고 ES모듈 시스템을 사용하기 위해서 프로젝트 종류에 따랄 웹팩(webpack)이나 ts-node같은 도구가 필요한 경우도 있습니다.
ES모듈 시스템은 타입스크립트에서도 잘 동작하며, 모듈 단위로 전환할 수 있게 해 주기 때문에 점진적 마이그레이션이 원활해집니다 ([아이템 61](https://github.com/Pyotato/effective_typescript/blob/item61/README.md)).

CommonJS 모듈 시스템을 사용한 전형적인 예제는 다음과 같습니다.

```js
// a.js
const b = require('./b');
console.log(b.name);


// b.js
const name = 'Module B';
module.exports = {name};
```

동일한 기능을 하는 코드를 ES 모듈로 표현하면 다음과 같습니다.

```js
// a.js
import *  as b from './b';
console.log(b.name);

// b.js
export const name = 'Module B';
```

## 프로토타입 대신 클래스 사용하기

과거에는 자바스크립트에서 프로토타입 기반의 객체 모델을 사용했습니다.
그러나 많은 개발자가 사용하기 애매한 프로토타입 모델보다는 견고하게 설계된 클래스 기반 모델을 선호했기 때문에, 결국 ES2015에 class 키워드를 사용하는 클래스 기반 모델이 도입되었습니다.

마이그레이션하려는 코드에서 단순한 객체를 다룰 때 프로토타입을 사용하고 있었다면 클래스로 바꾸는 갓이 좋습니다.
다음은 단순 객체를 프로토타입으로 구현한 예제입니다.

```js
function Person(first,last){
  this.first = first;
  this.last = last;
}

Person.prototype.getName = function(){
  return this.first + ' ' + this.last;
}

const marie = new Person("Marie", "Curie");
const personName = marie.getName();
```

프로토타입 기반 객체를 클래스 기반 객체로 바꾸면 다음과 같습니다.

```js
class Person{
  first : string ;
  last : string t;

  constructor (){
    this.first = first;
    this.last = last;
  }

  getName = function(){
    return this.first + ' ' + this.last;
  }

}

const marie = new Person("Marie", "Curie");
const personName = marie.getName();
```

프로토타입으로 구현한 Person 객체보다 클래스로 구현한 Person 객체가 문법이 간결하고 직괒적입니다.
클래스 문법에 익숙하지 않더라도 타입스크립트 언어 서비스를 활용하면 틀래스를 간단히 작성할 수 있습니다.

----
> ⚠️ 원본에서는 언어 서비스가 작동했지만, tsplayground 또는 vscode에서 실행 시 동작하지 않았습니다 (ts 설정이 다른 걸 수도?)
----

편집기에서 프로토타입 객체에 마우스를 올려, 타입스크립트 언어 서비스인 '함수를 ES2015로 변환 (Convert function to an ES2015 class)'을 선택하면 간단히 클래스 객체로 변환할 수 있습니다. 

## var 대신 let/const 사용하기

자바스크립트 `var` 키워드의 스코프 (scope) 규칙에 문제가 있다는 것은 널리 알려진 사실입니다.
스코프 문제를 자세히 알고 싶다면 (이펙트 자바스크립트 <<안사이트>>) 책을 참고하시길 바랍니다.
스코프 문제를 자세히 알지 못하더라도 `var`대신 `let/const`를 사용하면 스코프 문제를 피할 수 있습니다.
`let/const`는 제대로 된 블록 스코프 규칙을 가지며, 개발자들이 일반적으로 기대하는 방식으로 동작합니다.

만약 `var` 키워드를 `let/const`로 변경하면, 일부 코드에서 타입스크립트가 오류를 표시할 수도 있습니다.
오류가 발생한 부분은 잠재적으로 스코프 문제가 존재하는 코드이기 때문에 수정해야 합니다.

중첩된 함수 구문에도 `var`의 경우와 비슷한 스코프 문제가 발생합니다. 
예를 들어 보겠습니다.

```js
function foo(){
  bar();
  function bar(){
    console.log('hello');
  }
}
```

foo 함수는 호출하면 bar 함수의 정의가 호이스팅 (hoisting) 되어 가장 먼저 수행됙 때문에 bar 함수가 문제 없이 호출되고 hello가 출력됩니다.
호이스팅은 실행 순서를 예상하기 어렵게 만들고 직관적이지 않습니다.
대신 함수 표현식 (`const bar = ()=> {/** */}`)을 사용하여 호이스팅 문제를 파하는 것이 좋습니다. 

## for(;;)대신 for-of 또는 배열 메서드 사용하기

과거에는 자바스크립트에서 배열을 순회할 때 C 스타일의 for 루프를 사용했습니다.

```js
for(var i=0; i<array.length; i++){
  const el = array[i];
  // ...
}
```

모던 자바스크립트에는 `for-of`루프가 존재합니다.

```js
for(const el of array){
  // ...
}
```

`for-of`루프는 코드가 짧아지고 인덱스 변수를 사용하지도 않기 떄문에 실수를 줄일 수 있습니다.
인덱스 변수가 필요한 경우엔 forEach 메서드를 사용하면 됩니다.

```js
array.forEach((el, i)=>{
  // ...
});
```

`for-in` 문법도 존재하지만 몇 가지 문제점이 있기 때문에 사용하지 않는 것이 좋습니다.
자세한 내용은 [아이템 16](https://github.com/Pyotato/effective_typescript/blob/item16/README.md)에서 다룹니다.

## 함수 표현식보다는 화살표 함수 사용하기

`this`키워드는 일반적인 변수들과는 다른 스코프 규칙을 가지기 때문에, 자바스크립트에서 가장 어려운 개념 중 하나입니다.
일반적으로는 this가 클래스 인스턴스를 참조하는 것을 기대하지만 다음 예제처럼 예상치 못한 결과가 나오는 경우도 있습니다.

```ts
class Foo{
  method(){
    console.log(this);
    [1, 2].forEach(function(i){
      console.log(this);
    });
  }
}

const f = new Foo();
f.method(); // strict 모드에서는 Foo, undefined, undefined, non-strict에서믐 Foo, window, window 출력
```

대신 화살표 함수를 사용하면 상위 스코프의 this를 유지할 수 있습니다. 

```ts
class Foo{
  method(){
    console.log(this);
    [1, 2].forEach((i)=>{
      console.log(this);
    });
  }
}

const f = new Foo();
f.method(); // strict 모드에서는 Foo, undefined, undefined, non-strict에서믐 Foo, window, window 출력
```

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/50d669a2-c82b-451a-95e7-8e0a2cbf5abf"/>

인라인 (또는 콜백)에서는 일반함수보다 화살표 함수가 더 직관적이며 코드도 간결해지기 때문에 가급적 화살표 함수를 사용하는 것이 좋습니다.
그리고 컴파일러 옵션에 `noImplicitThis (또는 strict)`를 설정하면 타입스크립트가 this 바인딩 관련된 오류를 표시해주므로 설정하는 것이 좋습니다.
this 바인딩에 대한 자세한 내용은 [아이템 49](https://github.com/Pyotato/effective_typescript/blob/item49/README.md)에서 다룹니다.

# 단축 객체 표현과 구조 분해 할당 사용하기

pt 객체를 생성하는 다음 코드를 보겠습니다.

```js
const x = 1, y = 2, z = 3;
const pt = {
  x: x,
  y: y,
  z: z
};
```

변수와 객체 속성의 이름이 같다면 간단하게 다음 코드처럼 작성할 수 있습니다.

```js
const x = 1, y = 2, z = 3;
const pt = {x, y, z};
```

앞의 두 예제 중 후자의 코드가 더 간결하고 중복된 이름을 생략하기 때문에 가독성이 좋고 실수가 적습니다 ([아이템 36](https://github.com/Pyotato/effective_typescript/blob/item36/README.md))

화살표 한수 내에서 객체를 반환할 때는 소괄호로 감싸야 합니다. 
함수의 구현부에는 블록이나 단일 표현식이 필요하기 때문에 소괄호로 감싸서 표현식으로 만들어 준 것입니다.

```js
['A', 'B', 'C'].map((char,idx)=>({char, idx}));
```

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/be1cf741-0262-41ed-990f-fd5651f9554c"/>

객체의 속성 중 함수를 축약해서 표현하는 방법은 다음과 같습니다.

```js
const obj = {
  onClickLong: function (e){ /** */},
  onClickCompact: function (e){ /** */}
};
```

단순 객체 표현 (compact object literal)의 반대는 객체 구조 분해 (object deconstructuring)입니다.
다음 코드를 보겠습니다.

```js
const props = obj.props;
const a = props.a;
const b = props.b;
```

다음처럼 줄여서 작성할 수 있습니다.

```js
const {props} = obj;
const {a, b} = props;
```

극단적으로는 다음처럼 줄일 수도 있습니다.

```js
const {props: {a, b}} = obj;
```

참고로 a와 b는 변수로 선언되었지만 props는 변수 선언이 아니라는 것을 주의해야 합니다.

구조 분해 문법에서는 기본값을 지정할 수도 있습니다.
다음은 단순한 방식으로 if구문을 사용해 기본값을 지정하는 코드입니다.

```js
let {a} = obj.props;
if(a === undefined) a = 'default';
```

대신 다음처럼 구조 분해 문법 내에서 기본값을 지정할 수도 있습니다.

```js
const {a = 'default'} = obj.props;
```

배열에도 구조 분해 문법을 사용할 수 있습니다.
튜플처럼 사용하는 배열에서 특히 유용합니다.

```js
const point = [1, 2, 3];
const [x, y, z] = point;
const [, a, b] = point; // 첫번째 요소 무시
```

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1576b9cd-5fa7-4b8d-80d4-2e412338c105"/>

험수 매개변수에서도 구조분해 문법을 사용할 수 있습니다.

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/e315a2e5-5da7-4dd8-ba43-1cba615e30d6"/>

단축 객체 표현과 마찬가지로, 객체 구조 분해를 사용하면 문법이 간결해지고 변수를 사용할 때 실수를 줄일 수 있기 때문에 적극적으로 사용하는 것이 좋습니다.

## 함수 매개변수 기본값 사용하기

자바스크립트에서 함수의 모든 매개변수는 선택적 (셍략 가능)이며, 매개변수를 지정하지 않으면 undefined로 간주됩니다.

```js
function log2(a, b){
  console.log(a, b);
}
log2();
```

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/84bea78d-ac2b-4061-a2ab-b2f5d47e90af"/>

옛날 자바스크립트에서는 매개변수의 기본값을 지정하고 싶을 때, 다음 코드처럼 구현하곤 했습니다.

```js
function parseNum(str, base){
  base = base || 10;
  return parseInt(str, base);
}
```

모던 자바스크립트에서는 매개변수에 기본값을 직접 지정할 수 있습니다.

```js
function parseNum(str, base=10){
  return parseInt(str, base);
}
```

매개변수에 기본값을 지정하면 코드가 간결해질 뿐만 아니라 base가 선택적 매개변수라는 것을 명확히 나타내는 효과도 줄 수 있습니다.
그리고 기본값을 기반으로 타입 추론이 가능하기 때문에, 타입스크립트도 마이그레이션할 때 매개변수에 타입 구문을 쓰지 않아도 됩니다 ([아이템 19](https://github.com/Pyotato/effective_typescript/blob/item19/README.md))

## 저수준 프로미스나 콜백 대신 async/await 사용하기

[아이템 25](https://github.com/Pyotato/effective_typescript/blob/item19/README.md)에서 콜백과 프로미스보다 `async/await`가 권장되는 이유를 자세히 설명했습니다.
요점은 `async/await`를 사용하면 코드가 간결해져서 버그나 실수를 방지할 수 있고, 비동기 코드에 타입 정보가 전달되어 타입 추론을 가능하게 한다는 것입니다.

```ts
function getJSON(url: string){
  return fetch(url).then(response => response.json());
}

function getJSONCallback(url: string, cb : (result: unknown) => void){/** */}

```

콜백과 프로미스를 사용한 코드보다는 다음처럼 `async/await`로 작성한 코드가 훨씬 깔끔하고 직관적입니다.

```ts
async function getJSON(url: string){
  const response = await fetch(url);
  return response.json();
}
```

## 연관 배열에 객체 대신 Map과 Set 사용하기

[아이템 15](https://github.com/Pyotato/effective_typescript/blob/item15/README.md)애서 객체의 인덱스 시그니처를 사용하는 방법을 다뤘습니다.
인덱스 시그니처는 편리하지만, 몇 가지 문제점이 있습니다.
문자열 내의 단어 개수를 세는 함수를 예로 들어 보겠습니다.

```ts
function coundWords (text: string){
  const counts : {[word: string]: number} = {};
  for(const word of text.split(/[s,.]+/)){
    counts[word] = 1 + (counts[word] || 0);
  }
  return counts;
}
```

별다른 문제가 없어 보이는 코드지만, constructor라는 특정 문자열이 주어지면 문제가 발생합니다.

```ts
console.log(countWords('Objects have a constructor'));
```

실행하면 결과는 다음과 같습니다.

<img width="807" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/766438ee-43f9-4eaa-984b-81aa479669b5"/>


constructor의 초깃값은 `undefined`가 아니라 `Object.prototype`에 있는 생성자 함수입니다.
원치 않는 값일 뿐만 아니라, 타입도 number가 아닌 string입니다.
이런 문제를 방지하려면 Map을 사용하는 것이 좋습니다.

```ts
function countWordsMap(text: string){
  const counts = new Map<string, number>();
  for(const word of text.split(/[\s,.]+/)){
    counts.set(word, 1+ (counts.get(word) || 0);
  }
  return counts;
}
```

## 타입스크립트에 usestrict 넣지 않기

ES5에서는 버그가 될 수 있는 코드 패턴에 오류를 표시해주는 `엄격 모드(strictmode)`기 도입되었습니다.
다음 예제처럼 코드의 ㅈ일 처음에 `use strict`를 넣으면 엄격 모드가 활성화됩니다.

```js
'use strict';
function foo(){
  x = 10; //   strict 모드에서는 오류, non-strict 모드에서는 전역 선언
}
```

엄격 모드를 사용해 본 적이 없다면, 한번 시도해 보기 바랍니다.
오류로 표시된 부분은 타입스크립트에서도 오류일 가능성이 높습니다.

그러나 타입스크립트에서 수행되는 안전성 검사 (sanity check)가 엄격모드보다 훨씬 더 엄격한 체크를 하기 때문에, 타입스크립트 코드에서 'use strict'는 무의미합니다.

힐제로는 타입스크립트 컴파일러가 생성하는 자바스크립트 코드에서 'use strict'가 추가 됩니다.
'alwaysStrict' 또는 'strict' 컴파일러 옵션을 설정하면, 타입스크립트에서는 엄격 모드로 코드를 파싱하고 생상된 자바스크립트에 'use strict'를 추가합니다.
즉, 타입스크립트 코드에 'use strict'를 쓰지 않고, 대신 'alwaysStrict' 설정을 사용해야 합니다.

현재 아이템에서 나열된 기능들 외에도 모던 자바스크립트에는 수많은 기능이 존재하며, 타입스크립트에서도 사용가능합니다.
자바스크립트 표준 단체 TC39는 매년 새로운 기능들을 발표하고 있습니다.
한편 타입스크립트 개발팀은 자바스크립트 표준화 4단계 중 3단계 이상의 기능들을 타입스크립트 내에 구현하고 있습니다.
그러므로 표준화 완성 여부에 상관없이 자바스크립트 표준화 3단계 이상의 기능들은 타입스크립트 내에서도 사용할 수 있습니다.
[TC39의 깃헙 저장소](https://github.com/tc39/ecma262)에서 표준화 관련 최신 정보를 확인할 수 있습니다.
특히 파이프라인과 데코레이터 기능은 아직 3단계에 이르지 못했지만 큰 잠재력을 지니고 있으므로 조만간 타입스크립트에도 추가될 가능성이 큽니다.
