# macOS 머신 제안

macOS(Apple Silicon) 머신의 `~/.config/machine-context/local.md`에 넣을 만한 항목과 셋업 중 겪은 함정이다. 그대로 복사하지 말고 그 머신에서 확인한 것만 옮긴다. 아래 항목은 2026-11에 macOS 27, Homebrew 7, 화면 없이 SSH로만 쓰는 장비 한 대에서 확인했다. 버전이 다르면 다시 확인한다.

## local.md 예시

```markdown
## 셸

- macOS (Apple Silicon, arm64)다. 기본 셸은 zsh다. `/bin/bash`는 3.2라 bash 4 이상 기능을 쓰지 않는다.
- 기본 도구는 GNU가 아니라 BSD다: `sed -i ''`, `date -v-1d` (`date -d` 없음), `stat -f %z` (`stat -c` 없음), `grep -P` 없음.

## 패키지 관리

- 도구는 Homebrew(`/opt/homebrew`)로 관리한다. 각 도구의 자체 업데이트가 아니라 `brew upgrade`로 올린다.

## Docker

- Docker는 Linux VM(colima) 안에서 돈다. 컨테이너 안에는 GPU가 없으므로 GPU 작업은 네이티브(MPS, MLX)로 돌린다. 이미지는 arm64여야 한다.
```

## 함정

- **Command Line Tools가 없으면 `git`·`python3`이 껍데기다:** `/usr/bin/git`·`/usr/bin/python3`은 있지만 실행하면 설치를 요구한다. `command -v`로 "있다"고 판단하지 말고 `xcode-select -p`로 확인한다. Homebrew 설치 스크립트가 Command Line Tools를 같이 깐다.
- **Homebrew 최초 설치는 소유자에게 넘긴다:** 관리자 비밀번호를 묻는다. `sudo`로 실행하면 Homebrew가 root 실행을 거부한다.
- **Homebrew 7은 설치·업그레이드마다 y/n을 묻는다:** ask 모드가 기본값이다. 스크립트에서는 `export HOMEBREW_NO_ASK=1`로 끈다. TTY가 없으면 프롬프트는 건너뛴다.
- **비대화형 SSH는 `~/.zprofile`을 읽지 않는다:** Homebrew 안내대로 `~/.zprofile`에만 `brew shellenv`를 넣으면 `ssh host '명령'`에서 brew로 깐 도구가 보이지 않는다. `~/.zshenv`에도 넣는다. 로그인 셸에서는 `/etc/zprofile`의 `path_helper`가 PATH 순서를 바꾸므로 `~/.zprofile`에도 둔다.
- **brew로 깐 도구는 자체 업데이트가 막혀 있다:** `uv self update`는 "installed through an external package manager"로 거부된다. `brew upgrade`로 올린다.
- **brew로 깐 `docker-compose`·`docker-buildx`는 플러그인 경로를 알려 줘야 한다:** `~/.docker/config.json`에 `"cliPluginsExtraDirs": ["/opt/homebrew/lib/docker/cli-plugins"]`가 있어야 `docker compose`·`docker buildx`로 불린다.
- **시스템 Python은 3.9다:** `/usr/bin/python3`은 3.9.6이다. 다른 버전은 uv가 필요할 때 받는다 (`suggestions-python.md`).
- **화면 없이 쓰는 장비 — 재부팅 뒤 SSH:** FileVault가 꺼져 있으면 화면 로그인 없이도 SSH가 올라온다 (Wi-Fi에서도 확인). 원격 로그인(SSH)은 시스템 서비스라 로그인 세션과 무관하다.
- **화면 없이 쓰는 장비 — 잠자기:** 시스템 잠자기, 덮개 닫힘, 전원별(배터리·어댑터) 설정이 각각 따로다. `pmset -g custom`으로 전원별 값을, `ioreg -r -k AppleClamshellState -d 4`로 덮개 상태와 덮개가 잠자기를 일으키는지를 본다. 어댑터 전원에서 잠자기 0이어도 배터리 전원 값은 다를 수 있다.
- **성능·효율 코어:** `hw.ncpu`는 두 종류를 합친 수다 (`sysctl hw.perflevel0.physicalcpu hw.perflevel1.physicalcpu`가 성능·효율 코어 수). 워커 수를 코어 수로 정할 때 구분한다 (`suggestions-lowend.md`).

## 확인하지 못한 것

- 사용자 서비스(colima, `brew services`로 띄운 것)가 화면 로그인 없이 재부팅 뒤 뜨는지.
- FileVault를 켠 장비에서 재부팅 뒤 화면에서 암호를 넣기 전에 SSH가 올라오는지.
- 개인정보 보호 권한(전체 디스크 접근 등) 창이 SSH 세션에서 어떻게 나오는지.
- Intel Mac(`/usr/local` 경로). 위 경로는 Apple Silicon 기준이다.
