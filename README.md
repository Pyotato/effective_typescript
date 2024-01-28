# 🧵 ITEM 33 : string 타입보다 더 구체적인 타입 사용하기

string 타입의 범위는 매우 넓습니다.
"x"나 "y"같은 한 글자도, '모비 딕(Moby Dick', "Call me Ishmael..."로 시작하는 약 120만 글자의 소설)의 전체 내용도 string 타입입니다.
string타입으로 변수를 선언하려 한다면, 혹시 그보다 더 좁은 타입이 적절하지 않을지 검토해 보야야 합니다.

```ts
interface Album {
  artist: string;
  title: string;
  releaseDate: string;       //  YYYY-MM-DD
  recordingType: string;     // 예를 들어, 'live' 또는 'studio' 
}
```

string 타입이 남발된 모습입니다.
게다가 주석에 타입 정보를 적어 둔 걸 보면 ([아이템 30](https://github.com/Pyotato/effective_typescript/blob/item30/README.md)) 현재 인터페이스가 잘못되었다는 것을 알 수 있습니다.
다음 예시처럼 Album 타입에 엉뚱한 값을 설정할 수 있습니다.

```ts
const kindOfBlue: Album = {
  artist: 'Miles Davis',
  title: 'Kind of Blue',
  releaseDate: 'August 17th, 1959',      
  recordingType: 'Studio',   
}
```

`releaseDate` 필드의 값은 주석에 설명된 혁식과 다르며, `recordingType` 필드의 값 "Studio"는 소문자 대신 대문자가 쓰였습니다.
그러나 이 두 값 모두 문자열이고, 해당 객체는 Album 타입에 할당 가능하며 타입 체커를 통과합니다.

또한 string 타입의 범위가 넓기 때문에 제대로 된 Album 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않습니다.

```ts
function recordRelease(title: string, date: string){/** */}
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title); // 오류여야 하지만 정상
```

`recordRelease` 함수의 호출에서 매개변수들의 순서가 바뀌었지만, 둘 다 문자열이기 때문에 타입 체커가 정상으로 인식합니다.
앞의 예제처럼 string 타입이 남용된 코드를 "문자열을 남발하여 선언되었다 (stringly typed)'고 표현하기도 합니다.

앞의 오류를 방지하기 위해 타입의 범위를 좁히는 방법을 생각해 보겠습니다.
억지스러운 예시일 수 있지만. 가수 이름이나 앨범 제목으로 '모비 딕'의 전체 텍스트가 쓰일 수도 잇는 일입니다.
그러므로 artist나 title 같은 필드에는 string 타입이 적절합니다.
그러나 releaseDate 필드는 Date 객체를 사용해서 날짜 형식으로만 제한하는 것이 좋습니다.
recordingType 필드는 "live"외 "studio", 단 두 개의 값으로 유니온 타입을 정의할 수 있습니다.
(enum을 사용할 수도 있지만 일반적으로는 추천하지 않습니다. [아이템 53](https://github.com/Pyotato/effective_typescript/blob/item53/README.md) 참고).

```ts
type RecordingType = 'studio' | 'live';

interface Album {
  artist: string;
  title: string; 
  releaseDate: Date;
  recordingType: RecordingType;
}
```

위의 코드처럼 바꾸면 타입스크립트는 오류를 더 세밀하게 체크합니다. 

<img width="789" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/559633d4-0009-4f92-a722-a4b4da98a730"/>

이러한 방식에는 세 가지 장점이 더 있습니다.

첫 번째, 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전닫되어도 타입 정보가 유지됩니다.
예를 들어, 특정 레코딩 타입의 앨범을 찾는 함수를 작성한다면 다음처럼 정의할 수 있습니다.

```ts
function getAlbumsOfType(recordingType: string): Album [] {/** */}
```

`getAlbumsOfType`함수를 호출하는 곳에서 `recordingType`의 값이 string 타입이어야 한다는 것 외에는 다른 정보가 없습니다.
주석으로 써놓은 "studio" 또는 "live" Album의 정의에 숨어 있고, 함수를 사용하는 사람은 recordingType "studio" 또는 "live"여야 한다는 것을 알 수 없습니다.

두 번째, 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있습니다 ([아이템 48](https://github.com/Pyotato/effective_typescript/blob/item48/README.md)).

`getAlbumsOfType`이 받는 매개변수를 string 대신 RecordingType으로 바꾸면, 함수를 사용하는 곳에서 RecordingType의 설명을 볼 수 있습니다. 

<img width="624" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/d85fe9f4-2325-44c8-b214-149657294e4f"/>

세 번째, `keyof` 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해집니다.
(`keyof` 관련 내용이 이번 아이템 마지막까지 이어지므로 집중하시길 바랍니다.)

함수의 매개변수에 string을 잘못 사용하는 일은 흔합니다.
어떤 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 생각해 보겠습니다.
실제로 언더스코어(underscore)라이브러리에는 pluck이라는 함수가 있습니다.

```ts
function pluck(records, key){
  return records.map(r => r[key]);
}
```

pluck 함수의 시그니처를 다음처럼 작성할 수 있습니다.

```ts
function pluck(records: any[], key: string): any[]{
  return records.map(r => r[key]);
}
```

타입 체크가 되긴 하지만 any 타입이 있어서 정밀하지 못합니다.
특히 반환 값에 any를 사용하는 것은 매우 좋지 않은 설계입니다 ([아이템 38](https://github.com/Pyotato/effective_typescript/blob/item38/README.md)).

```ts
function pluck<T>(records: T[], key: string): any[]{
  return records.map(r => r[key]);
                        // ----- : '{}' 형식에 인덱스 시그니처가 없으므로 요소에 암묵적으로 'any' 형식이 있습니다.
}
```

이제 타입스크립트는 key의 타입이 string이기 때문에 범위가 너무 넓다는 오류를 발생시킵니다.
Album의 배열을 매개변수로 전달하면 기존의 string 타입의 넓은 범위와 반대로, key는 단 네 개의 값 ("artist","title","releaseDate", "recordingType") 만이 유효합니다.
다음 예시는 `keyof Album` 타입으로 얻게 되는 결과입니다.

```ts
type K = keyof Album;
// 타입이 "artist" | "title" | "releaseDate" | "recordingType"
```

그러므로 string을 `keyof T`로 바꾸면 됩니다.

```ts
function pluck<T>(records: T[], key: keyof T): any[]{
  return records.map(r => r[key]);
}
```

이 코드는 타입 체커를 통과합니다.
또한 타입스크립트가 반환 타입을 추론할 수 있게 해 줍니다.
pluck 함수에 마우스를 올려 보면, 추론된 타입을 알 수 있습니다. 

```ts
function pluck<T>(records: T[], key: keyof T): T[keyof T][]{
  return records.map(r => r[key]);
}
```

`T[keyof T][]`는 T객체 내의 가능한 모든 값의 타입입니다. 

그런데 key의 값으로 하나의 문자열을 넣게 되면, 그 범위가 너무 넓어서 적절한 타입이라고 보기 어렵습니다.
예를 들어 보겠습니다. 

```ts
const releaseDates = pluck(albums, 'releaseDate'); // 타입이 (string | Date)[]
```

releaseDates의 타입은 (string | Date)[]가 아니라 Date[]이어야 합니다.
`keyof T`는 string에 비하면 훨씬 범위가 좁기는 하지만 그래도 여전히 넓습니다.
따라서 범위를 더 좁히기 위해서, `keyof T`의 부분 집합 (아마도 단일 값)으로 두 번째 제너릭 매개변수를 도입해야 합니다.

```ts
function pluck<T, K extends keyof T>(records: T[], key: K):T[K][]{
  return records.map(r => r[key]);
}
```

([아이템 14](https://github.com/Pyotato/effective_typescript/blob/item14/README.md)에서 extends에 대해서 다룹니다.)

이제 타입 시그니처가 완벽해졌습니다.
pluck을 여러 가지 방법으로 호출하면서 제대로 반호나 타입이 추론되는지, 무효한 매개변수를 방지할 수 있는지 확인해 볼 수 있습니다.

```ts
pluck(albums, 'releaseDate'); // Date[] 타입
pluck(albums, 'artist'); // string[] 타입
pluck(albums, 'recordingType'); // RecordingType [] 타입
pluck(albums, 'recordingDate');
              // ------------ : "'recordingDate'" 형식의 인수는 ... 형식의 매개변수에 할당될 수 없습니다.
```

매개변수 타입이 정밀해진 덕분에 언어 서비스는 Album의 키에 자동 완성 기능을 제공할 수 있게 해줍니다 

<img width="539" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/96bb1e9c-0d6d-4f9e-a5b1-f352783fb277"/>

string은 any 타입과 비슷한 문제를 가지고 있습니다.
따라서 잘못 사용하게 되면 무효한 값을 허용하고 타입 간의 관계도 감추어 버립니다. 
이러한 문제점은 타입 체커를 방해하고 실제 버그를 차지 못하게 만듭니다.
타입스크립트에서 string의 부분 집합을 정의할 수 있느 기능은 자바스크립트 코드에 타입 안전성을 크게 높입니다.
보다 정확한 타입을 사용하면 오류를 방지하고 코드의 가독성도 향상시킬 수 있습니다.






