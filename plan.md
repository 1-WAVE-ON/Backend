# PLAN — SilentOrchestra 2.0

[요구사항](spec.md)을 구현하기 위한 구조·실행 설정·검증 방법입니다.

## 구현 구조

백엔드는 `backend/src/silent_orchestra/`의 `routers/`·`services/` 두 계층, 프런트는 빌드 없는 `frontend/` HTML/CSS/JS입니다.

| 책임 | 백엔드 경로 |
|---|---|
| API·맥락 | `routers/agent.py`, `services/context_resolver.py` |
| 학습·승인 | `services/pattern_learning.py`, `services/confirmation.py` |
| 특징·추론 | `services/gesture_encoder.py`, `services/intent_reasoner.py` |
| 실행·피드백 | `services/action_executor.py`, `services/feedback_service.py` |
| 실제 키 관측 | `services/input_observer.py` |
| 초기화 | `routers/demo.py`, `services/demo_service.py` |
| 데이터·설정 | `models.py`, `schemas.py`, `database.py`, `config.py` |

- 카메라: `frontend/app.js`, `backend/scripts/webcam_gesture_client.py`.
- DB: 기본 SQLite, 배포 시 `SO_DATABASE_URL`로 PostgreSQL 지정. Vercel 진입점은 루트 `main.py`.
- 데이터: 사용자·맥락·관찰·행동·패턴·제안·실행·피드백. 컬럼·제약은 [schema.sql](backend/sql/schema.sql), API 스키마는 실행 서버의 `/docs` 참조.
- `space`·`device`는 메타데이터이며 `activity`가 학습 단위. `speed`·`amplitude`는 함께 NULL이거나 함께 값이 있어야 합니다.

### UI 규칙

값은 [tokens.css](frontend/tokens.css)에서 관리합니다.

- 데스크톱은 맥락·작업·기억 3열, 모바일은 작업 우선. 단일 레이어·얇은 구분선을 사용합니다.
- 장식 구체·동심원·그라데이션·홍보 문구는 제외하고 Orb는 작은 상태 표시로 제한합니다.
- cyan은 주 동작(화면 5% 미만), violet은 학습·제안, error는 오류에 사용하며 색만으로 상태를 전달하지 않습니다.
- 제목은 IBM Plex Sans KR 700, 본문은 Pretendard Variable 400–600, Mono는 wordmark·실시간 지표에만 사용합니다.
- 모션은 버튼 press·상태 crossfade만 사용하고 reduced motion은 opacity 120ms 이하로 제한합니다. 포커스 표시·44px 이상 터치 영역을 유지합니다.
- 주 버튼은 solid cyan·한국어 동사, 보조 버튼은 어두운 표면·선, 삭제 버튼은 error 텍스트만 사용합니다.
- 성공 토스트는 생략하고 오류·화면 밖 비동기 결과는 고정 안내합니다.

## 실행 설정

기본 설치·실행은 [README](README.md#시작하기)를 따릅니다.

### 웹캠

```bash
python -m pip install -e "./backend[camera]"
# 서버 실행 후 별도 터미널에서 Windows 실제 키 관측
python backend/scripts/webcam_gesture_client.py --learn
# 첫 프레임 점검
python backend/scripts/webcam_gesture_client.py --check-camera --camera 0
# Windows 관측을 사용할 수 없는 경우 명시적 라벨 입력
python backend/scripts/webcam_gesture_client.py --input-mode labels --activity presentation --learn
```

- 기본 옵션: `--input-mode observe --activity auto`. 원래 키 입력은 차단하지 않으며 수정키 조합은 관측에서 제외합니다.
- PowerPoint Slide Show: Right/PageDown/N/Space는 다음, Left/PageUp/P는 이전, Escape는 종료. 편집 화면은 제외합니다.
- Spotify/VLC/iTunes/Music: Media Next/Previous/PlayPause만 관측하며 일반 Space는 제외합니다.
- 라벨 모드: N/B는 다음·이전, music의 Space는 재생·일시정지. 실제 앱 조작은 관측·수행하지 않습니다.
- 장치 진단: `--camera INDEX`, `--camera-backend dshow|msmf|default`.

### OS 실행

```bash
python -m pip install pyautogui
export SO_ENABLE_OS_ACTIONS=true  # PowerShell: $env:SO_ENABLE_OS_ACTIONS = "true"
python run_demo.py
```

설정은 서버 시작 시 읽습니다. `SO_REQUIRE_ACTIVE_WINDOW=true`가 기본입니다. macOS는 접근성 권한이 필요하며 Windows는 활성 창 제목으로 판정합니다. Linux는 활성 창 확인을 지원하지 않습니다. OS 실행을 끄려면 환경 변수를 false로 바꾸고 서버를 재시작합니다.

## 검증 전략

자동 검증 명령은 [README 테스트](README.md#테스트)를 따릅니다.

| 대상 | 확인 항목 |
|---|---|
| API·SQL | 미정의 영상 필드 거부, `frame_stored=0`, 이미지/BLOB 컬럼 부재, 모션 특징 저장 계약 |
| 학습·실행 | 승인 전 실행 금지, 맥락 분기, 활성 창 차단, SPEC의 임계값·피드백 규칙 |
| 초기화 | 7개 종속 테이블 삭제 건수, 다른 사용자 보존, 재생성·commit 실패 롤백, 초기화 후 3회 학습·재제안 |
| UI | 3초 갱신·요청 병합, 편집 보존, 오류 표시·복구, 외부 학습 진행률, 실행 결과·피드백, dialog 비차단, Reset 복구 |
| 카메라 | 방향·특징·payload·맥락·키 연결, 실패 시 자원 해제, 정지 제외, 손바닥 재무장, 완전한 원형 궤적 |

## 남은 과제

실제 장비에서 별도로 확인합니다.

- Python·브라우저 카메라의 네 몸짓 인식과 성공·오탐·미탐 기록.
- 브라우저 장치 전환, 중지·페이지 종료 시 MediaStream 해제.
- 네트워크 요청 캡처에서 이미지·원본 프레임 전송 0건 확인. 정책 API·UI 문구로 대체하지 않습니다.
- Windows 실제 키 관측 → 제안 → 승인 → 재인식 E2E.
- OS 실행을 사용할 PC의 권한·활성 창·키 매핑 확인.
