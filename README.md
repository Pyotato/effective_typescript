#  🌗 ITEM 20 : 다른 타입에는 다른 변수 사용하기

자바스크립트에서는 한 변수를 다른 목저긍로 가지는 다른 타입으로 재사용해도 됩니다.

```js
let id = "12-34-56";
fetchProduct(id);    // string으로 사용
id = 123456;
fetchProductBySerialNumber(id); // number로 사용
```

반면 타입스크립트에서는 두 가지 오류가 발생합니다.

<img width="1321" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/4d1bb2ec-51f1-4fc4-bdd4-0078f3d29028"/>

편집기에서 첫번째 id위에 마우스 커서를 올려 놓으면 무엇이 문제인지 알 수 있스빈다. 

<img width="428" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/b0d466ce-bf5b-44fa-a1de-1847cee180ac"/>

타입스크립트는 "12-34-56"이라는 값을 보고, id의 타입을 string으로 추론했습니다.
string 타입에는 number 타입을 할당할 수 없기 때문에 오류가 발생했습니다.

여기서 "변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다"는 중요한 관점을 알 수 있습니다.
타입을 바꿀 수 잇는 한 가지 방법은 범위를 좁히는 것인데 ([아이템 22](https://github.com/Pyotato/effective_typescript/blob/item22/README.md)), 
새로운 변수값을 포함하도록 확장하는 것이 아니라 타입을 더 작게 제한하는 것입니다.
이 관점에서 반하는 타입 지정 방법 ([아이템 41](https://github.com/Pyotato/effective_typescript/blob/item41/README.md))이 있는데, 이 방법은 어디까지나 예외이지 규칙은 아닙니다.

이제 오류가 발생한 앞의 예제를 고쳐 보겠습니다.
id의 타입을 바꾸지 않으려면, string과 number 모두 포함할 수 있도록 타입을 확장하면 됩니다.
`string | number` 로 표현하며, 유니온(union)타입이라고 합니다.

```ts
let id : string | number= "12-34-56";
fetchProduct(id);   
id = 123456;
fetchProductBySerialNumber(id); 
```

타입스크립트는 첫 번째 함수 호출에서 id는 string으로, 두 번째 호출에서는 number라고 제대로 판단합니다.
할당문에서 유니온 타입으로 범위가 좁혀졌기 때문입니다.

유니온 타입으로 코드가 동작하기는 하겠지만 더 많은 문제가 생길 수 있습니다. 
id를 사용할 때마다 값이 어떤 타입인지 확인해야 하기 때문에 유니온 타입은 string이나 number같은 간단한 타입에 비해 다루기 더 어렵습니다.

차라리 별도의 변수를 도입하는 것이 낫습니다.

```ts
let id = "12-34-56";
fetchProduct(id);   
const serial = 123456;
fetchProductBySerialNumber(serial); 
```

앞의 예제에서 첫 번째 id와 재사용한 두 번째 id는 서로 관련이 없습니다.
그냥 변수를 재사용했을 뿐입니다.
변수를 무분별하게 재사용하면 타입 체커와 사람 모두에게 혼란을 줄 뿐입니다.

다른 타입에는 별도의 변수를 사용하는 게 바람직한 이유는 다음과 같습니다.

- 서로 관련이 없는 두 개의 값을 분리합니다. (id와 serial)
- 변수명을 더 구체적으로 지을 수 있습니다.
- 타입 추론을 향상시키며, 타입 구문이 불필요해집니다.
- 타입이 좀더 간결해집니다 (`string | number` 대신 string과 number 사용). 
- let 대신 const로 변수를 선언하게 됩니다. const로 변수를 선언하면 코드가 간결해지고, 타입 체커가 타입 추론하기에도 좋습니다.

타입이 바뀌는 변수는 되도록 피해야 하며, 목적이 다른 곳에는 별도의 변수명을 사용해야 합니다.
그런데 지금까지 이야기한 재사용되는 변수와, 다음 예제에 나오는 '가려지는(shadowed)' 변수를 혼동해서는 안 됩니다.

```ts
let id = "12-34-56";
fetchProduct(id);   
{
  const id = 123456;
  fetchProductBySerialNumber(id);
}
```

여기서 두 id는 이름이 같지만 실제로는 서로 아무런 관계가 없습니다.
그러므로 각기 다른 타입으로 사용되어도 문제 없습니다.
그런데 동일한 변수명에 타입이 다르다면, 타입스크립트 코드는 잘 동작할지 몰라도 사람에게 혼란을 줄 수 있습니다.
다시 한번 말하지만 목적이 다른 곳에는 별도의 변수명을 사용하기 바랍니다. 
많은 개발팀이 린터 규칙을 통해 '가려지는' 변수를 사용하지 못하도록 하고 있습니다.

이번 아이템에서는 변수에 기본형(primmitive)타입을 할당하는 방법을 다뤘습니다. 
변수에 객체를 할당하는 방법은 [아이템 23](https://github.com/Pyotato/effective_typescript/blob/item23/README.md)에서 다룹니다.
































