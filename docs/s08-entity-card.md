---
title: "S8. 입력 구조화 — 엔티티 + AI 프롬프트"
parent: "📗 심화과정"
nav_order: 9
has_children: true
---

# HR·총무 요청 접수 Copilot — Entity에서 AI까지
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 15:25 | 50분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 이 모듈에서 만들 것

> **HR/총무 요청을 자연어로 받아서  
> 구조화된 신청서/티켓으로 만들어주는 Copilot**

- **Entity** — 업무 분류(HR/GA, 출장/연차/회의실)를 **확실하게** 잡는다
- **입력변수(Inputs)** — 지역·목적·시기 등 슬롯을 **모듈화**해 수집한다
- **AI 프롬프트** — Entity가 못 하는 **자연어 날짜 변환·요약·티켓 생성**을 해결한다
- **Adaptive Card** — AI가 실패할 때의 **폴백 UI**(날짜 직접 선택)

---

## 왜 2단계로 나누는가

> "Entity로 **'되는 것'과 '안 되는 것'**이 있습니다.  
> 1단계에서 한계를 직접 체험하고, 2단계에서 AI로 해결합니다."

```
┌─────────────────────────────────────┐
│   1단계: Entity + 입력변수            │
│   · Custom Entity로 업무 분류         │
│   · 입력변수로 슬롯 수집/모듈화        │
│   · 날짜는 텍스트로만 받음 (한계 체험)  │
└──────────────┬──────────────────────┘
               ↓
┌─────────────────────────────────────┐
│   2단계: AI 프롬프트                  │
│   · 날짜 자연어 → 실제 날짜 변환      │
│   · 누락 체크 + 요약/티켓 생성        │
│   · 사용자 확인 + Adaptive Card 폴백 │
└─────────────────────────────────────┘
```

→ 실습은 두 단계로 분리되어 있습니다.

- [실습 ① — Entity + Router + 입력변수](s08-1-entity-router)
- [실습 ② — AI 프롬프트(DateParser + Summarizer)](s08-2-ai-prompt)

---

## 토픽 맵 (전체)

```
T0_Router (입구)
  ├─ [Entity] Domain (HR / GA)
  ├─ [Entity] HR_Task / GA_Task
  └─ Redirect → 처리 토픽
        │
        ├─ T1_HR_BusinessTrip (출장)
        │    ├─ [입력변수] location, purpose, rawDatePhrase
        │    ├─ [AI 프롬프트] DateParser → 날짜 변환
        │    ├─ [확인] 맞나요? / Adaptive Card 폴백
        │    └─ [AI 프롬프트] Summarizer → 티켓 생성
        │
        ├─ T1_HR_AnnualLeave (연차)
        │    └─ (동일 패턴)
        │
        └─ T1_GA_MeetingRoom (회의실)
             └─ (동일 패턴)
```

---

## Entity 목록 (전체)

| Entity 이름 | Type | 값 |
|:----------|:-----|:--|
| `Domain` | Closed list | HR (인사, HR) / GA (총무, 관리) |
| `HR_Task` | Closed list | BusinessTrip (출장) / AnnualLeave (연차, 휴가) / Certificate (재직증명서) |
| `GA_Task` | Closed list | MeetingRoom (회의실) / ITAsset (노트북, 장비) / AccessCard (출입증) |
| `LeaveUnit` | Closed list | FullDay (종일) / HalfAM (오전반차) / HalfPM (오후반차) |

---

## Entity = "밑줄 치기"

엔티티는 사용자의 **자연어 발화에서 정보를 알아서 뽑아내는** 기능입니다.

| | 변수 (기본) | 엔티티 (심화) |
|:--|:-----------|:-------------|
| 비유 | 빈칸 채우기 | **밑줄 치기** |
| 입력 방식 | 사용자에게 하나씩 질문 | 한 문장에서 **자동 추출** |
| 결과 일관성 | 사용자가 입력한 그대로 | **Closed list로 값이 고정** |

이번 실습에서는 **Closed list 커스텀 엔티티**로 업무 분류를 잡습니다.

---

## Entity vs AI — 역할 분담

| 기준 | Entity | AI (선택지 제한) |
|:-----|:-------|:--------------|
| 속도 | 즉시 | 1~3초 |
| 비용 | 무료 | 토큰 과금 |
| 결과 일관성 | 100% | 99% |
| 감사 추적 | 가능 | 어려움 |
| 자연어 유연성 | 낮음 (정의된 값만) | 높음 |

> **확실한 건 Entity, 유연한 건 AI.**  
> 1단계에서 Entity의 한계를 직접 체험한 뒤, 2단계에서 AI로 보완합니다.

---

## 산출물 체크리스트

| 항목 | 내용 |
|:-----|:-----|
| Entity 3개 | `Domain`, `HR_Task`, `GA_Task` |
| 토픽 4개 | `T0_Router`, `T1_HR_BusinessTrip`, `T1_HR_AnnualLeave`, `T1_GA_MeetingRoom` |
| AI 프롬프트 2개 | `DateParser`, `Summarizer` |
| Adaptive Card 1개 | 날짜 선택 폴백용 |
| 테스트 발화 세트 | 1단계용 + 2단계용 |

---

## 다음 단계

- [실습 ① — Entity + Router + 입력변수](s08-1-entity-router) (35분)
- [실습 ② — AI 프롬프트(DateParser + Summarizer)](s08-2-ai-prompt) (25분)
