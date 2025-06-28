# ESM Only Commandline Tool

## 요구사항

1. 개발할 때는 `bin/dev-cli.js` 를 참조하고 패킹한 후에는 `bin/cli.js` 를 참조한다.
2. `bin/dev-cli.js` 는 ts 파일(`src/index.ts`)을 참조하여 변경이 발생해도 매번 컴파일을 하지않아 개발하는데 편리하다.
3. `bin/cli.js` 는 번들된 결과물(`dist/index.js`)을 참조하여 불필요한 내용이 없어진 번들링된 파일을 써서 최적화되어있다.
4. 소스코드 `src/index.ts` 의 경우, esm 표준을 준수하여 각 ts 파일을 js 라는 확장자로 작성된 코드로 import 한다.

## dev-cli.js 에서 register 로 쓸 후보군

1. ts-node
2. tsx

## register란?

Node.js에서 특정 파일을 실행할 때 **즉시 변환(Just-In-Time 컴파일)**을 가능하게 해 주는 require 훅을 등록(register)해 주는 모듈입니다.

e.g.)

```js
import { register } from "node:module";
import { pathToFileURL } from "node:url";

register("ts-node/esm", pathToFileURL("./"));

import("../src/index.ts");
```

## ts-node 로 구성하기

> bash

```bash
node --experimental-specifier-resolution=node --experimental-loader ts-node/esm.mjs ./src/index.ts
```

> dev-cli.js

```js
// #!/usr/bin/env node

import { register } from "node:module";
import { pathToFileURL } from "node:url";

register("ts-node/esm", pathToFileURL("./"));

import("../src/index.ts");
```

> cli.js

```js
import("../dist/index.js");
```

## tsx 로 구성하기

> bash

```bash
yarn tsx ./src/index.ts
```

> dev-cli.js

```js
import { tsImport } from "tsx/esm/api";

await tsImport("../src/index.ts", import.meta.url);
```

> cli.js

```js
import("../dist/index.js");
```

## 결론

ts-node 나 tsx 나 모두 사용가능하나 각각 장단점이 있다. ts-node 의 인터페이스는 좀 더 열려있어서 node.js 의 맥락을 이해할 수 있는 대신 사용하기 조금 어려울 수 있다. 다만 tsx 는 추상화가 잘 되어 있어 정말 쉽게 도입할 수 있는 대신 추상화 레벨이 높다 보니 디버깅이 어려울 수도 있을 것 같다. tsx가 코드가 간결한 느낌이 있다.

ts-node 는 tsc 기반으로 동작되므로 느릴 수 있지만 tsc 의 컴파일 옵션을 그대로 사용하므로 디버깅이 쉽다. tsx 는 esbuild 기반으로 동작되어 빠르기 때문에 개발 환경에서는 적절한 선택일 수 있다.

실제로 시간을 비교해보면 ts-node 로 dev-cli.js 를 실행한 경우 0.98s 가 걸렸고 tsx 로 실행한 경우 0.21s 가 걸렸다.

```
➜  hello-tsx-cli git:(main) ✗ time yarn hello-tsx-cli-dev
Hello, World!
yarn hello-tsx-cli-dev  0.21s user 0.04s system 123% cpu 0.204 total

➜  hello-ts-node-cli git:(main) ✗ time yarn hello-ts-node-cli-dev
Hello, World!
yarn hello-ts-node-cli-dev  0.98s user 0.08s system 215% cpu 0.495 total
```
