##  C++ string 헤더 및 string 자료형

###  개요

C++에서 문자열을 처리할 때는 `std::string`과 C 스타일 문자열인 `char 배열`을 사용할 수 있다. `std::string`은 `<string>` 헤더를 포함시켜야 사용할 수 있는 C++ 표준 라이브러리 타입이다.

---

### string 헤더

```cpp
#include <string>
````

* `<string>`은 C++의 `std::string` 클래스 사용을 위한 헤더.
* 내부적으로는 `std::basic_string<char>`의 typedef이다.


### string vs char\[]

| 구분    | `std::string`               | `char[]` / `char*`             |
| ----- | --------------------------- | ------------------------------ |
| 정의    | C++ 문자열 클래스                 | C의 문자열 (문자 배열)                 |
| 메모리   | 동적 할당 (자동 관리)               | 정적 / 수동 동적 할당 필요               |
| 크기 변경 | 자동 조정됨                      | 직접 관리해야 함                      |
| 연산    | `+`, `==`, `.substr()` 등 지원 | `strcpy`, `strcat`, `strcmp` 등 |
| 안정성   | 상대적으로 안전                    | Null 종료 누락 위험 있음               |
| 헤더    | `<string>`                  | `<cstring>`                    |

e.g.

```cpp
std::string s = "hello";
char c[] = "hello";
```

---

### 왜 `.c_str()`을 써야 할까?

`std::string`은 C 스타일 문자열(`char*`)과는 다르다. C 표준 함수(`printf`, `strcmp`, `fopen` 등)는 `const char*`를 요구하므로 `std::string`을 넘길 수 없다. 이때 `.c_str()`을 사용해야 한다.

```cpp
std::string s = "example";
printf("%s\n", s.c_str());  // OK
// printf("%s\n", s);      // 컴파일 에러
```

* `s.c_str()`은 `const char*`를 반환한다.
* 반환된 포인터는 `s`가 변경되거나 소멸되면 무효화된다.

<br/>

----

### 자주 사용하는 string 멤버 함수

| 함수                      | 설명             |
| ----------------------- | -------------- |
| `.length()` / `.size()` | 문자열 길이         |
| `.substr(pos, len)`     | 부분 문자열 추출      |
| `.find("word")`         | 문자열 검색         |
| `.append("추가")` / `+`   | 문자열 결합         |
| `.c_str()`              | C 스타일 문자열로 변환  |
| `.empty()`              | 문자열이 비어 있는지 확인 |
| `.clear()`              | 문자열 초기화        |

e.g.

```cpp
std::string a = "hello";
std::string b = "world";
std::string c = a + " " + b;  // "hello world"
```

---

### 기타

* `std::getline(std::cin, s)` -> 공백 포함 문자열 입력 가능
* C++11 이후: `std::to_string(123)` -> `"123"`
* 문자열 비교: `==` 연산자로 가능 (내용 비교)