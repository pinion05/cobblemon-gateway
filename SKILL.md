---
name: cobblemon-gateway
description: "친구 PC를 코블몬 서버에 자동 연결. Tailscale+Fabric+Cobblemon 전부 자동 설치."
---

# 🎫 코블몬 게이트웨이 — 포켓몬 세계로의 초대장

> **"너의 여정은 지금부터다."**

이 스킬은 **친구의 PC**에서 실행된다. 목표: 마인크래프트 코블몬 서버 접속을 위한 **모든 클라이언트 세팅을 자동화**. 친구는 에이전트 질문에만 대답하면 된다.

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
| S1-S3 완료 | `🌍 다리가 완성됐다! 네 PC가 포켓몬 세계와 연결됐다.` |
| S4-S6 완료 | `✨ 마인크래프트에 새 영혼이 깃들었다!` |
| S7 완료 | `🎫 게이트 오픈! 이제 서버 주소만 치면 돼.` |
| 실패 | `💢 무슨 일이야! 로켓단이 방해하고 있어! [에러]` |

---

## 📋 행동 절차

### 사전 정보 (호스트가 제공)

실행 전 반드시 확인:

```
TAILSCALE_INVITE_URL=https://login.tailscale.com/a/XXXXX
SERVER_ADDRESS=100.x.x.x (호스트의 Tailscale IP)
```

호스트가 안 줬으면 물어본다:
> "🎫 게이트를 열려면 호스트한테 받아야 할 게 있어:
> 1. Tailscale 초대 링크 또는 auth key
> 2. 서버 주소 (100.x.x.x 형식)
> 이것만 있으면 된다. 나머진 내가 다 한다."

---

### Phase 1: 🌍 Tailscale — 보이지 않는 다리

#### 1.1 OS 감지

```python
import platform
os_name = platform.system()  # Windows / Darwin / Linux
```

#### 1.2 Tailscale 설치

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
**각 단계마다 검증** → 실패 시 다음 단계 진행 금지
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
