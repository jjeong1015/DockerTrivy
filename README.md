# DockerTrivy

## 🛠️ 목표
Docker를 통해 Spring Boot 애플리케이션을 컨테이너로 배포하고, Trivy를 사용해 보안 스캔을 자동화한 워크플로우를 제공한다. 또한 GitHub Actions를 이용해 CI/CD 파이프라인을 구축하여, 코드를 커밋하는 순간 자동으로 보안 스캔까지 진행하는 환경을 구성한다.
## 🐋 Docker 설치
```bash
# 1. apt 인덱스 업데이트
$ sudo apt-get update
$ sudo apt-get install ca-certificates curl gnupg lsb-release

# 2. Docker 공식 GPG 키 추가
$ sudo mkdir -m 0755 -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 3. Docker 저장소를 APT 소스에 추가
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 4. APT 패키지 캐시 업데이트
$ sudo apt-get update

# 5. Docker 서비스 상태 확인
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 6. 사용자 권한 설정
$ sudo usermod -aG docker $USER # docker 명령어 사용시 sudo 권한 부여하는 설정(재부팅 필수)
$ newgrp docker    # 설정한 그룹 즉각 인식하는 명령어, 생략시 재부팅 후에만 group 적용
$ groups
$ tail /etc/group

# 7. 설치 확인
$ docker --version
```
## 🚀 Docker에 Spring Boot 애플리케이션 배포
1. 프로젝트 디렉토리 생성 및 이동
```bash
$ mkdir dockerTrivy

$ cd dockerTrivy
```
```bash
$ cd scanDockerImage

$ touch Dockerfile
```
2. Dockerfile 작성
```bash
# Dockerfile

# Step 1: Build stage
FROM openjdk:17-jdk-alpine AS build

# 작업 디렉토리 생성
WORKDIR /usr/src/myapp

# Gradle Wrapper와 관련 파일 복사
COPY gradlew /usr/src/myapp/
COPY gradle /usr/src/myapp/gradle/

# 소스 코드 복사 (src 디렉토리 및 필요한 다른 파일 복사)
COPY . /usr/src/myapp

# Gradle Wrapper에 실행 권한 부여
RUN chmod +x ./gradlew

# Gradle 빌드 실행
RUN ./gradlew clean build --no-daemon

# Step 2: Run stage
FROM openjdk:17-jdk-alpine

# 작업 디렉토리 생성
WORKDIR /usr/src/myapp

# 빌드된 JAR 파일을 복사 (ScanDockerImageApplication.jar로 이름 지정)
COPY --from=build /usr/src/myapp/build/libs/*.jar /usr/src/myapp/ScanDockerImageApplication.jar

# 포트 노출 (Spring Boot 기본 포트)
EXPOSE 8080

# JAR 파일 실행 명령어
CMD ["java", "-jar", "/usr/src/myapp/ScanDockerImageApplication.jar"]
```
3. Docker 이미지 빌드 및 실행
```bash
$ docker build -t scan-docker-image .
```
```bash
$ docker run -d -p 8080:8080 scan-docker-image
```
## 🔍 Trivy로 보안 스캔하기
```bash
# Trivy 이미지 다운로드
$ docker pull aquasec/trivy

# 이미지 보안 스캔 실행
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image scan-docker-image
```
## 🛠️ GitHub Actions 연동하여 CI/CD 자동화
```bash
# .github/workflows/trivy-scan.yml
name: Security Scan

on:
  push:
    branches:
      - main
  pull_request:

jobs:
  trivy-scan:
    runs-on: ubuntu-latest

    steps:
      # 소스 코드 체크아웃
      - name: Checkout code
        uses: actions/checkout@v2

      # Docker 이미지 빌드
      - name: Build Docker image
        run: docker build -t ScanDockerImageApplication .

      # Trivy 설치
      - name: Install Trivy
        run: |
          sudo apt-get update && sudo apt-get install -y wget
          wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_Linux-64bit.deb
          sudo dpkg -i trivy_Linux-64bit.deb

      # Docker 이미지 보안 스캔 실행
      - name: Scan Docker image for vulnerabilities
        run: trivy image ScanDockerImageApplication
```
## 🌐 GitHub 리포지토리 연동
```bash
# Git 설치
$ sudo apt update
$ sudo apt install git

# 사용자 정보 설정
$ git config --global user.name "사용자 이름"
$ git config --global user.email "사용자 이메일"

# 리포지토리 초기화 및 원격 저장소 연동
$ git init
$ git remote add origin https://github.com/jjeong1015/DockerTrivy.git

# 커밋 및 푸시
$ git add .
$ git commit -m "feat: commit message"
$ git branch -M main
$ git push -u origin main
```
