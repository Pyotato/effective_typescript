# 🏁 ITEM 62 : 마이그레이션의 완성을 위해 noImplicitAny 설정하기

프로젝트 전체를 <i>.ts</i>로 전환했다면 매우 큰 진척을 이룬 것이지만, 마지막 단계가 남아 있습니다.
바로 `noImplicitAny`를 설정하는 것입니다 ([아이템 2](https://github.com/Pyotato/effective_typescript/blob/item2/README.md)).
`noImplicitAny`가 설정되지 않은 상태에서는 타입 선언에서 비롯되는 실제 오류가 숨어 있기 때문에 마이그레이션이 완료되었다고 하 수 없습니다.

예를 들어 보겠습니다. 
Chart 클래스에 속성 타입 선언을 추가 할 때 '누락된 모든 멤버 추가'라는 빠른 수정 기능을 사용했고 ([아이템 61](https://github.com/Pyotato/effective_typescript/blob/item61/README.md)), 문맥 정보가 부족해서 indices는 any타입으로 추론되었습니다.

```ts
class Char{
  indices: any;
  // ...
}
```

indices는 숫바의 배열인 것처럼 보이므로 number[] 타입으로 수정해 보겠습니다.

```ts
class Char{
  indices: number[];
  // ...
}
```

오류는 사라졌고 문제가 없는 것처럼 보이지만, 사실 number[]는 잘못된 타입이었습니다.
다음은 클래스의 다른 부분에 있는 함수입니다.

```ts
getRanges(){
  for(const r of this.indices){
    const low = r[0];
    const high = r[1];
  }
}
```

<img width="276" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/06fb1e70-2f43-4e2f-9c34-6dde224a2a7e"/>

명백히 `number[][]` 또는 `[number, number][]`가 정확한 타입입니다.
그러나 현재 indices가 number[]로 선언되어 있기 때문에 r은 number 타입으로 추론됩니다.
r이 number 타입인데 배열 인덱스 접근에 오류가 없다는 것에 주목합시다.
이처럼 `noImplicitAny`을 설정하지 않으면, 타입 체크는 매우 허술해집니다.

`noImplicitAny`를 설정하면 다음과 같은 오류가 발생합니다.

<img width="1599" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f73682fd-dd69-48f1-944b-c368e8671d16"/>

처음에는 `noImplicitAny`를 로컬에만 설정하고 작업하는 것이 좋습니다.
왜냐하면 원격에서는 설정에 변화가 없기 때문에 빌드가 실패하지 않기 때문입니다.
로컬에서만 오류로 인식되기 ㅒㄸ문에 수정된 부분만 커밋할 수 있어서 점진적 마이그레이션이 가능합니다.
한편, 타입 체커가 발생하는 오류의 개수는 `noImplicitAny`와 관련된 작업의 진척도를 나타내는 지표로 활용할 수 있습니다.

타입 체크의 강도를 높이는 설정에는 여러가지가 있습니다.
이번 아이템에서 설명하고 있는 `noImplicitAny`는 상당히 엄격한 설정이며, `strictNullChecks`같은 설정을 적용하지 않더라도 대부분의 타입 체크를 적용한 것으로 볼 수 있습니다.
그리고 최종적으로 가장 강력한 설정은 `"strict": true`입니다. 
타입 체크의 강도는 팀 내의 모든 사람이 타입스크립트에 익숙해진 다음에 조금씩 높이는 것이 좋습니다.
