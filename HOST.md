# 🏠 코블몬 홈서버 구축 — 호스트 전용 가이드

> **이 파일은 호스트만 읽는다.** 플레이어에게는 보이지 않는다.

이 가이드는 Linux 노트북을 안전한 Cobblemon 홈서버로 구성하는 절차를 다룬다. 서버 구축이 완료되면 `SKILL.md`를 플레이어에게 전달하여 클라이언트 접속을 자동화한다.

---

## 🔒 네트워크 불변 조건

- 플레이어를 호스트 tailnet의 `Member`로 초대하지 않는다.
- 호스트 auth key를 클라이언트 PC에 전달하거나 사용하지 않는다.
- Tailscale **Device Sharing**으로 Cobblemon Linux 노드 하나만 공유한다.
- 공유 사용자는 자기 Tailscale 계정과 tailnet을 유지한다.
- 공유 노드의 `tcp:25565`만 허용하고 SSH, 관리 UI, 파일 공유 등 다른 서비스는 허용하지 않는다.
- Minecraft 서버를 공개 인터넷에 직접 노출하지 않는다.
- 기존 `* → * : *` allow-all 정책이 있으면 좁은 규칙을 추가하는 것만으로 제한되지 않는다. 공유 사용자와 일치하는 광범위 규칙을 제거하거나 축소한다.

---

## 🎯 호스트 성공 기준

| 기준 | 체크항목 | 검증 방법 |
|------|----------|-----------|
| H1 | Linux 서버에 Java 21과 Fabric 1.21.1 설치 | Java 버전 및 Fabric 서버 파일 확인 |
| H2 | Cobblemon과 필수 의존 모드 설치 | Modrinth 호환 jar와 required dependencies 확인 |
| H3 | 서버가 Tailscale IP의 TCP 25565에서 실행 | `ss`와 로컬 접속 검사 |
| H4 | 재부팅 후 자동 실행 및 월드 영속성 | systemd 활성화와 `/srv/cobblemon` 확인 |
| H5 | 공유 사용자는 TCP 25565만 접근 | Tailscale Grants 검토 및 공개 인터페이스 미노출 확인 |
| H6 | 서버 노드 Device Share 링크 생성 | 공유 대상이 Linux 서버 한 대인지 확인 |

**H1~H6 전부 PASS 후에만 플레이어에게 공유 링크를 전달한다.**

---

## H1-H2: Fabric 서버와 모드 설치

서버에 최소 8GB RAM이 필요하며 16GB 이상을 권장한다. 36GB RAM 호스트에서는 Java 힙을 8~12GB로 설정한다.

1. CPU, RAM, 디스크와 Linux 배포판을 감지한다.
2. Tailscale이 호스트 본인 계정으로 연결되어 있는지 확인한다.
3. Java 21을 설치한다.
4. 전용 사용자와 영속 디렉터리를 준비한다.

```bash
id cobblemon >/dev/null 2>&1 || sudo useradd --system --home /srv/cobblemon --shell /usr/sbin/nologin cobblemon
sudo mkdir -p /srv/cobblemon/mods
sudo chown -R cobblemon:cobblemon /srv/cobblemon
```

공식 Fabric Installer로 서버를 설치한다:

```bash
java -jar <FABRIC_INSTALLER> server -dir /srv/cobblemon -mcversion 1.21.1 -loader 0.17.2 -downloadMinecraft
```

Modrinth API에서 Minecraft `1.21.1`, loader `fabric` 조건으로 Cobblemon과 `required` dependencies를 재귀적으로 해석해 `/srv/cobblemon/mods/`에 설치한다. 현재 필수 의존성에는 Fabric API가 포함된다. 각 jar의 크기와 API 응답 파일명을 검증한다.

Minecraft EULA는 호스트가 명시적으로 동의한 경우에만 `eula=true`로 설정한다.

---

## H3-H4: Tailscale 전용 바인딩과 자동 실행

```bash
tailscale ip -4
```

`server.properties`에 다음을 설정한다:

```properties
server-ip=<HOST_TAILSCALE_IPV4>
server-port=25565
white-list=true
enforce-whitelist=true
```

서버를 `/srv/cobblemon`에서 실행하고 systemd 서비스에 다음 불변 조건을 적용한다:

- `User=cobblemon`
- `WorkingDirectory=/srv/cobblemon`
- `After=network-online.target tailscaled.service`
- `Requires=tailscaled.service`
- `ExecStart=/usr/bin/java -Xms8G -Xmx12G -jar fabric-server-launch.jar nogui`
- `Restart=on-failure`

검증:

```bash
systemctl is-enabled cobblemon
systemctl is-active cobblemon
ss -ltnp | grep ':25565'
```

---

## H5: 공유 사용자 포트 격리

호스트 방화벽에서 Minecraft 포트를 Tailscale 인터페이스에만 허용한다. 기존 규칙을 먼저 읽고 충돌 없이 병합한다.

```bash
sudo ufw allow in on tailscale0 proto tcp to any port 25565 comment 'Cobblemon via Tailscale'
```

공개 인터페이스에 대한 `25565/tcp` allow 규칙이 없어야 한다. 가능하면 Minecraft도 Tailscale IPv4에만 바인딩한다.

Tailscale 관리 콘솔의 Grants에 다음 최소 권한을 병합한다. Device Sharing 사용자는 애초에 자신에게 공유된 노드만 볼 수 있으므로 목적지는 `*`로 두되, 네트워크 권한은 Minecraft의 TCP 25565로 제한한다:

```json
{
  "grants": [
    {
      "src": ["autogroup:shared"],
      "dst": ["*"],
      "ip": ["tcp:25565"]
    }
  ]
}
```

Grants는 누적 적용된다. 기존 `src: ["*"]` 또는 다른 allow-all 규칙이 공유 사용자에게 일치하면 반드시 제거하거나 축소한다. 정책 미리보기에서 공유 사용자가 자신에게 공유된 노드의 TCP 25565만 접근하고, 그 노드의 SSH·관리 UI·파일 공유 등 다른 포트에는 접근하지 못하는지 확인한다. 기존 호스트 방화벽 규칙도 감사해 `tailscale0`에서 다른 관리 포트를 허용하지 않도록 한다.

---

## H6: 서버 노드 공유

Tailscale Machines 화면에서 **Cobblemon Linux 서버 노드 하나만** Share한다. 소규모 초대 그룹에는 사람마다 일회용 링크를 따로 발급한다. 공유 링크는 비밀번호처럼 취급하고 공개 채널이나 로그에 남기지 않는다. 사용되지 않은 링크는 30일 후 만료된다.

플레이어는 호스트 tailnet 멤버가 되지 않고 자기 Tailscale 계정과 tailnet에 로그인한 상태로 공유를 수락한다. 공유를 수락하려면 플레이어가 자기 tailnet의 Owner, Admin 또는 IT admin이어야 하며, 개인 tailnet에서는 보통 본인이 Owner다. 공유된 노드는 플레이어 쪽 tailnet으로 연결을 먼저 시작할 수 없는 격리 상태로 유지된다.

### 화이트리스트 등록

화이트리스트에 플레이어의 Minecraft 닉네임을 추가한다. H3에서 `white-list=true`를 설정했으므로 화이트리스트에 없는 플레이어는 접속이 거부된다:

```bash
# 서버 콘솔 (RCON 또는 서버 터미널)에서 실행
whitelist add <player_minecraft_username>
```

또는 `whitelist.json`에 직접 추가 후 `whitelist reload` 실행. 플레이어에게 Minecraft 닉네임을 미리 받아서 등록한다.

### 플레이어에게 전달할 것

```text
TAILSCALE_SHARE_URL=<device-share-link>
SERVER_ADDRESS=<HOST_TAILSCALE_IPV4>:25565
```

이 두 값과 `SKILL.md`를 플레이어에게 전달한다. tailnet 초대 링크나 auth key는 전달하지 않는다.

---

## 📎 버전 정보 (2026-07 기준)

| 항목 | 버전 |
|------|------|
| Minecraft | 1.21.1 |
| Java | 21 |
| Cobblemon | 1.7.3 |
| Fabric Loader | 0.17.2+ |
| Fabric API | 0.116.14+1.21.1 |
