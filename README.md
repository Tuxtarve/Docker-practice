# Docker 실습 기록

## 프로젝트 목표
- Docker 이미지 다운로드 → 컨테이너 실행 → 테스트
- VSCode 환경에서 문제 해결 과정 기록

## 디렉토리 구조
- `src/` : Dockerfile, index.html 등 코드
- `notes/` : 오류/설명/메모 기록
- `README.md` : 전체 실습 개요

# 🐳 Docker 실습: 볼륨 관리 (Volume vs Bind-Mount)

## 1. 실습 개요
도커 컨테이너의 데이터 영속성을 관리하는 두 가지 핵심 방식인 **Volume**과 **Bind-Mount**의 차이점을 실습하고 장단점을 비교한다.

## 2. 실습 시나리오
- **환경**: MacBook M3 Pro (macOS), VS Code, Docker Desktop
- **대상**: Nginx 및 MySQL 컨테이너

## 3. 실습 내용 및 명령어
### [EX 1] 볼륨(Volume) 방식 - Nginx
Docker가 관리하는 가상 저장소 생성 및 연결
```zsh
docker volume create my-vol
docker run -d --name docker1 -p 8081:80 -v my-vol:/usr/share/nginx/html nginx



---

# 🐳 Docker 실습 기록

## 📌 프로젝트 목표

* Docker 사용자 정의 네트워크(User Defined Bridge Network) 생성
* 컨테이너 간 통신 구조 이해
* 기본 bridge 네트워크와 사용자 정의 bridge 네트워크 차이 이해
* 컨테이너 이름 기반 통신(Service Discovery) 확인

---

## 💻 실습 환경

* OS: macOS (MacBook M3 Pro)
* Tool: VSCode, Docker Desktop
* Image: alpine
* Network Driver: bridge

---

## 📂 디렉토리 구조

```
docker-practice/
│
├── src/        # 실습 코드 및 설정
├── notes/      # 오류 및 개념 정리
├── README.md   # 전체 실습 기록
└── commands.sh # 실행 명령어 기록
```

---

# 🚀 실습: 사용자 정의 브리지 네트워크

---

## 1️⃣ 사용자 정의 네트워크 생성

네트워크 생성:

```bash
docker network create alpine-net
```

확인:

```bash
docker network ls
```

출력 예:

```
NETWORK ID     NAME         DRIVER
98ec17774ee0   alpine-net   bridge
f2e215926624   bridge       bridge
```

---

## 2️⃣ 컨테이너 생성 및 네트워크 연결

```bash
docker run --name alpine1 -itd --network alpine-net alpine ash

docker run --name alpine2 -itd --network alpine-net alpine ash

docker run --name alpine3 -itd alpine ash

docker run --name alpine4 -itd --network alpine-net alpine ash
```

bridge 네트워크 추가 연결:

```bash
docker network connect bridge alpine4
```

---

## 3️⃣ 컨테이너 IP 확인

```bash
docker ps -q | xargs -I {} docker inspect -f '{{.Name}}: {{range .NetworkSettings.Networks}}{{.IPAddress}} {{end}}' {}
```

예시 출력:

```
/alpine1: 172.20.0.2
/alpine2: 172.20.0.3
/alpine3: 172.17.0.2
/alpine4: 172.20.0.4 172.17.0.3
```

---

## 📊 네트워크 구성 분석

| 컨테이너    | 연결 네트워크            | IP                     |
| ------- | ------------------ | ---------------------- |
| alpine1 | alpine-net         | 172.20.0.2             |
| alpine2 | alpine-net         | 172.20.0.3             |
| alpine3 | bridge             | 172.17.0.2             |
| alpine4 | alpine-net, bridge | 172.20.0.4, 172.17.0.3 |

---

# 🔍 네트워크 통신 테스트

---

## 4️⃣ alpine1 → alpine-net 컨테이너 통신

접속:

```bash
docker attach alpine1
```

ping 테스트:

```bash
ping -c 2 alpine2

ping -c 2 alpine4
```

결과:

```
정상 통신 성공
```

✔ 이유:

사용자 정의 네트워크는 자동 DNS 지원

즉, 이름으로 통신 가능

---

## 5️⃣ alpine1 → bridge 컨테이너 통신

```bash
ping alpine3
```

결과:

```
ping: bad address 'alpine3'
```

하지만 IP는 가능:

```bash
ping 172.17.0.2
```

✔ 이유:

기본 bridge 네트워크는 DNS 지원 안함

---

## 6️⃣ alpine4 → 모든 컨테이너 통신

alpine4는 두 네트워크 연결됨

가능:

```
ping alpine1
ping alpine2
ping 172.17.0.2
```

---

## 7️⃣ 외부 네트워크 테스트

```bash
ping google.com
```

결과:

```
정상 통신 성공
```

✔ 컨테이너는 외부 인터넷 접근 가능

---

# 🧠 핵심 개념 정리

---

## User Defined Bridge Network

특징:

* 자동 DNS 지원
* 컨테이너 이름으로 통신 가능
* 격리된 네트워크 환경 제공

예:

```
ping alpine2  ← 가능
```

---

## Default Bridge Network

특징:

* DNS 지원 안함
* IP로만 통신 가능

예:

```
ping alpine3 ← 실패
ping 172.17.0.2 ← 성공
```

---

## Service Discovery

Docker가 자동으로 이름 → IP 변환

예:

```
alpine2 → 172.20.0.3
```

---

# 🔥 사용자 정의 네트워크 사용하는 이유

| 기능       | default bridge | user-defined bridge |
| -------- | -------------- | ------------------- |
| 이름 통신    | ❌              | ✅                   |
| DNS      | ❌              | ✅                   |
| 네트워크 격리  | ❌              | ✅                   |
| 운영 환경 사용 | ❌              | ✅                   |

---

# 🧹 정리 작업

컨테이너 중지:

```bash
docker stop alpine1 alpine2 alpine3 alpine4
```

컨테이너 삭제:

```bash
docker rm alpine1 alpine2 alpine3 alpine4
```

네트워크 삭제:

```bash
docker network rm alpine-net
```

---

# 📌 실습 핵심 요약 (시험 + 실무 중요)

| 개념                     | 설명        |
| ---------------------- | --------- |
| docker network create  | 네트워크 생성   |
| docker network connect | 네트워크 연결   |
| user-defined network   | 이름 통신 가능  |
| default bridge         | IP만 통신 가능 |
| docker inspect         | IP 확인     |
| Service Discovery      | 자동 이름 해석  |

---

# 👤 Author

Tuxtarve

---

# ✅ 이제 GitHub 업로드

```bash
git add .

git commit -m "feat: 사용자 정의 bridge 네트워크 및 컨테이너 통신 실습 완료"

git push origin main
```








