## C++ `<regex>` 헤더 파일

### C++ `<regex>` 헤더

C++11부터 표준 라이브러리에 추가된 `<regex>` 헤더는 정규 표현식(Regular Expressions)을 사용하여 문자열을 검색, 일치, 대체하는 기능을 제공한다.

-----

### 주요 클래스 및 함수

`<regex>` 헤더는 주로 아래와 같은 클래스와 함수들로 구성된다.

### 1. `std::regex`

정규 표현식 객체를 나타내는 클래스이다. 이 객체를 사용하여 문자열에서 찾을 패턴을 정의한다.

```cpp
#include <regex>
#include <iostream>

// 숫자로만 이루어진 문자열을 찾는 정규식
std::regex num_regex("[0-9]+"); 
```

### 2. `std::smatch`

정규 표현식과 일치하는 결과를 저장하는 컨테이너이다. `std::match_results`의 일종이며, 문자열(string)을 대상으로 할 때 사용된다.

```cpp
#include <regex>
#include <iostream>

std::string s = "안녕하세요, 12345입니다.";
std::smatch m;

// regex_search 함수로 패턴을 찾고 결과를 m에 저장
std::regex_search(s, m, std::regex("[0-9]+")); 

if (!m.empty()) {
    std::cout << "일치하는 문자열: " << m[0] << std::endl; // 출력: 12345
}
```

### 3. 정규 표현식 함수

  * **`std::regex_match()`**: 문자열 **전체**가 정규 표현식 패턴과 일치하는지 확인한다.

    ```cpp
    std::string s1 = "12345";
    std::string s2 = "12345abc";

    // s1은 전체가 숫자이므로 true 반환
    bool result1 = std::regex_match(s1, std::regex("[0-9]+")); 
    // s2는 전체가 숫자가 아니므로 false 반환
    bool result2 = std::regex_match(s2, std::regex("[0-9]+")); 
    ```

  * **`std::regex_search()`**: 문자열 **일부**가 정규 표현식 패턴과 일치하는지 확인한다. 일치하는 첫 번째 부분을 찾는다.

    ```cpp
    std::string s = "안녕하세요, 12345입니다.";

    // 문자열 일부에 숫자가 있으므로 true 반환
    bool result = std::regex_search(s, std::regex("[0-9]+"));
    ```

  * **`std::regex_replace()`**: 정규 표현식과 일치하는 부분을 다른 문자열로 대체한다.

    ```cpp
    std::string s = "전화번호: 010-1234-5678, 이메일: example@test.com";
    std::regex reg("010-[0-9]{4}-[0-9]{4}"); // 전화번호 패턴

    // 전화번호를 "***-****-****"로 대체
    std::string result = std::regex_replace(s, reg, "***-****-****");

    std::cout << result << std::endl; 
    // 출력: 전화번호: ***-****-****, 이메일: example@test.com
    ```

-----

### 정규 표현식 문법 

| 문자 | 의미 | 예시 |
| :--- | :--- | :--- |
| `.` | 모든 단일 문자 | `a.c`는 `abc`, `a#c`와 일치 |
| `[]` | 괄호 안의 문자 중 하나 | `[abc]`는 `a`, `b`, `c` 중 하나와 일치 |
| `^` | 문자열의 시작 | `^abc`는 `abc...`와 일치 |
| `$` | 문자열의 끝 | `abc$`는 `...abc`와 일치 |
| `*` | 0회 이상 반복 | `a*`는 `` , `a`, `aa`, `aaa`와 일치 | | `+` | 1회 이상 반복 | `a+`는 `a`, `aa`, `aaa`와 일치 | | `?` | 0회 또는 1회 반복 | `a?`는  ``, `a`와 일치 |
| `{n}` | 정확히 n번 반복 | `a{3}`는 `aaa`와 일치 |
| `{n,}`| n회 이상 반복 | `a{2,}`는 `aa`, `aaa`와 일치 |
| `{n,m}`| n회에서 m회 반복 | `a{1,3}`은 `a`, `aa`, `aaa`와 일치 |
| `\|` | OR 연산자 | `(ab)\|(cd)`는 `ab` 또는 `cd`와 일치 |
| `()` | 그룹핑 | `(abc)+`는 `abc`, `abcabc`와 일치 |
| `\` | 특수 문자 이스케이프 | `\+`는 리터럴 `+`와 일치 |

-----
