# SilentOrchestra 2.0

몸짓 뒤의 행동을 활동별로 학습하고, 사용자가 승인한 연결만 실행하는 로컬 우선 Agent입니다.

## 기술 스택

Python 3.11+ · FastAPI · SQLAlchemy · SQLite/PostgreSQL · Vanilla HTML/CSS/JS · 선택적 OpenCV/PyAutoGUI

## 시작하기

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\Activate.ps1
python -m pip install -e ./backend
python run_demo.py
```

브라우저에서 <http://127.0.0.1:8000>을 엽니다.

## 사용 방법

웹 UI에서 몸짓과 후속 행동을 반복 입력하고 제안을 승인한 뒤, 재인식 결과에 피드백을 남깁니다. 기본 실행은 `DRY_RUN`입니다.

카메라·OS 제어 설정은 [실행 설정](plan.md#실행-설정), API는 실행 서버의 [/docs](http://127.0.0.1:8000/docs)를 참고합니다.

## 테스트

```bash
python -m pip install -e "./backend[dev]"
python -m pytest backend/tests -q
node --test frontend/tests/app.test.cjs
python backend/scripts/validate_sqlite.py --schema backend/sql/schema.sql --seed backend/sql/seed.sql --queries backend/sql/queries.sql --tests backend/sql/tests.sql --report /tmp/sqlite-validation.json
```

CI는 [.github/workflows/ci.yml](.github/workflows/ci.yml)에서 관리합니다.

## 관련 문서

- [요구사항](spec.md) · [구현 계획](plan.md) · [작업 지침](AGENTS.md)
