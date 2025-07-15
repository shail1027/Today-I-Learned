## C++ `<queue>`와 `<stack>` 헤더 파일

### 개요

C++ STL(Standard Template Library)에는 기본적인 자료구조를 제공하는 컨테이너 어댑터(container adapter)가 존재한다.  
그 중에서 `<queue>`와 `<stack>`은 FIFO(선입선출), LIFO(후입선출) 구조를 각각 제공하는 큐와 스택을 구현하는 데 사용된다.  
이들은 내부적으로 `deque` 또는 `list` 등의 컨테이너를 기반으로 동작하며, 인터페이스만 간소화되어 있다.

---

### `<stack>` 헤더

`<stack>`은 LIFO(Last-In First-Out) 구조를 제공하는 자료구조이다.

#### 주요 멤버 함수

| 함수 | 설명 |
|------|------|
| `push(elem)` | 요소를 스택의 맨 위에 추가한다 |
| `pop()` | 맨 위의 요소를 제거한다 |
| `top()` | 맨 위의 요소를 반환한다 |
| `empty()` | 스택이 비어 있는지 확인한다 |
| `size()` | 스택에 들어 있는 요소 개수를 반환한다 |

#### e.g

```cpp
#include <iostream>
#include <stack>

int main() {
    std::stack<int> st;
    st.push(1);
    st.push(2);
    st.push(3);
    std::cout << st.top() << std::endl; // 3
    st.pop();
    std::cout << st.top() << std::endl; // 2
}
```

---

### `<queue>`헤더

`<queue>`는 FIFO(First-In-First-Out) 구조를 제공하는 자료구조이다.

#### 주요 멤버 함수

| 함수           | 설명                   |
| ------------ | -------------------- |
| `push(elem)` | 요소를 큐의 뒤에 추가한다       |
| `pop()`      | 큐의 앞 요소를 제거한다        |
| `front()`    | 큐의 앞 요소를 반환한다        |
| `back()`     | 큐의 마지막 요소를 반환한다      |
| `empty()`    | 큐가 비어 있는지 확인한다       |
| `size()`     | 큐에 들어 있는 요소 개수를 반환한다 |


#### e.g.

```cpp
#include <iostream>
#include <queue>

int main() {
    std::queue<int> q;
    q.push(10);
    q.push(20);
    std::cout << q.front() << std::endl; // 10
    std::cout << q.back() << std::endl;  // 20
    q.pop();
    std::cout << q.front() << std::endl; // 20
}
```

---

### 내부 구조

- `stack`과 `queue`는 **container adapter**로, 기본 컨테이너 위에서 인터페이스만 제공한다.
- 기본적으로 내부 컨테이너는 `deque`가 사용된다.
- 다음과 같이 사용자 정의 컨테이너를 지정할 수 있다.
  - ```cpp
    std::stack<int, std::vector<int>> myStack;
    std::queue<int, std::list<int>> myQueue;
    ```


  