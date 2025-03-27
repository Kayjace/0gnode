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

## 3. Storage 노드 설정 및 실행


### 환경 변수 설정

`.env` 파일을 편집기로 열어 Storage 노드 관련 설정을 수정합니다:

```bash
nano .env
```

다음 중요 변수들을 적절히 설정합니다:

- **STORAGE_MINER_PRIVATE_KEY**: 테스트넷 토큰이 있는 지갑의 개인키(프라이빗키) (0x 접두사 제외)
  - 메타마스크에서 노드용 버너 지갑을 생성하고 개인키를 내보낸 후 맨 앞의 0x 제거
  - 또는 `0gchaind keys unsafe-export-eth-key $WALLET_NAME` 명령어로 기존 검증자 계정의 개인키 사용
- **STORAGE_ENR_ADDRESS**: 서버의 공개 IP 주소
- **STORAGE_BLOCKCHAIN_RPC_ENDPOINT**: 이 Docker Compose 파일로 노드/검증자를 실행 중이면 `http://node:8545`를 사용하거나, 그렇지 않으면 공개 RPC 엔드포인트 사용
- **STORAGE_NETWORK_ENR_TCP_PORT**와 **STORAGE_NETWORK_ENR_UDP_PORT**: 사용 가능한 포트로 설정 (두 값이 동일해야 함, 예: 31234)

### 지갑 자금 충전

Storage 노드 지갑에는 a0gi가 필요합니다. Faucet에서 자금을 받을 수 있습니다.

0gchaind 바이너리로 지갑을 생성한 경우, 다음 명령어로 EVM 주소를 확인할 수 있습니다 (메타마스크 등에 지갑 주소 갖고있으면 필요 없는 절차):

```bash
echo "0x$(0gchaind debug addr $(0gchaind keys show $WALLET_NAME -a) | grep hex | awk '{print $3}')"
```

### Storage 노드 실행

```bash
docker compose --profile storage up -d storage-node
```

### 로그 확인

```bash
docker compose --profile storage logs -f --since 1m storage-node
```

### 이후 필요시 노드 업데이트 방법

```bash
# 저장소 업데이트 및 이미지 가져오기
git pull
docker compose --profile storage pull

# storage 컨테이너 제거
docker compose --profile storage down

# storage 노드 설정 볼륨 제거
docker volume rm 0g-docker_storage_config

# Storage 노드 다시 실행
docker compose --profile storage up -d storage-node
```
