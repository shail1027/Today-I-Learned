# Docker란

## Docker란?

Docker는 애플리케이션을 **컨테이너(Container)** 라는 격리된 환경에 패키징하여 실행할 수 있도록 해주는 오픈소스 플랫폼이다. 개발 환경과 운영 환경의 차이를 없애는 것을 목표로 한다.

컨테이너 안에는 애플리케이션 코드, 런타임, 라이브러리, 환경변수, 설정 파일 등 실행에 필요한 모든 것이 포함된다. 따라서 Docker가 설치된 어떤 환경에서도 동일하게 동작한다.

---

## VM vs 컨테이너

Docker를 이해하려면 먼저 기존 가상머신(VM)과의 차이를 알아야 한다.

```
[ VM 구조 ]                        [ 컨테이너 구조 ]

┌──────────┬──────────┐            ┌──────────┬──────────┐
│  App A   │  App B   │            │  App A   │  App B   │
├──────────┼──────────┤            ├──────────┼──────────┤
│  Guest   │  Guest   │            │Container │Container │
│   OS A   │   OS B   │            │ Runtime  │ Runtime  │
├──────────┴──────────┤            ├──────────┴──────────┤
│     Hypervisor      │            │    Docker Engine     │
├─────────────────────┤            ├─────────────────────┤
│      Host OS        │            │      Host OS        │
├─────────────────────┤            ├─────────────────────┤
│     Hardware        │            │     Hardware        │
└─────────────────────┘            └─────────────────────┘
```

| 항목 | VM | 컨테이너 |
|---|---|---|
| 부팅 시간 | 수 분 | 수 초 (밀리초) |
| 용량 | 수 GB (Guest OS 포함) | 수 MB ~ 수백 MB |
| 격리 수준 | 완전 격리 (커널 분리) | 프로세스 수준 격리 (커널 공유) |
| 성능 | 오버헤드 큼 | 호스트와 거의 동일 |
| 이식성 | 상대적으로 낮음 | 매우 높음 |

VM은 Guest OS까지 통째로 띄우는 반면, 컨테이너는 Host OS의 커널을 공유하되 **네임스페이스(Namespace)** 와 **cgroups** 로 프로세스를 격리한다. 가볍고 빠르지만, VM보다 격리 수준은 낮다.

---

## 핵심 개념

### 이미지 (Image)

컨테이너를 만들기 위한 **읽기 전용 템플릿**이다. 클래스와 인스턴스의 관계처럼, 이미지에서 컨테이너가 만들어진다.

이미지는 **레이어(Layer)** 구조로 이루어져 있다. 각 레이어는 파일 시스템의 변경사항이며, 레이어들이 쌓여 하나의 이미지를 형성한다. 공통 레이어는 여러 이미지가 공유하므로 디스크 공간을 절약할 수 있다.

```
ubuntu:22.04 이미지
    ↓ (python 설치 레이어)
python:3.11-slim
    ↓ (requirements.txt 복사 레이어)
    ↓ (pip install 레이어)
    ↓ (소스코드 복사 레이어)
my-app:latest  ← 최종 이미지
```

### 컨테이너 (Container)

이미지를 실행한 **인스턴스**다. 이미지 위에 읽기-쓰기 레이어가 하나 추가되어 실행 중 변경사항을 저장하지만, 컨테이너가 삭제되면 해당 레이어도 사라진다. (영속적인 데이터는 볼륨으로 관리해야 한다.)

### 레지스트리 (Registry)

이미지를 저장하고 배포하는 저장소다. **Docker Hub**가 대표적인 퍼블릭 레지스트리이며, AWS ECR, GitHub Container Registry 같은 프라이빗 레지스트리도 있다.

```bash
docker pull python:3.11-slim        # Docker Hub에서 이미지 다운로드
docker push myusername/my-app:1.0   # 레지스트리에 이미지 업로드
```

---

## Dockerfile

이미지를 만들기 위한 **설계도(빌드 스크립트)** 다. 어떤 베이스 이미지에서 시작해서 어떤 파일을 넣고, 어떤 명령을 실행할지를 순서대로 기술한다.

```dockerfile
# 베이스 이미지 지정 (레이어 1)
FROM python:3.11-slim

# 작업 디렉터리 설정 (이후 명령의 기준 경로)
WORKDIR /app

# 의존성 파일만 먼저 복사 (캐시 최적화 핵심!)
COPY requirements.txt .

# 패키지 설치 (레이어 2)
RUN pip install --no-cache-dir -r requirements.txt

# 소스코드 복사 (레이어 3 — 자주 바뀌므로 마지막에)
COPY . .

# 컨테이너 실행 시 열어줄 포트 (문서 목적, 실제 포트 개방은 run 명령에서)
EXPOSE 8000

# 컨테이너가 시작될 때 실행할 기본 명령
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 주요 명령어

| 명령어 | 설명 |
|---|---|
| `FROM` | 베이스 이미지 지정. 모든 Dockerfile의 시작 |
| `WORKDIR` | 작업 디렉터리 설정. 없으면 자동 생성 |
| `COPY` | 호스트 파일을 이미지로 복사 |
| `ADD` | COPY + URL 다운로드 + tar 자동 압축 해제 (대부분 COPY 권장) |
| `RUN` | 이미지 빌드 시점에 명령 실행 (새 레이어 생성) |
| `CMD` | 컨테이너 기본 실행 명령 (덮어쓰기 가능) |
| `ENTRYPOINT` | 컨테이너 고정 실행 명령 (덮어쓰기 어려움) |
| `ENV` | 환경변수 설정 |
| `ARG` | 빌드 시점에만 사용하는 변수 (`--build-arg`로 전달) |
| `EXPOSE` | 컨테이너가 사용할 포트 명시 (문서 역할) |
| `VOLUME` | 마운트 포인트 지정 |

### CMD vs ENTRYPOINT

```dockerfile
# CMD만 사용: docker run image echo "hello" 하면 CMD가 통째로 교체됨
CMD ["python", "app.py"]

# ENTRYPOINT + CMD 조합: CMD는 ENTRYPOINT의 기본 인자로 동작
ENTRYPOINT ["python"]
CMD ["app.py"]
# docker run image train.py  →  python train.py  (CMD만 교체됨)
```

### 레이어 캐시 최적화

Dockerfile의 각 명령은 레이어를 생성하고, 레이어가 변경되면 이후 레이어는 모두 캐시가 무효화된다. 따라서 **자주 바뀌는 파일일수록 뒤에 배치**하는 것이 빌드 속도의 핵심이다.

```dockerfile
# 나쁜 예: 소스코드 한 줄 바꿔도 pip install이 매번 새로 실행됨
COPY . .
RUN pip install -r requirements.txt

# 좋은 예: requirements.txt가 바뀌지 않으면 pip install 레이어 캐시 재사용
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

### 멀티 스테이지 빌드

빌드 환경과 실행 환경을 분리해 **최종 이미지 크기를 대폭 줄이는** 패턴이다.

```dockerfile
# 스테이지 1: 빌드 환경 (컴파일러, 빌드 도구 포함)
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build           # dist/ 생성

# 스테이지 2: 실행 환경 (빌드 결과물만 복사)
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
# node_modules, 소스코드 등은 포함되지 않음 → 이미지 수백 MB 절감
```

---

## 주요 CLI 명령어

### 이미지 관련

```bash
# 이미지 빌드 (-t: 태그, .: Dockerfile 위치)
docker build -t my-app:1.0 .
docker build -t my-app:latest --no-cache .   # 캐시 무시하고 빌드

# 이미지 목록 확인
docker images
docker image ls

# 이미지 삭제
docker rmi my-app:1.0
docker image prune       # 태그 없는(dangling) 이미지 전체 삭제

# 레지스트리에서 받기/올리기
docker pull nginx:alpine
docker push myrepo/my-app:1.0
```

### 컨테이너 실행

```bash
# 기본 실행
docker run nginx

# 자주 쓰는 옵션 조합
docker run \
  -d \                            # 백그라운드(detached) 실행
  --name my-nginx \               # 컨테이너 이름 지정
  -p 8080:80 \                    # 호스트:컨테이너 포트 매핑
  -v $(pwd)/html:/usr/share/nginx/html \   # 볼륨 마운트
  -e APP_ENV=production \         # 환경변수 주입
  --restart unless-stopped \      # 자동 재시작 정책
  nginx:alpine

# 컨테이너 내부 셸 접속
docker exec -it my-nginx sh       # 실행 중인 컨테이너에 접속
docker run -it ubuntu bash        # 새 컨테이너로 바로 접속
```

### 컨테이너 관리

```bash
# 상태 확인
docker ps                  # 실행 중인 컨테이너
docker ps -a               # 중지된 것 포함 전체

# 시작 / 중지 / 재시작
docker start my-nginx
docker stop my-nginx       # SIGTERM → 10초 후 SIGKILL (graceful)
docker kill my-nginx       # 즉시 SIGKILL
docker restart my-nginx

# 삭제
docker rm my-nginx
docker rm -f my-nginx      # 실행 중인 컨테이너 강제 삭제

# 로그 확인
docker logs my-nginx
docker logs -f my-nginx    # 실시간 스트림 (tail -f 같은 효과)
docker logs --tail 100 my-nginx

# 리소스 사용량 실시간 모니터링
docker stats

# 컨테이너 내부 파일 확인
docker inspect my-nginx    # 컨테이너 상세 정보 (JSON)
```

### 청소

```bash
# 사용하지 않는 리소스 일괄 삭제
docker system prune        # 중지된 컨테이너, 미사용 이미지/네트워크 삭제
docker system prune -a     # 위 + 실행 중이 아닌 이미지까지 전부
```

---

## 볼륨 (Volume)

컨테이너는 삭제되면 내부 데이터도 사라진다. **DB 데이터, 업로드 파일** 같이 컨테이너 수명과 무관하게 유지해야 할 데이터는 볼륨을 사용한다.

### 볼륨 종류

```
1. Named Volume     — Docker가 관리하는 볼륨 (권장)
2. Bind Mount       — 호스트 디렉터리를 직접 마운트
3. tmpfs Mount      — 메모리에만 저장 (컨테이너 종료 시 삭제)
```

```bash
# Named Volume
docker volume create db-data
docker run -v db-data:/var/lib/postgresql/data postgres

# Bind Mount (개발 시 소스 코드 실시간 반영)
docker run -v $(pwd):/app my-app

# tmpfs (민감 정보를 디스크에 남기지 않을 때)
docker run --tmpfs /tmp my-app
```

```bash
# 볼륨 관리
docker volume ls
docker volume inspect db-data
docker volume rm db-data
docker volume prune          # 미사용 볼륨 전체 삭제
```

---

## 네트워크 (Network)

컨테이너들이 서로 통신하거나 외부와 연결되는 방식을 제어한다.

### 네트워크 드라이버

| 드라이버 | 설명 | 사용 시점 |
|---|---|---|
| `bridge` | 기본값. 호스트와 격리된 가상 네트워크 | 단일 호스트에서 컨테이너 간 통신 |
| `host` | 호스트 네트워크 직접 사용 | 성능 최우선, 포트 격리 불필요 시 |
| `none` | 네트워크 완전 비활성화 | 완전 격리 필요 시 |
| `overlay` | 여러 호스트에 걸친 가상 네트워크 | Docker Swarm, Kubernetes |

```bash
# 사용자 정의 브리지 네트워크 생성 (컨테이너 이름으로 DNS 통신 가능)
docker network create my-network

docker run -d --name api --network my-network my-api
docker run -d --name db  --network my-network postgres

# api 컨테이너 안에서 'db'라는 호스트명으로 PostgreSQL에 접근 가능
# postgres://db:5432/mydb
```

> 기본 브리지 네트워크는 컨테이너 이름으로 DNS 통신이 **불가**하다. 반드시 사용자 정의 네트워크를 만들어야 컨테이너 이름으로 접근할 수 있다.

---

## Docker Compose

여러 컨테이너를 **하나의 yaml 파일로 정의하고 한 번에 관리**하는 도구다. 웹 서버 + 앱 서버 + DB처럼 여러 서비스가 함께 동작하는 구성을 쉽게 다룰 수 있다.

### docker-compose.yml 예시

```yaml
version: "3.9"

services:
  # FastAPI 애플리케이션
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: fastapi-app
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - SECRET_KEY=supersecretkey
    volumes:
      - .:/app                  # 개발 시 소스코드 실시간 반영
    depends_on:
      db:
        condition: service_healthy   # DB가 healthy 상태가 될 때까지 대기
    restart: unless-stopped
    networks:
      - app-network

  # PostgreSQL 데이터베이스
  db:
    image: postgres:15-alpine
    container_name: postgres-db
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data   # 데이터 영속성
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # Redis 캐시
  redis:
    image: redis:7-alpine
    container_name: redis-cache
    ports:
      - "6379:6379"
    networks:
      - app-network

  # Nginx 리버스 프록시
  nginx:
    image: nginx:alpine
    container_name: nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certbot/conf:/etc/letsencrypt
    depends_on:
      - api
    networks:
      - app-network

volumes:
  db-data:           # 네임드 볼륨 선언

networks:
  app-network:
    driver: bridge
```

### Compose 주요 명령어

```bash
# 전체 서비스 빌드 + 시작 (백그라운드)
docker compose up -d

# 빌드 강제 (코드 변경 시)
docker compose up -d --build

# 특정 서비스만 재시작
docker compose restart api

# 로그 확인
docker compose logs -f api

# 실행 중인 서비스 상태
docker compose ps

# 서비스 중지 (컨테이너만 제거, 볼륨은 유지)
docker compose down

# 볼륨까지 전부 삭제
docker compose down -v

# 특정 서비스 컨테이너 안으로 접속
docker compose exec api sh

# 서비스 스케일 조정
docker compose up -d --scale api=3   # api 컨테이너를 3개로 늘림
```

---

## .dockerignore

`.gitignore`처럼 빌드 컨텍스트에서 제외할 파일을 지정한다. 불필요한 파일이 이미지에 포함되거나 빌드 속도가 느려지는 것을 방지한다.

```
# .dockerignore
.git
.gitignore
.env
.env.*
__pycache__
*.pyc
*.pyo
.pytest_cache
.coverage
node_modules
*.log
*.md
README.md
docker-compose*.yml
.DS_Store
```

---

## 환경변수와 시크릿 관리

```yaml
# docker-compose.yml에서 env 파일 참조
services:
  api:
    env_file:
      - .env          # 파일 통째로 주입
    environment:
      - DEBUG=false   # 개별 변수 오버라이드
```

```bash
# .env 파일 (Compose가 자동으로 읽음)
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
SECRET_KEY=dev-secret-key-change-in-prod
```

> `.env` 파일은 절대 git에 커밋하지 않는다. `.env.example`을 만들어 필요한 키 목록만 공유하는 것이 관례다.

---

## 실전 패턴

### 개발 vs 프로덕션 분리

```
docker-compose.yml          # 공통 기반
docker-compose.dev.yml      # 개발 전용 (볼륨 마운트, --reload 등)
docker-compose.prod.yml     # 프로덕션 전용 (리소스 제한, 로깅 등)
```

```bash
# 개발 환경 실행
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d

# 프로덕션 환경 실행
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

### 헬스체크 (Healthcheck)

```dockerfile
# Dockerfile에 직접 정의
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```

```yaml
# Compose에서 정의
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s    # 초기 기동 시간 여유
```

### 리소스 제한

```yaml
services:
  api:
    deploy:
      resources:
        limits:
          cpus: "0.5"       # CPU 최대 50% 사용
          memory: 512M      # 메모리 최대 512MB
        reservations:
          cpus: "0.25"
          memory: 256M
```
