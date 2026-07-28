# 🎫 Cobblemon Gateway

Linux 노트북을 Cobblemon 홈서버로 구성하고 친구 PC의 클라이언트 설치까지 자동화하는 스킬입니다. 친구는 호스트의 tailnet에 가입하지 않고 Tailscale Device Sharing으로 공유된 서버 노드의 TCP 25565만 사용합니다.

실행은 번호순 단계가 아니라 의존성 웨이브로 진행됩니다:

1. Linux 호스트에 Fabric/Cobblemon 서버 설치
2. 서버를 Tailscale IP의 TCP 25565에만 바인딩
3. Tailscale Grants와 Device Sharing으로 서버 노드만 공유
4. 친구 PC에서 Tailscale과 Minecraft 클라이언트 병렬 준비
5. 공유 링크 수락과 서버 연결 검증

## 사용법

호스트 또는 친구의 에이전트에게 `SKILL.md` 파일을 주고 역할에 맞게 실행시키세요.

호스트가 제공해야 할 정보:
- Cobblemon 서버 노드의 Device Share 링크
- 서버 주소 (`100.x.x.x:25565`)

친구는 호스트 tailnet의 멤버가 되지 않으며, 자신에게 공유된 Linux 서버 노드만 볼 수 있습니다. 호스트의 다른 기기와 subnet route는 공유되지 않습니다. Tailscale Grants는 공유 사용자가 해당 노드의 `tcp:25565`만 사용할 수 있도록 제한합니다.

친구는 자신의 Tailscale 계정과 tailnet으로 공유 링크를 수락합니다. 링크는 비밀번호처럼 취급하고, 호스트 tailnet 초대 링크나 auth key는 친구에게 전달하지 않습니다.
