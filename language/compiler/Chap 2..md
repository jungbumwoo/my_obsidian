 

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

|용어|의미|
|---|---|
|**Terminal**|실제 코드에 등장하는 고정된 기호 (`if`, `else`, `(`, `)`, `{`, `}` 등)|
|**Non-terminal**|문법 구조를 표현하는 변수 (`stmt`, `expr`, `decl` 등)|
|**Production**|하나의 문법 규칙 (`stmt → if ( expr ) stmt else stmt`)|
|**Grammar**|이런 Production 들의 모음|
|**CFG**|문맥 자유 문법, 대부분의 프로그래밍 언어의 구조를 표현하기 위한 문법 형식|