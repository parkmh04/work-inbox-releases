# work-inbox releases

개인 업무 비서 앱 **work-inbox**의 macOS 설치 파일(DMG)만 배포하는 저장소입니다. 소스 코드는 없습니다.

## 받기
[Releases](../../releases)에서 최신 버전의 `work-inbox-<버전>-arm64.dmg`를 받습니다(Apple Silicon 전용).

```sh
gh release download --repo parkmh04/work-inbox-releases --pattern '*.dmg'
```

## 설치
1. DMG를 열어 `work-inbox.app`을 응용 프로그램 폴더로 옮깁니다.
2. 처음 열 때 "확인되지 않은 개발자" 경고가 나오면 시스템 설정 → 개인정보 보호 및 보안 → **그래도 열기**를 누릅니다. (공증되지 않은 자체 서명 앱입니다. 회사 관리 맥이 막으면 IT 부서에 문의하세요.)
3. 캘린더 권한을 묻으면 허용합니다.

## 요구 사항
- macOS (Apple Silicon)
- Claude Code CLI가 설치·로그인되어 있어야 추출이 동작합니다.
  - 개인 맥: Claude Max 구독 로그인(기본).
  - 회사 맥(회사 Bedrock/게이트웨이): `~/Library/Application Support/work-inbox/config.yaml`에
    ```yaml
    model:
      billing: company
    ```
    를 넣고 앱을 다시 시작합니다(1.0.1부터).

## 데이터
- 모든 자료는 이 맥의 로컬 폴더에만 저장됩니다.
- 모델로는 민감정보를 가린 입력 텍스트만 보냅니다.
- 남에게 보내는 동작(초대·공유·게시)은 없습니다.
- 실제 업무 자료를 넣기 전에 소속 회사의 보안 정책을 확인하세요.
