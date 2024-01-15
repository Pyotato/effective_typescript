#  🎁 ITEM 10 : 객체 래퍼 타입 피하기

자바스크립트에는 객체 이외에도 기본형 값들에 대한 일곱 가지 타입(`string`,`number`,`nunber`,`boolean`,`null`,`undefined`,`symbol`,`bigint`)이 있습니다. 
string, number, boolean, null은 자바스크립트 초창기부터 존재해 왔으며, symbol 기본형은 ES2015에서 추가되었고, bigint는 최종 확장 단계에 있습니다.

기본형들은  `불변(immutable)`이며 메서드를 가지지 않는다는 점에서 객체와 구분됩니다. 그런데 기본형인 string의 경우 메서드를 가지고 있는 것처럼 보입니다.

```js
'primitive'.charAt(3) // "m"
```

하지만 사실 charAt은 string의 메서드가 아니며, string을 사용할 때 자바스크립트 내부적으로 많은 동작이 일어납니다.
string '기본형'에는 메서드가 없지만, 자바스크립트에는 메서드를 가지는 String '객체' 타입이 정의 되어 있습니다.
자바스크립트는 기본형과 객체 타입을 서로 자유롭게 변환합니다.
string 기본형에 charAt같은 메서드를 사용할 때, 자바스크립트는 기본형을 String 객체로 래핑(wrap)하고, 메서드를 호출하고, 마지막에 래핑한 객체를 버립니다.

만약 String.prototype을 몽키-패치(monkey-patch)한다면 앞서 설명한 내부적인 동작들을 관찰할 수 있습니다 ([아이템 43](https://github.com/Pyotato/effective_typescript/blob/item43/README.md))

> 🐒 몽키-패치란 런타임에 프로그램의 어떤 기능을 수정해서 사용하는 기법을 의미합니다. 
자바스크립트에서는 주로 프로토타입을 변경하는 것이 해당됩니다. 여담이지만 몽키-패치의 유래가 꽤 재미있으니 검색해보길 바랍니다. [몽키패치 위키(영문)](https://en.wikipedia.org/wiki/Monkey_patch)

```ts
// 실제로는 이렇게 하지 마세요!
const originalCharAt = String.prototype.charAt;
String.prototype.charAt = function(pos){
  console.log(this, typeof this, pos);
  return originalCharAt.call(this, pos);
};
console.log('primitive'.charAt(3));
```

<img width="1253" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4f1f4e3e-c7c3-4b23-920c-5b4384acfd3c"/>

메서드 내의 this는 string 기본형이 아닌 String 객체 래퍼입니다. String 객체를 직접 생성할 수도 있으며, string 기본형처럼 동작합니다.
그러나 string 기본형과 String객체 래퍼가 항상 동일하게 동작하는 것은 아닙니다. 예를 들어, String 객체는 오직 자기 자신하고만 동일합니다.

```ts
console.log("hello" === new String("hello"));
console.log(new String("hello") === new String("hello"));
```

<img width="680" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/6f08f942-ec02-4c88-a044-e6cd1cc9cb90"/>

객체 래퍼 타입의 자동 변환은 종종 당황스러운 동작을 보일 때가 있습니다.
예를 들어 어떤 속성을 기본형에 할당한다면 그 속성이 사라집니다.

```ts
x = "hello"; // x가 String로 변환
x.language = "English" // "English" // language 속성이 추가
console.log(x.language); // undefined // language 속성이 추가된 객체는 버려짐
```

다른 기본형에도 동일하게 객체 래퍼 타입이 존재합니다. number에는 Number, boolean에는 Boolean, symbol에는 Symbol, bigint에는 BigInt가 존재합니다 (null과 undefined에는 객체 래퍼가 없습니다). 

이 래퍼 타입들 덕분에 기본형 값에 메서드를 사용할 수 있고 정적 메서드 (String.fromCharCoder 같은)도 사용할 수 있습니다. 그러나 보통은 래퍼 객체를 직접 생성할 필요가 없습니다.

타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링합니다.

- string과 String
- number와 Number
- boolean과 Boolean
- symbol과 Symbol
- bigint와 BigInt

그런데 string을 사용할 때는 특히 유의해야 합니다. string을 String이라고 잘못 타이핑하기 쉽고 (특히, 자바나 C#을 사용하던 사람이라면), 실수를 하더라도 처음에는 잘 동작하는 것처럼 보이기 때문입니다.

```ts
function getStringLen(foo: String){
  return foo.length;
}
console.log(getStringLen("hello"));
console.log(getStringLen(new String("hello")));

```

그러나 string을 매개변수로 받는 메서드에 String 객체를 전달하는 순간 문제가 발생합니다.

```ts
function isGreeting(phrase: String){
  return ['hello','good day'].includes(phrase);
}

```

<img width="1358" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0ebf0e5d-df98-4e7e-8cf9-abd03ed29fd5"/>

string은 String에 할당할 수 있지만 String은 string에 할당할 수 없습니다.
오류 메세지대로 string타입을 사용해얗 합니다. 대부분의 라이브러리와 마찬가지로 타입슼릡트가 제공하는 타입 선언은 전부 기본형 타입으로 되어 있습니다.

래퍼 객체는 타입 구문의 첫 글자를 대문자로 표기하는 방법으로도 사용할 수 있습니다.

```ts
const s: String = "primitive";
const n: Number = 12;
const b: Boolean = true;
```

당연히 런타임의 값은 객체가 아니고 기본형입니다. 
그러나 기본형 타입은 객체 래퍼에 할당할 수 있기 때문에 타입스크립트는 기본형 타입을 객체 래퍼에 할당하는 선언을 허용합니다.
그러나 기본형 타입을 객체 래퍼에 할당하는 구문은 오해하기 쉽고, 굳이 그렇게 할 필요도 없습니다 ([아이템 19](https://github.com/Pyotato/effective_typescript/blob/item19/README.md)).
그냥 기본형 타입 사용하는 것이 낫습니다.

그런데 new 없이 BigInt와 Symbol를 호출하는 경우는 기본형을 생성하기 때문에 사용해도 좋습니다. 

```ts
console.log(typeof BigInt(1234));
console.log(typeof Symbol('sym'));

```

<img width="590" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/18ba99ea-3afb-47d8-8467-d940673f8ac3"/>

이들은 BigInt와 Symbol '값'이지만, 타입스크립트 타입은 아닙니다 ([아이템 8](https://github.com/Pyotato/effective_typescript/blob/item8/README.md)).
앞의 예제의 결과는 bigint와 symbol 타입의 값이 됩니다.

