# Voltix 심볼·자재 기준표 초안

- 문서명: MVP 심볼·자재 기준표 초안
- 목적: 전기도면 심볼 자동 분류와 자재 매핑의 초기 기준 정의
- 작성일: 2026-07-09
- 버전: v0.1 Draft

---

## 1. 기준 정의 목적

본 문서는 Voltix MVP에서 우선 인식할 전기 심볼과 실제 자재 품목의 매핑 기준을 정의한다. 초기 기준은 공동주택·오피스텔 세대 전기 평면도에서 반복적으로 등장하는 배선기구와 통신 관련 자재를 중심으로 한다.

이 기준표는 다음 기능의 기반 자료로 사용한다.

1. CAD 블록·레이어 기반 심볼 자동 분류
2. 심볼-자재 매핑 규칙 생성
3. 수량산출 결과 집계 기준
4. 검수 화면 필터와 색상 구분
5. 자재코드 초기 데이터 구성

---

## 2. MVP 심볼 분류

| 대분류 | 심볼 유형 | 예시 표기 | MVP 포함 | 비고 |
|---|---|---|---|---|
| 스위치 | 일반 스위치 | SW, S, 1SW | Y | 구 수 구분은 후속 세분화 |
| 스위치 | 2구 스위치 | 2SW, SW2, 2S | Y | 블록명 또는 속성 기준 |
| 스위치 | 3구 스위치 | 3SW, SW3, 3S | Y | 블록명 또는 속성 기준 |
| 스위치 | 조광 스위치 | DIM, DSW | 보류 | 샘플 도면 확인 후 결정 |
| 콘센트 | 일반 콘센트 | CON, C, OUTLET | Y | 가장 우선 검증 |
| 콘센트 | 접지 콘센트 | E-CON, GCON | Y | 표기 차이 확인 필요 |
| 콘센트 | 전용 콘센트 | AC, REF, WM | Y | 에어컨, 냉장고, 세탁기 등 |
| 콘센트 | 방우 콘센트 | WP, WPCON | 보류 | 외부/발코니 도면 확인 필요 |
| 통신 | LAN 인출구 | LAN, DATA | Y | 통신 레이어와 함께 판단 |
| 통신 | 전화 인출구 | TEL, PHONE | Y | 요즘 도면에서 빈도 확인 필요 |
| 통신 | TV 수구 | TV, CATV | Y | 통합수구와 구분 필요 |
| 통신 | 통합수구 | MULTI, 통합, M.OUT | Y | 현장 표기 차이가 클 수 있음 |
| 분전 | 세대분전반 | PANEL, 분전반, DB | Y | 수량은 적지만 필수 |
| 기타 | 조명기구 | LIGHT, LGT | 후순위 | MVP 배선기구 이후 |
| 기타 | 전열기기 | HEATER, FAN | 후순위 | 품목 범위 확정 필요 |

---

## 3. 초기 자동 분류 규칙

### 3.1 판단 우선순위

1. 사용자가 저장한 매핑 규칙과 `block_name`이 일치하는지 확인한다.
2. `layer_name`에 포함된 전기·통신 키워드를 확인한다.
3. `block_name`에 포함된 심볼 키워드를 확인한다.
4. CAD 속성값에 품목명 또는 기호가 있는지 확인한다.
5. 복수 후보가 있으면 신뢰도 점수로 정렬한다.
6. 신뢰도가 낮으면 `ReviewRequired` 또는 `Unmapped` 상태로 둔다.

### 3.2 키워드 초안

| 키워드 | 우선 분류 | 보조 조건 | 예상 상태 |
|---|---|---|---|
| SW | 스위치 | 전등, 조명, LIGHT 레이어 | Classified |
| 1SW | 1구 스위치 | 스위치 계열 레이어 | Classified |
| 2SW | 2구 스위치 | 스위치 계열 레이어 | Classified |
| 3SW | 3구 스위치 | 스위치 계열 레이어 | Classified |
| CON | 콘센트 | 전열, POWER 레이어 | Classified |
| OUTLET | 콘센트 | 전열, POWER 레이어 | Classified |
| LAN | LAN 인출구 | 통신, DATA 레이어 | Classified |
| TEL | 전화 인출구 | 통신, TEL 레이어 | Classified |
| TV | TV 수구 | 통신, CATV 레이어 | Classified |
| PANEL | 세대분전반 | 분전, 전력 레이어 | ReviewRequired |
| DB | 세대분전반 | 분전, 전력 레이어 | ReviewRequired |
| MULTI | 통합수구 | 통신 레이어 | ReviewRequired |

### 3.3 신뢰도 기준

| 점수 구간 | 상태 | 처리 |
|---|---|---|
| 0.85 이상 | Classified | 자동 분류 후 수량산출 후보 반영 |
| 0.60~0.84 | ReviewRequired | 검수 화면에서 우선 표시 |
| 0.60 미만 | Unmapped | 수량산출 제외, 사용자가 매핑 필요 |

### 3.4 점수 산정 초안

```text
confidence_score =
  block_name_match_score
  + layer_name_match_score
  + attribute_match_score
  + saved_mapping_score
```

| 항목 | 권장 점수 |
|---|---:|
| 저장된 매핑 규칙 일치 | 0.50 |
| block_name 키워드 일치 | 0.30 |
| layer_name 키워드 일치 | 0.20 |
| attributes 품목 키워드 일치 | 0.20 |
| 복수 후보 충돌 | -0.20 |
| 금지 레이어 또는 비전기 레이어 | -0.30 |

---

## 4. 자재 품목 기준

### 4.1 자재 기본 필드

| 필드 | 설명 | 필수 |
|---|---|---|
| category | 자재 분류 | Y |
| material_name | 품명 | Y |
| specification | 규격 | N |
| brand | 제조사 | N |
| model_code | 품번 | N |
| unit | 단위 | Y |
| description | 설명 | N |
| is_active | 사용 여부 | Y |

### 4.2 초기 자재 품목 예시

| category | material_name | specification | unit | MVP 여부 |
|---|---|---|---|---|
| 스위치 | 1구 스위치 | 매입형 | EA | Y |
| 스위치 | 2구 스위치 | 매입형 | EA | Y |
| 스위치 | 3구 스위치 | 매입형 | EA | Y |
| 콘센트 | 2구 콘센트 | 접지형 | EA | Y |
| 콘센트 | 2구 방우 콘센트 | 방우형 | EA | 보류 |
| 콘센트 | 에어컨 전용 콘센트 | 전용회로 | EA | Y |
| 통신 | LAN 인출구 | 1구 | EA | Y |
| 통신 | TV 수구 | 1구 | EA | Y |
| 통신 | 전화 인출구 | 1구 | EA | Y |
| 통신 | 통합수구 | LAN/TV/TEL 통합 | EA | Y |
| 분전반 | 세대분전반 | 세대용 | EA | Y |

---

## 5. 심볼-자재 매핑 초안

| symbol_type | 예상 block_name | 예상 layer_name | 기본 material_name | 검수 필요 |
|---|---|---|---|---|
| SWITCH_1 | 1SW, SW1 | 전등, LIGHT | 1구 스위치 | N |
| SWITCH_2 | 2SW, SW2 | 전등, LIGHT | 2구 스위치 | N |
| SWITCH_3 | 3SW, SW3 | 전등, LIGHT | 3구 스위치 | N |
| OUTLET_NORMAL | CON, OUTLET | 전열, POWER | 2구 콘센트 | N |
| OUTLET_DEDICATED | AC, REF, WM | 전열, POWER | 전용 콘센트 | Y |
| LAN_OUTLET | LAN, DATA | 통신, DATA | LAN 인출구 | N |
| TEL_OUTLET | TEL, PHONE | 통신, TEL | 전화 인출구 | N |
| TV_OUTLET | TV, CATV | 통신, CATV | TV 수구 | N |
| MULTI_OUTLET | MULTI, M.OUT | 통신 | 통합수구 | Y |
| UNIT_PANEL | PANEL, DB | 분전, POWER | 세대분전반 | Y |

---

## 6. 수량 집계 기준

| 집계 단위 | 설명 | 예시 |
|---|---|---|
| 도면별 | 단일 도면 내 자재별 수량 | 84A 전기 평면도 내 콘센트 28EA |
| 세대 타입별 | 세대 타입 기준 자재별 수량 | 84A 타입 1세대 기준 |
| 층별 | 층 정보가 있는 경우 층별 집계 | 10층 콘센트 280EA |
| 프로젝트별 | 세대 타입별 수량에 세대 수를 곱해 합산 | 84A 100세대 × 28EA |
| 견적 기준 | 검수 완료 후 확정 수량 | 수량표 확정 후 견적 반영 |

---

## 7. 중복 심볼 판단 기준

중복 의심 심볼은 자동 삭제하지 않고 검수 대상으로 표시한다.

| 조건 | 설명 | 처리 |
|---|---|---|
| 동일 material_id | 같은 자재로 매핑됨 | 중복 판단 후보 |
| 동일 block_name | 동일 CAD 블록명 | 중복 판단 후보 |
| 근접 좌표 | 좌표 거리 기준값 이하 | 검수필요 표시 |
| 동일 layer_name | 같은 레이어 | 보조 판단 |
| 같은 attributes | 속성값 동일 | 보조 판단 |

초기 처리:

```text
if material_id same and block_name same and distance < threshold:
    status = ReviewRequired
    duplicate_candidate = true
```

---

## 8. 검수 상태 기준

| 상태 | 의미 | 수량 반영 |
|---|---|---|
| Detected | CAD 분석으로 감지됨 | N |
| Classified | 자동 분류됨 | 조건부 |
| Unmapped | 자재 매핑 필요 | N |
| Mapped | 자재 매핑 완료 | Y |
| ReviewRequired | 사용자 검수 필요 | 조건부 |
| Verified | 검수 완료 | Y |
| Excluded | 산출 제외 | N |

---

## 9. 확정 전 확인할 질문

1. 현장에서 주로 쓰는 스위치·콘센트 블록명 규칙은 무엇인가?
2. 통합수구와 개별 LAN/TV/TEL 수구를 어떤 기준으로 구분하는가?
3. 전용 콘센트는 블록명, 레이어명, 텍스트 중 무엇으로 구분되는가?
4. 세대분전반은 수량산출과 견적에서 어떤 규격으로 분류하는가?
5. 세대 타입별 총 수량 산출 시 적용 세대 수는 어디에서 입력받는가?
6. 같은 좌표 또는 근접 좌표의 심볼은 실제 중복인가, 복합 표기인가?

---

## 10. 다음 작업

1. 실제 샘플 도면에서 블록명과 레이어명을 추출한다.
2. 본 기준표의 키워드와 실제 도면 키워드를 비교한다.
3. 자동 분류 가능한 심볼과 검수필요 심볼을 구분한다.
4. 자재 품목표와 제조사 품번표를 연결한다.
5. 초기 매핑 규칙을 DB seed 데이터로 전환한다.
