# 💻 Medical SSH Tunnel

이 프로젝트는 로컬 윈도우 서버(레노버 PC)의 **SSH 서비스**를 Cloudflare Tunnel을 통해 안전하게 외부로 노출하여, 공인 IP나 포트 포워딩 없이 어디서든 원격 접속이 가능하도록 설정하는 가이드입니다.

---

## 🚀 아키텍처 개요

1.  **Windows SSH Server**: 윈도우 내장 OpenSSH 서버 (포트 22).
2.  **Cloudflare Tunnel (Docker)**: 로컬 포트 22를 Cloudflare 네트워크를 통해 지정된 도메인(`medical.naranja.my`)으로 우회(Proxy) 연결합니다.

---

## 🛠 서버측 설정 (Windows)

터널을 가동하기 전, 윈도우 서버에서 SSH 서비스가 실행 중이어야 합니다.

### 1. OpenSSH 서버 설치 및 시작
관리자 권한으로 PowerShell을 열고 아래 명령어를 실행합니다.

```powershell
# SSH Server 설치 여부 확인 및 설치
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

# 서비스 시작 및 자동 실행 설정
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'

# 22번 포트 방화벽 허용 (이미 되어있을 수 있음)
New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
```

---

## 🐳 터널 실행 방법 (Docker)

1.  `.env` 파일에 발급받은 `TUNNEL_TOKEN`을 입력합니다.
2.  아래 명령어를 통해 터널 컨테이너를 실행합니다.

```powershell
docker-compose up -d
```

---

## 🔒 클라이언트 접속 방법 (외부 기기)

### 1. cloudflared 설치
접속하려는 클라이언트 컴퓨터에 `cloudflared`가 설치되어 있어야 합니다.

*   **Windows (winget):** `winget install Cloudflare.cloudflared`
*   **Mac (brew):** `brew install cloudflare/cloudflare/cloudflared`

### 2. 터미널에서 직접 접속
```bash
ssh -o "ProxyCommand=cloudflared access ssh --hostname %h" [윈도우-계정명]@medical.naranja.my
```

### 3. SSH Config 설정 (권장)
`~/.ssh/config` 파일에 아래 내용을 추가하면 `ssh medical` 키워드만으로 간편하게 접속할 수 있습니다.

```text
Host medical
    HostName medical.naranja.my
    ProxyCommand cloudflared access ssh --hostname %h
    User [윈도우-계정명]
```

---

## 💻 VS Code로 접속하기

1.  VS Code에 **Remote - SSH** 확장을 설치합니다.
2.  `F1` 키를 누르고 `Remote-SSH: Connect to Host...`를 선택합니다.
3.  위에서 설정한 `medical` 호스트를 선택하거나, `[윈도우-계정명]@medical.naranja.my`를 입력하여 접속합니다.

---

## 📁 주요 파일

*   `docker-compose.yaml`: Cloudflare Tunnel 서비스를 정의합니다.
*   `.env`: 보안 토큰(`TUNNEL_TOKEN`)을 관리합니다.