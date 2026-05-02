---
title: "실습① — Entity + Router + 입력변수"
parent: "S8. 입력 구조화 — 엔티티 + AI 프롬프트"
grand_parent: "📗 심화과정"
nav_order: 1
---

# 실습 ①: Entity + Router + 입력변수
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 10:30 | 35분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 이 실습의 목표

- Entity로 업무를 **분류**한다 (HR/GA → 세부 업무)
- 입력변수(Inputs)로 필요한 정보를 **수집**한다
- 날짜는 **텍스트로만** 받는다 → **한계 체험** (실습 ②로 이어짐)

---

## 1-1. Entity 만들기 (10분)

Copilot Studio → **Entities → + New entity**

### Entity 1: `Domain` (Closed list)

| 항목 | 동의어 |
|:-----|:------|
| HR | 인사, HR |
| GA | 총무, 관리, 사무 |

### Entity 2: `HR_Task` (Closed list)

| 항목 | 동의어 |
|:-----|:------|
| BusinessTrip | 출장 |
| AnnualLeave | 연차, 휴가 |
| Certificate | 재직증명서, 경력증명서 |

### Entity 3: `GA_Task` (Closed list)

| 항목 | 동의어 |
|:-----|:------|
| MeetingRoom | 회의실, 미팅룸 |
| ITAsset | 노트북, 모니터, 장비 |
| AccessCard | 출입증, 사원증 |

---

## 1-2. Router 토픽 만들기 (10분)

### `T0_Router` 노드 구성

```
[Question] "어떤 업무를 도와드릴까요?"
  · Identify → Entity: Domain
  · 변수: bizDomain
       ↓
[Condition] bizDomain == HR
       ↓
  [Question] "HR에서 어떤 업무인가요?"
    · Identify → Entity: HR_Task
    · 변수: hrTask
       ↓
  [Condition] hrTask == BusinessTrip
    → Redirect → T1_HR_BusinessTrip
  [Condition] hrTask == AnnualLeave
    → Redirect → T1_HR_AnnualLeave

[Condition] bizDomain == GA
       ↓
  [Question] "총무에서 어떤 업무인가요?"
    · Identify → Entity: GA_Task
    · 변수: gaTask
       ↓
  [Condition] gaTask == MeetingRoom
    → Redirect → T1_GA_MeetingRoom
```

### 테스트 발화

| 발화 | 결과 |
|:-----|:-----|
| "인사 문의요" | Domain = HR ✅ |
| "출장" | HR_Task = BusinessTrip ✅ |
| "회의실 예약" | GA_Task = MeetingRoom ✅ |
| "좀 쉬고 싶어요" | ❌ 인식 실패 → 재질문 |

{: .highlight }
> **체험 포인트**: Entity가 정확하게 잡히는 것을 확인.  
> 인식 실패 시 "모르겠다"고 다시 묻는 게 **안전한 설계**라는 걸 체감합니다.

---

## 1-3. 처리 토픽 + 입력변수 (15분)

### `T1_HR_BusinessTrip` (출장) — Inputs 정의

| 변수명 | Type | 질문 프롬프트 | 비고 |
|:------|:-----|:------------|:-----|
| `location` | String | "출장 지역은 어디인가요?" | 필수 |
| `purpose` | String | "출장 목적을 간단히 적어주세요." | 필수 |
| `rawDatePhrase` | String | "출장 시기를 말씀해 주세요. (예: 다음주 수요일, 5/10~12 등)" | **텍스트로만** 받음 |

### 토픽 내부 노드 구성

```
[자동] location 비어있으면 → 질문
[자동] purpose 비어있으면 → 질문
[자동] rawDatePhrase 비어있으면 → 질문
       ↓
[Message] 요약 출력
```

### 1단계 요약 메시지 (아직 날짜 변환 전)

```
📋 출장 요청 접수 (초안)

- 업무: 출장
- 지역: {location}
- 목적: {purpose}
- 시기: {rawDatePhrase}  ← 아직 "텍스트 그대로"

⚠️ 정확한 날짜는 아직 확정되지 않았습니다.
```

> 같은 패턴으로 `T1_HR_AnnualLeave`, `T1_GA_MeetingRoom`도 만들어 둡니다.  
> (입력변수 이름만 업무에 맞게 변경)

---

## 1-4. "한계 체험" (5분) ⭐ 전환점

### 의도적으로 시도시킬 발화

| 발화 | 결과 |
|:-----|:-----|
| "다음주 수요일 부산 출장" | `location` = 부산 ✅, `rawDatePhrase` = "다음주 수요일" (텍스트 그대로) |
| | **하지만 실제 날짜(예: 2026-05-06)로는 변환 안 됨** ❌ |

### 강사 멘트

> "보셨듯이 Entity와 입력변수로  
> '무슨 업무인지'와 '지역/목적'은 잘 잡았습니다.  
>
> 하지만 **'다음주 수요일'이 실제로 몇 월 며칠인지**는  
> Copilot Studio가 자체적으로 해결하지 못합니다.
>
> 이제 [실습 ②](s08-2-ai-prompt)에서 **AI 프롬프트로 이걸 해결**합니다."

---

## 체크리스트

- [ ] Entity 3개 생성 (`Domain`, `HR_Task`, `GA_Task`)
- [ ] `T0_Router` 토픽이 HR/GA → 세부 업무로 분기됨
- [ ] `T1_HR_BusinessTrip` 토픽이 입력변수 3개를 수집함
- [ ] "다음주 수요일 부산 출장"으로 **한계 체험** 완료

---

## 다음 단계

[실습 ② — AI 프롬프트(DateParser + Summarizer)](s08-2-ai-prompt)로 이동해서  
"다음주 수요일"을 실제 날짜로 변환하고, 누락 체크 + 티켓 생성을 자동화합니다.
