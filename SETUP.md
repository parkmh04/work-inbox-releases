# work-inbox 1.1 설치·설정 가이드 (회사 맥, AI용)

대상 버전: **1.1** (`v1.1.0-rc2`). 1.0.1에서 올리는 경우는 맨 아래 "1.0.1에서 올릴 때"를 먼저 본다.

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

## ① 환경 확인과 회사 AI 경로 고르기

```
sw_vers -productVersion        # macOS 버전 (13 이상 권장)
uname -m                       # arm64 여야 함 (Apple Silicon). x86_64면 멈추고 사람에게 보고: 배포 DMG는 arm64 전용
```

회사 맥에서 AI를 부르는 방법은 셋 중 하나다. **사람에게 어느 것을 쓰는지 묻거나 아래 확인 명령으로 판별**하고, 해당 경로의 확인이 통과해야 ④로 간다. 둘 이상이면 사람이 하나를 고른다.

| 경로 | 이럴 때 | config.yaml (④) | 먼저 통과해야 할 확인 |
|---|---|---|---|
| A. ccr (Claude Code Router) | 회사에서 `ccr code`로 Claude Code를 쓴다 | `billing: company` + `company.router: ccr` | `ccr status`, `claude --version` |
| B. 사내 게이트웨이·Bedrock (`~/.claude/settings.json`의 `env`) | Claude Code가 Bedrock·사내 게이트웨이로 바로 붙는다 | `billing: company` | `claude --version`, `claude auth status`의 authMethod, `claude -p "hello"` |
| C. opencode (사내 모델) | Claude Code 대신 opencode + 사내 모델을 쓴다 | `mode: opencode` + `opencode.*` | `opencode --version`, `echo hi \| opencode run "답하세요"` |

경로별 확인 (공통: **비밀값이 보이면 옮겨 적지 말고, "정상/오류"만 보고**한다):

- **A. ccr**
  ```
  claude --version
  ccr status                     # 라우터 서비스가 떠 있어야 함. 꺼져 있으면 사람에게 알리고 ccr start
  ls ~/.claude-code-router/config.json   # 존재 여부만. 내용 출력 금지
  ```
- **B. 게이트웨이·Bedrock**
  ```
  claude --version
  claude auth status             # 결과 중 authMethod 값만 본다. 나머지(이메일·키 등)는 출력·기록하지 않는다
  claude -p "hello"              # 짧은 응답이 오면 통과
  ```
  - `authMethod`가 `third_party`·`api_key`·`api_key_helper`면 회사 경로로 인식된다. `claude.ai`·`oauth_token`이면 개인 구독 로그인이므로 앱이 막는다(`auth_not_company`) — 멈추고 사람에게 알린다.
  - 회사 설정(Bedrock·게이트웨이 환경변수)은 `~/.claude/settings.json`의 `env` 블록에 있어야 한다. 셸 `export`만으로는 Finder로 켠 앱에서 동작하지 않는다. 이 파일은 **값을 출력하지 않는다.** `env` 키 존재만 본다: `python3 -c "import json,os;print('env' in json.load(open(os.path.expanduser('~/.claude/settings.json'))))"`. 없으면 멈추고 사람에게 묻는다.
- **C. opencode**
  ```
  opencode --version
  echo hi | opencode run "답하세요"     # 응답이 오면 통과. 안 되면 앱에서도 안 된다
  ```
  - 안 되면 **[사람]** 이 터미널에서 `opencode auth login` 또는 사내 `opencode.json`을 준비한다. AI는 키를 요구하거나 다루지 않는다.
  - 사용할 모델 ID는 `opencode models`로 목록을 보고 사람이 정한다(`provider/model` 형식).

경로 확인이 안 되면 여기서 멈춘다. 설치·회사 설정은 IT 안내를 따른다. A·B는 work-inbox가 이 Claude Code를, C는 opencode를 불러 쓴다.

## ② DMG 받기

대상: GitHub 공개 저장소 `parkmh04/work-inbox-releases`의 Release `v1.1.0-rc2`(prerelease), 파일 `work-inbox-1.1.0-rc2-arm64.dmg`.

**[사람]** `gh` 로그인이 필요하면 사람이 한다: `gh auth login` (브라우저 인증). AI는 토큰을 요구하거나 다루지 않는다.

```
gh auth status
mkdir -p ~/Downloads/work-inbox-dmg && cd ~/Downloads/work-inbox-dmg
gh release download v1.1.0-rc2 --repo parkmh04/work-inbox-releases --pattern '*.dmg' --pattern 'SHA256SUMS.txt'
ls -l
shasum -a 256 -c SHA256SUMS.txt     # OK가 나와야 한다
```

`gh`가 없거나 로그인이 어려우면 **[사람]** 브라우저로 저장소 Releases 페이지에서 `v1.1.0-rc2`의 DMG를 `~/Downloads`에 받는다. 공개 저장소라 로그인 없이도 받을 수 있다.

## ③ 설치·첫 실행

1. DMG를 연다: `open ~/Downloads/work-inbox-dmg/*.dmg` 후 `work-inbox.app`을 `/Applications`로 복사한다 (Finder 드래그 또는 `cp -R "/Volumes/<볼륨 이름>/work-inbox.app" /Applications/`). 다 쓰면 `hdiutil detach "/Volumes/<볼륨 이름>"`.
2. **[사람]** 첫 실행: Finder에서 `/Applications/work-inbox.app` 우클릭 → 열기. 막히면 시스템 설정 → 개인정보 보호 및 보안 → "그래도 열기"를 사람이 누른다. 이 앱은 자체 서명(ad-hoc)이라 이 경로가 정상이다.
3. AI는 `xattr -dr com.apple.quarantine /Applications/work-inbox.app`를 쓰기 전에 사용자 확인을 받는다 (보안 설정 성격이므로).
4. **MDM·보안 솔루션이 실행을 막으면 멈춘다.** 우회하지 말고 사람에게 "IT 보안팀에 work-inbox.app 허용을 문의하세요"라고 안내한다.
5. 정상이면 메뉴바에 work-inbox 아이콘이 뜨고 창이 열린다. 확인: `curl -s 127.0.0.1:4823/healthz` (응답이 오면 코어 동작).

## ④ 회사 경로 설정 (config.yaml)

회사 경로 설정은 **config.yaml에서만** 한다 (설정 화면 입력줄·자연어로는 바꿀 수 없다). 설정 화면의 "모델 경로와 과금" 행과 opencode 카드는 현재 상태를 보여 주기만 한다. 앱이 개인 Max 구독 확인을 건너뛰고 회사 경로로만 호출한다.

파일: `~/Library/Application Support/work-inbox/config.yaml`

1. 앱을 종료한다 (메뉴바 아이콘 → 종료).
2. 파일이 있으면 먼저 백업: `cp config.yaml config.yaml.bak`. 없으면 만든다. **기존 내용은 보존**하고 아래 중 ①에서 고른 경로 하나만 넣는다.
   - 이미 최상위 `model:` 키가 있으면 새로 만들지 말고 그 아래에 키만 들여쓰기로 추가한다 (`model:`이 두 번 나오면 YAML이 깨진다). 들여쓰기는 공백 2칸, 탭 금지.
   - **config는 엄격하다.** 모르는 키(오타 포함)가 있으면 오류로 앱이 뜨지 않는다. 아래 키 이름을 그대로 쓴다.

   **A. ccr**
   ```yaml
   model:
     billing: company
     company:
       router: ccr
   ```
   앱이 `~/.claude-code-router/config.json`에서 라우터 주소와 토큰을 읽는다(값은 화면·로그에 나오지 않는다). 설정 파일 구조가 달라 "ccr 설정을 찾지 못함"이 나면 사람에게 라우터 주소를 확인받아 직접 지정한다(토큰은 **사람이 직접 입력**, AI는 출력 금지):
   ```yaml
   model:
     billing: company
     company:
       env:
         ANTHROPIC_BASE_URL: "http://127.0.0.1:3456"
         ANTHROPIC_AUTH_TOKEN: "<사람이 입력>"
   ```

   **B. 게이트웨이·Bedrock**
   ```yaml
   model:
     billing: company
   ```
   앱이 `~/.claude/settings.json`의 `env` 중 허용 키(Bedrock·`ANTHROPIC_BASE_URL`·`ANTHROPIC_AUTH_TOKEN`·`AWS_*`·프록시 등)만 골라 CLI에 넘긴다. 그 파일을 읽게 하지 않으려면 `company.settingsEnv: false`.

   **C. opencode**
   ```yaml
   model:
     mode: opencode
     opencode:
       model: <provider/model>   # `opencode models` 목록의 값. 생략하면 opencode 기본 모델
       path: null                # 생략 시 PATH와 ~/.opencode/bin 등에서 찾음
       timeoutMs: 180000
       envPass: []               # opencode.json이 {env:NAME}으로 읽는 변수 이름(값 아님)만 적는다
   ```
   - C에서는 `billing: company`를 넣지 않는다(앱이 `billing: company`는 `mode: claude-cli`에서만 허용한다). 화면에는 "opencode(사내 모델) · 회사 과금"으로 표시된다.
   - opencode 모드는 이미지를 직접 받지 않고 OCR 글자만 모델에 간다.

   공통 선택 항목 (A·B):
   - 모델 고정: `model.company.model: <회사에서 쓰는 모델 ID>` (생략하면 Claude Code가 회사 설정대로 고름).
   - `--json-schema`가 회사 경로에서 안 될 때: `model.company.structuredOutput: prompt` (기본 `json-schema`).
   - 1.0.1의 키 `companyModel`·`companyEnv`·`companyRouter`가 남아 있어도 앱이 `company.model|env|router`로 옮겨 읽는다. 새로 쓸 때는 `company.*`로 쓴다.
3. 확인: `grep -n -A8 '^model:' ~/Library/Application\ Support/work-inbox/config.yaml` (토큰·키를 넣었다면 그 줄 값은 사람에게 보여 주거나 기록하지 않는다).
4. 앱을 다시 실행한다 (`open -a work-inbox`). 설정 화면의 "모델 경로와 과금" 행(C는 opencode 카드)에서 회사 경로 표시를 확인한다.

### 1.1 기능과 설정 키

1.1에서 늘어난 사용자 기능 중 설정과 닿는 것만 적는다. 아래 키는 모두 선택이며, 모르는 키·오타는 앱이 시작 오류로 막는다(바로 위의 엄격성 규칙). 기본값으로 쓰려면 아무것도 넣지 않는다.

| 기능 | 설정 키 (config.yaml) | 기본 | 비고 |
|---|---|---|---|
| 업로드 포맷 확대 | 없음 | — | 이미지·PDF·텍스트·Word·Excel·PowerPoint·Numbers·Pages·Keynote 등. 전부 이 맥 안에서 읽는다. 한도는 `limits.maxFileBytes`(25MB), `limits.maxInputChars`(150000자), `limits.maxPdfPages`(30) |
| 폴더 올리기 | `limits.batchMaxFiles` / `batchMaxBytes` / `batchCallGapMs` / `scanMaxEntries` / `scanMaxDepth` | 100개 / 500MB / 2000ms / 2000개 / 깊이 6 | 올리기 화면에서 폴더를 고르면 확인 전표가 먼저 뜬다 |
| URL 입력 + 1단계 링크 | `ingest.url.maxLinks` / `maxBytes` / `timeoutMs` / `perHostGapMs` | 20개 / 2MB / 15000ms / 1000ms | 올리기에서 링크를 붙이면 미리보기에서 따라갈 링크를 고르고, 고른 것만 한 단계 가져온다. `maxLinks` 범위 0~50 |
| 사내 주소 허용 | `ingest.url.allowPrivateHosts` | `true` | 아래 참고 |
| 자연어 설정 | 없음 (설정 화면 맨 위 입력줄) | — | 아래 참고 |
| 파일 트리 | 없음 | — | 보관함에서 날짜·종류·"폴더·사이트" 기준으로 묶어 본다 |
| 분류 색인 | `index.enabled` | `true` | vault의 `index/` 아래에 종류·주제·월별 목록을 Markdown으로 자동 생성. 읽기 전용이며 지워도 다시 생긴다. 만들지 않으려면 `index: { enabled: false }` |

**사내 URL**: 올리기의 "링크로 올리기"는 사내 주소(사설망)도 기본 허용이다(`ingest.url.allowPrivateHosts: true`). 막으려면 사람이 아래처럼 바꾼다. AI는 임의로 바꾸지 않는다. 로그인이 필요한 페이지는 읽지 못한다.
```yaml
ingest:
  url:
    allowPrivateHosts: false
```

**자연어 설정**: 설정 화면 맨 위 입력줄에 문장을 쓰면 변경안을 보여 주고, 확인하면 config.yaml의 해당 키만 바꾼다(다른 키·주석은 유지). 바꿀 수 있는 것은 캘린더 읽기(`calendar.read.enabled`·`calendar.read.exclude`), 캘린더 알람(`calendar.timedAlarmMinutesBefore`), 브리핑(`briefing.times`·`briefing.calendar`), 알림(`reminders.enabled`·`reminders.meetingMinutesBefore`·`reminders.overdueAt`)뿐이다. 모델 경로·`billing`·`model.company.*`·`model.opencode.*`·`calendar.name`·`server.port`·`vaultDir`·`ingest.url.allowPrivateHosts`는 과금·외부 전송·데이터 위치에 닿아 **자연어로 바꿀 수 없고 config.yaml에서만** 바꾼다. 따라서 AI가 이 문서대로 회사 경로를 설정할 때는 파일을 직접 편집한다.

## ⑤ 캘린더 권한

첫 실행 시 "work-inbox" 캘린더 접근 대화상자가 뜬다. **[사람]** 허용을 누른다. 안 떴거나 거부했다면 메뉴바 아이콘 → "캘린더 권한 요청…". 그래도 안 되면 사람이 시스템 설정 → 개인정보 보호 및 보안 → 캘린더에서 work-inbox를 켠다. 앱이 쓰는 것은 전용 캘린더 `work-inbox` 하나이며 참석자를 넣지 않는다. 회사 캘린더 계정 정책이 막으면 멈추고 사람에게 묻는다. `tccutil reset Calendar`는 모든 앱의 권한을 초기화하므로 사람 승인 없이 쓰지 않는다.

## ⑥ 가짜 데이터로 동작 확인

1. 앱의 **올리기** 화면에 아래 가짜 메모 한 줄을 붙여 넣고 올리기(⌘↵)를 누른다.
   ```
   김가짜 팀장에게 금요일까지 양식 보내기
   ```
2. 잠시 뒤 같은 화면에 후보 카드(할 일/약속)가 뜨는지 본다. 뜨면 추출 경로(A·B: Claude Code → 회사 경로, C: opencode → 사내 모델)가 정상이다.
3. 후보를 승인하지 않아도 된다. 승인 시 캘린더에 `work-inbox` 일정이 생기므로, 승인했다면 확인 후 캘린더 앱에서 그 일정을, 앱에서 해당 항목을 삭제한다.
4. 올린 시험 자료는 올리기/보관함 카드의 ✕("이 자료 지우기")로 지운다.
5. (선택) 올리기 화면의 폴더 올리기와 링크로 올리기도 가짜 파일 몇 개가 든 빈 폴더, 사내 위키 공개 페이지 하나로 확인할 수 있다. 지원 형식은 이미지·PDF·텍스트·Word·Excel·PowerPoint 등이며 파일 25MB·글자 15만 자 한도다. 폴더는 확인 전표가 뜨면 내용을 보고 올리거나 취소한다. 올린 뒤 보관함의 파일 트리와 vault의 `index/`에 목록이 생겼는지 본다.
6. 실제 업무 메모는 위 확인이 끝난 뒤 사람이 쓴다. 고객·개인신용정보·회사 기밀은 넣지 않는 것이 이 앱의 원칙이다.

## ⑦ 문제 해결

| 증상 | 확인 | 조치 |
|---|---|---|
| 앱이 설정 오류로 안 뜸 | `config.yaml` 키 오타(예: `biling`), `model:` 중복, 탭 들여쓰기, C에서 `billing: company` 동시 지정 | 키 이름·구조 수정 (config는 모르는 키를 오류로 처리한다). 안 되면 `config.yaml.bak`으로 복원 |
| `auth_not_company` ("개인 구독 로그인이 감지됐습니다" / "회사 경로를 확인할 수 없습니다") | `claude auth status`의 authMethod만 확인. `claude.ai`·`oauth_token`이면 개인 로그인. 판별 불가면 회사 env·라우터 신호 없음 | 개인 로그인으로는 호출하지 않는 것이 정상 동작이다. A는 `company.router: ccr`와 `ccr status`, B는 `~/.claude/settings.json`의 `env` 키 존재, 둘 다 아니면 사람에게 회사 경로 확인을 요청. 회사 경로가 아닌 개인 로그인으로 우회하지 않는다 |
| 추출이 "로그인" 오류 (배너 "Claude Code 로그인 필요", `auth_required`) | 터미널에서 `claude -p "hello"` | 앱과 같은 사용자 계정에서 Claude Code가 회사 설정으로 응답해야 한다. 안 되면 사람이 `~/.claude/settings.json`의 회사 설정(IT 안내값) 점검. 비밀값은 AI가 출력하지 않는다. 고친 뒤 보관함 "다시 시도" |
| "ccr 설정을 찾지 못함" | `ccr status`, `~/.claude-code-router/config.json` 존재 여부 | 라우터 시작(`ccr start`) 또는 `company.env`로 직접 지정 |
| "회사 경로에서 구조화 출력 실패" | Claude Code 버전·게이트웨이 지원 | `model.company.structuredOutput: prompt`로 바꿔 재시도. 안 되면 관리자에게 Claude Code 버전 확인 |
| opencode: `cli_not_found` ("opencode를 찾을 수 없습니다") | `which opencode`, `opencode --version` | PATH에 두거나 `model.opencode.path` 지정 후 재시작 |
| opencode: 로그인/자격증명 필요 (`auth_required`) | `echo hi \| opencode run "답하세요"`, `opencode auth list` | **[사람]** `opencode auth login` 또는 사내 `opencode.json` 준비. 공급자 키 환경변수를 쓰면 이름을 `envPass`에 추가 |
| opencode: 모델을 찾을 수 없음 (`bad_request`) | `opencode models` | `model.opencode.model`을 목록의 `provider/model` 값으로 수정 |
| opencode: 응답이 비었거나 JSON 검증 실패 (`bad_output`) | 터미널에서 `echo hi \| opencode run "답하세요"` | 모델이 지시를 못 따르는 경우. 앱이 1회 재시도한다. 계속되면 다른 사내 모델로 바꿔 보거나 `timeoutMs`를 늘린다 |
| opencode: 이미지가 안 읽힘 | 이미지 단독 자료 | opencode 모드는 OCR 글자만 쓴다. 글자가 없는 이미지는 지원하지 않는다 |
| "구독 아님" 오류 | `config.yaml`의 `model:` 아래 `billing: company` 유무·들여쓰기 | 설정을 넣거나 고친 뒤 앱 재시작 (A·B) |
| 401 (로컬 서버 인증 실패) | 크롬 확장을 쓰는 경우만 해당. 확장 옵션의 토큰 | 앱 설정 화면의 "토큰 복사" 값을 확장 옵션에 다시 붙여넣기. AI는 토큰 파일을 읽어 출력하지 않는다 |
| 401 (모델 호출, 회사 쪽) | `claude -p "hello"` 또는 opencode 확인 결과 | 회사 인증 만료일 수 있음. 사람이 회사 방식으로 재인증 |
| 링크로 올리기가 사내 주소에서 실패 | `ingest.url.allowPrivateHosts` 값, 사내 프록시·인증서 | 기본은 허용이다. 로그인이 필요한 페이지는 읽지 못한다. 사설 CA는 `NODE_EXTRA_CA_CERTS`가 `~/.claude/settings.json` env에 있는지 사람이 확인 |
| 앱이 안 뜸 / 메뉴바만 / 포트 | `curl -s 127.0.0.1:4823/healthz` | 응답 없으면 `~/Library/Logs/work-inbox/` 로그 확인 후 재실행. 포트 충돌이면 `lsof -i :4823`로 점유 프로세스 확인 (다른 프로세스를 임의로 종료하지 말고 사람에게 알린다) |
| 캘린더에 안 올라감 | 시스템 설정 → 캘린더 권한 | 허용 후 앱 재시작 |
| 앱 실행이 막힘 | Gatekeeper/MDM 메시지 | ③ 참고. MDM이면 IT 문의 |

로그는 `~/Library/Logs/work-inbox/`에 있다. 로그를 사람에게 보여 줄 때도 토큰·키 줄은 가린다.

## ⑧ 되돌리기

- 회사 경로 설정만 되돌리기: 앱 종료 → `config.yaml`에서 ④에서 넣은 줄(`billing: company`와 `company:` 블록, 또는 `mode: opencode`와 `opencode:` 블록)을 지운다 (`model:` 아래에 다른 키가 없으면 `model:` 줄도 지운다). 백업이 있으면 `config.yaml.bak`으로 복원. 앱 재시작.
- 앱 삭제: 앱 종료 → `/Applications/work-inbox.app`을 휴지통으로. 로그인 항목에 남아 있으면 시스템 설정 → 일반 → 로그인 항목에서 제거.
- 데이터까지 완전 제거 (사람 확인 후): `~/Library/Application Support/work-inbox`, `~/Library/Logs/work-inbox` 삭제, 캘린더 앱에서 `work-inbox` 캘린더 삭제. vault(사용자가 쌓은 메모·할 일)는 사람이 판단해 지운다.

## 1.0.1에서 올릴 때

- 같은 `/Applications/work-inbox.app`을 새 DMG의 앱으로 교체한다(앱 종료 → 교체 → 실행). vault·`config.yaml`·토큰·캘린더는 그대로 쓴다.
- **설정 자동 이전**: 1.0.1의 `model.companyModel`·`model.companyEnv`·`model.companyRouter`가 `config.yaml`에 남아 있어도 1.1이 읽을 때 `model.company.model|env|router`로 옮겨 읽는다. 파일은 고쳐 쓰지 않으니 그대로 둬도 되고, 새로 쓸 때만 `company.*`로 쓴다. `company`와 옛 키가 둘 다 있으면 `company.*` 값이 이긴다.
- `billing: company`는 그대로 동작한다. 1.1에서 새 키(`company.settingsEnv`·`company.structuredOutput`, opencode 모드, 위 표의 키)는 모두 선택이다.
- 올린 뒤 ⑥으로 가짜 데이터 확인을 한 번 한다.

사용법은 앱 안 설정·도움말과 이 저장소 README를 참고한다.
