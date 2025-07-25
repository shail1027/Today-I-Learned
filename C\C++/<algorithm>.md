## C++ `<algorithm>` 헤더


### 개요

C++의 `<algorithm>` 헤더는 STL 컨테이너(`vector`, `array`, `deque` 등)와 함께 사용할 수 있는 다양한 **범용 알고리즘 함수**들을 포함하는 표준 라이브러리이다.  
정렬, 탐색, 조건 검사, 변형 등 여러 종류의 반복자 기반 연산을 지원하며, 코드의 재사용성과 가독성을 크게 향상시킨다.


```cpp
#include <algorithm>
```

---

### 자주 사용하는 함수

| 함수                              | 설명                         | 시간 복잡도        |
| ------------------------------- | -------------------------- | ------------- |
| `sort(first, last)`             | 구간을 오름차순으로 정렬              | 평균 O(n log n) |
| `reverse(first, last)`          | 구간을 반전                     | O(n)          |
| `max_element(first, last)`      | 최대값 반복자 반환                 | O(n)          |
| `min_element(first, last)`      | 최소값 반복자 반환                 | O(n)          |
| `find(first, last, value)`      | 해당 값을 찾고 반복자 반환            | O(n)          |
| `count(first, last, value)`     | 값의 등장 횟수 세기                | O(n)          |
| `accumulate(first, last, init)` | 합계 구하기 (헤더는 `<numeric>`)   | O(n)          |
| `all_of(first, last, pred)`     | 모두 조건 만족 여부                | O(n)          |
| `any_of(first, last, pred)`     | 하나라도 조건 만족 여부              | O(n)          |
| `none_of(first, last, pred)`    | 모두 조건 불만족 여부               | O(n)          |
| `unique(first, last)`           | 연속 중복 제거 (반환된 반복자로 잘라야 함)  | O(n)          |
| `lower_bound(first, last, val)` | 정렬된 구간에서 이진탐색으로 첫 ≥ val 찾기 | O(log n)      |
| `upper_bound(first, last, val)` | 정렬된 구간에서 첫 > val 찾기        | O(log n)      |

---

