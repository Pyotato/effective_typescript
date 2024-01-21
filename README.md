# 🔢 ITEM 16 : number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

자바스크립트는 이상하게 동작하기로 유명한 언어입니다.
그중 가장 악명 높은 것은 암시적 타입 강제와 관련된 부분입니다.

```js
console.log("0" == 0); // true
```

다행히도 암시적 타입 강제와 관련된 문제는 대부분 `===`와 `!==`를 사용해서 해결이 가능합니다.

자바스크립트 객체 모델애도 이상한 부분들이 있으며, 이 중 일부는 타입스크립트 타입 시스템으로 모델링 되기 떄문에 자잡스크립트 객체 모델을 이해하는 것이 중요합니다.
이미 [아이템 10](https://github.com/Pyotato/effective_typescript/blob/item10/README.md)에서 다룬 객체 래퍼 타입에서 이러한 이상한 특징을 살펴봤습니다.
이번 아이템에서는 또 다른 이상한 점들을 다룹니다.

자바스크립트에서 객체란 키/값 쌍의 모음입니다.
키는 보통 문자열입니다(ES2015 이후로는 심벌일 수 있습니다).
그리고 값은 어떤 것이든 될 수 있습니다.

파이썬이나 자바에서 볼 수 있는 '해시 가능' 객체라는 표현이 자바스크립트에는 없습니다.
만약 더 복잡한 객체를 키로 사용하려고 하면, toString 메서드가 호출되어 객체가 문자열로 변환됩니다.

```js
> x = {}; // {}
> x[[1,2,3]] = 2 // 2
> x // {'1,2,3' : 1 }
```

특히, 숫자는 키로 사용할 수 없습니다.
만약 속성 이름으로 숫자를 사용하려고 하면, 자바스크립트는 런타임은 문자열로 변환할 겁니다.

```js
{ 1: 2, 3: 4 }
> { '1' : 2, '3': 4 }
```

이번엔 배열을 알아보겠습니다. 배열은 분명히 객체입니다.

```js
> typeof [] //'object'
```

그러니 숫자 인덱스를 사용하는 것이 당연합니다.

```js
> x = [1,2,3] // [1,2,3]
// x[0] // 1
```

이상하게 보일지 모르지만, 앞의 인덱스들은 문자열로 변환되어 사용됩니다.
문자열 키를 사용해도 역시 배열의 요소에 접근할 수 있습니다.

```js
>  x['1'] // 2
```

`Object.keys`를 이용해 배열의 키를 나열해 보면, 키가 문자열로 출력됩니다.

```js
Object.keys(x)l // ['0', '1', '2']
```

타입스크립트는 이러한 혼란을 바로잡기 위해 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식합니다.
Array에 대한 타입 선언은 ([아이템 6](https://github.com/Pyotato/effective_typescript/blob/item6/README.md]),<i>libs.es5.d.ts</i>에서 확인할 수 있습니다.

```ts
interface Array<T>{
  // ...
  [n: number]: T;
}
```

런타임에는 ECMAScript 표준이 서술하는 것처럼 문자열 키로 인식하므로 이 코는 완전히 가상이라고 할 수 있지만, 타입 체크 시점에 오류를 잡을 수 있어 유용합니다.

```ts
const xs = [1,2,3];
const x0 = xs[0];
const x2 = xs['1'];


function get<T>(array: T[], k: string):T {
  return array[k];
}
```

---
⚠️ 아래는 원문에 해당되는 내용입니다. (작성자의 tsplayground과 ts작성 파일에서는 정상 동작합니다.)

```ts
const xs = [1,2,3];
const x0 = xs[0];
const x2 = xs['1'];
      //   ~~~~~~~ 인덱스 식이 'number' 형식이 아니므로 요소에 암시적으로 'any'형식이 있습니다.


function get<T>(array: T[], k: string):T {
  return array[k];
  //   ~~~~~~~ 인덱스 식이 'number' 형식이 아니므로 요소에 암시적으로 'any'형식이 있습니다.
}
```

---

다시 한번 말하지만, 이 코드는 실제로 동작하지 않습니다.
그리고 타입스크립트 타입 시스템의 다른 것들과 마찬가지로, 타입 정보는 런타임에 제거됩니다 ([아이템 3](https://github.com/Pyotato/effective_typescript/blob/item3/README.md)).
한편 `Object.keys`같은 구문은 여전히 문자열로 반환됩니다.

```ts
const keys = Object.keys(xs);
for(const key in xs){
  key;
  const x= xs[key];
}
```

<img width="226" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/d483ccbc-d455-402b-8ea4-dd0cd65bc18d"/>
<img width="226" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/76e342f7-eed3-4e4a-8bc1-d24b7f36f0be"/>
<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/1302c35d-7c4c-4cd2-842a-a6c608495516"/>

string이 number에 할당될 수 없기 때문에, 예제의 마지막 줄이 동작하는 것이 이상하게 보일 겁니다.
배열을 순회하는 코드 스타일에 대한 실용적인 허용이라고 생각하는 것이 좋습니다.
자바스크립트에서는 흔한 일이지만, 이 예제가 배열을 순회하기에 좋은 방법은 아닙니다.
인덱스에 신경 쓰지 않는 다면, `for-of`를 사용하는 게 더 좋습니다.

```ts
const keys = Object.keys(xs);
for(const x of xs){
  x;
}
```

<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/0bb21e2e-6c16-480f-83f6-1b82bd267853"/>

만약 인덱스의 타입이 중요하다면, number타입을 제공해줄 Array.prototype.forEach를 사용하면 됩니다. 

```ts
xs.forEach((x, i)=>{
  i;
  x;
});
```

<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/7e7930f5-4235-43a6-91e8-9422b9319012"/>
<img width="211" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/504464a2-2048-4f77-8efd-9604279a3708"/>


루프 중간에 멈춰야 한다면, C 스타일인 for(;;) 루프를 사용하는 것이 좋습니다.

```ts
for(let i=0; i< xs.length; i++){
  const x = xs[i];
  if(x<0) break;
}
```

타입이 불확실하다면, (대부분의 브라우저와 자바스크립트 엔진에서) `for-in`루프는 `for-of` 또는 `for`루프에 비해 몇 배나 느립니다.

다시 배열 이야기로 돌아오겠습니다.
인덱스 시그니처가 number로 표현되어 있다면 입력한 값이 number여야 한다는 것을 의미하지만 (`for-in`루프는 확실히 제외하고), 실제 런타임에 사용되는 키는 string 타입입니다.

이 부분이 혼란스럽게 느껴질 수 있습니다.
일반적으로 string 대신 number를 타입의 인덱스 시그니처로 사용할 이유는 많지 않습니다.
만약 숫자를 사용하여 인덱스할 항목을 지정한다면 Array 또는 튜플 타입을 대신 사용하게 될 겁니다.
number를 인덱스 타입으로 사용하면 숫자 속성이 어떤 특별한 의미를 지닌다는 오해를 불러 일으킬 수 있습니다.

한편 Array 타입이 사용하지도 않을 push나 concat 같은 다른 속성 (프로토타입에서 온)을 가지는 게 납득하기 어려울 수 있습니다.
납득하기 어렵다는 것은 구조적인 고려를 하고 있다는 뜻이기 때문에 타입스크립트를 잘 이해하고 있다고 볼 수 있습니다 ([아이템 4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md)에서 다룹니다).

어떤 길이를 가지는 배열과 비슷한 형태의 튜플을 사용하고 싶다면 타입스크립트에 있는 ArrayLike 타입을 사용합니다.


```ts
function checkedAccess<T>(xs: ArrayLike<T>, i: number):T{
  if(i<xs.length){
    return xs[i];
  }
  throw new Error(`배열의 끝을 지나서 ${i}를 접근하려고 했습니다.`);
}
```

이 예제는 길이와 숫자 인덱스 시그니처만 있스빈다.
이런 경우가 실제로는 드물기는 하지만 필요하다면 ArrayLike를 사용해야 합니다.
그러나 ArrayLike를 사용하더라도 키는 여전히 문자열이라는 점을 잊지 말아야 합니다.

```ts
const tupleLike: ArrayLike<string>= {
  '0':'A';
  '1'': 'B';
  length: 2,
}  // 정상
```
