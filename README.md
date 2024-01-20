# 🏡 ITEM 31 : 타입 주변에 null 값 배치하기

`strictNullChecks` 설정을 처음 켜면, null이나 undefined 관련된 오류들이 갑자기 나타나기 때문에, 오류를 걸러 내는 if 구문을 코드 전체에 추가해야 한다고 생각할 수 있습니다.
왜냐하면 어떤 변수가 null이 될 수 있는지 없는지를 타입만으로는 명확하게 표현하기 어렵기 때문입니다.
에를 들어 B변수가 A변수의 값으로부터 비롯되는 값이라면, A가 null이 될 수 없을 때 B역시 null이 될 수 없고, 그 반대로 A가 null이 될 수 있다면 B 역시 null이 될 수 있습니다.
이러한 관계들은 겉으로 드러나지 않기 때문에 사람과 타입 체커 모두에게 혼란스럽습니다.

값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분된다면, 값이 섞여 있을 때보다 다루기 쉽습니다. 타입에 null을 추가하는 방식으로 이러한 경우를 모델링할 수 있습니다.

숫자들의 최솟값과 최댓삾을 계산하는 extent함수를 가정해 보겠습니다.

```ts
function extent(nums: number[]){
  let min, max;
  for(const num of nums){
    if(!min){
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min,max];
}
```

이 코드는 타입 체코를 통과하고 (strictNullChecks 없이), 반환타입은 number[]로 추론됩니다.
그러나 여기에는 버그와 함께 설계적 결함이 있습니다.

- 최솟값이나 최댓값이 0인 경우, 값이 덧쓰워져 버립니다. 예를 들어, extent([0,1,2])의 결과는 [0,2]가 아니라 [1,2]가 됩니다.
- nums 배열이 비어 있다면 함수는 [undefined, undefined]를 반환합니다. undefined를 포함하는 객체는 다루기 어렵고 절대 권장하지 않습니다. 
코드를 살펴보면 min과 max가 동시에 둘 다 undefined가 아니라는 것을 알 수 있지만, 이러한 정보는 타입 시스템에서 표현할 수 없습니다.

strictNullChecks 설정을 켜면 앞의 두 가지 문제점이 드러납니다.

<img width="1594" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/973243e9-4bdf-4826-8567-574f52708e56"/>

extent의 반환 타입이 (number | undefined)[]로 추론되어서 설계적 결함이 분명해졌습니다.
이제는 extent를 호출하는 곳마다 타입 오류의 형태로 나타납니다.

<img width="1142" alt="image" src="https://github.com/Pyotato/effective_typescript/assets/102423086/ad7942e0-06c9-4c2b-a0f0-cd7819cd47de"/>

extent 함수의 오류는 undefined를 min에서만 제외했고 max에서는 제외하지 않았기 때문에 발생했습니다.
두 개의 변수는 동시에 초기화되지만, 이러한 정보는 타입 시스템에서 표현할 수 없습니다.
max에 대한 체크를 추가해서 오류를 더 해결할 수도 있지만 버그가 두 배로 늘어날 겁니다.

더 나은 해법을 찾아보겠습니다. min과 max를 한 객체 안에 넣고 null이거나 null이 아니게 하면 됩니다.

```ts
function extent(nums: number[]){
    let result: [number,number] | null = null;
    for(const num of nums){
        if(!result){
            result = [num,num];
        } else {
            result = [Math.min(num, result[0]),Math.max(num, result[1])];
        }
    }
    return result;
}
```

이제는 반환 타입이 `[number,number] | null`이 되어서 사용하기가 더 수월해졌습니다.
null이 아님 단언(!)을 사용하면 min과 max를 얻을 수 있습니다.

```ts
const range = extent([0,1,2]);
if(range){
   const [min,max] = range;
   const span = max - min; 
}  // 정상
```

extent의 결괏값으로 단일 객체를 사용함으로써 설계를 개선했고, 타입스크립트가 null값 사이의 관계를 이해할 수 있도록 했으며 버그도 제거했습니다.
`if(!result)` 체크는 이제 제대로 동작합니다.

null과 null이 아닌 값을 섞어서 사용하면 클래스에서도 문제가 생깁니다.
예를 들어, 사용자와 그 사용자의 포럼 게시글을 나타내는 클래스를 가정해 보겠습니다.

```ts
class UserPosts {
  user : UserInfo | null;
  posts: Post[] | null;

  constructor(){
    this.user = null;
    this.post = null;
  }

  async init(userId: string): Promise<UserPosts>{
    const [user, postId] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userid)
    ]);
    return new userPosts(user, posts);
  }

  getUsername(){
    return this.user.name;
  }
}

```

이제 UserPosts 클래스는 완전히 null이 아니게 되었고, 메서드를 작성하기 쉬워졌습니다.
물론 이 경우에도 데이터가 부분적으로 준비되었을 때 작업을 시작해야 한다면, null과 null이 아닌 경우의 상태를 다루어야 합니다.

(null인 경우가 필요한 속성은 프로미스로 바꾸면 안 됩니다. 
코드가 매우 복잡해지며 모든 메서드가 비동기로 바뀌어야 합니다. 
프로미스는 데이터를 로드하는 코드를 단순하게 만들어주지만, 데이터를 사용하는 클래스에서는 반대로 코드가 복잡해지는 효과를 내기도 합니다.)
