 

2.2 Syntex Definition
프로그래밍 언어의 문법(Syntax)을 **형식적으로 정의하는 방법, 즉 문맥 자유 문법(Context-Free Grammar, CFG)**을 소개

**문맥 자유 문법(CFG)**은 프로그래밍 언어의 구조(문법)를 표현하는 형식적인 방법.
주로 **컴파일러의 프론트엔드**에서 사용되며, **if-else, while, 함수 호출 등 언어 구조를 계층적으로 표현**

문법(grammar)이란?
프로그래밍 언어의 문장을 형식적으로 정의하기 위한 규칙들의 모음.
이 문법은 다음 두 가지 구성요소로 이루어져 있음.

- **터미널(Terminal)**: 실제 코드에 나오는 고정된 단어들  
    예: `if`, `else`, `(`, `)` 같은 키워드, 기호 등
    
- **비터미널(Non-terminal)**: 문법 구조를 표현하는 변수  
    예: `expr`(표현식), `stmt`(문장), `decl`(선언) 등


ex)
Java if else
```Java
if (expression) statement else statement
```

```bash
stmt → if ( expr ) stmt else stmt
```

- 여기서 `stmt`는 비터미널 → 하나의 문장
- `expr`도 비터미널 → 표현식
- `if`, `else`, `(`, `)`는 터미널 → 실제 코드에 나오는 고정된 키워드/기호

 이 문법 규칙을 **생산 규칙(Production)**이라고 부름
- `→` 는 "**~는 ~로 구성될 수 있다**"는 의미
    
- 하나의 비터미널이 어떤 터미널/비터미널 조합으로 확장될 수 있는지 정의함
- 
stmt는 'if (expr) stmt else stmt' 형태일 수 있다"는 뜻

| 용어               | 의미                                                      |
| ---------------- | ------------------------------------------------------- |
| **Terminal**     | 실제 코드에 등장하는 고정된 기호 (`if`, `else`, `(`, `)`, `{`, `}` 등) |
| **Non-terminal** | 문법 구조를 표현하는 변수 (`stmt`, `expr`, `decl` 등)               |
| **Production**   | 하나의 문법 규칙 (`stmt → if ( expr ) stmt else stmt`)         |
| **Grammar**      | 이런 Production 들의 모음                                     |
| **CFG**          | 문맥 자유 문법, 대부분의 프로그래밍 언어의 구조를 표현하기 위한 문법 형식              |

2.2 Definition of Grammers

덧셈과 뺄셈이 포함된 숫자 표현식을 **문맥 자유 문법(CFG)**으로 표현하는 방법을 설명

- **list → list + digit**:  
    예: `3+5`, 또는 `3+5+2`, 즉 **앞부분이 list이고 마지막에 + 숫자가 추가됨**
    
- **list → list - digit**:  
    예: `9-2`, 또는 `9-2-1`
    
- **list → digit**:  
    한 자리 숫자 하나일 수도 있음 (예: `7`)
    
- **digit → 0 | 1 | ... | 9**:  
    한 자리 숫자는 이렇게 정의됨

하나로 묶으면,
`list → list + digit | list - digit | digit`

ex)  `9-5+2`는 아래처럼 분석
```scss
list
→ list + digit
→ (list - digit) + digit
→ ((digit) - digit) + digit
→ ((9) - 5) + 2
```

2.2.2 Derivation
**Derivation(도출)**이란?

> **Derivation**은 문법의 **시작 기호(start symbol)**에서 시작해서  
> **생산 규칙(production rule)**을 반복적으로 적용하면서  
> **문자열을 생성해 나가는 과정**

