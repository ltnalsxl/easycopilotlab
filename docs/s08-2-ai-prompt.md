---
title: "실습② — AI 프롬프트 (DateParser + Summarizer)"
parent: "S8. 입력 구조화 — 엔티티 + AI 프롬프트"
grand_parent: "📗 심화과정"
nav_order: 2
---

# 실습 ②: AI 프롬프트 — DateParser + Summarizer
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 11:05 | 25분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 이 실습의 목표

- [실습 ①](s08-1-entity-router)에서 못 한 **자연어 날짜 변환**을 AI로 해결
- **누락 체크 + 티켓 문구 생성**까지 AI로 고도화
- AI 결과는 반드시 **사용자 확인** + 실패 시 **Adaptive Card 폴백**

---

## 2-1. AI 프롬프트 ①: 날짜 변환 (DateParser) (15분)

### 프롬프트 액션 생성

Copilot Studio → **Actions → + Prompt**

### Inputs

| 변수명 | Type |
|:------|:-----|
| `rawDatePhrase` | String |
| `todayDate` | String |

### 프롬프트 본문

```
당신은 날짜 변환기입니다.

오늘 날짜: {todayDate}

사용자 표현: "{rawDatePhrase}"

위 표현을 실제 날짜로 변환하세요.

규칙:
- YYYY-MM-DD 형식
- 범위면 start_date, end_date 모두 출력
- 단일이면 start_date만 (end_date는 동일값)
- 변환 불가면 UNKNOWN

출력 형식:
start_date: YYYY-MM-DD
end_date: YYYY-MM-DD
confidence: HIGH 또는 LOW
```

### Outputs

| 변수명 | Type |
|:------|:-----|
| `start_date` | String |
| `end_date` | String |
| `confidence` | String |

---

### 날짜 변환 후 분기

```
[Condition] confidence == HIGH
  ↓ True
  [Message]
  "📅 '{rawDatePhrase}'
   → {start_date} ~ {end_date} 맞나요?"
  [Question] 맞습니다 / 다시 선택
     ↓ 맞습니다 → date_start, date_end 확정
     ↓ 다시 선택 → Adaptive Card (폴백)

  ↓ False (LOW / UNKNOWN)
  [Message]
  "날짜를 파악하기 어렵습니다.
   달력에서 직접 선택해 주세요."
  → Adaptive Card (폴백)
```

### Adaptive Card 폴백 (날짜 직접 선택)

```json
{
  "type": "AdaptiveCard",
  "version": "1.5",
  "body": [
    {
      "type": "TextBlock",
      "text": "📅 날짜를 선택해 주세요",
      "weight": "Bolder"
    },
    {
      "type": "Input.Date",
      "id": "input_start_date",
      "label": "시작일",
      "isRequired": true
    },
    {
      "type": "Input.Date",
      "id": "input_end_date",
      "label": "종료일",
      "isRequired": true
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "확인"
    }
  ]
}
```

### 테스트 발화

| 발화 | AI 결과 | 사용자 확인 |
|:-----|:-------|:----------|
| "다음주 수요일" | 2026-05-06 (HIGH) | "맞나요?" → 확정 ✅ |
| "다음주 월~수" | 05-04 ~ 05-06 (HIGH) | "맞나요?" → 확정 ✅ |
| "5월 말쯤" | UNKNOWN (LOW) | → Adaptive Card 폴백 ✅ |

{: .highlight }
> **체험 포인트**: 실습 ①에서 안 되던 "다음주 수요일"이 AI로 해결되는 순간.

---

## 2-2. AI 프롬프트 ②: 요약 + 누락 체크 + 티켓 생성 (15분)

### 프롬프트 액션: `Summarizer`

### Inputs

| 변수명 | 값 |
|:------|:--|
| `task` | 출장 / 연차 / 회의실 등 |
| `location` | 지역 |
| `purpose` | 목적 |
| `date_start` | 확정된 시작일 |
| `date_end` | 확정된 종료일 |
| `original_message` | 사용자 원문 |

### 프롬프트 본문

```
당신은 사내 업무 요청 정리 도우미입니다.

아래 정보를 바탕으로 두 가지를 출력하세요.

[수집된 정보]
- 업무: {task}
- 지역: {location}
- 목적: {purpose}
- 기간: {date_start} ~ {date_end}
- 원문: "{original_message}"

[출력 1: 누락 체크]
위 정보 중 비어있거나 불명확한 항목을 나열하세요.
모두 채워져 있으면 "누락 없음"

[출력 2: 티켓 문구]
위 정보를 아래 형식의 신청서 문구로 작성하세요.

---
■ 업무 요청 티켓
- 요청 유형:
- 기간:
- 지역:
- 목적:
- 비고:
---
```

### 실제 출력 예시

#### 케이스 A: 정보 완전

```
[누락 체크]
누락 없음

[티켓]
■ 업무 요청 티켓
- 요청 유형: 출장
- 기간: 2026-05-04 ~ 2026-05-06
- 지역: 부산
- 목적: 고객사 미팅
- 비고: 없음
```

#### 케이스 B: 목적 누락

```
[누락 체크]
- purpose(출장 목적)이 비어 있습니다.

[티켓]
⚠️ 목적이 누락되어 티켓을 완성할 수 없습니다.
```

→ 누락 시 Copilot이 해당 슬롯을 **다시 질문**하도록 분기 추가.

---

## 정리 & 토론 (선택, 15분)

### ① 발전 과정 비교

| | 1단계 (Entity + 입력변수) | 2단계 (AI 프롬프트) |
|:--|:---------------------|:-----------------|
| 업무 분류 | ✅ | ✅ |
| 지역/목적 수집 | ✅ | ✅ |
| 날짜 변환 | ❌ | ✅ |
| 누락 자동 체크 | ❌ | ✅ |
| 티켓 문구 생성 | ❌ | ✅ |
| 속도/비용 | ⚡ 즉시/무료 | ⏱️ 1~3초/토큰 비용 |
| 결과 일관성 | 🔒 100% 동일 | ⚠️ 99% (확인 필요) |
| 감사 추적 | 📋 가능 | 📋 블랙박스 |

### ② 토론 주제

> **"AI한테 선택지를 제한하면 Entity 없이 AI로 다 할 수 있지 않나요?"**

→ 가능합니다. 그래도 Entity를 쓰는 이유는 **속도·비용·일관성·감사**.  
→ **이 트레이드오프를 알고 선택하는 게 설계자의 판단**입니다.

### ③ 최종 메시지

> **Entity는 '확실한 것을 확실하게' 잡는 도구**  
> **AI는 'Entity가 못 하는 것을 유연하게' 해결하는 도구**  
>
> 둘 다 쓸 줄 알아야 하고,  
> **어디에 뭘 쓸지 판단하는 게 설계자의 역할**입니다.

---

## 체크리스트

- [ ] `DateParser` 프롬프트 액션 생성 (Inputs/Outputs/본문)
- [ ] `confidence` 분기로 사용자 확인 / Adaptive Card 폴백 구현
- [ ] `Summarizer` 프롬프트 액션 생성 (누락 체크 + 티켓 문구)
- [ ] "다음주 수요일 부산 출장 고객사 미팅" 한 문장으로 티켓까지 완성

---

다음 모듈: [S9. 설계 패턴 정리](s09-design-patterns)
