---
title: "실습① — 흐름 완성 (Step 1~12)"
parent: "S2. Excel 분석 17단계"
grand_parent: "📗 심화과정"
nav_order: 1
---

# 실습 ①: Excel을 받아 분석하는 흐름 만들기 (Backend)
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 10:35 | 15분 | 🟡 따라보기 |

이 페이지에서는 **Power Automate 흐름**을 완성합니다. 사용자 입구는 다음 페이지에서 만듭니다.

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Step 1. 샘플 Excel 준비

- `샘플/과일판매.xlsx` 를 OneDrive 작업 폴더로 업로드
- 첫 번째 시트 이름은 그대로 두기 (`Sheet1`)
- 컬럼 확인: `연도`, `분기`, `과일`, `판매량` (헤더는 1행)

> **포인트**: "교육용 샘플"이지만, 실제 적용 시에는 컬럼명이 깔끔한지·빈 행이 없는지·시트가 하나로 정리됐는지가 정확도를 좌우합니다.

---

## Step 2. 에이전트 흐름 (Power Automate) 만들기

1. [make.powerautomate.com](https://make.powerautomate.com) 접속
2. **만들기 → 인스턴트 클라우드 흐름**
3. 트리거: **Copilot Studio (V2)** 선택 → "Copilot에서 흐름을 호출할 때"
4. 흐름 이름: `엑셀분석_흐름`
5. 트리거 노드의 **입력** 추가:
   - `요청문` (텍스트)
   - `파일` (파일) — 콘텐츠와 파일명 모두 받을 수 있도록 **파일** 형식 선택

> **포인트**: 입력 파라미터의 **이름과 타입**은 토픽에서 호출할 때 동일하게 매핑되어야 합니다.

---

## Step 3. OneDrive에 임시 저장

1. 새 작업 추가: **OneDrive for Business → 파일 만들기**
2. 폴더 경로: `/Apps/엑셀분석_임시/`
3. 파일 이름: `@{triggerOutputs()?['body/파일/name']}` 또는 단순화하여 `temp_@{utcNow('yyyyMMddHHmmss')}.xlsx`
4. 파일 콘텐츠: 트리거 입력의 **파일 콘텐츠**

> **포인트**: 임시 폴더는 미리 OneDrive에 만들어두면 깔끔합니다. 이후 Office Script가 이 파일을 대상으로 실행됩니다.

---

## Step 4. 여기까지 테스트

1. 흐름 저장 → **테스트** → "수동으로"
2. 입력: 요청문에 "test", 파일에 `과일판매.xlsx` 첨부
3. 실행 후 OneDrive `/Apps/엑셀분석_임시/`에 파일이 생성됐는지 확인

> **체크포인트**: 여기서 실패하면 OneDrive 권한이나 트리거 입력 매핑을 다시 확인하세요.

---

## Step 5. Office Script 작성 (시트 → JSON)

1. OneDrive에서 방금 업로드한 `과일판매.xlsx` 열기 (Excel for the web)
2. 리본 메뉴: **자동화 → 새 스크립트**
3. **Excel Copilot 사용 권장**: 우측 Copilot 창에서 다음 프롬프트 입력
   > "이 통합 문서의 첫 번째 시트에 있는 모든 데이터를 헤더 포함한 객체 배열 JSON 문자열로 반환하는 Office Script를 만들어줘. main 함수의 반환 타입은 string이어야 해."
4. 생성된 코드 검토 후 저장
5. 스크립트 이름: `시트_to_JSON`

샘플 코드:

```typescript
function main(workbook: ExcelScript.Workbook): string {
  const sheet = workbook.getFirstWorksheet();
  const range = sheet.getUsedRange();
  if (!range) return "[]";
  const values = range.getValues();
  if (values.length < 2) return "[]";
  const headers = values[0].map(String);
  const rows = values.slice(1).map(row => {
    const obj: { [key: string]: string | number | boolean } = {};
    headers.forEach((h, i) => obj[h] = row[i]);
    return obj;
  });
  return JSON.stringify(rows);
}
```

> **포인트**: 반환 타입을 `string`으로 잡아 JSON 문자열로 받으면, 다음 단계의 AI 프롬프트가 처리하기 가장 쉽습니다.

---

## Step 6. 흐름에 "스크립트 실행" 노드 추가

1. Power Automate 흐름으로 돌아옴
2. 새 작업: **Excel Online (Business) → 스크립트 실행**
3. 매개변수
   - 위치: `OneDrive for Business`
   - 문서 라이브러리: `OneDrive`
   - 파일: 동적 콘텐츠에서 **Step 3에서 만든 파일의 ID** 선택
   - 스크립트: `시트_to_JSON`

---

## Step 7. AI 프롬프트 실행 노드 추가

1. 새 작업: **AI Builder → 프롬프트로 텍스트 만들기 (Create text with GPT using a prompt)**
2. 프롬프트는 아직 없으니 **새 프롬프트 만들기** 클릭

---

## Step 8. 사용자 지정 프롬프트 작성

새 창에서 프롬프트 디자이너가 열립니다.

**프롬프트 이름**: `엑셀_분석_프롬프트`

**프롬프트 본문**:

```
당신은 Excel 데이터 분석가입니다. 아래 [데이터]는 Excel 시트의 행을
JSON 배열로 변환한 것입니다. [요청문]에 따라 데이터를 분석하고,
한국어로 친절하게 답변하세요.

규칙:
- 숫자는 천 단위 콤마로 표기합니다.
- 평균/합계 등 계산이 필요하면 정확히 계산합니다 (코드 인터프리터 사용).
- 데이터에 없는 값은 추측하지 말고 "데이터에 없습니다"라고 답하세요.
- 답변은 결론 한 줄 + 근거 표(또는 목록)로 구성하세요.

[요청문]
{{요청문}}

[데이터]
{{데이터JSON}}
```

**입력 변수 추가**:

- `요청문` (텍스트)
- `데이터JSON` (텍스트)

**모델 설정 (중요)**:

- 모델: **GPT-5** (또는 사용 가능한 최신 모델)
- **코드 인터프리터: 켜기** ← 평균/합계/그룹화 등을 정확히 계산하기 위해 필수

**테스트**:

- 요청문: "사과의 연도별 평균 판매량은?"
- 데이터JSON: Step 5에서 한 번 실행해본 JSON 결과를 그대로 붙여넣기

테스트 결과가 그럴듯하면 **프롬프트 저장**.

> **포인트**: 코드 인터프리터를 켜지 않으면 LLM이 숫자 계산을 "추측"으로 해서 틀릴 수 있습니다. 표/숫자 분석에는 거의 필수입니다.

---

## Step 9. 프롬프트 단독 테스트

프롬프트 디자이너의 테스트 모드에서 다양한 요청문으로 점검:

- "사과의 연도별 평균 판매량은?"
- "가장 많이 팔린 과일 Top 3는?"
- "2024년 전체 판매량 합계는?"

---

## Step 10. 흐름으로 돌아와 입력 파라미터 매핑

1. AI 프롬프트 노드에 두 입력이 보임
2. `요청문`에 → 트리거의 `요청문` 동적 콘텐츠 매핑
3. `데이터JSON`에 → Step 6 "스크립트 실행"의 **결과(result)** 동적 콘텐츠 매핑

---

## Step 11. 흐름의 응답 노드

1. 마지막 작업: **Copilot Studio에 응답 (Respond to Copilot)**
2. 출력 추가:
   - 이름: `결과텍스트` (텍스트)
   - 값: AI 프롬프트 노드의 **응답 텍스트** 동적 콘텐츠

---

## Step 12. 흐름 전체 테스트

1. 저장 후 **테스트 → 수동으로**
2. 요청문: "사과의 연도별 평균 판매량은?"
3. 파일: `과일판매.xlsx`
4. 결과 텍스트가 자연어 분석 결과로 나오는지 확인

> **체크포인트**: 여기까지 성공하면 흐름은 완성. 다음 페이지에서 토픽 입구를 만들고 끝냅니다.

---

## 다음 페이지

[실습② — 토픽 연결 (Step 13~17)](s02-2-excel-topic)
