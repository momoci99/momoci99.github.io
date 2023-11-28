---
layout: single
title: "[Typescript] Effective Typescript 4장 - 타입 설계"
tag:
  - Effective Typescript
---

이펙티브 타입스크립트 4장 내용을 정리하였습니다.

# 4장 타입 설계

## 28. 유효한 상태만 표현하는 타입을 지향하기

- 타입을 잘 설계하면 코드는 직관적으로 작성할 수 있음.
- 그러나 타입 설계가 엉망이라면 어떠한 기억이나 문서도 도움이 되지 않음.
- 효과적인 타입 설계는 유효한 상태만 표현할 수 있는 타입을 만들어내는 것.
- 해당 챕터에서 타입 설계가 잘못된 상황을 바로잡을 것.

e.g: 웹 어플리케이션을 만든다고 가정 - 페이지의 상태 설계

```tsx
interface State {
  pageText: string;
  isLoading: boolean;
  error?: string;
}
```

e.g : 페이지를 그리는 renderPage함수 작성시 상태 객체의 필드를 전부 고려해서 상태 표시 분기

```tsx
function renderPage(state: State) {
  if (state.error) {
    return `Error! Unable to load ${currentPage}: ${state.error}`;
  } else if (state.isLoading) {
    return `Loading ${currentPage}...`;
  }
  return `<h1>${currentPage}</h1>\n${state.pageText}`;
}
```

- 해당 코드는 분기 조건이 명확히 분리되어있지 않음.
- isLoading이 true이고 동시에 error값이 존재하면 로딩 중인 상태인지, 오류가 발생한 상태인지 명확히 구분할 수 없음. (필요한 정보가 부족함)

e.g: 페이지를 전환하는 changePage함수 정의

```tsx
async function changePage(state: State, newPage: string) {
  state.isLoading = true;

  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ko) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const text = await response.text();
    state.isLoading = false;
    state.pageText = text;
  } catch (e) {
    state.error = "" + e;
  }
}
```

- 오류가 발생했을 때 state.isLoading을 false로 설정하는 로직이 빠져있음.
- state.error를 초기화하지 않았기 때문에, 페이지 전환 중에 로딩 메시지 대신 과거의 오류 메시지를 보여줌
- 페이지 로딩 중에 사용자가 페이지를 바꿔 버리면 어떤 일이 벌어질지 예상하기 어려움.
- 새페이지에 오류가 뜨거나, 응답이 오는 순서에 따라 두 번째 페이지가 아닌 첫 번째 페이지로 전환될 수도 있음.

문제점 +

- 상태 값의 두 가지 속성이 동시에 정보가 부족하거나(요청이 실패한 것인지, 여전히 로딩중인지 알 수 없음)
- 두가지 속성이 충돌(오류이면서 동시에 로딩 중일 수 있음)할 수 있다는것.
- State 타입은 isLoading이 true이면서 동시에 error값이 설정되는 무효한 상태를 허용함.
- 무효한 상태가 존재하면 render()와 changePage() 둘 다 제대로 구현할 수 없게됨.

e.g : 애플리케이션의 상태를 좀 더 제대로 표현

```tsx
interface RequestPending {
	state: 'pending';
}
interface RequestError {
	state: 'error';
	error: string;
}
interface RequestSuccess {
	state: 'ok';
	pageText: string;
}
type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
	currentPage: string;
	requests:{[page:string]:RequestState);
}
```

- 네트워크 요청 과정 각각의 상태를 명시적으로 모델링하는 태그된 유니온(또는 구별된 유니온)이 사용됨.
- 상태를 나타내는 타입의 코드 길이가 서너 배 길어지긴 했으나, 무효한 상태를 허용하지 않도록 크게 개선.
- 현재 페이지는 발생하는 모든 요청의 상태로서, 명시적으로 모델링.
- 그 결과로 개선된 renderPage와 changePage함수는 쉽게 구현할 수 있음.

e,g: 개선된 renderPage와 changePage

```tsx
function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.requests[currentPage];
  switch (requestState.state) {
    case "pending":
      return `Loading ${currentPage}...`;
    case "error":
      return `Error! Unable to load ${currentPage}: ${requestState.error}`;
    case `ok`:
      return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
  }
}

async function changePage(state: State, newPage: string) {
  state.requests[newPage] = { state: "pending" };
  state.currentPage = newPage;

  try {
    const response = await fetch(getUrlForPage(newPage));
    if (!response.ok) {
      throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
    }
    const pageText = await response.text();
    state.requests[newPage] = { state: "ok", pageText };
  } catch (e) {
    state.requests[newPage] = { state: "error", error: "" + e };
  }
}
```

- 처음에 있었던 renderPage와 changePage의 모호함은 완전히 사라짐.
- 현재 페이지가 무엇인지 명확하며, 모든 요청은 정확히 하나의 상태로 맞아 떨어짐.
- 요청이 진행 중인 상태에서 사용자가 페이지를 변경하더라도 문제 없음.
- 무효가 된 요청이 실행되긴 하겠으나 UI에는 영향이 없음.

### 요약

- 유효한 상태와 무효한 상태를 둘 다 표현하는 타입은 혼란을 초래하기 쉽고 오류를 유발하게됨
- 유효한 상태만 표현하는 타입을 지향해야함. 코드가 길어지거나 표현하기 어렵지만 결국은 시간을 절약하고 고통을 줄일 수 있음.

## 29. 사용할때는 너그럽게, 생성할 때는 엄격하게

```
TCP 구현체는 견고성의 일반적인 원칙을 따라야 한다.
당신의 작업은 엄격하게 하고, 다른 사람의 작업은 너그럽게 받아들여야 한다
견고성의 원칙 (포스텔의 법칙) - 존 포스텔
```

- 함수의 시그처에도 위와 비슷한 규칙을 적용해야함
- 함수의 매개변수의 타입의 범위는 넓어도 되지만, 결과를 반환할 때는 일반적으로 타입의 범위가 더 구체적이어야함.

e.g : 3d 매핑 API는 카메라의 위치를 지정하고 경계 박스의 뷰포트를 계산하는 방법을 제공.

```tsx
declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds): CameraOptions;
```

e.g : 카메라의 위치를 잡기 위해 viewportForBounds의 결과가 setCamera로 바로 전달 될 수 있도록 함.

```tsx
interface CameraOptions{
	center? :LngLat;
	zoom? :number;
	bearing? :number;
	pitch? : number;
}
type LngLat = {
	{
		lng:number, lat:number; |
	}
	{
		lon:number, lat:number; |
	}
	[number, number];
}
```

- 일부 값은 건드리지 않으면서 동시에 다른 값을 설정할 수 있어야 하므로 CameraOptions의 필드는 모두 선택적.
- LngLat 타입도 setCameara의 매개변수 범위를 넓혀줌.
- 매개변수로 {lng, lat} 객체, {lon, lat} 객체, 또는 순서만 맞다면 [lng, lat] 쌍을 넣을 수도 있음.
- 이러한 편의성을 제공하여 함수 호출을 쉽게 할 수 있음.

viewportForBounds 함수는 또 다른 자유로운 타입을 매개변수로 받을 수 있음.

```tsx
type LngLatBounds =
  | { northeat: LngLat; southwest: LngLat }
  | [LngLat, LngLat]
  | [number, number, number, number];
```

- 이름이 주어진 모서리, 위도/경도 쌍, 또는 순서만 맞다면 4-튜플을 사용하여 경계 지정 가능.
- LngLat는 세 가지 형태를 받을 수 있으므로 LngLatBounds의 가능한 형태는 19가지 이상으로 매우 자유로운 타입이 됨.

e.g GeoJSON 기능을 지원하도록 뷰포트를 조절하고, 새 뷰포트를 URL에 저장하는 함수 정의

```tsx
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const {
    center: { lat, lng },
    zoom,
  } = camera;
  // error - 형식에 lat, lng 속성이 없습니다.
  zoom;
  window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

- lat, lng 속성이 없고 zoom속성만 존재하기 때문에 발생하였으나, 타입이 number | undefined로 추론되는 것 역시 문제.
- 근본적인 문제는 viewportForBounds의 타입 선언이 사용될 때 뿐만이 아니라 만들어질 때도 너무 자유로움.
- camera값을 안전한 타입으로 사용하는 유일한 방법은 유니온 타입의 각 요소별로 코드를 분기하는것.

- 수많은 선택적 속성을 가지는 반환 타입과 유니온 타입은 viewportForBounds를 사용하기 어렵게 만듬.
- 매개변수 타입의 범위가 넓으면 사용하기 편리하지만, 반환 타입의 범위가 넓으면 불편함.
- **즉 사용하기 편리한 API일수록 반환 타입이 엄격함.**

- 유니온 타입의 요소별 분기를 위한 한 가지 방법은, 좌표를 위한 기본 형식을 구분하는 것
- 배열과 배열 같은 것(array-like)의 구분을 위해, 자바스크립트의 관례에 따라 LngLat와 LngLatLike를 구분할 수 있음.
- setCamera함수가 매개 변수로 받을 수 있도록, 완전히 정의된 Camera 타입과 Camera타입이 부분적으로 정의된 버전을 구분할 수도 있음.

```tsx
interface LngLat = { lng:number; lat:number;};
type LngLatLike = LngLat | { lon:number; lat:number;} | [number, number];

interface Camera {
	center: LngLat;
	zoom: number;
	bearing: number;
	pitch: number;
}

interface CameraOptions extends Omit<Partial<Camera>, 'center'> {
	center?: LngLatLike;
}
type LngLatBounds =
	{northeat: LngLatLike, southwest: LngLatLike} |
	[LngLatLike, LngLatLike] |
	[number, number, number, number];

declare function setCamera(camera: CameraOptions): void;
declare function viewportForBounds(bounds: LngLatBounds):Camera;
```

- Camera가 매우 엄격하므로 조건을 완화하기 위해 느슨한 CameraOptions 타입으로 만듬.
- setCamera 매개 변수 타입의 center속성에 LngLatLike 객체를 허용해야하기 때문에 Partial<Camera>를 사용하면 코드가 동작하지 않음.
- LngLatLike가 LngLat의 부분 집합이 아니라 상위집합이기 때문에, Camera Options extends Partial<Camera>를 사용할 수 없어야함.
- 너무 복잡해보인다면, 약간의 반복 작업을 해야겠으나, 명시적으로 타입을 추출해서 개선할 수 있음.

e.g CameraOptions의 개선 버전

```tsx
interface CameraOptions {
  center?: LngLatLike;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
```

e.g : CameraOptions를 선언하는 두 가지 방식 모두 focusOnFeature함수가 타입 체커를 통과할 수 있게 함.

```tsx
function focusOnFeature(f: Feature) {
  const bounds = calculateBoundingBox(f);
  const camera = viewportForBounds(bounds);
  setCamera(camera);
  const {
    center: { lat, lng },
    zoom,
  } = camera; // 정상
  zoom; // 타입이 number
  window.location.search = `?v=@${lat},${lng}z${zoom}`;
}
```

- zoom의 타입이 number|undefined가 아니라 number임.
- 이제 viewportForBounds함수를 사용하기 훨씬 쉬워짐.
- 그리고 bounds를 생성하는 다른 함수가 있다면 LngLatBounds와 LngLatBoundsLike를 구분할 수 있도록 새로운 기본 형식을 도입해야함.
- 앞에 등장한 것 처럼 경계 박스의 형태를 19가지나 허용하는 것은 좋은 설계가 아님.
- 그러나 다양한 타입을 허용해야하만 하는 라이브러리의 타입 선언을 작성하고 있다면, 어쩔 수 없이 다양한 타입을 허용하는 경우가 생김. 하지만 그때도 19가지 반환 타입이 나쁜 설계라는 사실을 잊어서는 안됨.

### 요약

- 보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있음.
- 선택적 속성과 유니온 타입은 반환 타입보다 매개변수 타입에 더 일반적.
- 매개변수와 반환 타입의 재사용을 위해서 기본 형태(반환 타입)와 느슨한 형태(매개변수 타입)을 도입하는 것이 좋음.

## 30. 문서에 타입 정보를 쓰지 않기

e.g : 잘못된 코드 (코드와 주석 정보가 맞지 않음)

```tsx
/**
 * 전경색(foreground) 문자열을 반환.
 * 0 개 또는 1개의 매개변수를 받습니다.
 * 매개변수가 없을 때는 표준 전경색을 반환합니다.
 * 매갭변수가 있을 때는 특정 페이지의 젼경색을 반환합니다.
 */
function getForegroundColor(page?: string) {
  return page === "login" ? { r: 127, g: 127, b: 127 } : { r: 0, g: 0, b: 0 };
}
```

3가지 문제점이 존재

1. 함수가 string형태의 색깔을 반환한다고 적혀있으나 실제로는 {r, g, b} 객체를 반환.
2. 주석에는 함수가 0개 또는 1개의 매개변수를 받는다고 설명하고 있으나, 타입 시그니처만 보아도 명확하게 알 수 있는 정보.
3. 불필요하게 장황함. 함수 선언과 구현체보다 주석이 더 김.

- 주석보다 타입스크립트의 타입 구문 시스템이 더 나음.
- 주석은 코드와 동기화 되지 않음을 명심.

e.g : 주석을 개선

```tsx
/** 애플리케이션 또는 특정 페이지의 전경색을 가지고 옴*/
function getForegroundColor(page?: string): Color {
  // ...
}
```

- 특정 매개변수를 설명하고 싶다면 JSDoc의 @param 구문을 사용하면 됨

```tsx
/** nums를 변경하지 않습니다 **/
function sort(nums: number[]) {
  /*...*/
}
```

- 값을 변경하지 않는다고 설명하는 주석도 좋지 않음.
- 또한 매개 변수를 변경하지 않는다는 주석도 사용하지 않는게 좋음.

```tsx
function sort(nums: readonly number[]) {
  /* ... */
}
```

- readonly 로 선언하여 타입스크립트가 규칙을 강제할 수 있게하면 됨
- 주석에 적용한 규칙은 변수명에도 그대로 적용할 수 있음.
- 변수명에 타입 정보를 넣지 않도록 함.
- 단, 단위가 있는 숫자들은 예외. 단위가 무엇인지 확실하지 않다면 변수명 또는 속성 이름에 단위를 포함할 수 있음. 예를 들어 timesMs는 time 보다 훨씬 명확하고 temperatureC는 temperature보다 훨씬 명확함.

### 요약

- 주석과 변수명에 타입 정보를 적는것은 피해야함.
- 타입 선언이 중복되는 것으로 끝나면 다행이지만, 최악의 경우는 타입 정보에 모순 발생
- 타입이 명확하지 않는 경우는 변수명에 단위 정보를 포함하는 것을 고려하는것이 좋음.(ex timeMs 또는 temperatureC)

## 31 타입 주변에 null 값 배치하기

- 값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분된다면, 값이 섞여 있을 때 보다 다루기 쉬움.
- 타입에 null을 추가하는 방식으로 이러한 경우를 모델링 할 수 있음.

e.g : 숫자들의 최솟값과 최댓값을 계산하는 extent 함수를 가정

```tsx
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

- 타입 체커를 통과(strictNullChecks 없이), 반환 타입은 number[]로 추론됨.
- 그러나 여기에는 버그와 함께 설계적 결함이 있음.
  - 최솟값이나 최댓값이 0인 경우, 값이 덧씌어짐. 예를 들어 extent([0, 1, 2])의 결과는 [0, 2]가 아니라 [1, 2] 가 됨.
  - nums 배열이 비어있다면 함수는 [undefined, undefined] 를 반환.
- undefined를 포함하는 객체는 다루기 어렵고 절대 권장하지 않음.
- 코드를 살펴보면 min과 max가 동시에 둘 다 undefined이거나 둘 다 undefined 가 아니라는것을 알 수 있으나, 이러한 정보는 타입 시스템에서 표현할 수 없음.

e.g : strictNullChecks 설정을 켠 상태

```tsx
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
      //error -
      //Argument of type 'number | undefined' is not assignable to parameter of type 'number'.
      //Type 'undefined' is not assignable to type 'number'.
    }
  }
  return [min, max];
}
```

- extent의 반환 타입이 (number | undefined)[]로 추론되어서 설계적 결함이 분명해짐
- extent를 호출하는 곳 마다 타입 오류의 형태로 나타남

```tsx
const [min, max] = extent([0, 1, 2]);
const span = max - min;
//error - 'max' is possibly 'undefined'.
```

- extent 함수의 오류는 undefined를 min에서만 제외했고, max에서는 제외하지 않았기 때문에 문제가 발생.
- 두개의 변수는 동시에 초기화 되지만, 이러한 정보는 타입 시스템에 표현할 수 없음.
- max에 대한 체크를 추가해서 오류를 해결할 수도 있지만 버그가 두배로 늘어날 것.

e.g 더 나은 방법을 적용(min과 max를 한 객체 안에 넣고 null이거나 null이 아니게 하면 됨.

```tsx
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
    return result;
  }
}
```

- 이제는 반환 타입이 [number, number] | null이 되어서 사용하기가 더 수월해짐.

e.g : null 아님 단언(!)을 사용하면 min과 max를 얻을 수 있음.

```tsx
const [min, max] = extent([0, 1, 2]);
const span = max - min; //정상
```

e.g : null아님 단언 대신 단순 if 구문으로 체크할 수도 있음.

```tsx
const range = extent([0, 1, 2]);
if (range) {
  const [min, max] = range;
  const span = max - min; //정상
}
```

- extent의 결괏값으로 단일 객체를 사용함으로써 설계를 개선.
- 타입스크립트가 null 값 사이의 관계를 이해할 수 있도록 했으며, 버그도 제거.
- if(!result)체크는 이제 제대로 동작.

e.g null과 null이 아닌 값을 섞어서 사용하면 클래스에서도 문제가 생기는 예제

```tsx
class UserPosts {
  user: UserInfo | null;
  posts: Post[] | null;

  constructor() {
    this.user = null;
    this.posts = null;
  }

  async init(userId: string) {
    return Promise.all([
      async () => (this.user = await fetchUser(userId)),
      async () => (this.posts = await fetchPostForUser(userId)),
    ]);
  }

  getUserName() {
    // ...?
  }
}
```

- 두 번의 네트워크 요청이 로드되는 동안 user와 posts 속성은 null 상태.
- 어떤 시점에는 둘 다 null이거나, 둘중 하나만 null이거나, 둘다 null이 아닐 수 있음. 총 네가지 경우 존재
- 속성값의 불확실성이 클래스의 모든 메서드에 나쁜 영향을 미침. 결국 null체크가 난무하고 버그를 양산하게 될 것.

e.g : 설계를 개선

```tsx
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  async init(userId: string): Promise<UserPosts> {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostForUser(userId),
    ]);
    return new UserPosts(user, posts);
  }

  getUserName() {
    return this.user.name;
  }
}
```

- UserPosts 클래스는 완전히 null이 아니게 되었고, 메서드를 작성하기 쉬워졌음.
- 이 경우에도 데이터가 부분적으로 준비되었을 때 작업을 시작해야한다면, null과 null이 아닌 경우의 상태를 다루어야함.

주의

- null인 경우가 필요한 속성은 프로미스로 바꾸면 안됨. 코드가 매우 복잡해지며 모든 메서드가 비동기로 바뀌어야함.
- 프로미스는 데이터를 로드하는 코드를 단순하게 만들어 주지만, 데이터를 사용하는 클래스에서는 반대로 코드가 복잡해지는 효과를 내기도 함.

### 요약

- 한 값의 null 여부가 다른 값의 null 여부에 암시적으로 관련되도록 설계하면 안됨.
- API 작성 시에는 반환 타입을 큰 객체로 만들고, 반환 타입 전체가 null 이거나, null이 아니게 만들어야함. 사람과 타입 체커 모두에게 명료한 코드가 될 것.
- **클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는것이 좋음.**
- strictNullChecks를 설정하면 코드에 많은 오류가 표시되겠지만, null값과 관련된 문제점을 찾아낼 수 있기에 반드시 필요함.

## 32. 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 유니온 타입 속성을 가지는 인터페이스를 작성 중이라면 인터페이스의 유니온 타입을 사용하는 게 더 알맞지는 않을지 검토할 필요성이 있음.

e.g 벡터를 그리는 프로그램. 특정 기하학적(geometry)타입을 가지는 계층의 인터페이스를 정의한다고 가정.

```tsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

- layout 속성은 모양이 그려지는 방법과 위치(둥근 모서리, 직선)을 제어.
- paint속성은 스타일(파란선, 굵은선, 얇은선, 점섬)을 제어.
- layout이 LineLayout이면서 paint속성이 FillPaint 타입인 것은 성립되지 않음.

e.g : 오류를 줄이기 위해 더 나은 방법으로 모델링

```tsx
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

- layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지 함.
- 유효한 상태만을 표현함.

e.g: 태그된 유니온(또는 구분된 유니온)의 예

```tsx
//Layer의 경우 속성 중의 하나는 문자열 리터럴 타입의 유니온이 됨.
interface Layer {
  type: "fill" | "line" | "point";
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

e.g : type ‘fill’ 과 LineLayout과 PointPaint 타입이 같이 쓰는 경우를 방지하기 위해 Layer를 인터페이스의 유니온으로 변환

```tsx
interface FillLayer {
  type: "fill";
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  type: "line";
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  type: "paint";
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

- type 속성은 ‘태그’ 이며 런타임에 어떤 타입의 Layer가 사용되는지 판단하는데 쓰임.
- 타입스크립트는 태그를 참고하여 Layer의 타입 범위를 좁힐 수도 있음.

```tsx
function drawLayer(layer: Layer) {
  if (layer.type === "fill") {
    const { paint } = layer; // 타입 FillPaint
    const { layout } = layer; // 타입이 FillLayout
  } else if (layer.type === "line") {
    const { paint } = layer; //타입이 LinePaint
    const { layout } = layer; //타입이 LineLayout
  } else {
    const { paint } = layer; //타입이 PointPaint
    const { layout } = layer; //타입이 PointLayout
  }
}
```

- 각 타입의 속성들 간의 관계를 제대로 모델링하면, 타입스크립트가 코드의 정확성을 체크하는데 도움이 됨.
- 다만 타입 분기후 layer가 포함된 동일한 코드가 반복되는 것이 어수선하게 보임.
- 태그된 유니온은 타입스크립트 타입 체커와 잘 맞으므로 타입스크립트 코드 어디에서나 찾을 수 있음.
- 해당 패턴을 잘 기억하여 필요할 때 사용할 수 있도록 해야함.
- 어떤 데이터 타입을 태그된 유니온으로 표현할 수 있다면, 그렇게 하는게 좋음.
- 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 undefined 인 경우도 태그된 유니온 패턴이 잘 맞음.

e.g : 타입정보를 담고있는 주석으로 인해 문제가 있는 코드

```tsx
interface Person {
  name: string;
  //다음은 둘 다 동시에 있거나 동시에 없음.
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

- 타입 정보를 담고 있는 주석은 문제가 될 소지가 매우 높음.
- placeOfBirth와 dateOfBirth 필드는 실제로 관련되어 있지만, 타입 정보에는 어떠한 관계도 표현되지 않음.

e.g : 두개의 속성을 하나의 객체로 모아 더 나은 설계 적용.

```tsx
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  };
}
```

e.g : place만 있고 date가 없는 경우 오류 발생

```tsx
const alanT: Person = {
  name: "Alan Turing",
  birth: {
    //Property 'date' is missing in type '{ place: string; }'
    // but required in type '{ place: string; date: Date; }'.
    place: "London",
  },
};
```

e.g : Person 객체를 매개변수로 받는함수는 birth 하나만 체크하면 됨.

```tsx
function eulogize(p: Person) {
	console.log(p.name);
	const {birth} = p;
	if(birth){
		console.log('was born on ${birth.date} in ${birth.place}.`);
	}
}
```

```tsx
interface Name {
  name: string;
}

interface PersonWithBirth extends Name {
  placeOfBirth: string;
  dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```

- 타입의 구조를 손 댈 수 없는 상황(ex : API의 결과)이면, 앞서 다룬 인터페이스의 유니온을 사용해서 속성 사이의 관계를 모델링 할 수 있음.

e.g : 중첩된 객체에서도 동일한 효과를 볼 수 있음.

```tsx
function eulogize(p: Person) {
  if ("placeOfBirth" in p) {
    p; //타입이 PersonWithBirth
    const { dateOfBirth } = p; //정상, 타입이 Date
  }
}
```

## 요약

- 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않음.
- 이로 인해 실수가 발생할 수 있으며 주의해야함.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기도 좋음.
- 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야함. 태그된 유니온 타입은 타입스크립트와 매우 잘 맞기 때문에 자주 볼 수 있는 패턴.

## 33. string 타입보다 더 구체적인 타입 사용하기

- string의 타입의 범위는 매우 넓음
- “x” 혹은 “y” , “call me is…” 같은 내용도 string 타입.
- string타입으로 변수를 선언하려 한다면 더 좁은 타입이 적절한지 검토 필요.

e.g : 음악 컬렉션을 만들기 위해 앨범의 타입을 정의

```tsx
interface Album {
  artist: string;
  title: string;
  releaseDate: string; // YYYY-MM-DD
  recordingType: string; // "live" or "studio"
}
```

- string타입이 남발되어있음.
- 주석에 타입 정보를 적어두어 현재 인터페이스가 잘못되었음.

e.g : Album 타입에 엉뚱한 값을 설정

```tsx
const kindOfBlue: Album = {
  artist: "Miles Davis",
  title: "Kind of Blue",
  releaseDate: "August 17th, 1959", //날짜 형식이 다름.
  recordingType: "Studio", //오타(대문자 S)
}; //그러나 정상 동작함
```

- releaseDate 필드값은 주석에 설명된 형식과 다름.
- recordingType 필드의 값 “Studio”는 소문자 대신 대문자가 쓰임.
- 그러나 Album타입에 할당가능하며 타입 체커를 통과.

e.g : 제대로된 Album 객체를 사용하더라도 매개변수 순서가 잘못된 것이 오류로 드러나지 않음.

```tsx
function recordRelease(title: string, date: string) {
  /* ... */
}
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title); //오류이나 정상
```

- 매개변수들의 순서가 바뀌었으나 타입 체커는 정상으로 인식(string이므로)
- string 타입이 남용된 코드를 “문자열을 남발하여 선언되었다(stringly typed)라고 표현.

e.g : 타입의 범위를 좁히는 방법 사용

```tsx
type RecordingType = "studio" | "live";

interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

- recordingType 필드를 ‘live’, ‘studio’ 의 유니온 타입 정의
- releaseDate는 Date 타입 사용

e.g : 오류를 더 세밀하게 잡아내게 된 코드

```tsx
const kindOfBlue: Album = {
  artist: "Miles Davis",
  title: "Kind of Blue",
  releaseDate: new Date("1959-08-17"),
  recordingType: "Studio",
  //Error - Type '"Studio"' is not assignable to type 'RecordingType'.
  //Did you mean '"studio"'?
};
```

**string 타입을 좁혔을 때 얻는 장점들**

1. 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지됨
2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있음.

   ```tsx
   /** 이 녹음은 어떤 환경에서 이루어 졌는지? */
   type RecordingType = "live" | "studio";
   ```

3. keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능함.

e.g : 어떤 배열에서 한 필드의 값만 추출하는 함수를 작성한다고 가정 (실제 언더스코어 라이브러리에는 pluck이라는 함수가 있음)

```tsx
function pluck(records, key) {
	return records.map(r => r[key);
}
```

e.g: pluck 함수의 시그니처 예제

```tsx
function pluck(records: any[], key: string): any[] {
  return records.map((r) => r[key]);
}
```

- 타입 체크가 되긴 하지만, any 타입이 있어서 정밀하지 못함.
- 특히 반환값에 any를 사용하는 것은 매우 좋지 않은 설계.

e.g : 제너릭 타입을 도입하여 개선

```tsx
function pluck<T>(records: T[], key: string): any[] {
  return records.map((r) => r[key]);
  //Error - Element implicitly has an 'any' type because expression of
  // type 'string' can't be used to index type 'unknown'.
  // No index signature with a parameter of type
  // 'string' was found on type 'unknown'.
}
```

- 타입 체커는 key의 타입이 string이기 때문에 범위가 너무 넓다는 오류 발생.
- Album의 배열을 매개변수로 전달하면 기존의 string타입의 넓은 범위 반대로, key는 단 네 개의 값(”artist”, “title”, “releaseDate”, “recordingType”) 만이 유효함.

e.g : keyof Album 타입으로 얻게되는 결과

```tsx
type K = keyof Album;
//"artist" | "title" | "releaseDate" | "recordingType"
```

```tsx
//string을 keyOf T 로 변경
function pluck<T>(records: T[], key: keyof T) {
  return records.map((r) => r[key]);
}
```

- 이 코드는 타입 체커를 통과할 수 있으며, 타입스크립트가 반환 타입을 추론할 수 있게 해줌.

```tsx
//타입 스크립트가 추론한 pluck의 타입
function pluck<T>(records: T[], key: keyof T): T[keyof T][];
```

- T[keyof T]는 T객체 내의 가능한 모든 값의 타입임.

그런데 key의 값으로 하나의 문자열을 넣게 되면, 그 범위가 너무 넓어서 적절한 타입이라고 보기 어려움.

```tsx
const releaseDates = pluck(albums, "releaseDate"); //타입이 (string | Date)[]
```

- releaseDates의 타입은 (string | Date)[]가 아니라 Date[]이어야 함.
- keyof T는 string에 비하면 훨씬 범위가 좁기는 하지만 그래도 여전히 넓음.

e.g : 범위를 더 좁히기 위해 keyof T의 부분 집합(아마도 단일 값)으로 두 번째 제너릭 매개변수를 도입해야함

```tsx
function plck<T, K extends keyof T>(records: T[], key: K): T[K][] {
  return records.map((r) => r[key]);
}
```

e.g: 완벽해진 타입 시그니처

```tsx
pluck(albums, "releaseDate"); // 타입이 Date[]
pluck(albums, "artist"); //타입이 string[]
pluck(albums, "recordingType"); //타입이 RecordingType[]
//Error - "recordingDate"형식 인수는 할당될 수 없음.
```

## 요약

- 문자열을 남발하여 선언된 코드를 피할것. 모든 문자열을 할당할 수 있는 string타입 보다는 더 구체적인 타입을 사용하는 것이 좋음
- 변수의 범위를 보다 정확하게 표현하고 싶다면 string 타입보다는 문자열 리터럴 타입의 유니온을 사용하면 됨. 타입 체크를 더 엄격히 하고 생산성을 향상시킬 수 있음.
- 객체의 속성 이름을 함수 매개변수로 받을 때는 string보다는 keyof T를 사용하는것이 좋음.

## 34 부정확한 타입보다는 미완성 타입을 사용하기

- 타입 선언의 정밀도를 높이는 일에는 주의를 기울여야함.
- 실수가 발생하기 쉽고 잘못된 타입은 차라리 없는 것 보다 못할 수 있기 때문.

e.g GeoJSON형식의 타입 선언을 작성한다고 가정.

```tsx
interface Point {
  type: "Point";
  coordinates: number[];
}

interface LineString {
  type: "LineString";
  coordinates: number[][];
}

interface Polygon {
  type: "Polygon";
  coordinates: number[][][];
}

type Geometry = Point | LineString | Polygon;
```

- 큰 문제는 없으나 number[]가 약간 추상적.
- 여기서 number[]는 경도와 위도를 나타내므로 튜플타입으로 선언하는게 나음.

e.g : 튜플 타입으로 선언

```tsx
type GeoPosition = [number, number];
interface Point {
  type: "Point";
  coordinates: GeoPosition;
}
```

- 타입을 더 구체적으로 개선했기 때문에 더 나은 코드가 되었음. 그러나
- 코드에는 위도와 경도만 명시하였으나 GeoJSON의 위치정보에는 세 번째 요소인 고도가 있을 수 있음.
- 결과적으로 타입 선언을 세밀하게 만들고자 했으나 시도가 너무 과했고, 오히려 타입이 부정확해짐.
- 현재의 타입을 그대로 사용하려면 사용자들은 타입 단언문을 도입하거나 as any를 추가해서 타입 체커를 완전히 무시해야함.

e.g : JSON으로 정의된 Lisp와 비슷한 언어의 타입 선언을 작성한다고 가정

```lisp

["+", 1, 2]  //3
["/", 20, 2] //10
["case", [">", 20, 10], "red", "blue"] //"red"
["rgb", 255, 0, 127] // "#FF007F"
```

다양한 입력값을 허용하는 동작을 모델링

입력값 예제

1. 모두 허용
2. 문자열, 숫자, 배열 허용
3. 문자열, 숫자, 알려진 함수 이름으로 시작하는 배열 허용
4. 각 함수가 받는 매개변수의 개수가 정확한지 확인
5. 각 함수가 받는 매개변수의 타입이 정확한지 확인

e.g : 처음 1,2 옵션을 표현

```tsx
type Expression1 = any;
type Expression2 = number | string | any[];
```

e.g : 표현식의 유효성을 체크하는 테스트 세트 도입

```tsx
const tests: Expression2[] = [
  10,
  "red",
  true, //Error - Type 'boolean' is not assignable to type 'Expression2'.
  ["+", 10, 5],
  ["case", [">", 20, 10], "red", "blue", "green"],
  ["**", 2, 31],
  ["rgb", 255, 128, 64],
  ["rgb", 255, 0, 127, 0],
];
```

e.g : 정밀도를 한 단계 더 끌어 올리기 위해서 튜플의 첫 번째 요소에 문자열 리터럴 타입의 유니온을 사용

```tsx
type FnName = "+" | "-" | "/" | ">" | "<" | "case" | "rgb";
type CallExpression = [FnName, ...any[]];
type Expression3 = number | string | CallExpression;

const tests: Expression3[] = [
  10,
  "red",
  true, //Error
  ["+", 10, 5],
  ["case", [">", 20, 10], "red", "blue", "green"],
  ["**", 2, 31], //Error
  ["rgb", 255, 128, 64],
];
```

e.g : 매개변수 갯수가 정확한지 확인하기 위한 코드 구현

```tsx
type Expression4 = number | string | CallExpression;
type CallExpression = MathCall | CaseCall | RGBCall;

interface MatchCall {
  0: "+" | "-" | "/" | "*" | ">" | "<";
  1: Expression4;
  2: Expression4;
  length: 3;
}

interface CaseCall {
  0: "case";
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4 | 6 | 8 | 10 | 12 | 14 | 16; // 등등
}

interface RGBCall {
  0: "rgb";
  1: Expression4;
  2: Expression4;
  3: Expression4;
  length: 4;
}

const tests: Expression4[] = [
  10,
  "red",
  true, //Error
  ["+", 10, 5],
  ["case", [">", 20, 10], "red", "blue", "green"], //Error
  ["**", 2, 31], // Error
  ["rgb", 255, 128, 64],
  ["rgb", 255, 128, 64, 73], //Error
];
```

- 유효하지 않은 표현식에서 전부 오류 발생.
- 그러나 타입 정보는 정밀해졌을지는 몰라도 이전 버전보다 개선되었다고 보기 어려움.
- 오류 메시지는 더욱 난해해짐.

### 요약

- 타입 안정성에서 불쾌한 골짜기는 피해야합니다. 타입이 없는 것 보다 잘못된 게 더 나쁩니다.
- 정확하게 타입을 모델링 할 수 없다면, 부정확하게 모델링하지 말아야함.
- any와 unknown를 구별해서 사용해야함.

## 35 데이터가 아닌 API와 명세를 보고 타입 만들기

- 잘 설계된 타입은 타입스크립트사용을 즐겁게 해 주는 반면, 잘못된 타입은 비극을 불러옴. (😉)
- API 명세를 참고해 타입을 생성하면 타입스크립트는 사용자가 실수를 줄일 수 있게 도와줌.
- 반면, 예시 데이터를 참고해 타입을 생성하면, 눈앞에 있는 데이터들만 고려하게 되므로 예기치 않은 곳에서 오류가 발생할 수 있음.

e.g : 31장에서 Feature의 경계 상자를 계산하는 calculateBoundingBox함수 사용 및 실제 구현

```tsx
function calculateBoundingBox(f: Feature): BoundingBox | null {
  let box: BoundingBox | null = null;

  const helper = (coords: any[]) => {
    //...
  };

  const { geometry } = f;
  if (geometry) {
    helper(geometry.coordinates);
  }

  return box;
}
```

- Feature 타입은 명시적으로 정의된 적이 없음.
- 31장에 등장한 focusOnFeature 함수 예제를 사용하여 작성해 볼 수 있음.
- 그러나 GeoJSON명세를 사용하는게 더 나음.

e.g GeoJSON 선언 사용시 에러 발생

```tsx
import { Feature } from "geojson";

function calculateBoundingBox(f: Feature): BoundingBox | null {
  let box: BoundingBox | null = null;

  const helper = (coords: any[]) => {
    //...
  };

  const { geometry } = f;
  if (geometry) {
    helper(geometry.coordinates);
    //Error
    //Geometry 형식에 coordinates 속성이 없음.
    //GeometryCollection 형식에 coordinates 속성이 없음
  }
  return box;
}
```

- geometry에 coordinates 속성이 있다고 가정한 게 문제.
- 점, 선, 다각형을 포함한 많은 도형에서 맞는 개념. 그러나 GeoJSON은 다양한(heterogeneous)도형의 모음인 GeometryCollection일 수도 있음.
- 다은 도형 타입들과 다르게 GeometryCollection에는 coordinates속성이 없음.
- geometry가 GeometryCollection 타입인 Feature를 사용해서 calculatedBoundingBox를 호출하면 undefined의 속성 0을 읽을 수 없다는 오류 발생

e.g undefined 속성 참조 에러 해결

```tsx
const { geometry } = f;
if (geometry) {
  if (geometry.type === "GeometryCollection") {
    throw new Error("GeometryCollections are not supported.");
  }
  helper(geometry.coordinates); //정상
}
```

- 타입스크립트는 타입을 체크하는 방법으로 도형의 타입을 정제할 수 있음.
- 그러므로 정제된 타입에 한해서 geometry.coordinates의 참조를 허용하게됨.
- 차단된 GeometryCollection 타입의 경우, 사용자에게 명확한 오류 메시지를 제공.

e.g : 헬퍼 함수를 호출하여 모든 타입을 지원하는 코드

```tsx
const geometryHelper = (g: Geometry) => {
  if (geometry.type === "GeometryCollection") {
    geometry.geometries.forEach(geometryHelper);
  } else {
    helper(geometry.coordinates); //정상
  }
};

const { geometry } = f;
if (geometry) {
  geometryHelper(geometry);
}
```

e.g : GraphQL의 경우

```tsx
query {
	respository(owner: "Microsoft", name: "TypeScript") {
		createdAt
		description
	}
}
```

- GraphQL API는 타입스크립트와 비슷한 타입 시스템을 사용.
- 가능한 모든 쿼리와 인터페이스를 명세하는 스키마로 이뤄짐.
- 이러한 인터페이스를 사용해서 특정 필드를 요청하는 쿼리를 작성

e.g : 쿼리 결과

```tsx
{
	"data" : {
		"repository" : {
			"createdAt" : "2014-06-17T15:28:39Z",
				"description": "TypeScaript is a superset of Javascript that compiles to Javascript"
		}
	}
}

```

- GraphQL의 장점은 특정 쿼리에 대해 타입스크립트 타입을 생성할 수 있다는 점.
- GeoJSON 예제와 마찬가지로 GraphQL을 사용한 방법도 타입에 null이 가능한지 여부를 정확히 모델링할 수 있음.

e.g : GitHub 저장소에서 오픈 소스 라이선스를 조회하는 쿼리.

```tsx
query getLicense($owner:String!, $name:String!){

	respository(owner:$owner, name:$name) {
		description
		licenseInfo {
			spdxId
			name
		}
	}
}
```

- $owner와 $name은 타입이 정의된 GraphQL 변수.
- 타입 문법이 타입스크립트와 매우 비슷함.
- String은 GraphQL의 타입. (타입스크립트에서는 string)
- 타입스크립트에서 string 타입은 null이 불가능 하지만 GraphQL의 string 타입에서는 null이 가능함.
- 타입 뒤의 !는 null이 아님을 명시.

### 요약

- 코드의 세밀한 곳 까지 타입 안전성을 얻기 위해 API 또는 데이터 형식에 대한 타입 생성을 고려해야함.
- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있기 때문에 데이터보다는 명세로부터 코드를 생성하는 것이 좋음.

## 36 해당 분야의 용어로 타입 이름 짓기

- 이름 짓기 역시 타입 설계에서 중요한 부분.
- 엄선된 타입, 속성, 변수의 이름은 의도를 명확히 하고 코드와 타입의 추상화 수준을 높여줌.
- 잘못 선택한 타입 이름은 코드의 의도를 왜곡하고 잘못된 개념을 심어줌.

e.g : 동물들의 데이터베이스를 구축한다고 가정. 이를 표현하기 위한 인터페이스

```tsx
interface Animal {
	name: string;
	endangered: boolean;
	habitat: string;
}

const leopard: Animal = {
	name: string;
	endangered: false,
	habitat:'tundra',
};
```

- name은 매우 일반적인 용어. 동물의 학명인지 일반적인 명칭인지 알 수 없음
- endangered 속성의 의도를 `멸종 위기` 또는 `멸종`으로 생각했을지도 모름.
- 서식지를 나타내는 habitat 속성은 범위가 너무 넓은 string 타입일 뿐만 아니라 서식지라는 뜻 자체도 불분명함.
- 객체의 변수명이 leopard이지만, name 속성의 값은 ‘Snow Leopard’ 라 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 불분명함.

e.g : 의미가 분명한 타입 선언

```tsx
interface Animal {
	commonName: string;
	genus: string;
	species: string;
	status: ConservationStatus;
	climates: KoppenClimate[];
}
type ConvervationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate = 'Af' | 'Am' | 'As' | 'Aw' | .... | 'ET';

const snowLeopard: Animal = {
	commonName: 'Snow Leopard',
	genus: 'Panthera',
	species: 'Uncia',
	status: 'VU', //취약종
	climates: ['ET', 'EF', 'Dfd'] //고산대 또는 아고산대
};
```

- name은 commonName, genus, species등 더 구체적인 용어로 대체
- endangered는 동물 보호 등급에 대한 IUCN의 표준 분류 체계인 ConservationStatus 타입의 status로 변경됨.
- habitat은 기후를 뜻하는 climates로 변경. 쾨펜 기후 분류를 사용

- 코드로 표현하고자 하는 모든 분야에는 주제를 설명하기 위한 전문 용어들이 있음.
- 자체적으로 용어를 만들기 보다는 해당 분야에 이미 존재하는 용어를 사용해야함.
- 전문 분야의 용어는 정확하게 사용해야합니다. 특정 용어를 다른 의미로 잘못 쓰게되면, 직접 만든 용어보다 더 혼란을 주게됨.

**타입, 속성, 변수에 이름을 붙일 때 명심해야할 세 가지 규칙**

- 동일한 의미를 나타낼 때는 같은 용어를 사용해야함. 정말 의미적으로 구분이 되어야 하는 경우에만 다른 용어를 사용.
- data, info, thing, item, object, entity 같은 모호하고 의미 없는 이름은 피해야함. entity라는 용어가 해당 분야에서 특별한 이름을 가진다면 문제없음. 그러나 귀찮다고 무심코 의미 없는 이름을 붙여서는 안됨.
- 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지를 고려해야함. 예를 들어 INodeList보다는 Directory가 더 의미있는 이름. Directory는 구현의 측면이 아니라 개념적인 측면에서 디렉터리를 생각나게 함. 좋은 이름은 추상화의 수준을 높이고 의도치 않은 충돌의 위험성을 줄여줌.

## 37 공식 명칭에는 상표를 붙이기

e.g 구조적 타이핑 특성으로 인해 이상한 결과가 나오는 코드

```tsx
interface Vector2D {
  x: number;
  y: number;
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm({ x: 3, y: 4 }); //정상. 결과는 5
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D); //정상. 결과는 동일하게 5
```

- 구조적 타이핑 관점에서는 문제가 없으나, 수학적으로 따지면 2차원 벡터를 사용해야함.
- calculateNorm 함수가 3차원 벡터를 허용하지 않게 하려면 공식 명칭(nominal typing)을 사용하면 됨.
- 공식 명칭을 사용하는것은 타입이 아니라 값의 관점에서 Vector2D라고 말하는 것.
- 공식 명칭 개념을 타입스크립트에서 흉내 내려면 상표(brand)를 붙이면됨. (ex : 스타벅스가 아니라 커피)

```tsx
interface Vector2D {
  _brand: "2d";
  x: number;
  y: number;
}
function vec2D(x: number, y: number): Vector2D {
  return { x, y, _brand: "2d" };
}
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y); //기존과 동일
}

calculateNorm(vec2D(3, 4)); //정상, 5를 반환.
const vec3D = { x: 3, y: 4, z: 1 };
calcuateNorm(vec3D); //Error _brand 속성이 .. 없습니다.
```

- 상표(\_brand)를 사용해서 calculateNorm 함수가 Vector2D 타입만 받는 것을 보장함.
- 그러나 vec3D 값에 \_brand: ‘2d’ 라고 추가하는 것 같은 악의적인 사용을 막을 수 없음.
- 다만 단순한 실수를 방지하는데 충분.

- 상표 기법은 타입 시스템에서 동작하지만, 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있음.
- 타입 시스템이기 때문에 런타임 오버헤드를 없앨 수 있고, 추가 속성을 붙일 수 없는 string이나 number 같은 내장 타입도 상표화 할 수 있음.

e.g : 절대 경로를 사용해 파일 시스템에 접근하는 함수를 가정.

```tsx
type AbsolutePath = string & { _brand: "abs" };
function listAbsolutePath(path: AbsolutePath) {
  //...
}
function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith("/");
}
```

- string 타입이면서 \_brand 속성을 가지는 객체는 만들 수 없음. AbsolutePath는 온전히 타입 시스템의 영역.
- 만약 path값이 절대 경로와 상대 경로 둘 다 될수 있다면, 타입을 정제해 주는 타입 가드를 사용해서 오류를 방지할 수 있음.

```tsx
function f(path:string){
	if(isAbsolutePath(path)){
		listAbsolutePath(path));
	}
	listAbsolutePath(path); //Error - string형식의 인수는 AbsolutePath 형식의 매개변수에 할당 될 수 없음.
}
```

- if 체크로 타입을 정제하는 방식은 매개변수 path가 절대 경로인지 또는 상대경로인지에 따라 분기하기 떄문에 분기하는 이유를 주석으로 붙이기에도 좋음.
- 단 주석이 오류를 방지해주는것은 아니라는점에 주의.
- 반면 로직을 분기하는 대신 오류가 발생한 곳에 path as AbsolutePath를 사용해서 오류를 제거할 수도 있지만 단언문은 지양할 것.
- 단언문을 쓰지 않고 AbsolutePath타입을 얻기 위한 유일한 방법은 AbsolutePath타입을 매개변수로 받거나 타입이 맞는지 체크하는 것 뿐

e.g 이진 검색을 하는 경우

```tsx
function binarySearch(<T>(xs: T[], x: T): boolean {
	let low = 0, high = xs.length - 1;
	while(high >= low){
		const mid = low + Math.floor((high - low) / 2);
		const v = xs[mid];
		if (v === x) return true;
		[low, high] = x > v ? [mid + 1, high] : [low, mid - 1];
	}
	return false;
}
```

- 이진 검색은 이미 정렬된 상태를 가정하므로, 목록이 이미 정렬되어있다면 문제가 없음.
- 그러나 목록이 정렬되어 있지 않다면 잘못된 결과가 나옴.
- 타입스크립트 타입 시스템에서는 목록이 정렬되어 있다는 의도를 표현하기가 어려움.

e.g: 상표기법을 이용한 예제

```tsx
type SortedList<T> = T[] & { _brand: "sorted" };

function isSorted<T>(xs: T[]): xs is SortedList<T> {
  for (let i = 1; i < xs.length; i++) {
    if (xs[i] < xs[i - 1]) {
      return false;
    }
  }
  return true;
}

function binarySearch<T>(xs: SortedList<T>, x: T): boolean {
  //...
}
```

- binarySearch를 호출하려면, 정렬되었다는 상표가 붙은 SortedList 타입의 값을 이용하거나 isSorted를 호출하여 정렬되었음을 증명해야함.
- isSorted에서 목록 전체를 루프 도는것이 효율적인 방법은 아니나, 적어도 안전성은 확보할 수 있음.

e.g : number 타입에도 상표를 붙이는 예제

```tsx
type Meters = number & { _brand: "meters" };
type Seconds = number & { _brand: "seconds" };

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000); //타입이 Meters
const oneMin = seconds(60); //타입이 Seconds
```

```tsx
const tenKm = oneKm * 10; //타입이 number
const v = oneKm / oneMin; //타입이 number
```

- number 타입에 상표가 붙여도 산술 연산 후에는 상표가 없어지기 때문에 실제로 사용하기에는 무리가 있음

### 요약

- 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에, 값을 세밀하게 구분하지 못하는 경우가 있음.
- 값을 구분하기 위해 공식 명칭이 필요하다면 상표를 붙이는 것을 고려해야함.
- 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있음.
