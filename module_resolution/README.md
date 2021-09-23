# Module Resolution

module resolution을 번역하면 모듈 해결이다. 사전적의미는 이렇고 사람들이 많이 쓰는 표현으로는 모듈 해석이라한다.

모듈 해석은 컴파일러가 `import` 구문을 해석할 때 무엇을 참조하고 있는지 알아내는 프로세스를 의미한다.

예를 들어 `import { a } from 'moduleA'` 라는 구문이 있을 때 a의 사용을 검사하기 위해서 무엇을 참조하는지 정확히 알아야하며 `moduleA`의 정의를 검사할 필요가 있다.
