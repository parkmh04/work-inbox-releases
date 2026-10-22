# work-inbox 1.0.1 설치·설정 가이드 (회사 맥, AI용)

사용 방법: 회사 맥의 opencode 또는 Claude Code에 "이 문서(`SETUP.md`, https://github.com/parkmh04/work-inbox-releases/blob/main/SETUP.md)대로 work-inbox를 설치·설정해 줘"라고 붙여 넣는다. AI는 아래 규칙을 지키며 단계 순서대로 진행한다.

## AI에게 주는 규칙

**사용자 확인 없이 하지 말 것** (하려면 먼저 무엇을 왜 하는지 말하고 승인받는다)
- 시스템 보안 설정 변경 (Gatekeeper 끄기, SIP, 방화벽, 프라이버시 설정 등)
- 키체인 조작 (읽기·쓰기·인증서 생성 모두)
- `sudo` 사용

**절대 하지 말 것**
- 토큰 파일(`~/Library/Application Support/work-inbox/token`) 내용 출력·로그·복사. 존재 여부만 `ls`로 본다.
- 실제 고객·계좌·주민번호·회사 기밀 데이터를 시험에 사용. 시험은 가짜 데이터만.
- 앱 서명을 우회하는 도구 설치 (서명 제거·재서명 도구, 보안 솔루션 비활성화 도구 등)
- 명령 출력에 API 키·토큰 같은 비밀값이 보이면 그대로 옮겨 적지 말고 가린다.

**회사 보안 정책과 충돌하면 멈추고 사람에게 묻는다.** (MDM 차단, 설치 제한, 네트워크 차단 등. 우회를 시도하지 않는다.)

사람이 직접 해야 하는 단계는 문서에 **[사람]** 으로 표시했다. 그 단계에서는 안내만 하고 기다린다.

## ① 환경 확인

```
sw_vers -productVersion        # macOS 버전 (13 이상 권장)
uname -m                       # arm64 여야 함 (Apple Silicon). x86_64면 멈추고 사람에게 보고: 배포 DMG는 arm64 전용
claude --version               # Claude Code 설치 여부
claude -p "hello"              # 회사 설정(Bedrock 라우터)으로 응답이 오는지 최소 확인
```

- `claude -p "hello"`는 짧은 응답만 오면 통과다. 출력에 키·토큰·URL 쿼리가 섞여 있으면 사람에게 보여 주거나 기록하지 말고 "응답 정상/오류"만 보고한다.
- `claude`가 없거나 응답하지 않으면 여기서 멈춘다. work-inbox는 이 Claude Code를 불러 쓴다. 설치·회사 설정은 IT 안내를 따른다.
- 설정 파일 `~/.claude/settings.json`은 **내용을 출력하지 않는다.** 필요하면 파일이 있는지만 확인한다.

## ② DMG 받기

대상: GitHub 공개 저장소 `parkmh04/work-inbox-releases`의 Release `v1.0.1`, 파일 `work-inbox-1.0.1-arm64.dmg` (실제 파일명은 Release 화면에서 확인).

**[사람]** `gh` 로그인이 필요하면 사람이 한다: `gh auth login` (브라우저 인증). AI는 토큰을 요구하거나 다루지 않는다.

```
gh auth status
mkdir -p ~/Downloads/work-inbox-1.0.1 && cd ~/Downloads/work-inbox-1.0.1
gh release download v1.0.1 --repo parkmh04/work-inbox-releases --pattern '*.dmg'
ls -l
```

`gh`가 없거나 로그인이 어려우면 **[사람]** 브라우저로 저장소 Releases 페이지에서 `v1.0.1`의 DMG를 `~/Downloads`에 받는다.

## ③ 설치·첫 실행

1. DMG를 연다: `open ~/Downloads/work-inbox-1.0.1/*.dmg` 후 `work-inbox.app`을 `/Applications`로 복사한다 (Finder 드래그 또는 `cp -R "/Volumes/<볼륨 이름>/work-inbox.app" /Applications/`). 다 쓰면 `hdiutil detach "/Volumes/<볼륨 이름>"`.
2. **[사람]** 첫 실행: Finder에서 `/Applications/work-inbox.app` 우클릭 → 열기. 막히면 시스템 설정 → 개인정보 보호 및 보안 → "그래도 열기"를 사람이 누른다. 이 앱은 자체 서명(ad-hoc)이라 이 경로가 정상이다.
3. AI는 `xattr -dr com.apple.quarantine /Applications/work-inbox.app`를 쓰기 전에 사용자 확인을 받는다 (보안 설정 성격이므로).
4. **MDM·보안 솔루션이 실행을 막으면 멈춘다.** 우회하지 말고 사람에게 "IT 보안팀에 work-inbox.app 허용을 문의하세요"라고 안내한다.
5. 정상이면 메뉴바에 work-inbox 아이콘이 뜨고 창이 열린다. 확인: `curl -s 127.0.0.1:4823/healthz` (응답이 오면 코어 동작).

## ④ 회사 과금 설정

회사 Bedrock으로 과금되는 Claude Code를 쓰므로, 앱의 "Max 구독 확인"을 건너뛰고 Claude Code가 자기 `~/.claude/settings.json`의 회사 설정을 그대로 쓰게 한다. 이 설정은 **config.yaml에서만** 한다 (설정 화면 입력줄로는 바꿀 수 없다).

파일: `~/Library/Application Support/work-inbox/config.yaml`

1. 앱을 종료한다 (메뉴바 아이콘 → 종료).
2. 파일이 없으면 만들고, 있으면 **기존 내용을 보존한 채** 아래를 추가한다. 먼저 백업: `cp config.yaml config.yaml.bak` (파일이 있을 때).
   - **회사 Claude Code를 `ccr code`(Claude Code Router)로 쓰는 경우(권장 설정)**:
     ```yaml
     model:
       billing: company
       companyRouter: ccr
     ```
     앱이 `~/.claude-code-router/config.json`에서 라우터 주소와 토큰을 자동으로 읽는다(값은 화면·로그에 나오지 않는다). 라우터 서비스가 떠 있어야 하므로 먼저 `ccr status`로 확인하고, 꺼져 있으면 `ccr start`. 설정 파일 구조가 달라(ccr v3 등) "ccr 설정을 찾지 못함" 오류가 나면 사람에게 라우터 주소를 확인받아 아래처럼 직접 지정한다(토큰 값은 사람이 직접 입력, AI는 출력 금지):
     ```yaml
     model:
       billing: company
       companyEnv:
         ANTHROPIC_BASE_URL: "http://127.0.0.1:3456"
         ANTHROPIC_AUTH_TOKEN: "<사람이 입력>"
     ```
   - **회사 설정이 `~/.claude/settings.json`의 `env`에 있는 경우(Bedrock·게이트웨이)**:
     ```yaml
     model:
       billing: company
     ```
   - 이미 최상위 `model:` 키가 있으면 새로 만들지 말고 그 아래에 `billing: company` 한 줄만 들여쓰기로 추가한다 (`model:`이 두 번 나오면 YAML이 깨진다).
   - 들여쓰기는 공백 2칸, 탭 금지.
   - **전제 확인**: 회사 Claude Code 설정(Bedrock·게이트웨이 환경변수)이 `~/.claude/settings.json`의 `env` 블록에 있어야 한다. 셸에서 `export`로만 설정돼 있으면 Finder로 켠 앱에서는 동작하지 않는다. AI는 이 파일의 **값을 출력하지 말고**, `env` 키가 있는지만 확인한다(예: `python3 -c "import json,os;print('env' in json.load(open(os.path.expanduser('~/.claude/settings.json'))))"`). 없으면 멈추고 사람에게 회사 설정 방법을 묻는다.
   - 회사 모드에서 앱은 모델을 지정하지 않고 Claude Code가 회사 설정대로 고르게 한다. 특정 모델로 고정하려면 `model:` 아래 `companyModel: <회사에서 쓰는 모델 ID>`를 추가한다.
3. 확인: `grep -n -A3 '^model:' ~/Library/Application\ Support/work-inbox/config.yaml`
4. 앱을 다시 실행한다 (`open -a work-inbox`). 설정 화면의 "모델 경로와 과금" 행에서 회사 과금 표시를 확인한다.

## ⑤ 캘린더 권한

첫 실행 시 "work-inbox" 캘린더 접근 대화상자가 뜬다. **[사람]** 허용을 누른다. 안 떴거나 거부했다면 메뉴바 아이콘 → "캘린더 권한 요청…". 그래도 안 되면 사람이 시스템 설정 → 개인정보 보호 및 보안 → 캘린더에서 work-inbox를 켠다. 앱이 쓰는 것은 전용 캘린더 `work-inbox` 하나이며 참석자를 넣지 않는다. 회사 캘린더 계정 정책이 막으면 멈추고 사람에게 묻는다. `tccutil reset Calendar`는 모든 앱의 권한을 초기화하므로 사람 승인 없이 쓰지 않는다.

## ⑥ 가짜 데이터로 동작 확인

1. 앱의 **올리기** 화면에 아래 가짜 메모 한 줄을 붙여 넣고 올리기(⌘↵)를 누른다.
   ```
   김가짜 팀장에게 금요일까지 양식 보내기
   ```
2. 잠시 뒤 같은 화면에 후보 카드(할 일/약속)가 뜨는지 본다. 뜨면 추출 경로(Claude Code → 회사 Bedrock)가 정상이다.
3. 후보를 승인하지 않아도 된다. 승인 시 캘린더에 `work-inbox` 일정이 생기므로, 승인했다면 확인 후 캘린더 앱에서 그 일정을, 앱에서 해당 항목을 삭제한다.
4. 올린 시험 자료는 올리기/보관함 카드의 ✕("이 자료 지우기")로 지운다.
5. 실제 업무 메모는 위 확인이 끝난 뒤 사람이 쓴다. 고객·개인신용정보·회사 기밀은 넣지 않는 것이 이 앱의 원칙이다.

## ⑦ 문제 해결

| 증상 | 확인 | 조치 |
|---|---|---|
| 추출이 "로그인" 오류 (배너 "Claude Code 로그인 필요") | 터미널에서 `claude -p "hello"` | 앱과 같은 사용자 계정에서 Claude Code가 회사 설정으로 응답해야 한다. 안 되면 `~/.claude/settings.json`의 회사 설정(IT 안내값)을 사람이 점검. 비밀값은 AI가 출력하지 않는다. 고친 뒤 보관함 "다시 시도" |
| "회사 경로에서 구조화 출력 실패" | Claude Code 버전·회사 게이트웨이 지원 | 관리자에게 Claude Code 버전 확인 |
| "ccr 설정을 찾지 못함" | `ccr status`, `~/.claude-code-router/config.json` 존재 | 라우터 시작 또는 `companyEnv`로 직접 지정 |
| "개인 로그인으로 잡혔습니다" | 회사 경로가 아닌 개인 Claude 로그인 | `companyRouter`/`companyEnv` 설정 확인, 회사 경로로만 호출되게 |
| 앱이 설정 오류로 안 뜸 | `config.yaml` 키 오타(예: `biling`) | 키 이름 수정 |
| "구독 아님" 오류 | `config.yaml`의 `model:` 아래 `billing: company` 유무·들여쓰기 | 설정을 넣거나 고친 뒤 앱 재시작 |
| 401 (로컬 서버 인증 실패) | 크롬 확장을 쓰는 경우만 해당. 확장 옵션의 토큰 | 앱 설정 화면의 "토큰 복사" 값을 확장 옵션에 다시 붙여넣기. AI는 토큰 파일을 읽어 출력하지 않는다 |
| 401 (모델 호출, Bedrock 쪽) | `claude -p "hello"` 결과 | 회사 인증 만료일 수 있음. 사람이 회사 방식으로 재인증 |
| 앱이 안 뜸 / 메뉴바만 / 포트 | `curl -s 127.0.0.1:4823/healthz` | 응답 없으면 `~/Library/Logs/work-inbox/` 로그 확인 후 재실행. 포트 충돌이면 `lsof -i :4823`로 점유 프로세스 확인 (다른 프로세스를 임의로 종료하지 말고 사람에게 알린다) |
| 캘린더에 안 올라감 | 시스템 설정 → 캘린더 권한 | 허용 후 앱 재시작 |
| 앱 실행이 막힘 | Gatekeeper/MDM 메시지 | ③ 참고. MDM이면 IT 문의 |

로그는 `~/Library/Logs/work-inbox/`에 있다. 로그를 사람에게 보여 줄 때도 토큰·키 줄은 가린다.

## ⑧ 되돌리기

- 회사 과금 설정만 되돌리기: 앱 종료 → `config.yaml`에서 `billing: company` 줄을 지운다 (`model:` 아래에 다른 키가 없으면 `model:` 줄도 지운다). 백업이 있으면 `config.yaml.bak`으로 복원. 앱 재시작.
- 앱 삭제: 앱 종료 → `/Applications/work-inbox.app`을 휴지통으로. 로그인 항목에 남아 있으면 시스템 설정 → 일반 → 로그인 항목에서 제거.
- 데이터까지 완전 제거 (사람 확인 후): `~/Library/Application Support/work-inbox`, `~/Library/Logs/work-inbox` 삭제, 캘린더 앱에서 `work-inbox` 캘린더 삭제. vault(사용자가 쌓은 메모·할 일)는 사람이 판단해 지운다.

사용법은 앱 안 설정·도움말과 이 저장소 README를 참고한다.
