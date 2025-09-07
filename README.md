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
- 템플릿 기반 간단 UI

### 보안 및 설정
- CORS: `CORS_ALLOW_ORIGINS`, `CORS_ALLOW_CREDENTIALS`
- 기본 보안 헤더 추가: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`
- SSH 호스트 키 정책: DB `session_browser_config.host_key_policy`로 제어(`auto_add` | `reject`)
- 프록시 비밀번호는 API 응답에서 제외됩니다.
- 비밀정보는 가능하면 .env 대신 시크릿 매니저 사용 권장

### 데이터베이스
- 기본 SQLite 자동 생성
- 운영에서는 `DATABASE_URL`로 PostgreSQL/MySQL 등 외부 DB 사용 권장

### 최근 개선 사항(요약)
- 프록시: `host` 중복 등록 방지(유니크), 생성/수정 시 409 처리, 입력 검증 강화(호스트 정규식, `username` 빈값 금지, 비밀번호 필수). UI에서 포트 제거.
- 세션 브라우저: 동시 수집 `max_workers` 설정 추가 및 적용, 복합 인덱스(`proxy_id, collected_at`)로 조회 최적화
- 자원 사용률: 시계열 그래프(UI) 추가 및 집계 API 연동(평균/이동평균/누적평균, 자동 새로고침)
- API 응답에서 프록시 비밀번호 제거, `ProxyUpdate` 부분 업데이트 허용
- 목록 엔드포인트 페이지네이션 추가(`/api/proxies`, `/api/proxy-groups`, `/api/session-browser`, `/api/resource-usage`)
- 세션 브라우저: 호스트 키 정책(`auto_add/reject`) 준수
- CORS/보안 헤더/건강 체크(`/healthz`), `.env` 로딩 추가, 문서 노출 환경변수화(`ENABLE_DOCS`)
- DB 설정 개선: `DATABASE_URL` 지원, `pool_pre_ping` 활성화
- Pydantic 기본값 안전화: `default_factory` 적용(에러 맵 등)

### 로드맵 및 상세 개선 계획
자세한 개선 계획과 우선순위는 `docs/IMPROVEMENTS.md`를 참조하세요.

### 마이그레이션 노트
- 운영 환경에서는 `proxies.host` 유니크 인덱스 및 `session_records(proxy_id, collected_at)` 복합 인덱스를 마이그레이션 도구(Alembic 등)로 적용하세요.

### 세션 브라우저 설정
- 동시 수집 워커 수: DB 설정 `session_browser_config.max_workers`로 제어(기본 4)
- 타임아웃/SSH 포트/호스트 키 정책은 `/api/session-browser/config`로 조회/수정

### 자원 사용률 집계 API 예시
```bash
curl -X POST http://localhost:8000/api/resource-usage/series \
  -H 'Content-Type: application/json' \
  -d '{
    "proxy_ids": [1,2],
    "start": "2025-01-01T00:00:00Z",
    "end": "2025-01-01T12:00:00Z",
    "interval": "hour",
    "ma_window": 5
  }'
```

### 자원 사용률 그래프 사용법
- 페이지: `/resource`
- 대상 선택: 그룹/프록시 다중 선택 후 그래프 섹션에서 기간/버킷/이동평균 윈도우 선택
- 표시 지표: CPU/MEM/CC/CS/HTTP/HTTPS/FTP 체크박스로 선택
- 모드: 평균/이동평균/누적평균 토글
- 자동 새로고침: 체크 후 주기(초) 입력하면 현재 범위를 유지한 채 갱신

