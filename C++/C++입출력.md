## C++ 입력/출력 비교

### 개요

C++에서는 입력과 출력을 수행할 수 있는 다양한 함수들이 존재한다.  
대표적으로 C++ 스타일의 `cin`, `cout`, `getline`이 있으며, C 스타일의 `scanf`, `printf`도 여전히 많이 사용된다.  
이들은 기능과 성능, 사용 목적 면에서 중요한 차이를 보인다.

---

### `cin` vs `scanf`

| 항목 | `cin` | `scanf` |
|------|--------|---------|
| 스타일 | C++ | C |
| 헤더 | `<iostream>` | `<stdio.h>` |
| 네임스페이스 | `std` 필요 | 불필요 |
| 타입 안정성 | 높음 (템플릿 기반) | 낮음 (형식 문자열 직접 지정) |
| 포맷 문자열 | 불필요 | 필수 (`%d`, `%lf` 등) |
| 성능 | 느림 (기본 설정 시) | 빠름 |
| 문자열 입력 | 공백 이전까지만 읽음 (`cin >> str`) | 동일 |

e.g. :

```cpp
int a;
std::cin >> a;      // cin 사용
scanf("%d", &a);    // scanf 사용
```

---

### `cout` vs `printf`

| 항목    | `cout`                      | `printf`                 |
| ----- | --------------------------- | ------------------------ |
| 스타일   | C++                         | C                        |
| 포맷 제어 | `setprecision`, `setw` 등 사용 | 서식 문자열 (`%d`, `%.2f`) 사용 |
| 복합 출력 | 직관적 (연결 연산자 `<<`)           | 포맷 직접 조정                 |
| 성능    | 느림 (기본 설정 시)                | 빠름                       |
| 유연성   | 객체 출력 등 가능                  | 포맷 중심, 텍스트 출력에 최적화       |

e.g.

```cpp
int x = 5;
std::cout << "x = " << x << std::endl;    // C++ 스타일
printf("x = %d\n", x);                    // C 스타일
```

---

### `getline`

- `std::getline(std::cin, str)`은 줄 단위로 입력을 받으며, 공백을 포함한 문자열을 입력받는다.
- `cin >> str`은 공백 전까지만 읽는다.

```cpp
std::string name;
std::getline(std::cin, name);
```

**주의 사항**

`cin >> int`등 숫자 입력 후 `getline`을 바로 사용할 경우 `\n`이 버퍼에 남아있어 `getline`이 바로 종료된다. 이를 방지하기 위해서는 `cin.ignore()`을 사용해야 한다.

```cpp
int age;
std::cin >> age;
std::cin.ignore();            // \n 제거
std::getline(std::cin, name); // 정상 동작
```

---

### 성능 최적화 설정

기본적어로 `cin`과 `cout`은 `scanf`와`printf`와 동기화되어 있어 성능이 낮다. 입출력 속도를 높이기 위해서는 아래와 같은 설정을 사용해야 한다.

```cpp
std::ios::sync_with_stdio(false);
std::cin.tie(nullptr);
```

- `std::ios::sync_with_stdio(false);` : C와 C++의 스트림 동기화를 해제하여 cin, cout의 속도를 높인다.

- `std::cin.tie(nullptr);` : cin과 cout의 연결을 끊어 cin 후 cout이 자동 flush되는 것을 방지한다.

위 두 설정은 입출력 속도를 2~3배 이상 향상시킬 수 있다.



