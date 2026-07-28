---
name: cobblemon-gateway
description: "PC를 코블몬 서버에 자동 연결. 접속 정보를 기다리는 동안 Tailscale+Java+Fabric+Cobblemon 설치를 백그라운드에서 선행한다."
---

# 🎫 코블몬 게이트웨이 — 포켓몬 세계로의 초대장

> **"너의 여정은 지금부터다."**

목표: 마인크래프트 코블몬 서버 접속을 위한 **모든 클라이언트 세팅을 자동화**. 유저는 에이전트 질문에만 대답하면 된다.

---

## 🎯 성공 기준 (전부 PASS 해야 완료)

| # | 체크항목 | 검증 방법 |
|---|---------|----------|
| S1 | Tailscale 설치됨 | `tailscale version` 실행 시 버전 반환 |
| S2 | Tailscale 로그인 완료 | `tailscale status`에 `Logged in as` 표시 |
| S3 | 호스트 tailnet 접근 | `tailscale status`에 호스트 기기 보임 |
| S4 | Fabric Loader 설치됨 | 런처에 "fabric-loader-1.21.1" 프로필 존재 |
| S5 | Cobblemon mod 설치됨 | `.minecraft/mods/`에 Cobblemon jar 존재 |
| S6 | Fabric API 설치됨 | `.minecraft/mods/`에 Fabric API jar 존재 |
| S7 | 마인크래프트 실행 안내 완료 | 서버 주소까지 전달 |

**S1~S7 전부 PASS = 🎫 게이트 오픈.**

---

## ⚡ 강제 출력 규칙

**스킬 로드 즉시 출력:**

```
🎫 코블몬 게이트웨이 시작!

포켓몬 세계로 가는 3단계:
1️⃣ Tailscale — 보이지 않는 다리를 놓는다
2️⃣ Fabric — 마인크래프트에 새 영혼을 심는다
3️⃣ Cobblemon — 포켓몬을 불러온다

준비 됐어? 출발한다.
```

**각 단계 완료 시:**

| 단계 | 출력 문구 |
|------|----------|
| 사전 설치 시작 | `⚙️ 필요한 설치를 먼저 백그라운드에서 시작했다. 설치하는 동안 연결 정보만 받을게.` |
| S1-S3 완료 | `🌍 다리가 완성됐다! 네 PC가 포켓몬 세계와 연결됐다.` |
| S4-S6 완료 | `✨ 마인크래프트에 새 영혼이 깃들었다!` |
| S7 완료 | `🎫 게이트 오픈! 이제 서버 주소만 치면 돼.` |
| 실패 | `💢 무슨 일이야! 로켓단이 방해하고 있어! [에러]` |

---

## 📋 행동 절차

### Phase 0: ⚙️ 먼저 설치하고, 그다음 질문하기

**접속 정보가 없어도 설치 가능한 작업부터 즉시 시작한다. 질문 때문에 설치를 늦추지 않는다.**

반드시 아래 순서를 지킨다:

1. 시작 문구를 출력한다.
2. OS와 기존 설치 상태를 빠르게 감지한다.
3. 누락된 항목의 백그라운드 작업을 시작하고 PID/프로세스 핸들 및 로그 경로를 기록한다.
4. 백그라운드 작업이 실제로 시작된 것을 확인한 뒤 사전 설치 시작 문구를 출력한다.
5. 그제야 아직 받지 못한 호스트 정보를 질문한다.
6. 유저 답변이 오면 백그라운드 작업 결과를 회수하고 각 성공 기준을 검증한다.

#### 0.1 백그라운드 작업 구성

서로 독립적인 두 작업으로 실행한다. 이미 설치된 항목은 건너뛴다.

- **네트워크 작업:** Tailscale 패키지 설치 → S1 사전 검증
- **마인크래프트 작업:** Java 21 설치 → Fabric Loader 설치 → Cobblemon/Fabric API 다운로드 → S4-S6 사전 검증

네트워크 작업은 로그인, OAuth, 초대 수락을 시도하지 않는다. 인증이 필요한 작업은 유저 답변 이후 전면에서 진행한다.

macOS/Linux에서는 `nohup` 또는 에이전트 도구의 지속 실행 세션을 사용하고, Windows에서는 `Start-Process -PassThru`를 사용한다. 질문을 출력한 뒤에도 작업이 살아 있어야 한다. 각 작업의 stdout/stderr를 별도 로그에 저장한다.

관리자 권한이나 비밀번호 입력을 기다리는 명령은 백그라운드에서 무한 대기시키지 않는다. 해당 작업만 중단 상태로 기록하고, 독립적인 다른 설치는 계속한다. 비밀번호나 인증 토큰을 채팅으로 요구하거나 직접 입력하지 않는다.

#### 0.2 호스트 정보 받기

설치 시작 후 다음 정보의 제공 여부를 확인한다:

```
TAILSCALE_INVITE_URL=https://login.tailscale.com/a/XXXXX
# 또는 TAILSCALE_AUTH_KEY=tskey-auth-XXXXX
SERVER_ADDRESS=100.x.x.x (호스트의 Tailscale IP)
```

누락된 정보만 한 번에 물어본다:
> "⚙️ 설치는 이미 뒤에서 진행 중이야.
> 🎫 게이트를 연결하려면 호스트한테 받아야 할 게 있어:
> 1. Tailscale 초대 링크 또는 auth key
> 2. 서버 주소 (100.x.x.x 형식)
> 이것만 보내줘. 기다리는 동안 설치는 계속된다."

정보가 이미 제공되었으면 질문하지 말고 계속한다.

#### 0.3 답변 수신 후 작업 회수

- 기록한 PID/프로세스 핸들로 두 작업 상태를 확인하고 로그를 읽는다.
- 아직 실행 중이면 로그인 준비 등 독립적인 작업을 계속한 뒤 다시 확인한다.
- 실패한 작업은 로그에서 원인을 찾아 한 번 복구 시도한다. 같은 명령을 무한 반복하지 않는다.
- 네트워크 작업이 성공하면 Phase 1 인증으로, 마인크래프트 작업이 성공하면 S4-S6 검증으로 진행한다.
- 한 작업의 실패 때문에 독립적인 다른 작업을 취소하지 않는다.

---

### Phase 1: 🌍 Tailscale — 보이지 않는 다리

#### 1.1 OS 감지

Phase 0에서 감지한 값을 재사용한다. 아직 감지하지 않은 경우에만 실행한다.

```python
import platform
os_name = platform.system()  # Windows / Darwin / Linux
```

#### 1.2 Tailscale 설치

Phase 0의 네트워크 작업에서 실행한다. 이미 설치되었으면 건너뛰고, 백그라운드 설치가 실패했으면 로그를 확인한 뒤 전면에서 복구한다.

**Windows:**
```bash
winget install tailscale.tailscale
```

**macOS:**
```bash
brew install --cask tailscale
```

**Linux:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

**검증 (S1):** `tailscale version` → 버전 나오면 PASS

#### 1.3 Tailscale 회원가입 + 로그인

**⚠️ 핵심: 브라우저 자동화로 GitHub OAuth 로그인 수행**

절차:
1. `tailscale up` 실행 → 브라우저가 `login.tailscale.com` 열림
2. 에이전트가 브라우저 직접 제어 (Playwright/CDP/Selenium):
   - "Sign in with GitHub" 버튼 클릭
   - GitHub OAuth 페이지에서:
     - 이미 로그인되어 있으면 → "Authorize" 클릭
     - 안 되어 있으면 → 친구에게 "GitHub 아이디로 로그인 해줘. 브라우저 열어놨어." 라고 안내
     - **⚠️ 에이전트가 비밀번호를 직접 입력하지 않는다** — 보안상 절대 금지
   - Tailscale 대시보드에서 디바이스 등록 확인
3. 브라우저 제어가 안 되면:
   - `tailscale up --qr` → QR 표시 → 친구에게 스캔 안내
   - 또는 호스트가 auth key 발급: `tailscale up --authkey=tskey-auth-XXXXX`

**검증 (S2):** `tailscale status` → "Logged in as" 확인

#### 1.4 호스트 연결 확인

**검증 (S3):**
```bash
tailscale status  # 호스트 기기 리스트에 보임
ping -c 3 <SERVER_ADDRESS>  # Windows: ping -n 3
```

**S1-S3 전부 PASS → 출력:** `🌍 다리가 완성됐다!`

---

### Phase 2: ✨ Fabric — 새 영혼 심기

이 단계는 원칙적으로 Phase 0의 마인크래프트 작업에서 이미 시작되어 있어야 한다. 유저 답변을 기다리느라 이 단계를 보류하지 않는다.

#### 2.1 .minecraft 경로

| OS | 경로 |
|----|------|
| Windows | `%APPDATA%\.minecraft` |
| macOS | `~/Library/Application Support/minecraft` |
| Linux | `~/.minecraft` |

#### 2.2 Java 21 확인

```bash
java -version
```

없으면 설치:
- Windows: `winget install Microsoft.OpenJDK.21`
- macOS: `brew install openjdk@21`
- Linux: `sudo apt install openjdk-21-jre`

#### 2.3 Fabric Loader 설치

```bash
# Fabric Installer 다운로드
curl -o /tmp/fabric-installer.jar https://maven.fabricmc.net/net/fabricmc/fabric-installer/1.0.1/fabric-installer-1.0.1.jar

# 클라이언트 설치
java -jar /tmp/fabric-installer.jar client -mcversion 1.21.1 -loader 0.16.0
```

**검증 (S4):**
```python
import json, os
profiles_path = os.path.join(mc_dir, "launcher_profiles.json")
with open(profiles_path) as f:
    profiles = json.load(f)
assert any("fabric-loader-1.21.1" in k for k in profiles.get("profiles", {}))
```

#### 2.4 mods 폴더 + Cobblemon + Fabric API

```bash
mkdir -p ~/.minecraft/mods
```

**Cobblemon 최신 다운로드:**
```python
import requests, json, os

# 최신 버전 확인
resp = requests.get('https://api.modrinth.com/v2/project/cobblemon/version?game_versions=["1.21.1"]&loaders=["fabric"]')
ver = resp.json()[0]
cobblemon_url = ver['files'][0]['url']
cobblemon_name = ver['files'][0]['filename']

# 다운로드
r = requests.get(cobblemon_url)
with open(os.path.join(mc_dir, "mods", cobblemon_name), "wb") as f:
    f.write(r.content)
```

**Fabric API 다운로드:**
```python
resp2 = requests.get('https://api.modrinth.com/v2/project/fabric-api/version?game_versions=["1.21.1"]&loaders=["fabric"]')
ver2 = resp2.json()[0]
fab_url = ver2['files'][0]['url']
fab_name = ver2['files'][0]['filename']

r2 = requests.get(fab_url)
with open(os.path.join(mc_dir, "mods", fab_name), "wb") as f:
    f.write(r2.content)
```

**검증 (S5, S6):** `mods/` 폴더에 두 jar 존재 확인

**S4-S6 전부 PASS → 출력:** `✨ 마인크래프트에 새 영혼이 깃들었다!`

---

### Phase 3: 🎫 게이트 오픈

#### 3.1 최종 안내 출력

```
🎫 게이트 오픈!

이제 마인크래프트를 켜고:

1. 런처 왼쪽 → 프로필 선택 → "fabric-loader-1.21.1"

2. PLAY 클릭

3. 메인 메뉴 → Multiplayer → Add Server

4. 서버 주소: 100.x.x.x

5. Join Server!

포켓몬 세계에 오신 걸 환영한다 🎫
```

**S7 PASS → 완료.**

---

## 🚨 트러블슈팅

| 문제 | 해결 |
|------|------|
| tailscale up 안 됨 | OS 재부팅, 서비스 수동 시작 |
| GitHub OAuth 안 열림 | `tailscale login --url` → 브라우저 수동 열기 |
| Fabric Installer 실패 | Java 21 확인 (`java -version`) |
| Cobblemon 버전 안 맞음 | Modrinth API로 최신 재확인 |
| 서버 접속 실패 | `tailscale status`로 호스트 온라인 확인 |
| Incompatible mod list | MC 1.21.1용 jar인지 확인 |
| RAM 부족 | JVM 인수: `-Xmx4G -Xms2G` |
| 모드 폴더 위치 모름 | `.minecraft/mods/` (버전 하위 아님) |

---

## ⚠️ 에이전트 행동 가이드라인

**모든 다운로드는 공식 소스만** — Modrinth CDN, fabricmc.net, tailscale.com
**OS 먼저 감지** → 적절한 명령어 사용
**Java 21 없으면 먼저 설치**
**설치 먼저, 질문은 그다음** → 백그라운드 작업 시작 확인 후 필요한 정보만 질문
**각 단계마다 검증** → 실패 시 의존 작업만 중단하고 독립 작업은 계속
**백그라운드 작업 회수 필수** → PID/핸들, 종료 코드, 로그를 확인하지 않고 완료 선언 금지
**컨셉 문구 유지** — 기술적 용어 최소화, 친절하게

---

## 📎 버전 정보 (2026-07 기준)

| 항목 | 버전 |
|------|------|
| Minecraft | 1.21.1 |
| Java | 21 |
| Cobblemon | 1.7.3 |
| Fabric Loader | 0.16.0+ |
| Fabric API | 0.116.14+1.21.1 |
