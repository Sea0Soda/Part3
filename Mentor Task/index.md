# 🧪 TypeScript 과제: Router 경로에서 파라미터 추출 타입 만들기

## 📚 과제 목적

라우터 경로 문자열(예: `/users/:id`)에서 `:id`, `:userId` 같은 파라미터 이름을 추출하여  
타입스크립트 타입으로 `{ id: string }`, `{ userId: string; postId: string }` 형태로 만들어보세요.

이를 통해 `infer`, `template literal types`, `conditional types`, `mapped types`, `재귀 타입`을 실습합니다.

---

<details>
<summary>목적이해(과제만보려면 안보셔도됩니다)</summary>

### 경로문자열

https://instagram.com/users/김철수  
https://instagram.com/users/이영희

/users/김철수 → /users/:id  
/users/이영희 → /users/:id (같은 패턴)

여기서 id 같은부분을 추출하여 타입스크립 타입으로 지정해보는것이 목표

</details>

<details>
<summary>infer</summary>

### infer

```
type GetName<T> = T extends `Hello ${infer Name}` ? Name : never;
type Result = GetName<"Hello 철수">; // "철수"
```

- "Hello 철수"와 `Hello ${infer Name}` 비교:
- "Hello " 부분 → 일치
- "철수" 부분 → Name에 저장됨
- 결과: Name = "철수", 조건 = true

</details>

<details>
<summary>template literal types</summary>

### template literal types 사용예시

```
type UserPath = `users/${string}`;

// 이 타입에 맞는 것들:
let path1: UserPath = "users/123";     //
let path2: UserPath = "users/철수";    //
let path3: UserPath = "users/abc";     //

// 이 타입에 안 맞는 것들:
let path4: UserPath = "posts/123";     // users/로 시작하지않음
let path5: UserPath = "users";         // / 뒤에 아무것도없음
```

</details>

<details>
<summary>conditional types</summary>

### conditional types 설명

```
"if문 같은 것, 타입 버전"
type IsString<T> = T extends string ? "문자열임" : "문자열아님";

type Test1 = IsString<"hello">;  // "문자열임"
type Test2 = IsString<123>;      // "문자열아님"
```

</details>

</details>

<details>
<summary>mapped types</summary>

### mapped types 설명

```
interface LoginForm {
  username: string;
  password: string;
}

interface SignupForm {
  email: string;
  password: string;
  confirmPassword: string;
  name: string;
}

interface ContactForm {
  name: string;
  email: string;
  message: string;
}

추가적인게 생기면 타입을 계속 지정해줘야하지만

type Form<Fields extends string> = { [F in Fields]: string };

type LoginForm = Form<"username" | "password">;
type SignupForm = Form<"email" | "password" | "confirmPassword" | "name">;
type ContactForm = Form<"name" | "email" | "message">;

이런 방식으로 코드를 단축시킬수있음
```

</details>

</details>

<details>
<summary>재귀타입</summary>

### 재귀타입 예시

```
function countdown(n) {
  if (n <= 0) return "끝!";
  return countdown(n - 1);  // countdown 사용
}

--------------------------------------------------------------------

type DoStep<N> = N extends 1
  ? "끝!"
  : `단계${DoStep<PreviousN>}`;  // DoStep 자기 자신 사용

  함수내에서 바로 자기 자신을 호출할수있다로 이해함
```

</details>

## ✅ 최종 목표

| 경로 문자열                        | 기대 타입                             |
| ---------------------------------- | ------------------------------------- |
| `/users/:id`                       | `{ id: string }`                      |
| `/posts/:slug/comments/:commentId` | `{ slug: string; commentId: string }` |
| `/about`                           | `{}`                                  |
| `/users/:id?`                      | `{ id?: string }`                     |

---

## 📘 레벨 1: 단일 파라미터 추출

```ts
// 아래 타입을 완성하세요
type ExtractParams<S extends string> =
  S extends `${string}:${infer Param}`
    ? { [K in Param]: string }
    : {};

// 테스트 코드
type T1 = ExtractParams<'/users/:id'>;   // { id: string }
type T2 = ExtractParams<'/posts/:slug'>; // { slug: string }
type T3 = ExtractParams<'/about'>;       // {}

이해한것

//type ExtractParams<S extends string>
타입 지정하는 함수를 만들건데 이름은 ExtractParams 이고 여기 들어갈 S 는 문자열아무거나 들어올수있음
//= S extend ${string}:${infer Param}
S 에 있는 문자열: 이값 뒤에나오는건 Param 이란 이름으로 infer(값 추출해서 기억)
//? { [K in Param]: string }
    : {};
그리고 추출한 Param 값이 존재하면 string , 존재하지않으면 {}

${string}:${infer Param}
${/users/,/posts/,/about}:{id,slug,{} = param}

id,slug는 문자열로 비어있지않으니 string, T3 은 /about 뒤가 비어있으니 그대로 {} 빈값
```

---

## 📗 레벨 2: 여러 개 파라미터 추출

```ts
type ExtractParams<S extends string> =
  S extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param]: string } & ExtractParams<Rest>
  : S extends `${string}:${infer Param}`
    ? { [K in Param]: string }
  : {};


// 테스트 코드
type T4 = ExtractParams<'/users/:userId/posts/:postId'>;
// { userId: string; postId: string }

type T5 = ExtractParams<'/comments/:commentId/like/:reactionId'>;
// { commentId: string; reactionId: string }

문제 1과 같이 똑같이 작동하면서 userId 뒤에 오는 값(posts/:postId)을 Rest로 따로 기억하게함
다시 posts/:postId 가 ExtractParams 함수에 들어가면서 2~3번째 줄 코드에서 매칭이 실패함
(postId 뒤에 /가 없기때문)
2~3번 라인 코드와 매칭되지않았으니 3~4번 라인코드가 실행되면서 매칭성공함 (Param: PostID, Rest= {})



```

---

## 📙 레벨 3 (도전): 옵셔널 파라미터 지원

```ts
type ExtractParams<S extends string> =


  S extends `${string}:${infer Param}?/${infer Rest}`
    ? { [K in Param]?: string } & ExtractParams<Rest> //case 1

  : S extends `${string}:${infer Param}/${infer Rest}`
    ? { [K in Param]: string } & ExtractParams<Rest> //case 2

  : S extends `${string}:${infer Param}?`
    ? { [K in Param]?: string } //case 3

  : S extends `${string}:${infer Param}`
    ? { [K in Param]: string } //case 4
  : {};

// 테스트 코드
type T6 = ExtractParams<'/users/:id?'>;
// { id?: string }

type T7 = ExtractParams<'/users/:userId/posts/:postId?'>;
// { userId: string; postId?: string }

테스트 코드만을위한 결과값을 원하면 case 1, 4 코드는 필요없긴하지만

type T8 = ExtractParams<'/users/:userId?/posts/:postId'>; (case1),(case4)
type T9 = ExtractParams<'/users/:id'>; (case4)

같은상황을 생각해보면 4가지 케이스를 염두하고 사용하는게 좋을거같다고생각해서 작성

문제 1,2 와같이 작동하면서 T6 는 ?로 끝나기때문에 case3 에 매칭됨
T7 경우 case2 매칭되어 rest 값으로 post/:postId? 저장되어 다시 함수에 들어감
그후 case 3에 해당해서 결국 userId:string, postId?:string
```

---

## 💡 힌트

- `infer`를 사용해 문자열에서 파라미터 이름 추출하기
- `extends \`\${string}:\${infer Param}\`\` 구조 익히기
- `Mapped Type`으로 `{ [K in Param]: string }` 형태 만들기
- 여러 번 등장하는 경우 재귀적으로 처리
- `infer Name`, `infer Rest` 활용해 분리하기

---

```

```
