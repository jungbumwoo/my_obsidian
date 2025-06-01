
**Three-Address Code(3주소 코드)**는 컴파일러가 소스 코드를 처리할 때 중간 단계에서 사용하는 **간단한 명령어 형태**예요. 이 명령어는 마치 어셈블리 코드처럼 생겼고, 각 명령어에는 보통 **3개의 피연산자(operand)**가 들어갑니다.

**3-어드레스 코드**(Three-address code)는 [컴파일러](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC "컴파일러")에서 사용되는 [중간 언어](https://ko.wikipedia.org/wiki/%EC%A4%91%EA%B0%84_%EC%96%B8%EC%96%B4 "중간 언어")의 한 종류로, [컴파일러 최적화](https://ko.wikipedia.org/wiki/%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC_%EC%B5%9C%EC%A0%81%ED%99%94 "컴파일러 최적화")를 실현하는데 사용된다. '3-어드레스'로 불리는 것은 2개의 입력용과 1개의 출력용 [메모리 주소](https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EB%A6%AC_%EC%A3%BC%EC%86%8C "메모리 주소") 혹은 [레지스터](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EC%84%B8%EC%84%9C_%EB%A0%88%EC%A7%80%EC%8A%A4%ED%84%B0 "프로세서 레지스터")를 지정하는 형식이기 때문

t1 = inttofloat(60)   // 정수 60을 실수로 변환
t2 = id3 * t1         // 변수 id3와 t1을 곱함
t3 = id2 + t2         // 변수 id2와 t2를 더함
id1 = t3              // 결과를 id1에 저장


1.6.2 Environments and States

프로그램에서 변수 이름 → 메모리 위치 → 실제 값은 **순차적으로 바인딩(binding)** 됨
이 과정은 **정적(static)** 또는 **동적(dynamic)** 으로 이루어질 수 있음.


