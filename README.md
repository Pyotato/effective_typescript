# 🦆 ITEM 4 : 구조적 타이핑에 익숙해지기

자바스크립트는 본질적으로 **덕 타이핑(duck typing)** 기반입니다.

> 덕 타이핑이란, 개체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 적당 타입에 속하는 거으로 간주하는 방식입니다 덕 테스트에서 유래되었는데, 다음과 같은 명제로정의됩니다.
>
> "만약 어떤 새가 오리처럼 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다"

만약 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경쓰지 않고 사용합니다. 
타입스크립트는 이런 동작, 즉 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 동작을 그대로 모델링합니다. 
그런데 타입 체커의 타입에 대한 이해도가 사람과 조금 다르기 때문에 가끔 예상치 못한 결과가 나오기도 합니다. 
구조적 타이핑을 제대로 이해한다면 오류인 경우와 오류가 아닌 경우의 차이를 알 수 있고, 더욱 견고한 코드를 작성할 수 있습니다.

물리 라이브러리와 2D 벡터 타입을 다루는 경우를 가정해 보겠습니다.

```ts
interface Vertor2D{
   x: number;
   y: number;
}
```

벡터의 길이를 계산하는 함수는 다음과 같습니다.

```ts
function calculateLength(v: Vector2D){
  return Math.sqrt(v.x*v.x+v.y*v.y);
}
```

이제 이름이 들어간 벡터를 추가합니다. 

```ts
interface NamedVector{
  name: string;
  x: number;
  y: number;
}
```

NamedVector는 number 타입의 x와 y속성이 있기 때문에 calculateLength 함수로 호출 가능합니다. 타입스크립트는 다음 코드를 이해할 수 있을 정도로 충분히 영리합니다. 

```ts
const v: NamedVector = {x:3, y:4,name: 'Zee'};
calculateLength(v); // 정상, 결과는 5
```

흥미로운 점은 Vector2D와 NamedVector의 관계를 전혀 선언하지 않았다는 것입니다. 그리고 NamedVector를 위한 별도의 calculateLength를 구현할 필요도 없습니다.
타입스크립트 타입 시스템은 자바스크립트의 런타임 동작을 모델링합니다 ([아이템 1](https://github.com/Pyotato/effective_typescript/tree/item1)).
NamedVector의 구조가 Vectir2D와 호환되기 때문에 calculateLength 호출이 가능합니다. 여기서 '구조적 타이핑(structural typing)'이라는 용어가 사용됩니다. 

구조적 타이핑 때문에 문작 발생하기도 합니다. 3D벡터를 만들어 보겠습니다.

```ts

interface Vector3D{
   x: number;
   y: number;
   z: number;
}

```

그리고 벡터의 길이를 1로 만드는 정규화 함수를 작성합니다. 

```
function normalize(v: Vector3D){
   const length = calculateLength(v);
   return {
      x: v.x / length,
      y: v.y / length,
      z: v.z / length, 
   };
}
```

그러나 이 함수는 1보다 조금 더 긴 (1.41) 길이를 가진 결과를 출력할 것입니다.

<img width="1452" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/5ff95e59-f32b-43ac-bd53-e70470935537">

타입스크립트가 오류를 잡지 못한 이유를 생가해 보겠습니다.
calculateLength는 2D벡터를 기반으로 연산하는데, 버그로 인해 normalize가 3D로 연산되었습니다. z가 정규화에서 무시된 것입니다.
그런데 타입 체커가 이 문제를 잡아내지 못했습니다. calculateLength가 2D벡터를 받도록 선언되었음에도 불구하고 3D벡터를 받는 데 문제가 없었던 이유가 뭘까요?

Vector3D와 호환되는 {x,y,z} 객체로 calculateLength를 호출하면, 구조적 타이핑 관점에서 x와 y가 있어서 Vector2D와 호환됩니다. 따라서 오류가 발생하지 않았고, 타입 체커가 문제로 인식하지 않습니다. 
(이런 경우를 오류로 처리하기 위한 설정이 존재합니다 [아이템 37](https://github.com/Pyotato/effective_typescript/tree/item37)에서 자세히 다루겠습니다.)

함수를 작성할 때, 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질 거라고 생각하기 쉽습니다. 이러한 타입은 '봉인된(sealed)' 또는 '정확한(precise)' 타입이라고 불라며, 타입스크립트 타입 시스템에서는 표현할 수 없습니다.
좋든 싫든 타입은 '열려(open)'있습니다. 이러함 특성때문에 가끔 당황스러운 결과가 발생합니다. 

> 타입이 열려있다는 것은, 타입의 확장에 열려있다는 의미입니다. 즉, 타입에 선언된 속성 외에 임의의 속성을 추가하더라도 오류가 발샹허자 않는다는 것입니다.
> 
> 예를 들어, 고양이라는 타입에 크기 속성을 추가하여 '작은 고양이'가 되어도 고양이라는 사실은 변하지 않을 겁니다.
> 그래도 이해하기 어렵다면 [아이템 7](https://github.com/Pyotato/effective_typescript/tree/item7)을 참고하기 바랍니다.

```ts
function calculateLength1(v: Vector3D){
   let length = 0;
   for(const axis of Object.keys(v)){
      const coor = v[axis];
      length+= Math.abs(coord);
   }
   return length;
}
```

<img width="1721" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/cf0f6f72-69a5-4851-a0f4-11fe825d08ae">

오류룰 납득하기 어렵습니다. axis는 Vector3D 타입인 v의 키 중 하나이기 때문에 "x", "y", "z" 중 하나여야 합니다. 
그리고 Vector3D의 선언에 따르면, 이들은 모두 number이므로 coord의 타입이 number가 되어야 할 것으로 예상됩니다.

그러나 사실은 타입스크립트의 오류를 정확하게 찾아낸 것이 맞습니다. 앞에서 Vector3D는 봉인 (다른 속성이 없다)되었다고 가정했습니다. 
그런데 다음 코드처럼 작성할 수도 있습니다. 

```ts
const vec3D = {x:3, y:4, z: 1, address: '123 Broadway'};
calculatedLength1(vec3D); // 정상, NaN 반환
```

v는 어떤 속성이든 가질 수 있기 때문에, axis의 타입은 string이 될 수도 있습니다. 
그러므로 앞서 본 것처럼 타입스크립트는 v[axis]가 어던 속성이 될 지 알 수 없기 number라고 확정할 수 없습니다. 
정확한 타입으로 객체를 순회하는 것은 까다로운 문제입니다. 이 주제는 [아이템 54](https://github.com/Pyotato/effective_typescript/tree/item54)에서 다시 다루겠지만, 이번 경우의 결론을 말하자면 루프보다는 모든 속성을 각각 더하는 구현이 더 낫습니다.

```ts
function calculateLength(v: Vector3D){
   return Math.abs(v.x)+Math.abs(v.y)+Math.abs(v.z);
}
```

구조적 타이핑은 클래스와 관련된 할당문에서도 당황스러운 결과를 보여줍니다.

```ts
class C{
   foo:string;
   constructor(foo:string){
      this.foo = foo;
   }
}

const c = new C('instance Of C');
const d : C = {foo:'object literal'}; // 정상
```

d가 C타입에 할당되는 이유를 알아보겠습니다. d는 string 타입의 foo속성을 가집니다. 
게다가 하나의 매개변수(보통은 매개변수 없이 호출되지만)로 호출이 되는 생성자(Object.prototype으로부터 비롯된)를 가집니다. 
그래서 구조적으로는 필요한 속성과 생성자가 존재하기 떄문에 문제가 없습니다. 
만약 C의 생성자에 단순할당이 아닌 연산 로직이 존재한다면, d의 경우는 생성자를 실행하지 않으므로 문제가 발생합니다.
이러한 부분이 C타입의 매개변수를 선언하여 C 또는 서브클래스임을 보장하는 C++이나 자바같은 언어와 매우 다른 특징입니다.

테스트를 작성할 때는 구조적 타이핑이 유리합니다. 데이터베이스에 쿼리하고 결과를 처리하는 함수를 가정해 보겠습니다.

```ts
interface Author{
   first: string;
   last: string;
}

function getAuthors(database:PostgresDB): Author[]{
   const authorRows = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
   return authorRows.map(row => ({first: row[0], last: row[1]}));
}
```

getAuthors함수를 테스트하기 위해서는 모킹한 PostgresDB를 생성해야 합니다.
그러나 구조적 타이핑을 활용하여 더 구체적인 인터페이스를 정의하는 것이 더 나은 방법입니다.

runQuery 메서드가 있기 때문에 실제 환경에서도 getAuthors에 postgresDB를 사용할 수 있습니다. 
구조적 타이핑 덕분에 postgresDB가 DB 인터페이스를 구현하는지 명확히 선언할 필요가 없습니다.
타입스크립트는 그렇게 동작할 것이라는 것을 알아챕니다.

```ts
test('getAuthors',()=>{
   const authors = getAuthors({
      runQuery(sql:string){
      return [['Toni','Morrison'],['Maya','Angelou']];
   }
   });
   expect(authors).toEqual([
      {first:'Toni',last:'Morrison'},
      {first:'Maya',last:'Angelou'},
   ]);
});
```

타입스크립트는 테스트 DB가 해당 인터페이스를 충족하는지 확인합니다. 그리고 테스트 코드에는 실제 환경의 데이터베이스에 대한 정보가 불필요합니다. 
심지어 모킹한 라이브러리도 필요 없습니다. 추상화(DB)를 함으로써, 로직과 테스트를 특정한 구현(PostgresDB)으로부터 분리한 것입니다.

테스트 이외에 구저작 타이핑의 또 다른 장점은 라이브러리 간의 의존성을 완벽히 분리할 수 있다는 것입니다. 
이 내용은 [아이템 51](https://github.com/Pyotato/effective_typescript/tree/item51)에서 더 자세히 다룹니다.




