# 💻 Medical SSH Tunnel

이 프로젝트는 로컬 윈도우 서버(레노버 PC)의 **SSH 서비스**를 Cloudflare Tunnel을 통해 안전하게 외부로 전용(Exposing)하여 원격 접속을 가능하게 하는 설정입니다.

## 🚀 아키텍처 개요

1.  **Windows SSH Server**: 윈도우에서 실행 중인 기본 OpenSSH 서버 (포트 22).
2.  **Cloudflare Tunnel (SSH)**: 공인 IP나 포트포워딩 없이 도메인을 통해 SSH 우회(Proxy) 접속을 지원합니다.

---

## 🛠 실행 방법

`.env` 파일에 발급받으신 `TUNNEL_TOKEN`을 입력한 뒤 다음 명령어를 실행합니다.

```powershell
docker-compose up -d
```

---

## 🔒 접속 방법 (클라이언트 측)

외부 컴퓨터에서 해당 서버로 접속할 때 가장 간편한 방법은 `cloudflared`가 설치된 상태에서 `~/.ssh/config`에 다음과 같이 설정하는 것입니다.

```text
Host my-windows-server
    HostName [여러분의-주소.my]
    ProxyCommand /usr/local/bin/cloudflared access ssh --hostname %h
    User [윈도우-계정명]
```

이후 `ssh my-windows-server` 명령으로 접속이 가능합니다.

---

## 📁 주요 파일

*   `docker-compose.yaml`: `ssh_tunnel_server` 서비스 하나만 포함하고 있으며, 공식 `cloudflared` 이미지를 사용합니다.
*   `.env`: 터널링을 위한 보안 토큰(`TUNNEL_TOKEN`)을 보관합니다.