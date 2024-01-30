# 🧾 ITEM 35: 데이터가 아닌, API와 명세 보고 타입 만들기 

이 장의 다른 아이템들에서는 타입을 잘 설계하면 어떠한 이점이 있는지, 반대로 설계가 잘못되면 무엇이 잘못될 수 있는지에 대해서 다루었습니다.
잘 설계된 타입은 타입스크립트 사용을 즐겁게 해주는 반면, 잘못된 설계된 타입은 비극을 불러옵니다.
이러한 양면성 때문에 타입 설계를 잘 해야 한다는 압박감이 느껴질 수 있습니다.
이런 상황에서 타입을 직접 작성하지 않고 자동으로 생성할 수 있다면 매우 유용할 겁니다.

파일 형식, API, 명세 (specification) 등 우리가 다루는 타입 중 최소한 몇 개는 프로젝트 외부에서 비롯된 것 입니다.
이러한 경우는 타입을 직접 작성하지 않고 자동으로 생성할 수 있습니다.
여기서 핵심은, 예시 데이터가 아니라 명세를 참고해 타입을 생성한다는 것입니다.
명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있게 도와줍니다.
반면에 예시 데이터를 참고해 타입을 생성하면 눈앞에 있는 데이터들만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있습니다.

[아이템 29](https://github.com/Pyotato/effective_typescript/blob/item29/README.md)에서 Feature의 경계 상자를 계산하는 calcilateBoundingBox 함수를 사용했습니다.
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

Feature 타입은 명시적으로 정의된 적이 없습니다.
[아이템 29](https://github.com/Pyotato/effective_typescript/blob/item29/README.md)에 등장한 focusOnFeature 함수 예제를 사용하여 작성해 볼 수 있습니다.
그러나 공식 GeoJSON 명세를 사용하는 것이 더 낫습니다.

> [GeoJSON](https://geojson.org/)은 RFC 7946으로도 알려져 있습니다. 

다행히도 DefinitelyTyped에는 이미 타입스크립트 타입 선언이 존재합니다.
따라서 다음과 같이 익숙한 방법을 이용해 추가할 수 있습니다.

> $ npm install --save-dev @types/geojson <br/>
> @types/geojson@7946.0.7

GeoJSON 선언을 넣는 순간, 타입스크립트는 오류를 발생시킵니다.

```ts
import {Feature} from 'geojson';

function calculateBoundingBox(f:Feature):BoundingBox | null{
  let box: BoundingBox | null;
  const helper = (coords: any[]) => {
    // ...
  }
  const {geometry} = f;
  if(geometry){
    helper(geometry.coordinates);
                  // ----------: 'Geometry' 형식에 'coordinates' 속성이 없습니다.
                  //             'GeometryCollection' 형식에 'coordingates' 속성이 없습니다.
  }
  return box;
}
```

geometry에 coordinates 속성이 있다고 가정한 게 문제입니다.
이러한 관계는 점,선,다각형을 포함한 많은 도형에서는 맞는 개념입니다.
그러나 GeoJSON은 다양한 (heterogeneous) 도형의 모음인 GeometryCollection일 수도 있습니다.
다른 도형 타입들과 다르게 GeometryCollection에는 coordinate 속성이 없습니다.

geometry가 GeometryCollection 타입인 Feature를 사용해서 calculateBoundingBox를 호출하면 undefined의 0 속성을 읽을 수 없다는 오류를 발생합니다.

이 오류를 고치는 한 가지 방법은 다음 코드처럼 GeometryCollection을 명시적으로 차단하는 것입니다. 

```ts
const {geometry} = f;
  if(geometry){
    if(geomtry.type === 'GeometryCollection'){
      trhow new Error('Geometry Collections are not supported');
    }

    helper(geometry.coordinates); //   정상
  }
```

타입스크립트는 타입을 체크하는 방법으로 도형의 타입을 정제할 수 있으므로 정제된 타입에 한해서 geometry.coordinates의 참조를 허용하게 됩니다.
차단된 GeometryCollection 타입의 경우, 사용자에게 명확한 오류 메시지를 제공합니다.
그러나 GeometryCollection 타입을 차단하기보다는 모든 타입을 지원하는 것이 더 좋은 방법이기 때문에 조건을 분기해서 헬퍼 함수를 호출하면 모든 타입을 지원할 수 있습니다.

```ts

const geometryHelper = (g: Geomtry) => {
    if(geometry.type === 'GeometryCollection'){
      geometry.geometries.forEach(geometryHelper);
    } else {
      helper(geometry.coordinates); // 정상
    }
  }
  const {geometry} = f;
  if(geometry){
    geometryHelper(geometry);
  }

```

그 동안 GeoJSON을 사용해본 경험을 바탕으로 GeoJSON의 타입 선언을 직접 작성했을 수도 있습니다.
아마도 직접 작성한 타입 선언에는 GeometryCollection 같은 예외 상황이 포함되지 않았을 테고 완벽할 수도 없습니다.
반면, 명세를 기반으로 타입을 작성한다면 현재까지 경험한 데이터뿐만 아니라 사용 가능한 모든 값에 대해서 작동한다는 확신을 가질 수 있습니다.

API 호출에도 비슷한 고려 사항들이 적용됩니다.
API의 명세로부터 타입을 생성할 수 있다면 그렇게 하는 것이 좋습니다.
특히, GraphQL처럼 자체적으로 타입이 정의된 API에서 잘 동작합니다.

GraphQL API는 타입스크립트와 비슷한 타입 시스템을 사용하여, 가능한 모든 쿼리와 인터페이스를 명세하는 스키마로 이루어집니다.
우리는 이러한 인터페이스를 사용해서 특정 필드를 요청하는 쿼리를 작성합니다.
예를 들어, GitHub GraphQL API를 사용해서 저장소에 대한 정보를 얻는 코드는 다음처럼 작성할 수 있습니다.

```graphql
query {
  repository(owner: "microsoft", name: "Typescript"){
    createdAt
    description
  }
}
```

결과는 다음과 같습니다.

```json
{
  "data": {
    "repository": {
      "createdAt": "2014-06-17T15:28:39Z",
      "description": "Typescript is a superset of JavaScript that compiles to JavasScript."
    }
  }
}

```

GraphQL의 장점은 특정 쿼리에 대해 타입스크립트 타입을 생성할 수 있다는 것입니다.
GeoJSON 예제와 마찬가지로 GraphQL을 사용한 방법도 타입에 null이 가능한지 여부를 정확하게 모델링할 수 있습니다.

다음 예제는 Github 저장소에 오픈 소스 라이선스를 조회하는 쿼리입니다.

```graphql
query getLicense($owner: String!, $name: String!){
  repository(owner: $owner, name: $name){
    description
    licenseInfo {
      spdxId
      name
    }
  }
}
```

`$owner`와 `$name`은 타입이 정의된 GraphQL 변수입니다.
타입 문법이 타입스크립트와 매우 비슷합니다.
String은 GraphQL의 타입입니다.
타입스크립트애서는 string이 됩니다 ([아이템 10](https://github.com/Pyotato/effective_typescript/blob/item10/README.md)).
그리고 타입스크립트에서 stirng 타입은 null이 불가능하지만 GraphQL의 String 타입에서는 null이 가능합니다.
타입 뒤의 !은 null이 아님을 명시합니다.

GraphQL 쿼리를 타입스크립트 타입으로 변환해 주는 많은 도구가 존재합니다.
그중 하나는 Apollo 입니다.
다음은 Apollo를 어떻게 사용하는지 보여 줍니다.

> $ apollo client: codegen \ <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;--endpoint https://api.github.com/graphql\ <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;--includes license.graphql \ <br/>
> &nbsp;&nbsp;&nbsp;&nbsp;--target typescript <br/>
> Loading Apollo Project <br/>
> Generating query files with 'typescript' target - wrote 2 files <br/>

쿼리에서 타입을 생성하려면 GraphQL 스키마가 필요합니다.
Apollo는 `api.github.com/graphql`로부터 스키마를 얻습니다. 
실행의 결과는 다음과 같습니다.

```ts
export interface getLicense_repository_licenseInfo{
  __typename: 'Licence';
  /** Short identifier specified by <https://spdx.org/licenses> */
  spdxId : string | null;
 /** The license full name specified by <https://spdx.org/licenses> */
  name: string;
}

export interface getLicense_repository{
  __typename: 'Repository';
  /** The description of the repository */
  description : string | null;
 /** The license associated with the repository */
  licenseInfo: getLicense_repository_licenseInfo | null;
}

export interface getLicense{
  /** Lookup a given repository by the owner and repository name */
  repository: getLicense_repository | null;
}

export interface getLicenseVariables{
  owner: string;
  name: string;
}
```

주목할 만한 점은 다음과 같습니다.

- 쿼리 매개변수 (getLicenseVariables)와 응답(getLicense) 모두 인터페이스가 생성되었습니다.
- null 가능 여부는 스키마로부터 응답 인터페이스로 변환외었습니다.
`repository, description, licenseInfo, spdxId` 속성은 null이 가능한 반면, name과 쿼리에 사용된 변수들은 그렇지 않습니다.
- 편집기에서 확인할 수 있도록 주석은 JSDoc으로 변환되었습니다 ([아이템 48](https://github.com/Pyotato/effective_typescript/blob/item49/README.md)). 이 주석들은 GraphQL 스키마로부터 생성되었습니다.

자동으로 생성된 타입 정보는 API를 정확히 사용할 수 있돋록 도와줍니다.
쿼리가 바뀐다면 타입도 자동으로 바뀌며 스키마 바뀐다면 타입도 자동으로 바뀝니다.
타입은 단 하나의 원천 정보인 GraphQL 스키마로부터 생성되기 때문에 타입과 실제값이 항상 일치합니다.

만약 명세 정보가 공식 스키마가 없다면 데이터로부터 타입을 생성해야 합니다.
이를 위해 quicktype같은 도구를 사용할 수 있습니다.
그러나 생성된 타입이 실제 데이터와 일치하지 않을 수 있다는 점을 주의해야 합니다.
예외적인 경우가 존재할 수 있습니다.

우리는 이미 자동 타입 생성의 이점을 누리고 있습니다.
브라우저 DOM API에 대한 타입 선언은 공식 인터페이스로부터 생성되었습니다 ([아이템 55](https://github.com/Pyotato/effective_typescript/blob/item55/README.md))
이를 통해 복잡한 시스템을 정확히 모델링하고 타입스크립트가 오류나 코드 상의 의도치 않은 실수를 잡을 수 있게 합니다.
