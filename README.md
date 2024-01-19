#  🧑‍🤝‍🧑 ITEM 25 : 비동기 코드에는 콜백대신 async 함수 사용하기

과거의 자바스크립트에서는 비동기 동작을 모델링하기 위해 콜백을 사용했습니다.
그렇기 때문에 악명 높은 '콜백 지옥(callback hell)'을 필연적으로 마주할 수밖에 없었습니다.

```js
fetchURL(url1, function(response1){
  fetchURL(url2, function(response2)){
    fetchURL(url2, function(response2)){
      // ...
      console.log(1);
    }
    console.log(2);
  }
  console.log(3);
});
console.log(4);

// 4
// 3
// 2
// 1
```

로그에서 보면 알 수 있듯이, 실행의 순서는 코드의 순서와 반대입니다.
이러한 콜백이 중첩된 코드는 직괒적으로 이해하기 어렵습니다.
요청들은 병렬로 실행하거나 오류 상황을 빠져나오고 싶다면 더욱 혼란스러워집니다.

ES2015는 콜백 지옥을 극복하가 위해 프로미스(promise) 개념을 도입했습니다.
프로미스느 미래에 가능해질 어떤 것을 나타냅니다 (future라고 부르기도 합니다).
다음은 프로미스를 사용해 앞의 코드를 수정한 것입니다.

```js
const page1Promise = fetch(url);

page1Promise.then(response1 => {
  return fetch(url2);
}).then(response2 => {
  return fetch(url3);
}).then(response3 => {
  // ....
}).catch(error => {
  // ...
});
```

코드의 중첩도 적어졌고 실행 순서도 코드 순서와 같아졌습니다.
또한 오류를 처리하기도, `Promise.all`같은 고급 기법을 사용하기도 더 쉬워졌습니다.

ES2017에서는 async와 await 키워드를 도입하여 콜백 지옥응ㄹ 더욱 간단하게 처리할 수 있게 되었습니다.

```js
async funtion fetchPages(){
  const response1 = await fetch(url1);
  const response2 = await fetch(url2);
  const response3 = await fetch(url3);
  // ...
}
```

`await 키워드`는 각각의 프로미스가 처리 (resolve)될 때까지 fetchPages 함수의 실행을 멈춥니다.
async 함수 내에서 await 중인 프로미스가 거절(reject)되면 예외를 던집니다.
이를 통해 일반적인 `try/catch`구문을 사용할 수 있습니다.

```js
async funtion fetchPages(){
  const response1 = await fetch(url1);
  const response2 = await fetch(url2);
  const response3 = await fetch(url3);
  // ...
}
```

ES5 또는 더 이전 버전을 대상으로 할 때, 타입스크립트 컴파일러는 async와 await가 동작하도록 정교한 변환을 수행합니다.
다시 말해, 타입스크립트는 런타임에 관계없이 `async/await`를 사용할 수 있습니다.

콜백보다는 프로미스나 `async/await`를 사용해야 하는 이유는 다음과 같습니다.

- 콜백보다는 프로미스가 코드를 작성하기 쉽습니다.
- 콜백보다는 프로미스가 타입을 추론하기 쉽습니다.

예를 들어, 병렬로 페이지를 로드하고 싶다면 `Promise.all`을 사용해서 프로미스를 조합하면 됩니다.

```js
async funtion fetchPages(){
  const [response1, response2, response3] = await Promise.all([
    fetch(url1), fetch(url2), fetch(url3)
  ]);
  // ...
}
```

이런 경우는 await와 구조 분해 할당이 찰떡궁합닙니다.

타입스크립트는 세 가지 response 변수 각각의 타입을 Response로 추론합니다.
그러나 콜백 스타일로 동일한 코드를 작성하려면 더 많은 코드와 타입 구문이 필요합니다.

```ts
function fetchPagesCB(){
  let numDone = 0;
  const responses = string [] = [];
  const done = () => {
    const [response1, response2, response3] = responses;
    // ...
  }
  const urls = [url1, url2, url3];
  urls.forEach((url, i) => {
    fetchURL(url, r => {
      response[i] = url;
      numDone++;
      if(numDone === urls.length) done();
    });
  });
}
```

이 코드에 오류 처리를 포함하거나 `Promise.all` 같은 일반적인 코드로 확장하는 것은 쉽지 않습니다.

한편 입력된 프로미스들 중 첫 번째가 처리될 때 완료되는 `Promise.race`도 타입 추론과 잘 맞습니다.
`Promise.race`를 사용하여 프로미스에 타임아웃을 추가하는 방법은 흔하게 사용되는 패턴입니다.

```ts
function timeout(millis: number): Promise<never>{
  return new Promiuse({resolve, reject}=>{
    setTimeout(()=> reject('timeout', millis);
  });
}

async function fetchWithTimeout(url: string, ms: number){
  return Promise.race([fetch(url), timeout(ms)]);
}
```

타입 구문이 없어도 `fetchWithTimeout`의 반환타입은 `Promise<Response>`로 추론됩니다. 
추론이 동작하는 이유는 살펴보면 흥미로운 점을 발견할 수 있습니다.
Promise.race의 반환 타입은 입력 타입들의 유니온이고, 이번 경우는 `Promise<Response | never>`가 됩니다.
그러나 never(공집합)와의 유니온은 아무런 효과가 없으므로, 결과가 `Promise<Response>`인 타입 추론이 제대로 동작합니다.

가끔 프로미스를 직접 생성해야 할 때, 특히 setTimeout과 같은 콜백 API를 래핑할 경우가 있습니다.
그러나 선택의 여지가 있다면 일반적으로는 프로미스를 생성하기보다는 `async/await`를 사용해야 합니다. 
그 이유는 다음 두 가지입니다.

- 일반적으로 더 간결하고 직관적인 코드가 됩니다.
- async함수는 항상 프로미스를 반환하도록 강제됩니다.

```ts
// function getNumber(): Promise<number>{
  async function getNumber(){
    return 42;
  }
}
```

async 화살표 함수를 만들 수도 있습니다.

```ts
const getNumber = async () => 42; // 타입이 () => Promise<number>
```

프로미스르 직접 생성하면 다음과 같습니다.

```ts
const getNumber = Promise.resolve(42); // 타입이 () => Promise<number>
```

즉시 사용 가능한 값에도 프로미스를 반환하는 것이 이상하게 보일 수 있지만, 실제로는 비동기 함수로 통일하도록 강제하는 데 도움이 됩니다.
함수는 항상 동기 똔는 항상 비동기로 실행되어야 하며 절대 혼용해서는 안 됩니다.
예를 들어 fetchURL 함수에 캐시를 추가하기 위해 다음처럼 시도해 봤다고 가정해 보겠습니다.

```ts
// 이렇게 하지 맙시다!
const _cache : {[url: string]:string} = {};
function fetchWithCache(url: string, callback: (text: string) => void){
  if(url in _cache){
    callback(_cache);
  } else {
    fetchURl(url, text => {
      _cache[url] = text;
      callback(text);
    });
  }
}
```

코드가 최적화된 것처럼 보일지 몰라도, 캐시된 경우 콜백함수가 동기로 호출되기 때문에 fetchWithCache 함순는 이제 사용하기가 무척 어려워집니다.

```ts
let requestState = 'loading' | 'success' | 'error';
function getUser(userId: string){
  fetchWithCache(`/user/${userId}`, profile = > {
    requestStatus = 'success';
  });
    requestStatus = 'loading';
}
```

getUser를 호출한 후에 requestStatus의 값은 온전히 profile이 캐시되었는지 여부에 달렸습니다. 
캐시되어 있지 않다면 requestStatus는 조만간 'success'기 됩니다.
캐시되어 있다면 'success'가 되고 나서 바로 'loading'으로 다시 돌아가 버립니다.

async를 두 함수에 모두 사용하면 일관적인 동작을 강제하게 됩니다.

```ts
const _cache = {[url : string]: string} = {};
async function fetchWithCache(url : string){
  if(url in _cache){
    return _cacheUrl;
  }
  const response = await fetch (url);
  const text = await response.text();
  _cache[url] = text;
  return text;
}

let requestState: 'loading' | 'success' | 'error';
async function getUser(userId: string){
  requestStatus = loading;
  const profile = await fetchWithCache(`/user/${userId}`);
  requestStatus = 'success';
}

```

이 예제 requestState가 'success'로 끝나는 것이 명백해졌습니다.
콜백이나 프로미스를 사용하면 실수로 반(half)동기 코드를 작성 할 수 있지만, async를 사용하면 항상 비동기를 사용하는 셈입니다.

async함수에서 프로미스를 반환하면 또 다른 프로미스로 래핑되지 않습니다.
반환타입은 Promise<Promise<T>>가 아닌 Promise<T>가 됩니다.
타입스크립트를 사용하면 타입 정보가 명확히 드러나기 때문에 비동기 코드의 개념을 잡는데 도움이 됩니다. 

```ts
//function getJSOn(url : string): Promise<any>
async function getJSON(url:string){
  const response = null fetch(url);
  const jsonPromise = response.json();
  return jsonPromise;
}

```





















