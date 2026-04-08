---
title: "S7. 도구 — Excel × 도구 비교"
parent: "📗 심화과정"
nav_order: 9
---

# 도구 — Excel 데이터를 도구로 조회하는 3가지 방법
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 14:50 | 35분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 이 모듈에서 배우는 것

- 오전(S2)에서 배운 **지식 방식**과 **도구 방식**의 근본적 차이 이해
- Excel 커넥터 · CSV+AI 프롬프트 · Office Script — **3가지 도구 방식** 비교 실습
- SP 리스트 · Dataverse — **Excel 졸업 경로** 소개
- 상황별 **의사결정 가이드** 완성

{: .highlight }
> 오전 S2에서 "지식으로 읽는 4가지"를 봤습니다. 이번엔 **"도구로 정밀 조회하는 3가지"**입니다.

---

## 지식 vs 도구 — 근본적 차이

| | 지식 (오전 S2·S3) | 도구 (이번 S7) |
|:--|:-----------------|:--------------|
| **원리** | AI가 문서를 **읽고 해석** | 흐름이 데이터를 **조회하고 전달** |
| **비유** | 책을 읽고 대답하는 사서 | 데이터베이스를 검색하는 직원 |
| **데이터 변경** | 재인덱싱 필요 (지연) | **실시간 반영** |
| **정확도** | 의미 검색 (유사 답변) | **정확한 필터링** |
| **쓰기** | 읽기 전용 | 읽기 + **쓰기 가능** |
| **구축 비용** | 5~10분 | 30분~1시간 |

```
"운영 환경 Linux 서버 목록 알려줘"

지식 방식: "운영 환경에는 여러 Linux 서버가 있으며..." (의미 기반, 누락 가능)
도구 방식: 정확히 Env=LIVE AND OSType=Linux 필터 → 17건 목록 반환
```

---

## ④ Excel 커넥터 — 행 나열

가장 기본적인 도구 방식입니다. Power Automate에서 Excel Online 커넥터의 **"테이블에 있는 행 나열"**을 사용합니다.

### 아키텍처

```
👤 사용자 질문
    ↓
🤖 에이전트 → Power Automate 흐름 호출
    ↓
⚡ Excel Online 커넥터 — "테이블에 있는 행 나열"
    ↓ (OData 필터)
📊 SharePoint의 Excel 파일
    ↓
📋 필터링된 행 목록 반환
    ↓
🤖 에이전트가 결과를 정리하여 답변
```

### 특징

| 항목 | 내용 |
|:-----|:-----|
| 커넥터 | Excel Online (Business) |
| 액션 | 테이블에 있는 행 나열 (GetItems) |
| 필터링 | OData 필터 식 (예: `Env eq 'LIVE'`) |
| 장점 | **가장 간단**, 기본과정에서 이미 배운 패턴 |
| 단점 | 복잡한 조건 조합 어려움, 반환 행 수 제한 |
| 적합한 상황 | 단순 조건으로 **수십~수백 건** 조회 |

{: .note }
> 기본과정 M11에서 **행 추가**를 배웠습니다. 같은 커넥터로 **행 조회**도 할 수 있습니다.

### 실습: 흐름 만들기

1. Copilot Studio → 도구 → 흐름 → **새 에이전트 흐름** 생성
2. 트리거: **에이전트에서 흐름 실행** (입력: `filterCondition` 텍스트)
3. 액션: **Excel Online → 테이블에 있는 행 나열**
   - 위치: SharePoint 사이트
   - 파일: 실습용 Excel 파일
   - 테이블: AssetTable
   - 필터 쿼리: `triggerBody()?['text']`
4. 응답: 에이전트에 행 목록 반환

---

## ⑤ CSV 변환 + AI 프롬프트

Excel 전체 데이터를 CSV로 변환한 뒤, **AI Builder 프롬프트에 통째로 전달**하여 AI가 분석합니다.

### 아키텍처

```
👤 사용자 질문
    ↓
🤖 에이전트 → Power Automate 흐름 호출
    ↓
⚡ Excel Online — "테이블에 있는 행 나열"
    ↓
🔄 데이터 조작 — "CSV 테이블 만들기"
    ↓
🧠 AI Builder — "프롬프트 실행"
   (사용자 질문 + CSV 데이터를 함께 전달)
    ↓
📋 AI가 분석한 답변 반환
```

### 특징

| 항목 | 내용 |
|:-----|:-----|
| 핵심 | AI가 **전체 데이터를 보고 판단** |
| 장점 | 복잡한 질문도 자연어로 처리, 요약·비교·추세 분석 가능 |
| 단점 | **토큰 한계** — 데이터가 수백 건 넘으면 잘림 |
| 적합한 상황 | 소규모 데이터(수백 건 이하)의 **분석·요약** |

{: .warning }
> AI 프롬프트에 전달할 수 있는 토큰에 한계가 있습니다. 데이터가 많으면 ⑥ Office Script 방식을 사용하세요.

### 실습: 흐름 만들기

1. ④의 흐름을 복제하거나 새로 생성
2. **"테이블에 있는 행 나열"** 뒤에 **"CSV 테이블 만들기"** 추가
   - 원본: `outputs('테이블에_있는_행_나열')?['body/value']`
3. **AI Builder → 프롬프트 실행** 추가
   - 프롬프트 변수 1: 사용자 질문 (트리거의 text)
   - 프롬프트 변수 2: CSV 데이터 (CSV 테이블 만들기 결과)
4. 프롬프트 내용:
   ```
   아래 CSV 데이터에서 사용자 질문에 답하세요.
   결과가 없으면 없다고 말하세요. 데이터에 없는 내용을 만들지 마세요.
   
   ## 사용자 질문
   {prompt}
   
   ## 데이터
   {csvData}
   ```
5. 응답: AI 분석 결과 반환

### 팁: AI 프롬프트로 응답 포맷 제어

AI Prompt에 **출력 형식을 지정**하면, 에이전트 답변을 일관된 레이아웃으로 만들 수 있습니다.

```
아래 CSV 데이터에서 사용자 질문에 답하세요.
결과가 없으면 없다고 말하세요. 데이터에 없는 내용을 만들지 마세요.

## 응답 형식
반드시 아래 HTML 형식으로 답변하세요:

<b>🔍 검색 조건</b>: {사용자가 요청한 조건 요약}<br>
<b>📊 결과</b>: 총 {N}건<br>
<hr>
<table>
  <tr><th>항목1</th><th>항목2</th><th>항목3</th></tr>
  <tr><td>값</td><td>값</td><td>값</td></tr>
</table>
```

이렇게 하면 에이전트가 매번 **동일한 카드 형태**로 결과를 보여줍니다.

| 포맷 미지정 | 포맷 지정 |
|:-----------|:---------|
| 매번 다른 형태의 텍스트 | **일관된 HTML 테이블 레이아웃** |
| 사용자 경험 들쭉날쭉 | 깔끔한 카드 UI |

---

## ⑥ Office Script — 정밀 쿼리

LLM이 사용자 질문을 **JSON 쿼리로 변환**하고, Excel에 등록된 **Office Script가 필터링**을 수행합니다.

### 아키텍처

```
👤 "인프라서비스팀 리눅스 서버 목록"
    ↓
🤖 에이전트 (LLM이 JSON 변환)
   → {"queryType":"filtered_list","usingDep":"인프라서비스팀","osType":"Linux"}
    ↓
⚡ Power Automate 흐름
    ↓
📝 Parse JSON → Office Script 실행
    ↓
📊 Script가 Excel에서 조건 필터링
    ↓
📋 구조화된 JSON 결과 반환
   { "totalCount": 5, "data": [...] }
```

### 특징

| 항목 | 내용 |
|:-----|:-----|
| 핵심 | **LLM = 쿼리 생성**, **Script = 데이터 처리** — 역할 분리 |
| 장점 | 정확한 필터링, **대용량 가능**, 필요한 필드만 반환 |
| 단점 | Office Script 작성 필요, 설정이 가장 복잡 |
| 적합한 상황 | 수백~수천 건의 **정확한 조회** |

### S6과의 연결

| | S6 (영수증 → 엑셀) | S7 ⑥ (Excel → 에이전트) |
|:--|:------------------|:----------------------|
| Office Script 역할 | **쓰기** (행 추가) | **읽기** (필터링+반환) |
| 데이터 방향 | 에이전트 → Excel | Excel → 에이전트 |

### 실습: 흐름 만들기

1. Excel Online에서 실습 Excel 파일 열기
2. **자동화 → 새 스크립트** 생성:
   ```typescript
   function main(
     workbook: ExcelScript.Workbook,
     queryJson: string
   ): string {
     // 1. JSON 파싱
     const query = JSON.parse(queryJson);
     
     // 2. 테이블에서 데이터 읽기
     const table = workbook.getTable("AssetTable");
     const rows = table.getRangeBetweenHeaderAndTotal().getValues();
     const headers = table.getHeaderRowRange().getValues()[0] as string[];
     
     // 3. 조건에 맞는 행 필터링
     const filtered = rows.filter(row => {
       let match = true;
       if (query.env) match = match && String(row[headers.indexOf("Env")]) === query.env;
       if (query.osType) match = match && String(row[headers.indexOf("OSType")]) === query.osType;
       if (query.location) match = match && String(row[headers.indexOf("Location")]).includes(query.location);
       if (query.ownerName) match = match && String(row[headers.indexOf("OwnerName")]).includes(query.ownerName);
       if (query.usingDep) match = match && String(row[headers.indexOf("UsingDep")]).includes(query.usingDep);
       if (query.keyword) {
         const rowStr = row.map(String).join(",");
         match = match && rowStr.includes(query.keyword);
       }
       return match;
     });
     
     // 4. 결과를 JSON으로 반환 (최대 30건)
     const maxRows = 30;
     const data = filtered.slice(0, maxRows).map(row => {
       const obj: Record<string, string> = {};
       headers.forEach((h, i) => obj[h as string] = String(row[i]));
       return obj;
     });
     
     return JSON.stringify({
       totalCount: filtered.length,
       truncated: filtered.length > maxRows,
       data: data
     });
   }
   ```
3. Power Automate 흐름 생성:
   - 트리거: 에이전트에서 흐름 실행 (입력: `queryJson` 텍스트)
   - 액션 1: **Parse JSON** (`triggerBody()?['text']`)
   - 액션 2: **Excel Online → 스크립트 실행** (queryJson 파라미터 전달)
   - 응답: 스크립트 결과 반환

---

## 3가지 도구 방식 비교

| | ④ Excel 커넥터 | ⑤ CSV + AI | ⑥ Office Script |
|:--|:-------------|:-----------|:----------------|
| **난이도** | ★☆☆ | ★★☆ | ★★★ |
| **데이터 규모** | 중 (~수백 건) | 소 (~수백 건) | **대** (~수천 건) |
| **필터링** | OData (단순) | AI 판단 | **코드 (정밀)** |
| **분석·요약** | ✗ | **✓ (AI)** | ✗ |
| **쓰기** | ✓ | ✗ | **✓** |
| **토큰 비용** | 낮음 | **높음** | 낮음 |

---

## Excel 졸업: 다음 단계 소개

데이터가 더 커지거나, 여러 에이전트·앱에서 공유해야 하면 **Excel을 졸업**할 수 있습니다.

### ⑦ SharePoint 리스트

```
📊 Excel → 📋 SharePoint 리스트로 내보내기
```

| 항목 | 내용 |
|:-----|:-----|
| 하는 법 | Excel에서 **테이블 → 리스트로 내보내기** |
| 장점 | 뷰·필터·권한 관리, Power Apps 연동, API 기본 제공 |
| 적합한 상황 | Excel 구조 유지하면서 **관리 기능 강화** |

{: .tip }
> SP 리스트에 **항목이 추가되면 트리거 발동 → 에이전트가 자동 분석 후 메일 발송** 같은 자동화도 가능합니다. 기본과정에서 배운 Forms 트리거와 같은 원리지만, 데이터 소스가 리스트로 바뀌는 것입니다.

### ⑧ Dataverse

```
📊 Excel → 🗄️ Dataverse 테이블로 가져오기
```

| 항목 | 내용 |
|:-----|:-----|
| 하는 법 | Power Apps → **데이터 가져오기 → Excel** |
| 장점 | 관계형 DB, 보안 역할, ALM, 대용량 |
| 적합한 상황 | 엔터프라이즈급 **데이터 관리** |

---

## 최종 의사결정 가이드

```
내 데이터가 Excel이다
    │
    ├─ 매번 다른 파일? ──────── YES ──→ ⓪ 파일 첨부
    │
    ├─ "대략적인 답변"이면 충분?
    │   ├─ 파일 안 바뀜 ──────────────→ ① 직접 업로드
    │   ├─ 파일 가끔 바뀜 (1개) ───────→ ② SP 파일
    │   └─ 여러 파일 ─────────────────→ ③ SP 폴더
    │
    ├─ "정확한 조회"가 필요?
    │   ├─ 단순 조건 (~수백 건) ───────→ ④ Excel 커넥터
    │   ├─ AI 분석/요약 (~수백 건) ────→ ⑤ CSV + AI
    │   └─ 복잡 조건 (수천 건) ────────→ ⑥ Office Script
    │
    └─ 여러 앱/에이전트에서 공유?
        ├─ 간단한 구조 ──────────────→ ⑦ SP 리스트
        └─ 엔터프라이즈 ──────────────→ ⑧ Dataverse
```

{: .highlight }
> 이 의사결정 트리를 기억해두면, 앞으로 어떤 Excel 기반 에이전트를 만들든 **최적의 방법을 선택**할 수 있습니다.
