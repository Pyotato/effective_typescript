# 📛 ITEM 24 : 일관성있는 별칭 사용하기

어떤 값에 새 이름릉 할당하는 예제를 보겠습니다.

```ts
const borough = {name: 'Brooklyn', location: [40.688, -73.979]};
const loc = borough.location;
```

`borough.location` 배열에 loc이라는 별칭(alias)을 만들었습니다.
별칭의 값을 변경하면 원래 속성값에서도 변경됩니다.

```ts
loc[0] = 0;
borough.location
// [0,-73.979]
```

그런데 별칭을 남발해서 사용하면 제어 흐름을 분석하기 어렵습니다.
모든 언어의 컴파일러 개발자들은 무분별한 별칭 사용으로 골치를 찍고 있습니다.
타입스크립트에서도 마참가지로 별칭을 신중하게 사용해야 합니다.
그래야 코드를 잘 이해할 수 있고, 오류도 쉽게 찾을 수 있습니다.

다각형을 표현하는 자료구조를 가정해 보겠습니다.

```ts
interface Coordinate {
  x: number;
  y: number;
}

interface BoundingBox {
  x: [number, number];
  y: [number, number];
}

interface Polygon {
  exterior : Coordinate[];
  holes: Coordinate[][];
  bbox?: BoundingBox;
}
```

다각형의 기하학적 구조는 exterior와 holes 속성으로 정의됩니다.
bbox는 필수가 아닌 최적화 속성입니다. bbox 속성을 사용하면 어떤 점이 다각형에 포함된는지 빠르게 체크할 수 있습니다.

```ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate){
  if(polygon.bbox){
    if(pt.x < polygon.bbox.x[0] || pt.x > polygon.bbox.x[1] || pt.y < polygon.bbox.y[0] || pt.y > polygon.bbox.y[1] ){
      return false;
    }
  }
}
```

이 코드는 잘 동작하지만 (타입 체크도 통과) 반복되는 부분이 존재합니다.
특히 polygon.bbox는 3줄에 걸쳐 5번이나 등장합니다. 다음 코든는 중복을 줄이기 위해 임시 변수를 뽑아낸 모습입니다.

```ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate){
  const box = polygon.bbox;
  if(polygon.bbox){
    if(pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1] ){
      return false;
    }
  }
}
```

<img width="597" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/f4b222b4-093e-436a-99f9-f4302faa969e"/>

(strictNullChecks를 활성화했다고 가정했습니다.)

이 코드는 동작하지만 편집기에서 오류로 표시됩니다. 그 이유는 `polygon.bbox`를 별도의 box라는 별칭을 만들었고, 첫 번째 예제에서는 잘 동작했던 제어 흐름 분석을 방해했기 때문입니다.

어떤 동작이 이루어졌는지 box와 `polygon.bbox`의 타입을 조사해 보겠습니다.

<img width="448" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/77c1a66e-5576-46be-8188-36155f19145e"/>
<img width="448" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/fd55c29d-01be-4ffa-8cc8-0b5a95f5b243"/>
<img width="448" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/16f22496-91e2-41ef-8370-6cd47c8fca74"/>
<img width="448" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/45dbc8e1-dfda-48c1-8070-36c5c70d6747"/>

속성 체크는 `polygon.bbox`의 타입을 정제했지만 box는 그렇지 않았기 때문에 요류가 발생했습니다.
이러한 오류는 "별칭은 일관성 있게 사용한다"는 기본 원칙(golden rule)을 지키면 방지 할수 있습니다.

속성 체크에 box를 사용하도록 코드를 바꿔보겠습니다.

```ts
function isPointInPolygon(polygon: Polygon, pt: Coordinate){
  const box = polygon.bbox;
  if(box){
    if(pt.x < box.x[0] || pt.x > box.x[1] || pt.y < box.y[0] || pt.y > box.y[1] ){
      return false;
    }
  }
}
```

그러나 객체 비구조화를 이용할 때는 두 가지를 주의해야 합니다.

- 전체 bbox 속성이 아니라 x와 y가 선택적 속성일 경우에 속성 체크가 더 필요합니다. 따라서 타입의 경계에 null 값을 추가하는 것이 좋습니다. ([아이템 31](https://github.com/Pyotato/effective_typescript/blob/item31/README.md))
- bbox에는 선택적 속성이 적합했지만 holes는 그렇지 않습니다. holes가 선택적이라면, 값이 없거나 빈 배열([])이었을 겁니다. 차이가 없는데 이름을 구별한 것입니다. 빈 배열은 'holes 없음'을 나타내는 좋은 방법입니다.

별칭은 타입 체커뿐만 아니라 런타임에도 혼동을 야기할 수 있습니다. 

```ts
const {bbox} = polygon;
if(!bbox){
  calculatePolygonBbox(polygon);  // polygon.bbox가 채워집니다.
  // 이 때 polygon.bbox와 bbox는 다른 값을 참조합니다!
}
```

타입스크립트의 제어 흐름 분석은 지역 변수에는 꽤 잘 동작합니다. 
그러나 객체 속성에서는 주의해야 합니다.

```ts
function fn(p: Polygon){ /*...*/}
polygon.bbox   // 타입이 BoundingBox | undefined
if(polygon.bbox){
  polygon.bbox // 타입이 BoundingBox | undefined
  fn(polygon)
  polygon.bbox // 타입이 BoundingBox
}
```

`fn(polygon)` 호출은 `polygon.bbox`를 제거할 가능성이 있으므로 타입을 `BoundingBox | undefined`로 되돌리는 것이 안전할 것입니다.
그러나 함수를 호출할 때마다 속성 체크를 반복해야 하기 때문에 좋지 않습니다.
그래서 타입스크립트는 함수가 타입 정제를 무효화하지 않는다고 가정합니다.
그러나 실제로는 무효화될 가능성이 있습니다.
`polygon.bbox`로 사용하는 대신 bbox 지역 변수로 뽑아내서 사용하면 bbox의 타입은 정확히 유지되지만 `polygon.bbox`의 값과 같게 유지 되지 않을 수 있습니다.
