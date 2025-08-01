## C++ `map` 자료구조


### 개요

`std::map`은 C++ STL에서 제공하는 **정렬된 키-값 쌍(key-value pair)** 기반의 연관 컨테이너이다.  
각 키는 **중복될 수 없으며**, 자동으로 **오름차순 정렬**된 상태로 저장된다.  
내부적으로는 **이진 탐색 트리(Red-Black Tree)** 로 구현되어 있다.

---

### 헤더 및 선언

```cpp
#include <map>
std::map<std::string, int> m; // string -> int 매핑
```

---

### 주요 특징

- `O(log n)`의 시간 복잡도로 삽입, 탐색, 삭제 가능
- 키는 중복되지 않으며 자동 정렬됨
- `[]`연산자를 통해 배열처러 사용 가능
- 반복자를 사용해 순회 가능


---

### 주요 함수

| 함수                   | 설명                         |
| -------------------- | -------------------------- |
| `m[key]`             | 키에 대응되는 값을 반환 (없으면 0으로 생성) |
| `m.at(key)`          | 키의 값을 반환 (없으면 예외 발생)       |
| `insert({key, val})` | 쌍을 삽입                      |
| `erase(key)`         | 해당 키를 제거                   |
| `find(key)`          | 반복자 반환 (없으면 `end()`)       |
| `count(key)`         | 키 존재 여부 (0 또는 1)           |
| `size()`             | 원소 개수 반환                   |
| `clear()`            | 전체 삭제                      |
| `empty()`            | 비어 있는지 확인                  |
