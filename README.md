# 🤖 Isaac Sim 설치 및 실행 가이드

NVIDIA Isaac Sim 5.1.0 설치부터 기본 실행까지의 가이드입니다.

---

## 📋 목차

- [사전 설치](#-사전-설치)
- [시스템 요구사항 확인](#-시스템-요구사항-확인)
- [Isaac Sim 설치](#-isaac-sim-설치)
  - [Local 설치](#local-설치)
  - [Docker 설치](#docker-설치)

---

## 🔧 사전 설치

### Docker 설치

> Docker가 이미 설치되어 있다면 이 단계를 건너뛰세요.

- 공식 문서: [Docker Engine Install (Ubuntu)](https://docs.docker.com/engine/install/ubuntu)
- 설치 후 설정: [Linux Post-install](https://docs.docker.com/engine/install/linux-postinstall)

```bash
# Docker 편의 스크립트를 이용한 설치
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Post-install 설정 (sudo 없이 docker 실행)
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

# 설치 확인
docker run hello-world
```

### ROS2 설치

Ubuntu 버전에 맞는 ROS2를 설치해주세요.

| Ubuntu 버전 | ROS2 버전 | 설치 링크 |
|---|---|---|
| 22.04 | Humble | [설치 가이드](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html) |
| 24.04 | Jazzy | [설치 가이드](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html) |

---

## ✅ 시스템 요구사항 확인

Isaac Sim 설치 전, 아래 명령어로 시스템 호환성을 먼저 확인해주세요.

```bash
xhost +local:
docker run --entrypoint bash -it --gpus all --rm --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -v $HOME/.Xauthority:/isaac-sim/.Xauthority \
    -e DISPLAY \
    nvcr.io/nvidia/isaac-sim:5.1.0 ./isaac-sim.compatibility_check.sh
```

> ⚠️ Docker가 설치되어 있지 않을 경우 사전 설치 단계를 먼저 완료해주세요.

---

## 🚀 Isaac Sim 설치

### Local 설치

**1. 패키지 다운로드**

[공식 다운로드 페이지](https://docs.isaacsim.omniverse.nvidia.com/5.1.0/installation/download.html)에서 버전을 선택하여 다운로드합니다.

**2. 압축 해제**

```bash
mkdir ~/isaacsim
cd ~/Downloads
unzip "isaac-sim-standalone-5.1.0-linux-x86_64.zip" -d ~/isaacsim
```

**3. 설치 후 스크립트 실행**

```bash
cd ~/isaacsim
./post_install.sh

# post_install.sh로 설치가 안 될 경우 아래 중 하나를 실행하세요
./isaac-sim.sh -i
./isaac-sim.selector.sh -i
```

**4. Isaac Sim 실행**

```bash
# 아래 둘 중 하나를 선택하여 실행
./isaac-sim.sh
./isaac-sim.selector.sh
```

**`isaac-sim.selector.sh` 실행 시 ROS2 Bridge 설정**

앱 셀렉터 실행 후 아래 설정을 확인하세요.

- **ROS Bridge Extension**: `isaacsim.ros2.bridge`
- **Use Internal ROS2 Libraries**: 설치된 ROS2 버전 선택

---

### Docker 설치

> ⚠️ 현재 Docker를 이용한 설치 방법은 작업 중입니다.

**1. NVIDIA Container Toolkit 설치**

```bash
# 저장소 설정
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
    && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list \
    && sudo apt-get update

# NVIDIA Container Toolkit 설치
sudo apt-get install -y nvidia-container-toolkit

# Container Runtime 설정
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
sudo reboot
```

**2. NVIDIA 드라이버 및 설치 확인**

```bash
# NVIDIA GPU 드라이버 확인
nvidia-smi

# NVIDIA Container Toolkit 동작 확인
docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi
```

**3. Isaac Sim 이미지 Pull**

[NGC 컨테이너 페이지](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/isaac-sim)에서 확인 후 Pull합니다.

```bash
docker pull nvcr.io/nvidia/isaac-sim:5.1.0
```

**4. 캐시 볼륨 디렉토리 생성**

```bash
mkdir -p ~/docker/isaac-sim/cache/main/ov
mkdir -p ~/docker/isaac-sim/cache/main/warp
mkdir -p ~/docker/isaac-sim/cache/computecache
mkdir -p ~/docker/isaac-sim/config
mkdir -p ~/docker/isaac-sim/data/documents
mkdir -p ~/docker/isaac-sim/data/Kit
mkdir -p ~/docker/isaac-sim/logs
mkdir -p ~/docker/isaac-sim/pkg
sudo chown -R 1234:1234 ~/docker/isaac-sim
```

**5. Isaac Sim 컨테이너 실행**

```bash
docker run --name isaac-sim --entrypoint bash -it --gpus all -e "ACCEPT_EULA=Y" --rm --network=host \
    -e "PRIVACY_CONSENT=Y" \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v ~/docker/isaac-sim/cache/main:/isaac-sim/.cache:rw \
    -v ~/docker/isaac-sim/cache/computecache:/isaac-sim/.nv/ComputeCache:rw \
    -v ~/docker/isaac-sim/logs:/isaac-sim/.nvidia-omniverse/logs:rw \
    -v ~/docker/isaac-sim/config:/isaac-sim/.nvidia-omniverse/config:rw \
    -v ~/docker/isaac-sim/data:/isaac-sim/.local/share/ov/data:rw \
    -v ~/docker/isaac-sim/pkg:/isaac-sim/.local/share/ov/pkg:rw \
    -u 1234:1234 \
    nvcr.io/nvidia/isaac-sim:5.1.0
```

> 💡 `-e DISPLAY=$DISPLAY`와 `-v /tmp/.X11-unix:/tmp/.X11-unix` 옵션은 GUI 윈도우 출력을 위해 필요합니다. 누락 시 디스플레이 오류가 발생합니다.

**6. Isaac Sim 실행 (컨테이너 내부)**

```bash
./runapp.sh
```

---

## 📌 참고 링크

- [Isaac Sim 공식 문서](https://docs.isaacsim.omniverse.nvidia.com/5.1.0/)
- [NGC Isaac Sim 컨테이너](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/isaac-sim)
- [Docker 공식 문서](https://docs.docker.com/)
- [ROS2 Humble 설치](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)
- [ROS2 Jazzy 설치](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html)
