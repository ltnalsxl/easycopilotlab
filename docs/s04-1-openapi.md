---
title: "실습① — OpenAPI로 커스텀 커넥터 등록"
parent: "S4. 커스텀 커넥터"
grand_parent: "📗 심화과정"
nav_order: 1
---

# 실습 ①: OpenAPI로 커스텀 커넥터 등록
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 13:00 | 18분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 이 실습의 목표

- 사전 제공된 **OpenAPI 정의(YAML)** 를 가져와
- Power Platform에 **커스텀 커넥터를 생성**하고
- 커넥터의 **테스트 탭에서 단독 호출**까지 확인한다

---

## Step 1 — OpenAPI 정의 확인

강의 자료에서 `exchange-rate.openapi.yaml`을 받습니다. 핵심 내용은 다음과 같습니다.

```yaml
openapi: 3.0.0
info:
  title: ExchangeRate
  version: 1.0.0
servers:
  - url: https://api.frankfurter.app
paths:
  /latest:
    get:
      summary: 최신 환율 조회
      operationId: GetLatestRate
      parameters:
        - name: from
          in: query
          required: true
          schema: { type: string, example: USD }
        - name: to
          in: query
          required: true
          schema: { type: string, example: KRW }
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                type: object
                properties:
                  amount: { type: number }
                  base:   { type: string }
                  date:   { type: string }
                  rates:  { type: object, additionalProperties: { type: number } }
```

{: .note }
> Frankfurter API는 **무인증·공개·안정**의 조합이라 강의용으로 적합합니다.

---

## Step 2 — Power Apps에서 커스텀 커넥터 생성

[https://make.powerapps.com](https://make.powerapps.com) → 좌측 **데이터 → 사용자 지정 커넥터 → + 새 사용자 지정 커넥터 → OpenAPI 파일 가져오기**

| 항목 | 값 |
|:-----|:---|
| 커넥터 이름 | `ExchangeRate` |
| OpenAPI 파일 | `exchange-rate.openapi.yaml` |

업로드 후 **계속**.

---

## Step 3 — 일반 / 보안 탭

### 일반

| 항목 | 값 |
|:-----|:---|
| 호스트 | `api.frankfurter.app` |
| Base URL | `/` |
| 체계 | HTTPS |

### 보안

| 항목 | 값 |
|:-----|:---|
| 인증 유형 | **인증 없음** |

{: .tip }
> 사내 API라면 여기서 API 키 / OAuth 2.0 / Microsoft Entra ID 중 하나를 선택합니다. 이번에는 무인증으로 진행합니다.

---

## Step 4 — 정의 탭

OpenAPI에서 가져온 액션이 보입니다.

| 액션 | 값 |
|:-----|:---|
| `GetLatestRate` | GET `/latest` |

요약·설명을 한국어로 다듬어 두면 나중에 에이전트 도구 등록 시 그대로 노출됩니다.

| 항목 | 값 (권장) |
|:-----|:---|
| 요약 | 최신 환율 조회 |
| 설명 | 통화 코드 두 개를 받아 최신 환율을 반환합니다. (예: USD → KRW) |

오른쪽 상단 **커넥터 만들기**.

---

## Step 5 — 테스트 탭에서 단독 호출

테스트 탭 → **+ 새 연결**(인증 없음이라 즉시 생성) → 액션 선택 → 파라미터 입력.

| 파라미터 | 값 |
|:-----|:---|
| `from` | `USD` |
| `to` | `KRW` |

**테스트 작업** 클릭 → 200 응답이 와야 합니다.

### 기대 응답

```json
{
  "amount": 1.0,
  "base": "USD",
  "date": "2026-05-02",
  "rates": { "KRW": 1352.41 }
}
```

---

## 자주 나오는 문제

| 증상 | 원인 | 대응 |
|:-----|:-----|:-----|
| 가져오기 실패 | YAML 들여쓰기 오류 | 사전 제공 파일을 그대로 사용 |
| 401/403 | 보안 탭 인증 설정 잘못됨 | "인증 없음"으로 재설정 |
| 호스트 오류 | 호스트에 `https://` 포함 | 호스트는 도메인만 (`api.frankfurter.app`) |

---

## 체크리스트

- [ ] `exchange-rate.openapi.yaml` 가져오기
- [ ] 커넥터 이름 `ExchangeRate`
- [ ] 인증 없음으로 생성
- [ ] 테스트 탭에서 USD → KRW 200 응답 확인

---

## 다음 단계

[실습 ② — 에이전트에 도구로 연결](s04-2-connector-call)에서 이 커넥터를 Copilot Studio 에이전트에 도구로 등록합니다.
