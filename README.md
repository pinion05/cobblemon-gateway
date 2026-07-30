# 🎫 Cobblemon Gateway

친구들을 코블몬 서버로 초대하는 원클릭 스킬입니다. 호스트는 서버를 구축하고, 플레이어는 초대장 하나만으로 접속합니다.

## 구조

| 파일 | 대상 | 역할 |
|------|------|------|
| `HOST.md` | 호스트 | Linux 서버 구축: Fabric 서버 설치, Tailscale 포트 격리, Device Sharing 링크 생성 |
| `SKILL.md` | 플레이어 | 클라이언트 자동 세팅: Tailscale, Fabric, Cobblemon 모드 설치 및 서버 접속 |

플레이어는 서버 구축 과정을 전혀 볼 필요가 없습니다. `SKILL.md`만 받으면 됩니다.

## 사용법

### 호스트 (서버 운영자)

1. `HOST.md`를 따라 Linux 서버를 구축합니다.
2. 플레이어의 Minecraft 닉네임을 화이트리스트에 등록합니다.
3. 다음 두 값을 플레이어에게 전달합니다:
   - **Device Share 링크** (Tailscale Machines 화면에서 생성)
   - **서버 주소** (`100.x.x.x:25565`)
4. `SKILL.md`를 플레이어에게 보냅니다.

### 플레이어

1. `SKILL.md`를 코딩 에이전트에게 줍니다.
2. 호스트에게 받은 **공유 링크**와 **서버 주소**를 입력합니다.
3. 에이전트가 Tailscale, Fabric, Cobblemon을 자동으로 설치합니다.
4. 마인크래프트를 켜고 서버에 접속합니다.

## 보안 모델

- 플레이어는 호스트 tailnet에 가입하지 않습니다.
- Tailscale Device Sharing으로 서버 노드 하나만 공유됩니다.
- 공유 사용자는 `tcp:25565`만 사용할 수 있습니다 (Tailscale Grants로 제한).
- 호스트의 다른 기기, SSH, 관리 포트는 노출되지 않습니다.
- 공유 링크는 비밀번호처럼 취급합니다.
