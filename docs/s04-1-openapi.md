---
title: "실습① — REST API 선정 + Copilot으로 OpenAPI 작성"
parent: "S4. 커스텀 커넥터"
grand_parent: "📗 심화과정"
nav_order: 1
---

# 실습 ①: REST API 선정 + Copilot 으로 OpenAPI YAML 작성
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 13:00 | 15분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 이 실습의 목표

- 도구화할 **REST API 대상을 선정** 하는 기준을 익힌다
- **Copilot (또는 ChatGPT) 에게 OpenAPI 3.0 YAML 을 작성** 시킨다
- 생성된 YAML 을 **검토·다듬기** 한 뒤 다음 실습으로 가져간다

{: .important }
> 이 실습의 산출물은 **하나의 OpenAPI YAML 파일** 입니다. 강의 후 어떤 API 든 이 흐름으로 도구화할 수 있다는 것이 핵심.

---

## Step 1 — REST API 대상 선정

### 선정 체크리스트

| 항목 | 통과 기준 |
|:-----|:--------|
| 인증 | 무인증 또는 단순 API 키 (OAuth 2.0 은 강의 시간 부족) |
| 안정성 | 공식 운영 / SLA / 다운 빈도 |
| 엔드포인트 수 | 1~3 개로 시작 (단순 학습용) |
| 응답 JSON | 1~2 단계 깊이의 단순 구조 |
| 입력 변수 | 2~3 개 (학습 효과 최대) |

### 본 실습 대상 — Frankfurter 환율 API

| 항목 | 값 |
|:-----|:---|
| 베이스 URL | `https://api.frankfurter.app` |
| 엔드포인트 | `GET /latest` |
| 인증 | **없음** |
| 입력 변수 | `base`(기준 통화) · `symbols`(비교 통화, 콤마 구분) |
| 응답 예 | `{"amount":1.0,"base":"USD","date":"2026-05-09","rates":{"KRW":1352.41,"JPY":151.2}}` |

{: .note }
> Frankfurter 는 ECB(유럽중앙은행) 데이터를 무인증으로 공개하는 사이트입니다. 강의용으로 **인증·키 발급 시간이 0** 인 게 핵심 장점.

### 응용용 후보 (강의 후 자기주도 학습)

| 도메인 | 후보 |
|:------|:----|
| 공공 | data.go.kr (박스오피스 / 지하철 / 날씨) |
| 사내 | ITSM (ServiceNow / Jira) · HR · CRM |
| 외부 SaaS | 택배 추적 · 우편번호 · 주식 시세 |

---

## Step 2 — Copilot 에게 OpenAPI YAML 작성 요청

웹 Copilot · ChatGPT · Copilot Chat 등 어디서든 됩니다. 다음 프롬프트를 그대로 붙여넣으세요.

### 프롬프트 (그대로 사용)

```
나는 Copilot Studio 의 "도구 추가 → REST API" 기능에 붙여넣을 OpenAPI 3.0 YAML 이 필요해.

대상 API:
- 베이스 URL: https://api.frankfurter.app
- 엔드포인트: GET /latest
- 인증: 없음
- 쿼리 파라미터:
  * base (string, 선택, 예: USD) — 기준 통화 (ISO 4217)
  * symbols (string, 선택, 예: KRW,JPY,EUR) — 비교 통화들 (콤마 구분)
- 응답 200 (application/json):
  {
    "amount": 1.0,
    "base": "USD",
    "date": "2026-05-09",
    "rates": { "KRW": 1352.41, "JPY": 151.2 }
  }

요구사항:
1. openapi: 3.0.0 으로 시작
2. info.title 은 ExchangeRate, version 은 1.0.0
3. servers 에 위 URL 기입
4. operationId: GetLatestRate
5. summary/description 은 한국어로
6. 파라미터별 description 도 한국어로 (Copilot Studio 가 이 설명을 학습자에게 그대로 노출함)
7. responses 에는 200 만 정의 (스키마 포함)
8. 추가 코멘트 없이 YAML 본문만 출력
```

### 기대 출력 (참고)

```yaml
openapi: 3.0.0
info:
  title: ExchangeRate
  version: 1.0.0
  description: Frankfurter 무인증 환율 API. 기준 통화와 비교 통화들을 받아 최신 환율을 반환합니다.
servers:
  - url: https://api.frankfurter.app
paths:
  /latest:
    get:
      operationId: GetLatestRate
      summary: 최신 환율 조회
      description: ISO 4217 통화 코드를 받아 ECB 최신 환율을 반환합니다.
      parameters:
        - name: base
          in: query
          required: false
          description: 기준 통화 (ISO 4217, 예 USD)
          schema:
            type: string
            example: USD
        - name: symbols
          in: query
          required: false
          description: 비교 통화들 (콤마 구분, 예 KRW,JPY,EUR)
          schema:
            type: string
            example: KRW
      responses:
        "200":
          description: 환율 응답
          content:
            application/json:
              schema:
                type: object
                properties:
                  amount: { type: number }
                  base:   { type: string }
                  date:   { type: string }
                  rates:
                    type: object
                    additionalProperties: { type: number }
```

{: .tip }
> 출력이 위와 다소 달라도 괜찮습니다. 필수: ① `openapi: 3.0.0` ② `servers.url` ③ `paths./latest.get` ④ `operationId` ⑤ `parameters` 2개 ⑥ `responses.200`. 이 6가지만 있으면 다음 실습이 통과합니다.

---

## Step 3 — YAML 검토·다듬기 (3가지 점검)

Copilot 출력은 그대로 쓰지 말고 짧게 점검하세요.

### (A) 한국어 설명이 들어 있는가

`description` 필드는 Copilot Studio 가 도구 등록 시 **그대로 학습자에게 노출** 합니다. 영어로 나왔다면 한국어로 바꿔주세요.

| 위치 | 한국어로 바꿀 텍스트 |
|:-----|:-------------------|
| `info.description` | "Frankfurter 무인증 환율 API. …" |
| `paths./latest.get.summary` | "최신 환율 조회" |
| `paths./latest.get.description` | "ISO 4217 통화 코드를 받아 …" |
| 각 `parameter.description` | "기준 통화 (ISO 4217, 예 USD)" 등 |

### (B) `operationId` 가 영문 PascalCase 인가

Copilot Studio 는 `operationId` 를 **도구 함수명** 으로 그대로 사용합니다. 한국어/공백/하이픈은 호환성 이슈가 생깁니다.

- ✅ `GetLatestRate`
- ❌ `get-latest-rate`, `최신환율`, `Get Latest Rate`

### (C) 인증 블록이 없는가 (이번 API 는 무인증)

`securitySchemes` · `security` 가 들어 있다면 삭제. 무인증 API 에 인증 블록이 있으면 Studio 등록 단계에서 추가 입력을 요구합니다.

---

## Step 4 — 산출물 저장

YAML 을 클립보드에 복사해 두거나 `exchange-rate.openapi.yaml` 로 저장합니다. 다음 실습에서 **Copilot Studio 의 "도구 추가 → REST API"** 입력란에 그대로 붙여넣습니다.

---

## 자주 나오는 문제

| 증상 | 원인 | 대응 |
|:-----|:-----|:-----|
| YAML 들여쓰기 깨짐 | Copilot 출력에 마크다운 ``` 가 같이 복사됨 | 코드블록 표시 ` ``` ` 만 제거 |
| `operationId` 가 한글/공백 | 프롬프트에 명시 안 함 | 프롬프트에 `operationId: GetLatestRate` 못박기 |
| 응답 스키마가 너무 추상 (`type: object` 만) | 모델이 게으름 | "rates 의 additionalProperties 까지 명시" 라고 추가 요청 |
| 인증 블록 자동 추가 | 모델이 보수적 | 프롬프트에 "인증: 없음" 강조, 후처리로 제거 |

---

## 체크리스트

- [ ] 대상 API 선정 (이번엔 Frankfurter)
- [ ] Copilot 으로 OpenAPI 3.0 YAML 생성
- [ ] `openapi`, `servers`, `paths`, `operationId`, `parameters` × 2, `responses.200` 6가지 확인
- [ ] 한국어 description 채워짐
- [ ] `operationId` = `GetLatestRate` (영문 PascalCase)
- [ ] 인증 블록 없음
- [ ] YAML 클립보드 또는 파일로 보관

---

## 다음 단계

[실습 ② — Studio 에서 도구 등록 + 인터넷 검색 끄기 + 테스트](s04-2-connector-call) 에서 이 YAML 을 Copilot Studio 의 **도구 추가 → REST API** 에 붙여넣고, 한 번에 도구화 → 입력 변수 등록 → 인터넷 검색 차단 → 테스트까지 진행합니다.
