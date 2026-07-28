---
name: cobblemon-gateway
description: "PC를 코블몬 서버에 자동 연결. 접속 정보를 기다리는 동안 Tailscale+Java+Fabric+Cobblemon 설치를 백그라운드에서 선행한다."
---

# 🎫 코블몬 게이트웨이 — 포켓몬 세계로의 초대장

> **"너의 여정은 지금부터다."**

목표: 마인크래프트 코블몬 서버 접속을 위한 **모든 클라이언트 세팅을 자동화**한다. 유저는 필요한 인증과 질문에만 응답한다.

---

## 🎯 성공 기준과 의존성 웨이브

S 번호는 식별자일 뿐 실행 순서가 아니다. 아래 의존성 웨이브에 따라 실행한다.

| 웨이브 | 기준 | 체크항목 | 검증 방법 | 선행 조건 |
|--------|------|----------|-----------|-----------|
| W1-A | S1 | Tailscale 설치됨 | `tailscale version`이 버전을 반환 | 없음 |
| W1-B | S4 | Fabric Loader 설치됨 | 런처 프로필의 `lastVersionId`가 `fabric-loader-<버전>-1.21.1` 형식 | Java 21, Minecraft 경로 |
| W1-B | S5 | Cobblemon mod 설치됨 | `mods/`에 Minecraft 1.21.1 Fabric용 Cobblemon jar 존재 | Minecraft 경로 |
| W1-B | S6 | Fabric API 설치됨 | `mods/`에 Minecraft 1.21.1 Fabric API jar 존재 | Minecraft 경로 |
| W2 | S2 | Tailscale 로그인 완료 | `tailscale status --json`의 `BackendState`가 `Running` | S1, 초대 또는 인증 |
| W3 | S3 | 호스트 tailnet 접근 가능 | `tailscale ping <SERVER_ADDRESS>` 성공 | S2, 서버 주소 |
| W4 | S7 | 실행 안내 완료 | Fabric 프로필, Multiplayer 절차, 실제 서버 주소 전달 | S3, S4, S5, S6 |

```text
W0 시작
├─ W1-A: S1 ──→ W2: S2 ──→ W3: S3 ──┐
└─ W1-B: S4 + S5 + S6 ───────────────┴─→ W4: S7
```

웨이브는 전체 동기화 장벽이 아니라 **의존성 그룹**이다. 선행 조건이 충족된 작업은 다른 작업을 기다리지 말고 즉시 시작한다.

**S1~S7 전부 PASS = 🎫 게이트 오픈.**

---

## ⚡ 강제 출력 규칙

**스킬 로드 즉시 출력:**

```text
🎫 코블몬 게이트웨이 시작!

포켓몬 세계로 가는 3단계:
1️⃣ Tailscale — 보이지 않는 다리를 놓는다
2️⃣ Fabric — 마인크래프트에 새 영혼을 심는다
3️⃣ Cobblemon — 포켓몬을 불러온다

준비 됐어? 출발한다.
```

| 완료 시점 | 출력 문구 |
|-----------|-----------|
| W1 작업 시작 | `⚙️ 필요한 설치를 먼저 백그라운드에서 시작했다. 설치하는 동안 연결 정보만 받을게.` |
| S4-S6 완료 | `✨ 마인크래프트에 새 영혼이 깃들었다!` |
| S1-S3 완료 | `🌍 다리가 완성됐다! 네 PC가 포켓몬 세계와 연결됐다.` |
| S7 완료 | `🎫 게이트 오픈! 이제 서버 주소만 치면 돼.` |
| 실패 | `💢 무슨 일이야! 로켓단이 방해하고 있어! [에러]` |

---

## W0: ⚙️ 즉시 시작하고 질문하기

**접속 정보가 없어도 가능한 작업부터 시작한다. 질문 때문에 설치를 늦추지 않는다.**

1. 시작 문구를 출력한다.
2. OS, Minecraft 경로, 기존 설치 상태를 감지한다.
3. W1-A와 W1-B의 실행 가능한 작업을 백그라운드에서 시작한다.
4. PID/프로세스 핸들, 로그 경로, 시작 여부를 기록한다.
5. W1 작업 시작 문구를 출력한다.
6. 누락된 호스트 정보만 질문한다.
7. 답변이 오면 백그라운드 작업을 회수하면서 충족된 의존성부터 W2 이후를 진행한다.

### OS와 Minecraft 경로

```python
import os, platform

os_name = platform.system()  # Windows / Darwin / Linux
mc_dir = {
    "Windows": os.path.join(os.environ.get("APPDATA", ""), ".minecraft"),
    "Darwin": os.path.expanduser("~/Library/Application Support/minecraft"),
    "Linux": os.path.expanduser("~/.minecraft"),
}[os_name]
```

### 백그라운드 실행 규칙

- macOS/Linux에서는 `nohup` 또는 에이전트 도구의 지속 실행 세션을 사용한다.
- Windows에서는 `Start-Process -PassThru`를 사용한다.
- 질문 출력 후에도 작업이 살아 있어야 한다.
- stdout/stderr를 작업별 로그에 저장하고 종료 코드를 확인한다.
- 같은 패키지 관리자를 사용하는 설치는 잠금 충돌을 피하도록 직렬화한다. 모드 다운로드처럼 독립적인 네트워크 작업은 병렬화한다.
- 관리자 비밀번호나 OAuth 입력을 기다리는 작업은 백그라운드에서 무한 대기시키지 않는다. 해당 작업만 중단 상태로 기록하고 독립 작업은 계속한다.
- 비밀번호나 인증 토큰을 출력하거나 로그에 남기지 않는다.

### 호스트 정보 받기

설치 시작 후 다음 정보의 제공 여부를 확인한다:

```text
TAILSCALE_INVITE_URL=https://login.tailscale.com/a/XXXXX
# 또는 TAILSCALE_AUTH_KEY=tskey-auth-XXXXX
SERVER_ADDRESS=100.x.x.x
```

누락된 정보만 한 번에 질문한다:

> ⚙️ 설치는 이미 뒤에서 진행 중이야.
> 🎫 게이트를 연결하려면 호스트한테 받아야 할 게 있어:
> 1. Tailscale 초대 링크 또는 auth key
> 2. 서버 주소 (100.x.x.x 형식)
> 이것만 보내줘. 기다리는 동안 설치는 계속된다.

정보가 이미 제공되었으면 질문하지 않는다.

---

## W1: 🧰 로컬 클라이언트 준비

W1-A와 W1-B를 안전한 범위에서 병렬 실행한다. 같은 패키지 관리자를 사용하는 Tailscale과 Java 설치는 한 작업 안에서 직렬 실행해 잠금 충돌을 피한다.

### W1-A: Tailscale 설치 — S1

이미 설치되었으면 건너뛴다.

**Windows:**
```powershell
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

**S1 검증:**
```bash
tailscale version
```

### W1-B1: Cobblemon과 Fabric API 다운로드 — S5, S6

Java 설치와 독립적이므로 Minecraft 경로가 확인되는 즉시 시작한다. 두 파일 다운로드는 병렬 실행해도 된다.

Modrinth API의 `/v2/project/<project>/version`에서 `game_versions=["1.21.1"]`, `loaders=["fabric"]` 조건으로 최신 호환 버전을 조회한다. `primary: true`인 파일을 우선 선택하고, 없으면 첫 번째 파일을 `mc_dir/mods/`에 다운로드한다.

- Cobblemon 프로젝트: `cobblemon`
- Fabric API 프로젝트: `fabric-api`

```python
import os
os.makedirs(os.path.join(mc_dir, "mods"), exist_ok=True)
```

**S5/S6 검증:** `mods/`에 각 API 응답의 파일명과 일치하는 jar가 있고 크기가 0보다 큰지 확인한다.

### W1-B2: Java 21과 Fabric Loader — S4

```bash
java -version
```

Java 21이 없으면 설치한다:

- Windows: `winget install Microsoft.OpenJDK.21`
- macOS: `brew install openjdk@21`
- Linux: `sudo apt install openjdk-21-jre`

공식 Fabric Maven에서 Installer를 다운로드하고 Minecraft 경로를 명시해 설치한다:

```bash
curl -o <TEMP_DIR>/fabric-installer.jar https://maven.fabricmc.net/net/fabricmc/fabric-installer/1.0.1/fabric-installer-1.0.1.jar
java -jar <TEMP_DIR>/fabric-installer.jar client -dir "<MC_DIR>" -mcversion 1.21.1 -loader 0.16.0
```

**S4 검증:**

```python
import json, os, re

with open(os.path.join(mc_dir, "launcher_profiles.json"), encoding="utf-8") as f:
    profiles = json.load(f).get("profiles", {}).values()

assert any(
    re.fullmatch(r"fabric-loader-.+-1\.21\.1", p.get("lastVersionId", ""))
    for p in profiles
)
```

S4-S6가 모두 PASS이면 `✨ 마인크래프트에 새 영혼이 깃들었다!`를 출력한다.

---

## W2: 🔐 Tailscale 인증 — S2

S1과 인증 정보가 준비되는 즉시 시작한다. S4-S6 완료를 기다리지 않는다.

1. 초대 URL이 제공되었으면 유저가 해당 tailnet 초대를 수락하도록 브라우저를 연다.
2. `tailscale up`을 실행한다.
3. GitHub OAuth가 열리면:
   - 이미 로그인되어 있으면 승인 버튼을 누른다.
   - 로그인되어 있지 않으면 유저에게 열린 브라우저에서 직접 로그인하도록 안내한다.
   - 에이전트는 비밀번호를 직접 입력하지 않는다.
4. 브라우저 제어가 불가능하면 `tailscale up --qr`을 사용한다.
5. auth key가 제공되었으면 로그에 키를 남기지 않고 `tailscale up --authkey=<KEY>`를 실행한다.

**S2 검증:** 문자열 `Logged in as`에 의존하지 말고 구조화된 상태를 검사한다.

```python
import json, subprocess

status = json.loads(subprocess.check_output(
    ["tailscale", "status", "--json"], text=True
))
assert status.get("BackendState") == "Running"
```

---

## W3: 🌍 호스트 접근 확인 — S3

S2와 `SERVER_ADDRESS`가 준비되면 실행한다.

```bash
tailscale ping --c=3 --timeout=10s <SERVER_ADDRESS>
```

명령이 성공하고 대상 주소의 응답을 받으면 S3 PASS다. 실패하면 `tailscale status --json`에서 peer 목록과 호스트 온라인 상태를 확인한다.

S1-S3가 모두 PASS이면 `🌍 다리가 완성됐다! 네 PC가 포켓몬 세계와 연결됐다.`를 출력한다.

---

## W4: 🎫 게이트 오픈 — S7

S3, S4, S5, S6가 모두 PASS일 때만 실제 서버 주소를 포함해 출력한다:

```text
🎫 게이트 오픈!

이제 마인크래프트를 켜고:

1. 런처 왼쪽 → Minecraft 1.21.1용 Fabric Loader 프로필 선택
2. PLAY 클릭
3. 메인 메뉴 → Multiplayer → Add Server
4. 서버 주소: <SERVER_ADDRESS>
5. Join Server!

포켓몬 세계에 오신 걸 환영한다 🎫
```

Fabric 프로필 선택, Multiplayer 절차, 실제 서버 주소를 모두 전달하면 S7 PASS다.

---

## 백그라운드 작업 회수

- 기록한 PID/프로세스 핸들로 상태를 확인하고 로그와 종료 코드를 읽는다.
- 아직 실행 중이면 인증 등 독립적인 작업을 진행한 뒤 다시 확인한다.
- 실패 로그의 원인을 찾아 한 번 복구한다. 같은 명령을 무한 반복하지 않는다.
- 한 작업의 실패 때문에 독립적인 작업을 취소하지 않는다.
- 모든 S 기준의 검증 결과를 기록하지 않고 완료를 선언하지 않는다.

---

## 🚨 트러블슈팅

| 문제 | 해결 |
|------|------|
| 백그라운드 설치가 멈춤 | 로그와 권한 프롬프트 확인 후 해당 작업만 전면 재시도 |
| `tailscale up` 안 됨 | OS 재부팅, Tailscale 서비스 수동 시작 |
| GitHub OAuth 안 열림 | `tailscale up --qr` 또는 auth key 사용 |
| Fabric Installer 실패 | Java 21과 명시한 `-dir` 경로 확인 |
| Cobblemon 버전 안 맞음 | Modrinth API 필터와 게임 버전 재확인 |
| 서버 접속 실패 | `tailscale status --json`과 `tailscale ping`으로 호스트 확인 |
| Incompatible mod list | 두 mod가 Minecraft 1.21.1 Fabric용인지 확인 |
| RAM 부족 | JVM 인수에 `-Xmx4G -Xms2G` 설정 |
| 모드 폴더 위치 모름 | OS별 `mc_dir/mods/` 사용, 버전 하위 폴더에 넣지 않음 |

---

## ⚠️ 에이전트 행동 가이드라인

- **공식 소스만 사용:** Modrinth CDN, fabricmc.net, tailscale.com
- **설치 먼저, 질문은 그다음:** W1 시작 확인 후 필요한 정보만 질문
- **의존성 기준으로 진행:** S 번호를 직렬 단계로 해석하지 않음
- **독립 작업 계속:** 실패 시 의존 작업만 중단
- **백그라운드 작업 회수:** PID/핸들, 종료 코드, 로그 확인 필수
- **비밀 보호:** 비밀번호와 auth key를 출력하거나 로그에 저장하지 않음
- **컨셉 유지:** 기술 용어를 줄이고 친절한 문구 사용

---

## 📎 버전 정보 (2026-07 기준)

| 항목 | 버전 |
|------|------|
| Minecraft | 1.21.1 |
| Java | 21 |
| Cobblemon | 1.7.3 |
| Fabric Loader | 0.16.0+ |
| Fabric API | 0.116.14+1.21.1 |
