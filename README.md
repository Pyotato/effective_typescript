# 🧾 ITEM 35: 데이터가 아닌, API와 명세 보고 타입 만들기 

이 장의 다른 아이템들에서는 타입을 잘 설계하면 어떠한 이점이 있는지, 반대로 설계가 잘못되면 무엇이 잘못될 수 있는지에 대해서 다루었습니다.
잘 설계된 타입은 타입스크립트 사용을 즐겁게 해주는 반면, 잘못된 설계된 타입은 비극을 불러옵니다.
이러한 양면성 때문에 타입 설계를 잘 해야 한다는 압박감이 느껴질 수 있습니다.
이런 상황에서 타입을 직접 작성하지 않고 자동으로 생성할 수 있다면 매우 유용할 겁니다.

파일 형식, API, 명세 (specification) 등 우리가 다루는 타입 중 회소한 몇 개는 프로젝트 외부에서 비롯된 것 입니다.
이러한 경우는 타입을 직접 작성하지 않고 자동으로 생성할 수 있습니다.
여기서 핵심은, 예시 데이터가 아니라 명세를 참고해 타입을 생성한다는 것입니다.
명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있게 도와줍니다.
반면에 예시 데이터를 참고해 타입을 생성하면 눈앞에 있는 데이터들만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있습니다.

[아이템 29](https://github.com/Pyotato/effective_typescript/blob/item31/README.md)에서 Feature의 경계 상자를 계산하는 calcilateBoundingBox 함수를 사용했습니다.
실제 구현은 다음과 같은 모습입니다.

```ts
function calculateBoundingBox(f:Feature):BoundingBox | null{
  let box: BoundingBox | null;
  const helper = (coords: any[]) => {
    // ...
  }
  const {geometry} = f;
  if(geometry){
    helper(geometry.coordinates);
  }
  return box;
}
```






































