# 🎫 Cobblemon Gateway

친구의 코딩 에이전트에게 전달하는 스킬. 읽기만 하면 Tailscale 가입부터 Cobblemon 클라이언트 설치까지 전부 자동화합니다. 접속 정보를 묻기 전에 설치 가능한 항목을 백그라운드에서 먼저 시작하므로, 답변을 기다리는 시간도 설치에 사용합니다.

실행은 번호순 단계가 아니라 의존성 웨이브로 진행됩니다:

1. Tailscale과 Minecraft 클라이언트 구성 병렬 준비
2. Tailscale 인증
3. 호스트 연결 검증
4. Minecraft 실행 안내

## 사용법

친구의 에이전트에게 `SKILL.md` 파일을 주고 실행시키세요.

호스트가 제공해야 할 정보:
- Tailscale 초대 링크 또는 auth key
- 서버 주소 (100.x.x.x)
