## React Swiper Library


### 개요

**Swiper**는 현대적인 웹 모바일 터치 슬라이더를 구현하기 위한 가장 대중적인 자바스크립트 라이브러리이다. React 환경에서는 공식적으로 제공되는 Swiper React 컴포넌트를 사용하여 하드웨어 가속 기반의 부드러운 슬라이드, 터치 드래그, 루프 재생 등을 쉽게 구현할 수 있다.

---

### 설치 방법

먼저 프로젝트 터미널에서 `swiper` 패키지를 설치해야 한다.

```bash
npm install swiper
```

---

### 기본 사용법 및 구조

React에서 Swiper를 사용할 때는 메인 컨테이너인 `<Swiper>`와 각각의 슬라이드 아이템인 `<SwiperSlide>`를 사용한다.

```jsx
import { Swiper, SwiperSlide } from 'swiper/react';

// Swiper 기본 스타일 가져오기
import 'swiper/css';

export default () => {
  return (
    <Swiper
      spaceBetween={50} // 슬라이드 사이의 간격
      slidesPerView={3} // 한 번에 보여줄 슬라이드 개수
      onSlideChange={() => console.log('slide change')}
      onSwiper={(swiper) => console.log(swiper)}
    >
      <SwiperSlide>Slide 1</SwiperSlide>
      <SwiperSlide>Slide 2</SwiperSlide>
      <SwiperSlide>Slide 3</SwiperSlide>
      <SwiperSlide>Slide 4</SwiperSlide>
    </Swiper>
  );
};
```

---

### 모듈 추가 (Navigation, Pagination, Autoplay)

기본 슬라이더 외에 화살표(Navigation)나 하단 점(Pagination), 자동 재생 기능 등을 추가하려면 별도의 모듈을 불러와서 설정해야 한다.



#### 모듈 적용 예시

```jsx
import { Swiper, SwiperSlide } from 'swiper/react';
import { Navigation, Pagination, Autoplay } from 'swiper/modules';

// 필요한 스타일 가져오기
import 'swiper/css';
import 'swiper/css/navigation';
import 'swiper/css/pagination';

const MySlider = () => {
  return (
    <Swiper
      modules={[Navigation, Pagination, Autoplay]}
      spaceBetween={30}
      slidesPerView={1}
      navigation // 좌우 화살표 활성화
      pagination={{ clickable: true }} // 하단 페이지네이션 활성화 (클릭 가능)
      autoplay={{ delay: 3000 }} // 3초마다 자동 재생
      loop={true} // 무한 루프 모드
    >
      <SwiperSlide><img src="img1.jpg" alt="Slide 1" /></SwiperSlide>
      <SwiperSlide><img src="img2.jpg" alt="Slide 2" /></SwiperSlide>
      <SwiperSlide><img src="img3.jpg" alt="Slide 3" /></SwiperSlide>
    </Swiper>
  );
};
```

#### 1. 레이아웃 및 표시 관련 속성

슬라이더가 화면에 어떻게 보여질지를 결정하는 가장 기본적인 속성들이다.

* **`slidesPerView`**: 한 번에 보여줄 슬라이드의 개수를 설정한다. `'auto'`로 설정하면 슬라이드의 CSS 너비에 따라 자동으로 조절된다.
* **`slidesPerGroup`**: 한 번에 넘길 슬라이드의 개수를 설정한다. (예: 3으로 설정 시 화살표 클릭 시 3장씩 넘어간다.)
* **`spaceBetween`**: 슬라이드 사이의 간격(px 단위)을 설정한다.
* **`centeredSlides`**: 활성화된 슬라이드를 항상 가운데에 배치한다. 홀수 개의 `slidesPerView`와 함께 쓸 때 유용하다.



---

#### 2. 루프 및 자동 재생 (Loop & Autoplay)

* **`loop`**: 마지막 슬라이드에서 다음으로 넘기면 첫 번째 슬라이드가 나오도록 무한 루프를 활성화한다.
* **`loopedSlides`**: 루프 모드에서 복제될 슬라이드의 개수를 지정한다. (슬라이드 양이 적을 때 끊김 현상을 방지한다.)
* **`autoplay`**: 객체 형태로 상세 설정이 가능하다.
    * `delay`: 다음 슬라이드로 넘어가는 시간 (ms 단위, 3000 = 3초).
    * `disableOnInteraction`: 사용자가 슬라이드를 건드리면 자동 재생을 멈출지 여부 (기본값 `true`).
    * `pauseOnMouseEnter`: 마우스를 올렸을 때 일시 정지할지 여부.

---

#### 3. 반응형 설정 (Breakpoints)

화면 크기에 따라 속성값을 다르게 적용할 때 사용한다. C++의 조건문처럼 픽셀 단위로 분기 처리가 가능하다.

```jsx
<Swiper
  breakpoints={{
    // 320px 이상일 때 (모바일)
    320: {
      slidesPerView: 1,
      spaceBetween: 10
    },
    // 768px 이상일 때 (태블릿)
    768: {
      slidesPerView: 2,
      spaceBetween: 20
    },
    // 1024px 이상일 때 (데스크탑)
    1024: {
      slidesPerView: 4,
      spaceBetween: 30
    }
  }}
>
```

#### 4. 이벤트 및 컨트롤 (Events)

슬라이드의 상태 변화를 감지하여 특정 로직을 실행할 때 사용한다.

* **`onSlideChange`**: 슬라이드가 바뀔 때마다 실행되는 콜백 함수이다. 현재 인덱스를 파악할 때 유용하다.
* **`onSwiper`**: Swiper 인스턴스가 초기화될 때 실행된다. 이 인스턴스를 상태(state)에 저장하면 외부 버튼으로 슬라이드를 조작할 수 있다.
* **`grabCursor`**: 사용자가 슬라이드 위에 마우스를 올렸을 때 커서를 '손바닥' 모양으로 바꾸어 드래그 가능함을 알려준다.

---

#### 5. 전환 효과 (Effects)

기본적인 가로 슬라이드 외에 다양한 시각 효과를 줄 수 있다. (해당 효과 모듈을 추가로 `import` 해야 한다.)

* **`effect`**: `'fade'`, `'cube'`, `'coverflow'`, `'flip'`, `'cards'` 중 선택 가능하다.
* **`speed`**: 슬라이드가 전환되는 속도(ms)를 조절한다. (기본값 300).


