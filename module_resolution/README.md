# Module Resolution

module resolution을 번역하면 모듈 해결이다. 사전적의미는 이렇고 사람들이 많이 쓰는 표현으로는 모듈 해석이라한다.

모듈 해석은 컴파일러가 `import` 구문을 해석할 때 무엇을 참조하고 있는지 알아내는 프로세스를 의미한다.

예를 들어 `import { a } from 'moduleA'` 라는 구문이 있을 때 a의 사용을 검사하기 위해서 무엇을 참조하는지 정확히 알아야하며 `moduleA`의 정의를 검사할 필요가 있다.

이 시점에서 컴파일러는 moduleA가 어떻게 생겼는지 알아야한다.
한마디로 `module A`는 `.ts / .tsx / .d.ts` 파일에 정의가 돼있고 이 파일에 따라서 moduleA를 사용할 수 있다.

컴파일러는 우선적으로 import 하는 모듈이 어디에 있는지 찾는다. 이 과정에서 컴파일러는 두 가지 방법을 사용하는데 다음과 같다.

- Classic
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

##### Relative

상대적 imports 는 `/` 또는 `./` 또는 `../` 로 시작하는 방식을 의미한다.

다음의 예시를 확인해보자

```typescript
import Entry from "./components/Entry";

import { DefaultHeaders } from "../constants.http";

import "/mod";
```

상대적 imports 는 import 한 파일에 따라서 해결될 수 있으며
`ambient module declaration` 으로는 해결 될 수 없다.

이 방법은 실행시점에 파일의 상대적 위치가 보장될 경우에만 사용해야한다.

##### Non-relative

```typescript
import * as $ from "jquery";

import { Component } from "@angular/core";
```

비상대적 imports 는 `baseUrl` 또는 `path mapping` 에 따라서 해결이 된다. 또한 `ambient module declaration` 으로도 해결이 된다.

이 방법은 외부 패키지를 import 할 때 사용이 된다.

---

### module resolution 방법

위에서 언급했던 두 가지 방법에 대해서 알아보자.
두 가지 방법 중 `--moduleResolution` 플래그 값으로 한 가지 방법을 특정할 수 있으며 `--module commonjs` 에서는 default 는 `Node` 이다. 반면에 설정을 하면 `Classic` 방법이 특정이 된다.
Classic 방법은 `--module` 이 `amd, system, umd, es2015, esnext` 등의 값으로 설정된 경우를 포함한다.

> `Node` 방식은 대부분의 타입스크립트 커뮤니티에서 사용되는 방법이며 대부분의 프로젝트에서 추천되는 방식이다. 그러므로 module resolution 관련 문제가 발생하면 `moduleResolution: "node"` 로 설정하여 해결하면 된다.

##### Classic
