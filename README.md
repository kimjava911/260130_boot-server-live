1. 로컬에서 이미지 빌드하고 docker compose -> 로드 밸런싱
2. 로컬 이미지 -> ghcr 이미지로 푸시 (태그 변경 -> PAT login -> 푸시)
3. 원격 클라우드 -> docker 설치 -> ghcr 로그인 -> 이미지를 pull
4. 설정파일, .env -> git에서 pull할 수 있는 건 pull하고, 아닌 건 작성
5. 구동 후 접속(해당 ip:8080)으로 진행

### 이미지 빌드 및 테스트
```sh
# 이미 Dockerfile 파일이 있음
docker build -t boot-server:latest .
# arm64 빌드를 하고, ec2 -> amd64. -> 멀티 플랫폼을 위한 빌드
# macOS만 아래로 빌드...
docker buildx build --platform linux/arm64,linux/amd64 -t boot-server:latest .
```

### 이미지 레지스트리 푸시
```sh
docker images
# boot-server:latest
docker tag boot-server:latest ghcr.io/<github username>/boot-server:latest
# docker tag boot-server:latest ghcr.io/kimjava911/boot-server:latest
# https://github.com/settings/tokens
export CR_PAT=<token을 넣어주세요>
echo $CR_PAT | docker login ghcr.io -u <github username> --password-stdin
# echo $CR_PAT | docker login ghcr.io -u kimjava911 --password-stdin
docker push ghcr.io/<github username>/boot-server:latest
# docker push ghcr.io/kimjava911/boot-server:latest

# https://github.com/<username>
```

### 원격 SSH 연결
```sh
# pem은 .gitignore에 포함
chmod 400 *.pem
# ls -l *.pem
ssh -i "<지정한 키 이름>.pem" ubuntu@ec2-<여러분들의 주소>.compute.amazonaws.com
# ssh -i "my-key-260130.pem" ubuntu@ec2-43-200-1-0.ap-northeast-2.compute.amazonaws.com
```

### 도커 설치 및 권한 부여
```sh
# 패키지 매니저 업데이트
sudo apt update

# Docker 및 Docker Compose 설치
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 현재 사용자를 docker 그룹에 추가 (sudo 없이 사용 가능하도록)
sudo usermod -aG docker $USER
# 적용을 위해 로그아웃 후 재접속
exit
# 바로 위 방향키 눌러서 재로그인 시도하면 된다
```

```sh
docker -v
docker compose version
```

```sh
# docker pull ghcr.io/kimjava911/boot-server:latest
# unauthorized
export CR_PAT=<token을 넣어주세요>
echo $CR_PAT | docker login ghcr.io -u <github username> --password-stdin
# echo $CR_PAT | docker login ghcr.io -u kimjava911 --password-stdin
docker pull ghcr.io/<username>/boot-server:latest
# docker pull ghcr.io/kimjava911/boot-server:latest
docker tag ghcr.io/<username>/boot-server:latest boot-server:latest
# docker tag ghcr.io/kimjava911/boot-server:latest boot-server:latest
docker images
```