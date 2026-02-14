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
