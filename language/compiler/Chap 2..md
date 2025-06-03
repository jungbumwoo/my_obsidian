 

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


statement: 실행가능한 최소의 독립적인 코드 조각
expression: 수식. 하나 이상의 값으로 표현 될 수 있는 코드

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
- production body: 문법 규칙(->)의  오른쪽 부분 


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

2.2.5 연산자의 우선순위(precedence)

2.2.6 연산자의 결합 방향(associativity)

## 2.3 Syntax Directed Translation (구문 지향 번역)

> **어떻게 번역할지, 처리할지에 대한 규칙이나 코드**를 **직접 연결**해서 번역/처리를 수행하는 방식


| 용어                             | 의미                                                     |
| ------------------------------ | ------------------------------------------------------ |
| **Attribute** (속성)             | 문법 기호(비터미널/터미널)에 붙는 정보. 예: 타입, 위치, 코드 길이 등             |
| **Translation Scheme** (번역 스킴) | 문법의 각 규칙(production)에 실행할 코드 조각을 붙여, 파싱 도중 번역을 수행하는 방식 |

프로그래밍 언어의 **구성 요소(construct)**들에는 여러 가지 **속성(정보)**이 필요

예를 들어, 다음과 같은 것들이 "속성(attribute)"입니다:

- 변수의 **자료형** (`int`, `float` 등)
    
- 표현식의 **계산 결과 값**
    
- 구문 트리에서 어떤 노드의 **주소**, **오프셋**
    
- 생성된 **명령어 수**
    

이러한 속성은 문법 기호(예: `Expr`, `Stmt`)에 **붙여서 함께 관리**

- `Expr`에 붙을 수 있는 attribute 예시:
    
    - `Expr.val`: 계산된 값
        
    - `Expr.type`: 자료형
        
    - `Expr.code`: 생성된 코드

### 2. **Translation Scheme (번역 스킴)**

문법 규칙에다가 **실제 코드 조각을 붙여서**,  
**구문 분석(parsing)** 중에 그 코드들을 **차례로 실행**

| 목적     | 예시                                        |
| ------ | ----------------------------------------- |
| 계산     | `E.val = E₁.val + T.val`                  |
| 출력     | `print('+')`                              |
| AST 생성 | `E.node = new AddNode(E₁.node, T.node)`   |
| 코드 생성  | `E.code = concat(E₁.code, T.code, "ADD")` |

- 컴파일러에서 사용하는 문법 규칙 작성 시의 관례(convention)

우리가 문법 규칙(생성 규칙)을 작성할 때,  
동일한 비터미널(Non-terminal)이 한 규칙 안에 여러 번 등장하는 경우가 있음
```
E → E + E

E → E₁ + E₂
세 개의 `E`가 등장하지만, 서로 다른 역할을 하므로 이름을 구분
```

```
A → X₁ X₂ ... Xₙ

```

모두 같은 비터미널 X의 반복 사용을 의미하는 게 아니라,  
그냥 **임의의 여러 문법 기호들을 나타내는 표기**일 수 있습니다.

즉, 어떤 경우에는 `X₁`, `X₂`가 진짜 `X`의 다른 인스턴스일 수 있고,  
어떤 경우에는 그냥 각각 다른 기호일 수 있음

 2.3.3 간단한(SIMPLE) Syntax-Directed Definition (구문 지시 정의)

이 절에서는 **문법의 생성 규칙과 의미 규칙을 결합하여 번역(예: postfix 표기)하는 방식** 중에서도,  
특히 "간단한 방식(Simple SDD)"을 설명

---

 🔧 Simple SDD의 정의

**Simple Syntax-Directed Definition**은 다음과 같은 특징을 가집니다:

> "생성 규칙의 왼쪽 비터미널(결과)의 번역 결과는  
> 오른쪽에 있는 비터미널들의 번역 결과를 **그대로 순서대로 이어붙인 것(Concatenation)**이다.  
> (필요하다면 중간에 추가 문자열이 끼어들 수도 있음)"

---

#### 🧪 예시: postfix 변환

##### 문법 규칙:

`expr → expr + term`

##### 의미 규칙:

`expr.t = expr₁.t || term.t || '+'`

여기서 의미는 다음과 같습니다:

- `expr₁.t`: 왼쪽에 있는 표현식의 postfix 결과
    
- `term.t`: 오른쪽 항의 postfix 결과
    
- `'+'`: 마지막에 연산자 추가
    

**즉**, 중위 표현식 `a + b`는 → postfix로 `a b +`가 됩니다.

#### 2.3.5 Translation Schemes

프로그램 조각(코드)을 문법 규칙 안에 직접 넣어서 번역 과정을 정의하는 방법

- 기존방식과 비교
|구분|Syntax-Directed Definition (SDD)|Translation Scheme|
|---|---|---|
|방식|트리 노드에 "속성값"을 저장하고 계산|**프로덕션 안에 코드를 직접 작성**해서 번역을 수행|
|계산 시점|규칙 기반 (속성으로 계산)|**명시적으로 실행 위치 지정**|
|예시|`expr.t = term.t + '+'`|`expr → term { print('+') }`|

- 예시 설명

`rest → + term { print('+') } rest1`

여기서 `{ print('+') }`는 **semantic action (의미 동작)** 
이 코드 조각은 구문 분석 중 이 부분이 실행될 때 **즉시 실행됩니다.**

즉, 파서가 `rest → + term ...` 규칙을 사용할 때,  
`term` 뒤에 `+` 기호를 바로 출력하도록 지정한 것.