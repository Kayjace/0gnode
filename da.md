# 0G DA 노드 설치 및 실행 가이드

이 가이드는 0G Labs의 ZeroGravity 네트워크에서 DA(Data Availability) 노드를 설치하고 실행하는 방법을 설명합니다.

### 요구사항

linux 또는 wsl 환경

0G Labs에서 노드를 실행하기 위한 최소 하드웨어 요구사항:

- **운영 체제**: Ubuntu 20.04 이상
- **프로세서**: 4코어
- **메모리**: 16GB RAM
- **저장 공간**: 500GB SSD
- **네트워크**: 안정적인 연결과 최소 지연 시간

<br>

## 1. 기본 설정 (필요한 종속성 및 docker를 설치하신 분은 패스)

### 필요한 종속성 설치

서버에 SSH로 접속한 후 다음 명령어를 실행하여 필요한 패키지를 설치합니다:

```bash
sudo apt update
sudo apt install build-essential git curl jq -y
```

### Docker 및 Docker Compose 설치

0G Labs는 Docker를 사용하여 노드를 실행합니다:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo apt install docker-compose -y
```
<br>
<br>

## 2. EmberStake 저장소 클론하기 (마찬가지로 이미 하신분은 패스)

다음 명령어로 최신 버전의 저장소를 클론합니다:

```bash
cd ~
git clone https://github.com/EmberStake/0g-docker.git
cd 0g-docker
git checkout $(git tag -l | tail -1)
```

### 환경 설정 파일 준비

샘플 환경 파일을 복사합니다:

```bash
cp .env.sample .env
```

### Docker 이미지 가져오기

```bash
docker compose --profile '*' pull
```
<br>
<br>

## 4. DA 노드 설정 및 실행

### 전제 조건

- 최소 10 a0gi를 스테이킹할 수 있는 지갑이 필요합니다.

### BLS 키 생성

DA 노드를 실행하려면 BLS 개인 키가 필요합니다. 다음 명령어로 생성하고 안전한 곳에 백업해두세요:

```bash
docker compose --profile da-node run --rm --entrypoint /usr/local/bin/key-gen da-node
```

### 환경 변수 설정

`.env` 파일을 편집기로 열어 DA 노드 관련 설정을 수정합니다:

```bash
nano .env
```

다음 중요 변수들을 적절히 설정합니다:

- **DA_NODE_ETH_RPC_ENDPOINT**: 이 Docker Compose 파일로 노드/검증자를 실행 중이면 `http://node:8545`를 사용하거나, 그렇지 않으면 공개 RPC 엔드포인트 사용 (예: `https://testnet-v2-0g-api-rpc.emberstake.xyz`)
- **DA_NODE_SOCKET_ADDR**: 서버의 공개 IP 주소 또는 그에 연결된 DNS 레코드
- **DA_NODE_SOCKET_PORT**: 사용 가능한 포트 (예: 34180)
- **DA_NODE_SIGNER_BLS_PRIVATE_KEY**: 이전 단계에서 생성한 BLS 개인 키
- **DA_NODE_SIGNER_ETH_PRIVATE_KEY**: 최소 10 a0gi를 스테이킹한 계정의 개인 키 (0x 접두사 제외)
  - 메타마스크에서 새 지갑을 생성하고 개인키를 내보낸 후 맨 앞의 0x 제거
  - 또는 `0gchaind keys unsafe-export-eth-key $WALLET_NAME` 명령어로 기존 검증자 계정의 개인키 사용

### DA 인코더 환경 변수 설정 (선택사항)

DA 인코더를 실행하려면 다음 변수를 설정합니다:

- **DA_ENCODER_LISTEN_PORT**: 사용 가능한 포트 (예: 34190)

### DA 노드 실행

```bash
docker compose --profile da-node up -d da-node
```

### DA 인코더 실행 (선택사항)

```bash
docker compose --profile da-encoder up -d da-encoder
```

### 로그 확인

```bash
# DA 노드 로그
docker compose logs -f --since 1m da-node

# DA 인코더 로그 (선택사항)
docker compose --profile da-encoder logs -f --since 1m da-encoder
```

## 문제 해결 및 유지 관리

- 노드 상태가 정상인지 주기적으로 로그를 확인하세요.
- 문제가 발생하면 컨테이너를 재시작하거나 설정 파일을 확인하세요.


### 이후 노드 업데이트 방법

```bash
# 저장소 업데이트 및 이미지 가져오기
git pull
docker compose --profile da-node pull

# da노드 컨테이너 제거
docker compose --profile da-node down

# da 노드 설정 볼륨 제거
docker volume rm 0g-docker_da-node_config

# Storage 노드 다시 실행
docker compose --profile da-node up -d da-node
```
