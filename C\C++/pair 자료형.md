## 'pair' 자료형

### 개요

`std::pair`는 두 개의 값을 하나의 쌍으로 묶어서 표현할 수 있는 C++ 표준 템플릿 클래스이다. 값의 타입이 서로 달라도 묶을 수 있으며, 주로 맵(map) 자료구조, 반환값 튜플, 정렬용 묶음으로 사용된다.

---

### 선언 및 초기화

**기본 형태**
```cpp
std::pair<int, std::string> p1;
p1.first = 1;
p1.second = "apple";
```

<br>

**생성자 사용**

```cpp
std::pair<int, std::string> p2(2, "orange");
```

<br>


**`make_pair` 사용**

```cpp
auto p3 = std::make_pair(3, "cherry");
```
- `make_pair`는 타입을 자동으로 추론해준다.


---

### 접근 방법

- `first` : 첫 번째 원소에 접근
- `second` : 두 번째 원소에 접근

```cpp
std::pair<int, std::string> p = std::make_pair(10, "ten");
std::cout << p.first; // 10
std::cout << p.second; // ten
```

<br>

---

### 비교 및 정렬

- `pair`는 사전순(lexicographical order)으로 비교된다.
- 먼저 `first`를 비교하고, 같으면 `second`를 비교한다.

```cpp
std::pair<int, int> a = {1, 3};
std::pair<int, int> b = {1, 4};

if(a < b){
    // true : a.first == b.fist이고, a.second < b.second 이므로
}
```

<br>


**정렬 예시**

```cpp
std::vector<std::pair<int, int>> v = {{2, 3}, {1, 5}, {2, 1}};
std::sort(v.begin(), v.end());
// (1, 5) (2, 1) (2, 3) 순으로 정렬
```

---

### 활용 예시

1. 여러 값 반환
```cpp
std::pair<int, int> getMinMax(){
    return std::make_pair(1, 100);
}
```

2. map의 요소
```cpp
std::map<std::string, int> m;
m["apple"] = 5;

// map의 요소는 std::pair<const Key, T> 타입
for (auto it = m.begin(); it != m.end(); ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}
```


3. 좌표, 값 묶음 등

```cpp
std::pair<int, int> point = {3, 4}; // 좌표 표현
```


---

### 구조체와의 차이점

| 항목     | `pair`                 | 사용자 정의 `struct`        |
| ------ | ---------------------- | ---------------------- |
| 선언 편의성 | 높음                     | 낮음                     |
| 이름 가독성 | 낮음 (`first`, `second`) | 높음 (의미 있는 변수명)         |
| 정렬 기준  | 자동 (`first`, `second`) | 수동 정의 필요 (`operator<`) |

- 빠르게 사용할 때는 `pair`가 간편하지만, 의미 전달이 중요한 경우 구조체 사용이 더 명확하다.