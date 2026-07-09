# Voltix Sprint 1 백로그 초안

- 문서명: Sprint 1 개발 백로그 초안
- Sprint 목표: 회원/권한, 프로젝트 생성, 도면 업로드, 파일 저장, 분석 작업 상태 기반 구축
- 작성일: 2026-07-09
- 버전: v0.1 Draft

---

## 1. Sprint 1 목표

Sprint 1의 목표는 사용자가 로그인 후 프로젝트를 생성하고, 프로젝트에 도면 파일을 업로드하며, 업로드된 도면의 메타데이터와 분석 작업 상태를 관리할 수 있는 기반을 만드는 것이다.

CAD 파싱, 심볼 분류, 수량산출은 Sprint 2 이후 범위로 두고, Sprint 1에서는 “도면 분석 작업을 등록할 수 있는 상태”까지 구현한다.

---

## 2. 포함 범위

| 영역 | 포함 기능 |
|---|---|
| 인증 | 회원가입, 로그인, 로그아웃, 토큰 갱신 |
| 권한 | 기본 역할 Admin, Manager, Estimator, Viewer |
| 프로젝트 | 프로젝트 생성, 목록 조회, 상세 조회, 수정 |
| 도면 | DWG/DXF/PDF 업로드, 파일 검증, 메타데이터 저장 |
| 파일 저장 | 원본 파일 저장 경로 생성, 파일 메타데이터 관리 |
| 분석 작업 | 분석 Job 레코드 생성, queued 상태 등록 |
| 감사 로그 | 도면 업로드, 프로젝트 생성 이벤트 기록 |

---

## 3. 제외 범위

| 제외 항목 | 후속 Sprint |
|---|---|
| CAD Parser 실제 연동 | Sprint 2 |
| 블록·레이어·좌표 추출 | Sprint 2 |
| 심볼 자동 분류 | Sprint 3 |
| 심볼-자재 매핑 | Sprint 3 |
| 수량 자동 산출 | Sprint 3 |
| 도면 뷰어 검수 UI | Sprint 4 |
| 자재코드·단가표 | Sprint 5 |
| 견적서·발주 리스트 | Sprint 6 |

---

## 4. 사용자 스토리

### US-001. 회원가입

사용자는 이메일, 비밀번호, 이름, 회사명을 입력해 계정을 생성할 수 있다.

수용 기준:

1. 이메일 형식을 검증한다.
2. 중복 이메일은 가입을 차단한다.
3. 비밀번호는 해시로 저장한다.
4. 기본 권한은 `Estimator` 또는 `Viewer`로 생성한다.
5. 가입 성공 시 사용자 ID를 반환한다.

### US-002. 로그인

사용자는 이메일과 비밀번호로 로그인할 수 있다.

수용 기준:

1. 올바른 계정이면 access token과 refresh token을 발급한다.
2. 비밀번호가 틀리면 인증 실패 메시지를 반환한다.
3. 비활성 계정은 로그인할 수 없다.
4. refresh token으로 access token을 갱신할 수 있다.

### US-003. 프로젝트 생성

사용자는 현장 또는 견적 단위의 프로젝트를 생성할 수 있다.

수용 기준:

1. `project_name`, `site_name`, `building_type`은 필수다.
2. 생성자는 프로젝트 소유자로 등록된다.
3. 프로젝트 상태는 `Draft`로 시작한다.
4. 생성 시 감사 로그를 남긴다.

### US-004. 프로젝트 목록 조회

사용자는 권한이 있는 프로젝트 목록을 조회할 수 있다.

수용 기준:

1. 사용자는 본인이 속한 조직 또는 권한이 있는 프로젝트만 볼 수 있다.
2. 프로젝트명, 현장명, 고객사명으로 검색할 수 있다.
3. 상태별 필터를 적용할 수 있다.
4. 최근 생성일 기준 정렬을 기본값으로 한다.

### US-005. 도면 업로드

사용자는 프로젝트에 DWG, DXF, PDF 파일을 업로드할 수 있다.

수용 기준:

1. 허용 확장자는 `dwg`, `dxf`, `pdf`다.
2. MVP 기본 용량 제한은 50MB다.
3. 업로드 원본 파일은 프로젝트별 저장 경로에 저장한다.
4. 도면 메타데이터를 DB에 저장한다.
5. 도면 업로드 후 분석 Job을 `queued` 상태로 생성한다.
6. 프로젝트 상태를 `Uploaded`로 갱신한다.

### US-006. 도면 목록 및 상세 조회

사용자는 프로젝트에 등록된 도면 목록과 도면 상세 정보를 조회할 수 있다.

수용 기준:

1. 도면명, 버전, 파일 유형, 업로드일, 분석 상태를 표시한다.
2. 분석 상태는 `queued`, `processing`, `completed`, `failed` 중 하나다.
3. 업로드된 원본 파일 경로는 직접 노출하지 않는다.
4. 다운로드가 필요한 경우 서버 API 또는 서명 URL을 사용한다.

### US-007. 분석 작업 상태 관리

시스템은 도면 업로드 후 분석 작업 상태를 관리할 수 있다.

수용 기준:

1. 도면 업로드 시 `drawing_parse_job`을 생성한다.
2. 초기 상태는 `queued`다.
3. 작업 상태 변경 이력을 저장할 수 있다.
4. 실패 상태에는 실패 코드와 메시지를 저장할 수 있다.

---

## 5. API 초안

### 5.1 인증

```http
POST /api/auth/signup
POST /api/auth/login
POST /api/auth/logout
POST /api/auth/refresh
GET  /api/auth/me
```

### 5.2 프로젝트

```http
GET    /api/projects
POST   /api/projects
GET    /api/projects/{projectId}
PATCH  /api/projects/{projectId}
DELETE /api/projects/{projectId}
```

### 5.3 도면

```http
GET  /api/projects/{projectId}/drawings
POST /api/projects/{projectId}/drawings
GET  /api/drawings/{drawingId}
GET  /api/drawings/{drawingId}/analysis-status
```

### 5.4 작업

```http
GET   /api/jobs/{jobId}
PATCH /api/jobs/{jobId}/status
```

---

## 6. DB 테이블 초안

### 6.1 users

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | 사용자 ID |
| email | varchar | 이메일 |
| password_hash | varchar | 비밀번호 해시 |
| name | varchar | 이름 |
| company_name | varchar | 회사명 |
| role | varchar | 기본 권한 |
| is_active | boolean | 활성 여부 |
| created_at | timestamp | 생성일 |
| updated_at | timestamp | 수정일 |

### 6.2 projects

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | 프로젝트 ID |
| project_name | varchar | 프로젝트명 |
| client_name | varchar | 고객사명 |
| site_name | varchar | 현장명 |
| building_type | varchar | 건물 유형 |
| description | text | 설명 |
| status | varchar | 프로젝트 상태 |
| created_by | uuid | 생성 사용자 |
| created_at | timestamp | 생성일 |
| updated_at | timestamp | 수정일 |

### 6.3 drawings

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | 도면 ID |
| project_id | uuid | 프로젝트 ID |
| drawing_name | varchar | 도면명 |
| drawing_version | varchar | 도면 버전 |
| file_type | varchar | DWG/DXF/PDF |
| file_size | bigint | 파일 크기 |
| file_path | varchar | 원본 파일 저장 경로 |
| floor_info | varchar | 층 정보 |
| unit_type | varchar | 세대 타입 |
| analysis_status | varchar | 분석 상태 |
| uploaded_by | uuid | 업로드 사용자 |
| uploaded_at | timestamp | 업로드일 |

### 6.4 analysis_jobs

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | 작업 ID |
| drawing_id | uuid | 도면 ID |
| job_type | varchar | drawing_parse_job 등 |
| status | varchar | queued/processing/completed/failed |
| error_code | varchar | 실패 코드 |
| error_message | text | 실패 메시지 |
| started_at | timestamp | 시작일 |
| completed_at | timestamp | 완료일 |
| created_at | timestamp | 생성일 |

### 6.5 audit_logs

| 컬럼 | 타입 | 설명 |
|---|---|---|
| id | uuid | 로그 ID |
| user_id | uuid | 사용자 ID |
| entity_type | varchar | 대상 유형 |
| entity_id | uuid | 대상 ID |
| action | varchar | 작업 |
| before_data | jsonb | 변경 전 |
| after_data | jsonb | 변경 후 |
| created_at | timestamp | 생성일 |

---

## 7. 화면 초안

### 7.1 로그인 화면

필수 요소:

1. 이메일 입력
2. 비밀번호 입력
3. 로그인 버튼
4. 회원가입 이동
5. 오류 메시지 표시

### 7.2 프로젝트 목록 화면

필수 요소:

1. 프로젝트 생성 버튼
2. 검색 입력
3. 상태 필터
4. 프로젝트 목록 테이블
5. 프로젝트 상태 배지
6. 최근 업로드 도면 수

### 7.3 프로젝트 상세 화면

필수 요소:

1. 프로젝트 기본 정보
2. 도면 업로드 버튼
3. 도면 목록
4. 분석 상태 표시
5. 도면 상세 이동

### 7.4 도면 업로드 화면

필수 요소:

1. 파일 선택 또는 드래그 업로드
2. 도면명 입력
3. 도면 버전 입력
4. 층 정보 입력
5. 세대 타입 입력
6. 업로드 진행 상태
7. 업로드 실패 사유 표시

---

## 8. QA 시나리오

| 테스트 ID | 시나리오 | 기대 결과 |
|---|---|---|
| S1-QA-001 | 정상 이메일로 회원가입 | 사용자 생성 |
| S1-QA-002 | 중복 이메일 회원가입 | 가입 차단 |
| S1-QA-003 | 정상 계정 로그인 | 토큰 발급 |
| S1-QA-004 | 잘못된 비밀번호 로그인 | 인증 실패 |
| S1-QA-005 | 프로젝트 필수값 누락 | 생성 차단 |
| S1-QA-006 | 프로젝트 정상 생성 | Draft 상태 생성 |
| S1-QA-007 | DWG 파일 업로드 | 파일 저장, Drawing 생성, Job queued |
| S1-QA-008 | 허용되지 않은 파일 업로드 | 업로드 차단 |
| S1-QA-009 | 50MB 초과 파일 업로드 | 업로드 차단 |
| S1-QA-010 | 권한 없는 프로젝트 접근 | 접근 차단 |
| S1-QA-011 | 도면 상세 조회 | 메타데이터와 분석 상태 표시 |
| S1-QA-012 | 분석 Job 상태 조회 | queued 상태 반환 |

---

## 9. 오류 코드 초안

| 오류 코드 | 상황 | 처리 |
|---|---|---|
| AUTH_INVALID_CREDENTIALS | 로그인 실패 | 인증 실패 메시지 |
| AUTH_EMAIL_DUPLICATED | 이메일 중복 | 가입 차단 |
| PROJECT_REQUIRED_FIELD_MISSING | 프로젝트 필수값 누락 | 누락 필드 표시 |
| PROJECT_PERMISSION_DENIED | 프로젝트 접근 권한 없음 | 접근 차단 |
| FILE_INVALID_TYPE | 허용되지 않은 파일 형식 | 업로드 차단 |
| FILE_TOO_LARGE | 파일 용량 초과 | 업로드 차단 |
| FILE_UPLOAD_FAILED | 파일 저장 실패 | 재시도 안내 |
| JOB_NOT_FOUND | 작업 없음 | 조회 실패 |

---

## 10. Sprint 1 완료 기준

1. 사용자는 회원가입과 로그인을 할 수 있다.
2. 사용자는 프로젝트를 생성하고 목록을 조회할 수 있다.
3. 사용자는 프로젝트 상세에서 도면을 업로드할 수 있다.
4. 시스템은 도면 파일 형식과 용량을 검증한다.
5. 시스템은 원본 파일과 도면 메타데이터를 저장한다.
6. 시스템은 도면 업로드 후 분석 Job을 `queued` 상태로 생성한다.
7. 사용자는 도면 목록과 분석 상태를 조회할 수 있다.
8. 주요 이벤트는 감사 로그에 기록된다.
9. 권한 없는 사용자는 프로젝트와 도면에 접근할 수 없다.
10. Sprint 1 QA 시나리오가 통과한다.

---

## 11. 개발 착수 전 확인 필요

1. Backend를 NestJS로 할지 FastAPI로 할지 결정한다.
2. Frontend를 Next.js로 확정할지 결정한다.
3. 파일 저장소를 로컬 디스크, S3 호환 스토리지 중 어디로 둘지 결정한다.
4. 사용자 조직 또는 회사 단위 테이블을 Sprint 1에 포함할지 결정한다.
5. 도면 업로드 용량 제한 50MB가 적절한지 확인한다.
6. PDF 업로드를 Sprint 1에서 허용하되 분석은 후순위로 둘지 결정한다.

---

## 12. 다음 Sprint 연결

Sprint 1 완료 후 Sprint 2에서는 다음 기능을 진행한다.

1. 업로드된 DWG/DXF 파일을 분석 Worker로 전달한다.
2. DWG를 DXF로 변환한다.
3. DXF에서 블록, 레이어, 좌표, 속성을 추출한다.
4. 추출 결과를 `symbols` 테이블에 저장한다.
5. 심볼 목록 화면을 만든다.
6. 분석 실패 로그와 재시도 정책을 구현한다.
