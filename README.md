# effective_typescript

## Contents

| item |                                                  title                                                  | summary                                                                                                                                                                                                                                                                                             |
| :--: | :-----------------------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------ |
|  4  | [구조적 타이핑에 익숙해지기](https://github.com/Pyotato/effective_typescript/blob/item4/README.md) | 💡자바스크립트가 덕 타이핑(duck typing) 기반이고 타입스크립트가 이를 모델링하기 위해 구조적 타이핑을 사용함을 이해해야합니다. 어떤 인터페이스에 할당 가능한 값이라면 타입 선언에 명시적으로 나열된 속성들을 가지고 있을 겁니다. 타입은 '봉인'되어 있지 않습니다. <br/> 💡클래스 역시 구조적 타이핑 규칙을 따른다는 것을 명심해야 합니다. 클래스의 인스턴스가 예상과 다를 수 있습니다.  <br/> 💡구조적 타이핑을 사용하면 유닛 테스팅을 손쉽게 할 수 있습니다.                                                                                        |
|  40  | [함수 안으로 타입 단언문 감추기](https://github.com/Pyotato/effective_typescript/blob/item40/README.md) | 💡타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 합니다. 불가피하게 사용해야한다면, 정확한 정의를 가지는 함수 안으로 숨기도록 합니다.                                                                                                       |
|  54  |     [객체를 순회하는 노하우](https://github.com/Pyotato/effective_typescript/blob/item54/README.md)     | 💡객체를 순회할 때, 키가 어떤 타입인지 정확히 파악하고 있다면 `let k : keysof T`와 `for-in` 루프를 사용합시다. 함수의 매개변수로 쓰이는 객체에는 추가적인 키가 존재할 수 있다는 점을 명심합시다. <br/> 💡객체를 순회하며 키와 값을 얻는 가장 일반적인 방법은 `Object.entries` 를 사용하는 것입니다. |
