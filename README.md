## PPAT (Proxy Performance Analysis Tool)

FastAPI 기반의 프록시 성능 분석/운영 도구입니다. 프록시/그룹 관리, 세션 브라우저 수집(SSH), 자원 사용률 수집(SNMP), 간단한 UI와 API 문서를 제공합니다.

### 구성 요소
- FastAPI + SQLAlchemy + Pydantic v2
- Templates(Jinja2) + Static assets
- SQLite 기본(DB_URL로 교체 가능)
- SNMP 수집(aiosnmp), SSH 수집(paramiko)

### 빠른 시작
1) Python 3.10+ 준비, 가상환경 권장
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

2) 환경변수(.env 권장) 설정
- `DATABASE_URL`: 기본 `sqlite:///./ppat.db`
- `CORS_ALLOW_ORIGINS`: 기본 `*` (쉼표로 다중 허용)
- `CORS_ALLOW_CREDENTIALS`: 기본 `false` (와일드카드일 때 자동 비활성)
- `ENABLE_DOCS`: 기본 `true` (API 문서 노출)

3) 애플리케이션 실행
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

4) UI/엔드포인트
- 메인 설정 페이지: `/`
- 자원 사용률 페이지: `/resource`
- 세션 브라우저 페이지: `/session`
- API 라우트 프리픽스: `/api`
- API 문서: `ENABLE_DOCS=true` 상태에서 자동 제공

### 주요 기능
- 프록시/그룹 CRUD
- 세션 브라우저 수집(SSH) 및 파싱 저장
- 자원 사용률 수집(SNMP) 및 저장/조회
