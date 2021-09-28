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

이 방법이 Node 라고 이름이 붙은 이유는 Node 에서 module resolution 하는 방법을 흉내내기 떄문이다. 그렇다면 이 방법을 이해하기 위해서 node에서 어떻게 module resolution 하는지 알아봐야한다.

1. node에서 module resolution 하는 방법

node에서는 `import` 라는 구문을 사용하기 전에는 `require` 라는 키워드를 통해서 모듈을 불러왔다.

require 키워드는 상대적이냐 비상대적이냐에 따라서 다르게 동작을한다.

- 상대적인 경우
  상대적인 경우의 방법을 알아보자.
  예를 들어서 `/root/src/moduleA.js` 파일에서 `var x = require("./moduleB");` 라는 코드로 moduleB에 대해서 import 한다고 가정했을때 다음과 같은 방식으로 Node.js 는 module을 찾는다.

```
1. /root/src/moduleB.js 가 존재하는지 찾는다.

2. /root/src/moduleB 라는 폴더에 main 모듈이 특정이 되어있는
package.json 파일이 있는지 찾는다.
위 예시에서는 /root/src.moduleB/package.json 에서
{ "main": "lib/mainModule.js" } 를 포함하고 있는지 찾은 후 포함하고 있다면
Node.js는 /root/src/moduleB/lib/mainModule.js 를 참조한다.

3. /root/src/moduleB 가 index.js 를 포함하고 있는지 찾는다.
index.js 라는 파일은 암묵적으로 main 모듈이라고 간주된다.
```

- 비상대적인 경우
  반면 비상대적인 경우에는 `node_modules` 라는 폴더를 만들어 그 폴더 안에 모듈을 정의하고 사용한다. 이 폴더는 import 하는 파일과 동일한 위치에 존재하거나 상위 위치에 존재하기 때문에 Node.js 는 디렉토리 트리를 따라 올라가면서 `node_modules` 라는 폴더를 찾게된다.
  <br />
  상대적인 경우와 같은 예시로 비교해보자.

만약 `/root/src/moduleA.js` 파일에서 `x = require("moduleB")` 를 import 한다고 가정했을 때, 다음과 같은 위치에서 `mobuleB`를 찾는다.

```
1. /root/src/node_modules/moduleB.js
2. /root/src/node_modules/moduleB/package.json (main 속성이 있을 때)
3. /root/src/node_modules/moduleB/index.js

4. /root/node_modules/moduleB.js
5. /root/node_modules/moduleB/package.json (main 속성이 있을 때)
6. /root/node_modules/moduleB/index.js

7. /node_modules/moduleB.js
8. /node_modules/moduleB/packages.json (main 속성이 있을 때)
9. /node_modules/moduleB/index.js
```

> Node.js 는 4 ~ 7 번 과정을 진행하지 않음을 명시하자.

---

위에서 Classic 방식과 node 방식이 어떻게 module resolution 을 하는지 알아보았다. 그렇다면 TypeScript 에서는 어떻게 module resolution을 할까?

### TypeScript module resolution

타입스크립트의 module resolution은 컴파일 할 때 모듈을 찾는 것이 아니라 node js의 실행 시간 module resolution 방법을 따라한다.

이 방법을 따라하기 위해 타입스크립트는 TS 소스 파일`(.ts, .tsx, .d.ts)`을 Node가 모듈을 찾는 로직에 덮어쓴다.(js 파일을 찾는 대신 ts 파일을 찾는 거 같음) 그리고 package.json main 과 동일한 목적을 가진 `types` 라는 속성을 사용한다. (컴파일러는 이 types 속성을 이용하여 main 을 찾아낸다.)

다음 예시를 보자.
`import { b } from "./moduleB"` 를 `/root/src/moduleA.ts` 파일에서 실행되면 아래의 위치에서 `moduleB` 를 찾게된다.

```
1. /root/src/moduleB.ts
2. /root/src/moduleB.tsx
3. /root/src/moduleB.d.ts
4. /root/src/moduleB/package.json (types 라는 속성을 정의할 수 있을 시)
5. /root/src/moduleB/index.ts
6. /root/src/moduleB/index.tsx
7. /root/src/moduleB/index.d.ts

```
