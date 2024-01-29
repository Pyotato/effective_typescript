# 🪞 ITEM 51 : 의존성 분리를 위해 미러 타입 사용하기

CSV 파일을 파싱하는 라이브러리를 잓겅한다고 가정해 보겠습니다. parseCSV API는 간단합니다.
CSV 파일의 내용을 매개변수로 받고, 열 이름을 값으로 매핑하는 객체들을 생성하여 배열로 반환합니다.
그리고 NodeJS 사용자를 위해 매개변수에 Buffer 타입을 허용합니다.

```ts
function parseCVS(contents: string | Buffer):{[column: string ]: string}[]{
  if(typeof contents === 'object'){
    // 버퍼인 경우
    return parseCSV(contents.toString('utf8'));
  }
}
```

Buffer 타입 정의는 NodeJS 타입 선언을 설치해서 얻을 수 있습니다.

> $ npm install --save-dev @types/node

앞에서 작성한 CVS 파싱 라이브러리를 공개하면 타입 선언도 포함하게 됩니다. 그리고 타입 선언이 `@types/node`에 의존하기 때문에 `@types/node`는 devDependencies를 포함해야 합니다 ([아이템 45](https://github.com/Pyotato/effective_typescript/blob/item45/README.md)). 그러나 `@types/node`를 devDependencies로 포함하면 다음 두 그룹의 라이브러리 사용자들에게 문제가 생깁니다.

- `@types` 와 무관한 자바스크립트 개발자
- NodeJS와 무관한 타입스크립트 개발자

두 그룹의 사용자들은 각자가 사용하지 않는 모듈이 포함되어 있기 때문에 혼란스러울 겁니다.
Buffer는 NodeJS 개발자만 필요합니다. 그리고 `@types/node`는 NodeJS와 타입스크립트를 동시에 사용하는 개발자만 관련됩니다.

각자가 필요한 모듈만 사용할 수 있도록 구조적 타이핑 ([아이템 4](https://github.com/Pyotato/effective_typescript/blob/item4/README.md))을 적용할 수 있습니다. 
`@types/node`에 있는 Buffer 선언을 사용하지 않고, 필요한 메서드와 속성만 별도로 작성할 수 있습니다.
앞선 예제의 경우에는 인토딩 정보를 매개 변수로 받는 toString 메서드를 가지는 인터페이스르 별도로 만들어 사용하면 됩니다.

```ts
interface CsvBuffer{
  toString(encoding : string) :string;
}
function parseCVS(contents: string | CsvBuffer):{[column: string ]: string}[]{
  // ...
}
```

CsvBuffer는 Buffer 인터페이스보다 훨씬 짧으면서도 실제로 필요한 부분만을 떼어 내여 명시했습니다.
또한 해당 타입이 Buffer와 호환되기 때문에 NodeJS 프로젝트에서는 실제 Buffer 인스턴스로 parseCSV를 호출하는 것이 가능합니다.

```ts
parseCVS(new Buffer("column1,column2\nval1,val2","utf-8")); // 정상
```

만약 작성 중인 라이브러리가 의존하는 라이브러리의 구현과 무관하게 타입에만 의존한다면, 필요한 선언부만 추출하여 작성 중인 라이브러리에 넣는 것 (미러링, mirroring)을 고려해 보는 것도 좋습니다.
NodeJS 기반 타입스크립트 사용자에게는 변화가 없지만, 웹 기반이나 자바스크립트 등 다른 모든 사용자에게는 더 나은 사양을 제공할 수 있습니다.

다른 라이브러리의 타입이 아닌 구현에 의존하는 경우에도 동일한 기법을 적용할 수 있고 타입 의존성을 피할 수 있습니다.
그러나 프로젝트의 의존성이 다양해지고 필수 의존성이 추가됨에 따라 미러링 기법을 적용하기가 어려워집니다.
다른 라이브러리의 타입 선언의 대부분을 추출해야 한다면, 차라리 명시적으로 `@types` 의존성을 추가하는 게 낫습니다.

미러링 기법은 유닛 테스트와 상용 시스템 간의 의존성을 분리하는 데도 유용합니다. 자세한 내용은 [아이템 4의 getAuthors 예제](https://github.com/Pyotato/effective_typescript/blob/item4/README.md#getAuthors)를 참고하시길 바랍니다.


























