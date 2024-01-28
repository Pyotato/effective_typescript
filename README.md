#  ＠ ITEM 45 : devDependencies에 typescript와 @types 추가하기

npm(node package manager)은 자바스크립트 세장에서 필수적입니다.
npm은 자바스크립트 라이브러리 저장소 (npm 레지스트리)와, 프로젝트가 의존하고 있는 라이브러리들의 버전을 지정하는 방법 (<i>package.json</i>)을 제공합니다.

npm은 세 가지 종류의 의존성을 구분해서 관리하며, 각각의 의존성은 <i>package.json</i> 파일 내의 별도 영역에 들어 있습니다.

- dependencies
  - 현재 프로젝트를 실행하는데 필수적인 라이브러리들이 포함됩니다. 프로젝트의 런타임에 lodash가 사용된다면 dependencies에 포함되어야 합니다. 프로젝트를 npm에 공개하여 다른 사용자가 해당 프로젝트를 설치한다면, dependencies에 들어 있는 라이브러리도 함께 설치될 것입니다. 이러한 현상을 정이 (transitive) 의존성이라고 합니다.
- devDependencies
  - 현재 프로젝트를 개발하고 테스트하는 데 사용되지만, 런타임에는 필요 없는 라이브러리들이 포함됩니다. 예를 들어, 프로젝트에서 사용 중인 테스트 프레임워크가 devDependencies에 포함될 수 있는 라이브러리입니다. 프로젝트를 npm에 공개하여 다른 사용자가 해당 프로젝트를 설치한다면, devDependencies에 포함된 라이브러리들은 제외된다는 것이 dependencies와 다른 점입니다.
- peerDependencies
  - 런타임에 필요하긴 하지만, 의존성을 직접 관리하지 않는 라이브러리들이 포함됩니다. 단적인 에로 플러그인을 들 수 있습니다. 제이쿼리의 플러그인은 다양한 버전의 제이쿼리와 호환되므로 제이쿼리의 버전을 플러그인에서 직접 선택하지 않고, 플러그인이 사용되는 실제 프로젝트에서 선택하도록 만들 떄 사용합니다.
 
이 세 가지 의존성 중에서 dependencies와 devDependencies가 일반적으로 사용됩니다.
타입스크립트 개발자라면 라이브러리를 추가할 때 어떤 종류의 의존성을 사용해야 하는지 알고 있어야 합니다.
타입스크립트는 개발 도구일 뿐이고 타입 정보는 런타임에 존재하지 않기 때문에 ([아이템 3](https://github.com/Pyotato/effective_typescript/blob/item3/README.md)), 타입스크립트와 관련된 라이브러리는 일반저긍로 devDependencies에 속합니다. 

모든 타입스크립트 프로젝트에서 공통적으로 고려해야 할 의존성 두 가지를 살펴보겠습니다.

첫 번째, 타입스크립트 자체 의존성을 고려해야 합니다.
타입스크립트를 시스템 레벨로 설치할 수도 있지만, 다음 두 가지 이유때문에 추천하지는 않습니다.

- 팀원들 모두가 항상 동일한 버전을 설치한다는 보장이 없습니다.
- 프로젝트를 셋업할 때 별도의 단계가 추가됩니다.

따라서 타입스크립트를 시스템 레벨로 설치하기보다는 devDependencies에 넣는 것이 좋습니다.
devDependencies에 포함되어 있다면, `npm install`을 실행할 때 팀원들 모두 항상 정확한 버전의 타입스크립트를 설치할 수 있습니다.
그리고 타입스크립트 버전 업데이트는 다른 라이브러리의 업데이트와 같은 방법을 사용하게 됩니다.

대부분의 타입스크립트 IDE와 빌드 도구는 devDependencies를 통해 설치된 타입스크립트의 버전을 인식할 수 있도록 되어 있습니다. 
또한 커맨드 라인에서 npx를 사용해서 devDependencies룰 통해 설치된 타입스크립트 컴파일러를 실행할 수 있습니다.

> $ npx tsc

두 번째, 타입 의존성 (`@types`)을 고려해야 합니다.
사용하려는 라이브러리에 타입 선언이 포함되어 있지 않더라도, `DefinitelyTyped` (타입스크립트 커뮤니티에서 유지보수하고 있는 자바스크립트 라이브러리의 타입을 정의한 모음)에서 타입 정보를 얻을 수 있습니다. `DefinitelyTyped`의 타입 정의들은 npm 레지스트리의 `@types` 스코프에 공개됩니다. 즉, `@types/jquery`에는 제이쿼리의 타입 정의가 있고, `@types/lodash`에는 로대시의 타입 정의가 있습니다.
`@types` 라이브러리는 타입 정보만 포함하고 있으며 구현체는 포함하지 않습니다.

원본 라이브러리 자체가 dependencies에 있더라도 `@types` 의존성은 devDependencies에 있어야 합니다.
예를 들어, 리액트의 타입 선언과 리액트를 의존성에 추가하려면 다음처럼 실행합니다.

> $ npm install react <br/>
> $ npm install --save-dev @types/react

그러면 다음과 같은 <i>package.json</i> 파일이 생성됩니다. 

```json
{
  "devDependencies": {
      "@types/react": "^16.8.19",
      "typescript": "3.5.3",
  },
  "dependencies": {
      "react": "^16.8.19",
  },
}
```

이 예제의 의도는 런타임에 `@types/react`와 typescript에 의존하지 않겠다는 것입니다.
그러나 타입 의존성을 devDependencies에 넣는 방식이 항상 유효한 것은 아니며 `@types` 의존성과 관련된 몇 가지 문제점이 있습니다.
`@types` 의존성과 관련된 문제는 [아이템 46](https://github.com/Pyotato/effective_typescript/blob/item46/README.md)에서 자세히 다룹니다.
