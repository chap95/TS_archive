# Module Resolution

module resolution을 번역하면 모듈 해결이다. 사전적의미는 이렇고 사람들이 많이 쓰는 표현으로는 모듈 해석이라한다.

모듈 해석은 컴파일러가 `import` 구문을 해석할 때 무엇을 참조하고 있는지 알아내는 프로세스를 의미한다.

예를 들어 `import { a } from 'moduleA'` 라는 구문이 있을 때 a의 사용을 검사하기 위해서 무엇을 참조하는지 정확히 알아야하며 `moduleA`의 정의를 검사할 필요가 있다.

이 시점에서 컴파일러는 moduleA가 어떻게 생겼는지 알아야한다.
한마디로 `module A`는 `.ts / .tsx / .d.ts` 파일에 정의가 돼있고 이 파일에 따라서 moduleA를 사용할 수 있다.

컴파일러는 우선적으로 import 하는 모듈이 어디에 있는지 찾는다. 이 과정에서 컴파일러는 두 가지 방법을 사용하는데 다음과 같다.

- Classic
  - Relative
  - Non-Relative
- Node

이 두 가지 방법 모두 컴파일러에게 모듈이 어느 곳에 있는지 알려준다.

이 방법을 통해 모듈을 찾지 못했고 모듈이 상대적이지 않다면 컴파일러는 `ambient module declaration`의 위치를 찾는다.

결국에 컴파일러가 모듈의 위치를 찾아내지 못하면 에러를 발생시키게 된다.
위에서 들었던 예시에서 module A 를 찾이 못하면
`TS2307 : Cannot find module 'moduleA'`
형태의 에러를 발생키데 된다.

---

### Relative VS. Non-relative module imports

모듈 참조 방식이 상대적이냐 비상대적이냐에 따라서 module imports는 다른 방법으로 해결이 된다.

---

### module resolution 방법

위에서 언급했던 두 가지 방법에 대해서 알아보자.
두 가지 방법 중 `--moduleResolution` 플래그 값으로 한 가지 방법을 특정할 수 있으며 `--module commonjs` 에서는 default 는 `Node` 이다. 반면에 설정을 하면 `Classic` 방법이 특정이 된다.
Classic 방법은 `--module` 이 `amd, system, umd, es2015, esnext` 등의 값으로 설정된 경우를 포함한다.

> `Node` 방식은 대부분의 타입스크립트 커뮤니티에서 사용되는 방법이며 대부분의 프로젝트에서 추천되는 방식이다. 그러므로 module resolution 관련 문제가 발생하면 `moduleResolution: "node"` 로 설정하여 해결하면 된다.

---

##### Classic

classic 방법은 상대적 import 와 비상대적 import 가 있다. 원래는 classic 한 방법이 module resolution 방식 중 하나였지만 요즘에는 이전 버전들과의 호환성 문제를 해결하기 위해 자주 사용된다.

1. Relative

상대적 imports는 import를 하는 파일의 위치에 따라 모듈 검사를 실시하게 된다.
다음과 같이 `/` 또는 `./` 또는 `../` 로 시작하면 상대적 imports 이다.

```typescript
import Entry from "./components/Entry";

import { DefaultHeaders } from "../constants.http";

import "/mod";
```

동작하는 방식은 단순하다.
`/root/src/folder/A.ts` 에서 `import { b } from "./moduleB"` 를 수행하면 다음과 같은 위치에서 b 를 찾게된다.

```
1. /root/src/folder/moduleB.ts
2. /root/src/folder/moduleB.d.ts
```

2. Non-relative

비상대적 imports는 import 하는 파일의 위치에서 시작해 파일 디렉토리 트리를 따라 올라가면서 모듈을 찾는 방식이다.

상대적 imports와 달리 `/` 또는 `./` 또는 `../` 와 같은 상대적 위치를 나타내는 표시를 사용하지 않는다.

```typescript
import * as $ from "jquery";

import { Component } from "@angular/core";
```

비상대적 imports 는 `baseUrl` 또는 `path mapping` 에 따라서 해결이 된다. 또한 `ambient module declaration` 으로도 해결이 된다.

이 방법은 외부 패키지를 import 할 때 사용이 된다.

`/root/src/folder/A.ts` 에서 `import { b } from "./moduleB"` 를 수행하면 다음과 같은 위치에서 b 를 찾게된다.

```
/root/src/folder/moduleB.ts
/root/src/folder/moduleB.d.ts
/root/src/moduleB.ts
/root/src/moduleB.d.ts
/root/moduleB.ts
/root/moduleB.d.ts
/moduleB.ts
/moduleB.d.ts
```

---

##### Node
