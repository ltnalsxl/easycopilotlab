---
title: "실습③ — Tier 3: 책임 분리"
parent: "S1. 문서 자동화 5-Tier"
grand_parent: "📗 심화과정"
nav_order: 3
---

# 실습 ③: Tier 3 — 에이전트 추출 → 흐름 → Word 템플릿 채우기
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 09:58 | 22분 | 🟡 따라보기 (실습 환경에 따라 🟢 직접 실습) |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## 핵심 아이디어 — 책임 분리

![Tier 3 책임 분리 다이어그램](../assets/images/s03/05_세션1_Tier3_책임분리.png)

Tier 2 의 LLM 이 두 가지 일 — **(a) PDF에서 정보 추출** 과 **(b) HTML 디자인 만들기** — 을 한꺼번에 했습니다. Tier 3 는 이 둘을 분리합니다.

- **에이전트(LLM)**: PDF 에서 정보 추출만 — 결과는 단순 JSON
- **흐름(Power Automate)**: 정해진 Word 템플릿의 콘텐츠 컨트롤에 값을 꽂아 넣는 plumbing 만

```
[사용자] PDF 첨부 + "표준 회의록으로 만들어줘"
   ↓
[에이전트] PDF 추출 → JSON 결과
   ↓
[흐름] JSON 입력 받기
        → Word Online: 템플릿 채우기 (콘텐츠 컨트롤)
        → OneDrive: 새 파일 만들기
        → 공유 링크 반환
   ↓
[응답] 다운로드 링크 안내
```

이 분리의 효과:

- 에이전트 지침은 **단순 추출 지시** 만 — 분량이 1/3 로 줄어듦
- 디자인은 **Word 템플릿 .docx 를 디자이너가 직접 다듬을 수 있음** — 회사 표준 변경에 즉시 대응
- 액션 아이템 같은 표는 Word 의 **반복 섹션 컨트롤** 로 정확히 행 단위 매핑

---

## Step 3-1. Word 템플릿 — `회의록_템플릿_Tier3.docx`

{: .tip }
> 미리 만들어 둔 템플릿을 받아 쓰면 빠릅니다 — [회의록_템플릿_Tier3.docx 다운로드](../assets/files/회의록_템플릿_Tier3.docx). 직접 만드는 절차는 아래 순서대로.

직접 만들 때 순서:

1. Word 데스크톱에서 새 문서 → **파일 → 옵션 → 리본 사용자 지정 → 개발 도구** 활성화
2. V3 디자인 (네이비 표지, 라벨 셀 음영, 섹션 헤더) 으로 양식 디자인
3. 각 자리에 **개발 도구 → 일반 텍스트 콘텐츠 컨트롤** 삽입 후, 컨트롤 선택 → **속성 → 태그** 지정
   - 태그 이름: `title`, `meetingDate`, `location`, `attendees`, `agenda`, `decisions`, `nextMeeting`
4. 액션 아이템 표:
   - 헤더 행 (담당자/기한/내용) 만들기
   - 데이터 행 셀 안에 일반 텍스트 컨트롤 삽입 — 태그: `owner`, `dueDate`, `task`
   - 데이터 행 전체를 선택 → **개발 도구 → 반복 섹션 컨트롤** → 속성 → 태그: `actionItems`
5. 저장 → OneDrive `/Apps/회의록/회의록_템플릿.docx` 업로드

> **포인트**: 태그 이름 = 흐름의 입력 키. 오타 주의.

---

## Step 3-2. Power Automate 흐름 — `회의록_생성_흐름_Tier3`

- 트리거: **Copilot Studio (V2)**
- 입력 파라미터: 8 개
  - `title` (텍스트), `meetingDate` (텍스트), `location` (텍스트)
  - `attendees` (텍스트), `agenda` (텍스트), `decisions` (텍스트)
  - `nextMeeting` (텍스트)
  - `actionItems` (테이블/배열) — 객체 배열 `{ owner, dueDate, task }`

작업 순서:

1. **Word Online (Business) → Microsoft Word 템플릿 채우기 (Populate a Microsoft Word template)**
   - 위치: OneDrive
   - 라이브러리: OneDrive
   - 파일: `/Apps/회의록/회의록_템플릿.docx`
   - 컨트롤별 입력값 매핑:
     - `title` ← 트리거 입력 `title`
     - `meetingDate` ← 트리거 입력 `meetingDate`
     - `location` ← 트리거 입력 `location`
     - `attendees` ← 트리거 입력 `attendees`
     - `agenda` ← 트리거 입력 `agenda`
     - `decisions` ← 트리거 입력 `decisions`
     - `nextMeeting` ← 트리거 입력 `nextMeeting`
     - `actionItems` (반복 섹션) ← 트리거 입력 `actionItems` 통째로 매핑
       - 그 안의 `owner`, `dueDate`, `task` 는 자동으로 행별 매핑 UI 가 뜸
2. **OneDrive → 파일 만들기**
   - 폴더: `/Apps/회의록/생성결과/`
   - 파일 이름: `회의록_@{triggerBody()?['meetingDate']}_@{triggerBody()?['title']}.docx`
     (트리거 입력값 조합으로 안전한 파일명)
   - 콘텐츠: 1번 단계의 출력 (Microsoft Word 문서)
3. **OneDrive → 항목 링크 가져오기** (또는 공유 링크 만들기)
4. **Copilot Studio 에 응답**
   - 출력: `다운로드링크` (텍스트), `파일이름` (텍스트)

> **포인트**: 흐름 안에 LLM 이 한 번도 등장하지 않습니다. 흐름은 단순 plumbing 만 합니다. 모든 추출은 에이전트가 합니다.

---

## Step 3-3. 에이전트 지침 — 추출만 시키기

```
당신은 회의록 추출 에이전트입니다.
사용자가 회의록 PDF 를 첨부하고 표준화/Word 변환을 요청하면 다음을 수행하세요.

1. PDF 본문에서 아래 항목을 추출합니다 (없으면 빈 값, 추측 금지):
   - title (회의 제목)
   - meetingDate (일시. "2026-05-09 14:00" 형식)
   - location (장소)
   - attendees (참석자. 쉼표로 연결한 문자열)
   - agenda (안건. 줄바꿈으로 구분한 문자열)
   - decisions (결정 사항. 줄바꿈으로 구분한 문자열)
   - nextMeeting (다음 회의 일정)
   - actionItems (액션 아이템. 객체 배열로 [{owner, dueDate, task}, ...])

2. 추출 결과를 흐름 "회의록_생성_흐름_Tier3" 에 매개변수로 그대로 전달합니다.

3. 흐름 결과의 다운로드 링크를 사용자에게 한 줄로 안내합니다.
   "회의록 저장 완료: <다운로드 링크>"

PDF 가 아닌 파일이 들어오면 회의록 PDF 를 요청하세요.
원본에 없는 정보는 절대 추측하지 마세요.
```

지침 분량 약 500자 — Tier 2 의 1/15 입니다.

---

## Step 3-4. 토픽·도구 연결

- 흐름 `회의록_생성_흐름_Tier3` 을 도구로 추가 → 입력 매핑은 모델 자동 채움
- PDF 첨부는 채팅에서 자동 처리 (Excel 과 달리 별도 토픽 불필요)

---

## Step 3-5. 테스트

- 같은 두 PDF 첨부 → "표준 회의록으로"
- 흐름이 호출되어 OneDrive 에 .docx 가 떨어짐
- 액션 아이템이 반복 섹션을 따라 행 단위로 들어가 있는지 확인

---

## 결과 — 깔끔하게 분리됐다. 그런데 AI 빌더가 안 쓰였다

분리는 깔끔합니다. 회사 표준이 바뀌면 .docx 만 다시 디자인. 추출 로직이 바뀌면 지침만 수정. 책임이 명확합니다.

다만 — 이 세션의 본래 주제가 **AI 빌더** 였습니다. 그런데 Tier 3 까지 와도 AI 빌더는 한 번도 등장하지 않았습니다. **"AI 빌더의 진짜 강점은 어디서 쓰지?"** — 이 질문이 Tier 4·5 로 가는 동기입니다.

---

## 다음 페이지

[실습④ — Tier 4: AI 빌더 문서 출력](s01-4-tier4-aibuilder)

<!-- build-trigger: 2026-05-10 -->
