---
title: "실습③-A — Tier 3 백엔드: 템플릿 + 흐름"
parent: "S1. 문서 업로드 활용 실습"
grand_parent: "📗 심화과정"
nav_order: 3
---

# 실습 ③-A: Tier 3 백엔드 — Word 템플릿 + Power Automate 흐름
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 09:58 | 14분 | 🟡 따라보기 (실습 환경에 따라 🟢 직접 실습) |

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

## 이 실습에서 완성할 것

| 항목 | 내용 |
|:-----|:-----|
| **완성 산출물** | PDF에서 추출한 값을 Power Automate 흐름으로 보내 Word 템플릿을 채우는 에이전트 |
| **사용 도구** | Copilot Studio 도구 + Power Automate + Word Online 템플릿 채우기 |
| **성공 기준** | `actionItemsJson` 배열이 Word 반복 섹션에 여러 행으로 출력됨 |
| **다음 한계** | 추출은 여전히 에이전트 지침에 묶여 있어 재사용 가능한 AI 빌더 자산은 아직 없음 |

---

## Step 3-1. Word 템플릿 — `회의록_템플릿_Tier3.docx`

{: .tip }
> 미리 만들어 둔 템플릿을 받아 쓰면 빠릅니다 — [회의록_템플릿_Tier3.docx 다운로드](../assets/files/회의록_템플릿_Tier3.docx). 직접 만드는 절차는 아래 순서대로.

직접 만들 때 순서:

1. Word 데스크톱에서 새 문서 → **파일 → 옵션 → 리본 사용자 지정 → 개발 도구** 활성화

   {: .warning }
   > **호환성 모드 확인**: 제목 표시줄에 "호환성 모드" 라고 떠 있으면 **반복 섹션 콘텐츠 컨트롤이 비활성화** 됩니다 (Word 2013+ 기능). **파일 → 정보 → 변환** 클릭해서 최신 형식으로 변환한 뒤 진행하세요.

2. V3 디자인 (네이비 표지, 라벨 셀 음영, 섹션 헤더) 으로 양식 디자인
3. 각 자리에 **개발 도구 → 일반 텍스트 콘텐츠 컨트롤** 삽입 후, 컨트롤 선택 → **속성 → 태그** 지정
   - 태그 이름: `title`, `meetingDate`, `location`, `attendees`, `agenda`, `decisions`, `nextMeeting`
4. 액션 아이템 표:
   - 헤더 행 (담당자/기한/내용) 만들기
   - 데이터 행 셀 안에 일반 텍스트 컨트롤 삽입 — 태그: `owner`, `dueDate`, `task`
   - 데이터 행 **전체를 선택** (행 왼쪽 여백 클릭 또는 레이아웃 → 선택 → 행 선택)
   - **개발 도구 → 반복 섹션 콘텐츠 컨트롤** 클릭 (아이콘: 점 세 개가 세로로 쌓인 모양)
     - 행 위에 작은 ⊕ 핸들이 생기면 성공
   - 그 반복 섹션을 선택 → **속성 → 태그: `actionItems`**
   - {: .warning } **주의**: 표 바깥에 별도로 `actionItems` 라는 일반 텍스트 컨트롤을 따로 만들지 마세요. 반복 섹션 자체가 그 역할을 합니다.

   **① Word 템플릿**

   ![Word 템플릿 — 콘텐츠 컨트롤 배치 (개발 도구 탭, 디자인 모드)](../assets/images/s01-3/001_click.png)

5. 저장 → OneDrive 의 적당한 폴더 (예: `/myTemplates/회의록_템플릿_Tier3.docx`) 에 업로드

   **② OneDrive**

   ![OneDrive — 회의록_템플릿_Tier3.docx 업로드 완료](../assets/images/s01-3/002_click.png)

> **포인트**: 태그 이름 = 흐름의 입력 키. 오타 주의.

{: .important }
> **반복 섹션이 제대로 잡혔는지 검증하는 법**
>
> Power Automate 의 **Microsoft Word 템플릿 채우기** 액션이 매개변수를 어떻게 보여주는가로 확인할 수 있습니다.
>
> ✅ **정상**: `Title`, `MeetingDate`, ... `NextMeeting` 의 7 개 평면 입력 + `ActionItems` **하나** (그 안에 `owner`/`dueDate`/`task` 가 행별로 자동 반복 매핑 UI 가 뜸)
>
> ❌ **잘못됨**: `Owner`, `Task`, `OwnerdueDatetask` 같은 컨트롤이 상위에 평면으로 따로따로 노출되거나, `ActionItems` 가 그냥 텍스트 한 줄짜리 입력으로 보임 — 이건 **반복 섹션 콘텐츠 컨트롤이 안 만들어진 상태**. Word 로 돌아가 위 4번을 다시 수행하고, 흐름의 해당 액션을 **삭제 후 재추가** 해서 스키마를 새로 읽히세요 (액션 자체에 캐시된 스키마가 있어서 템플릿만 교체해서는 안 바뀜).

---

## Step 3-2. Power Automate 흐름 — `회의록_생성_흐름_Tier3`

- 트리거: **Copilot Studio (V2)**
- 입력 파라미터: 8 개 (**모두 텍스트**)
  - `title`, `meetingDate`, `location`, `attendees`, `agenda`, `decisions`, `nextMeeting`
  - `actionItemsJson` — 객체 배열을 **JSON 문자열**로 전달 (예: `[{"owner":"홍길동","dueDate":"2026-05-15","task":"초안 작성"}, ...]`)

> **왜 JSON 문자열인가**: Copilot Studio (V2) 트리거 입력 타입은 텍스트·숫자·부울·날짜·파일 수준이라, **객체 배열(행이 여러 개인 표)** 같은 구조를 그대로 받을 수 없습니다. 실전에서는 **텍스트로 JSON 문자열을 받고, 흐름 안에서 Parse JSON 으로 객체 배열로 변환**합니다. 이게 표준 패턴입니다.

Copilot Studio 좌측 **흐름** → **+ 새 에이전트 흐름** 으로 빈 흐름을 만듭니다.

**① 새 에이전트 흐름 만들기**

![Copilot Studio — 새 에이전트 흐름](../assets/images/s01-3/003_click.png)

빈 흐름은 **에이전트가 흐름을 호출할 때** (트리거) 와 **에이전트에게 응답** 두 노드로 시작합니다.

**② 빈 흐름 — 트리거 + 응답 노드 기본 상태**

![새 흐름 — 트리거 + 응답 기본 노드](../assets/images/s01-3/004_click.png)

트리거를 클릭 → **매개 변수 → + 입력 추가** 로 8 개 입력을 모두 **텍스트** 타입으로 추가합니다.

**③ 트리거 매개 변수 — `+ 입력 추가`**

![트리거 — 입력 추가](../assets/images/s01-3/005_click.png)

**④ 8개 텍스트 입력 추가 완료 (마지막은 `actionItemsJson`)**

![트리거 — 8 개 텍스트 입력 (title, meetingDate, location, attendees, agenda, decisions, nextMeeting, actionItemsJson)](../assets/images/s01-3/006_click.png)

작업 순서:

1. **데이터 조작 → JSON 구문 분석 (Parse JSON)**
   - 콘텐츠: 트리거 입력 `actionItemsJson`
   - 스키마 (【샘플 페이로드를 사용하여 스키마 생성】에 아래와 같은 샘플 붙여넣기):

     ```json
     [
       { "owner": "홍길동", "dueDate": "2026-05-15", "task": "초안 작성" }
     ]
     ```

     자동 생성된 스키마는 다음과 같아야 합니다:

     ```json
     {
       "type": "array",
       "items": {
         "type": "object",
         "properties": {
           "owner":   { "type": "string" },
           "dueDate": { "type": "string" },
           "task":    { "type": "string" }
         }
       }
     }
     ```

   트리거 아래 **+** 클릭 → 작업 추가:

   **⑤ 트리거 아래 + 클릭**

   ![트리거 아래 + 클릭](../assets/images/s01-3/007_click.png)

   검색창에 `json` → **JSON 구문 분석** 선택:

   **⑥ 작업 추가**

   ![작업 추가 — JSON 구문 분석](../assets/images/s01-3/008_click.png)

   Content 입력란 옆 동적 콘텐츠 픽커에서 트리거 입력 더 보기 (9):

   **⑦ Content**

   ![Content — 동적 콘텐츠 더 보기](../assets/images/s01-3/009_click.png)

   트리거 입력 목록에서 **`actionItemsJson`** 클릭:

   **⑧ 트리거 입력**

   ![트리거 입력 — actionItemsJson 선택](../assets/images/s01-3/010_click.png)

   Content 박스에 `actionItemsJson` 칩이 들어갑니다:

   **⑨ Content 에 actionItemsJson 칩 삽입 완료**

   ![Content 에 actionItemsJson 칩 삽입 완료](../assets/images/s01-3/011_click.png)

   Schema 박스 아래 **샘플 페이로드를 사용하여 스키마 생성** 링크 클릭:

   **⑩ 샘플 페이로드를 사용하여 스키마 생성**

   ![샘플 페이로드를 사용하여 스키마 생성](../assets/images/s01-3/012_click.png)

   대화상자에 위 샘플 JSON 한 줄 붙여넣고 **완료**:

   **⑪ 샘플 JSON 페이로드 붙여넣기 → 완료**

   ![샘플 JSON 페이로드 붙여넣기 → 완료](../assets/images/s01-3/013_click.png)

   자동 생성된 스키마 (array → items → object → owner/dueDate/task):

   **⑫ 자동 생성 스키마 확인**

   ![자동 생성 스키마 확인](../assets/images/s01-3/014_click.png)

2. **Word Online (Business) → Microsoft Word 템플릿 채우기 (Populate a Microsoft Word template)**
   - 위치: **OneDrive for Business**
   - 라이브러리: **OneDrive**
   - 파일: `/myTemplates/회의록_템플릿_Tier3.docx` (Step 3-1 5번에서 업로드한 파일)
   - 컨트롤별 입력값 매핑 (트리거 입력의 평면 7 개):
     - `title` ← 트리거 입력 `title`
     - `meetingDate` ← 트리거 입력 `meetingDate`
     - `location` ← 트리거 입력 `location`
     - `attendees` ← 트리거 입력 `attendees`
     - `agenda` ← 트리거 입력 `agenda`
     - `decisions` ← 트리거 입력 `decisions`
     - `nextMeeting` ← 트리거 입력 `nextMeeting`
     - `actionItems` (반복 섹션) ← **1번 단계 Parse JSON 의 산출 본문(Body)**
       - 그 안의 `owner`, `dueDate`, `task` 는 자동으로 행별 매핑 UI 가 뜨고, 각각 Parse JSON 출력의 동일 필드로 매핑

   JSON 구문 분석 아래 **+** 클릭 → 작업 추가:

   **⑬ JSON 구문 분석 아래 + 클릭**

   ![JSON 구문 분석 아래 + 클릭](../assets/images/s01-3/015_click.png)

   검색창에 `word` → **Microsoft Word 템플릿 채우기** 선택:

   **⑭ 작업 추가**

   ![작업 추가 — Microsoft Word 템플릿 채우기](../assets/images/s01-3/016_click.png)

   매개 변수 패널이 비어 있는 초기 상태:

   **⑮ Word 템플릿 채우기**

   ![Word 템플릿 채우기 — 위치/문서 라이브러리/파일 비어 있음](../assets/images/s01-3/017_click.png)

   위치·라이브러리·파일 선택 (캡쳐에서는 `/myTemplates/회의록_템플릿_Tier3.docx`):

   **⑯ 위치=OneDrive for Business / 라이브러리=OneDrive / 파일=회의록…**

   ![위치=OneDrive for Business / 라이브러리=OneDrive / 파일=회의록_템플릿_Tier3.docx](../assets/images/s01-3/018_click.png)

   파일을 선택하면 **고급 매개 변수** 영역이 0/8 로 나타납니다 → **모두 보기** 클릭해 8 개를 펼침:

   **⑰ 고급 매개 변수**

   ![고급 매개 변수 — 모두 보기](../assets/images/s01-3/019_click.png)

   평면 7 개 입력 (Title/MeetingDate/NextMeeting/Agenda/Decisions/Attendees/Location) 에 트리거 입력 동명을 매핑. **ActionItems 는 아직 손대지 않은 상태** (다음 단계에서 전체 배열 모드로 전환):

   **⑱ Title~Location 7 개 + ActionItems 박스 (개별 행 모드**

   ![Title~Location 7 개 + ActionItems 박스 (개별 행 모드 — 새 항목 추가)](../assets/images/s01-3/020_click.png)

   `ActionItems` 박스 **우측 상단** 의 **격자/배열 모양 아이콘** 클릭 → "전체 배열 입력으로 전환":

   **⑲ ActionItems**

   ![ActionItems — 격자 아이콘 클릭 (전체 배열 입력으로 전환)](../assets/images/s01-3/021_click.png)

   박스가 단일 입력란("배열 입력") 으로 바뀝니다:

   **⑳ ActionItems**

   ![ActionItems — 단일 배열 입력 모드](../assets/images/s01-3/022_click.png)

   동적 콘텐츠 픽커 (번개 아이콘) → **JSON 구문 분석 → Body** 클릭:

   **(21) 동적 콘텐츠**

   ![동적 콘텐츠 — JSON 구문 분석 Body](../assets/images/s01-3/023_click.png)

   `ActionItems` 에 **Body** 칩이 통째로 들어가면 완료:

   **(22) ActionItems ← Body 칩 매핑 완료**

   ![ActionItems ← Body 칩 매핑 완료](../assets/images/s01-3/024_click.png)

3. **OneDrive → 파일 만들기**
   - 폴더 경로: `/` (루트) 또는 `/Apps/회의록/생성결과/` 같은 본인 폴더
   - 파일 이름: 평문 `템플릿회의록_` + 함수 **`utcNow()`** + 평문 `.docx`
     - {: .note } `utcNow()` 가 반환하는 ISO 타임스탬프는 콜론(`:`) 을 포함하지만 OneDrive 가 자동으로 밑줄로 치환하여 매번 고유한 파일명이 보장됩니다.
     - {: .warning } **`@{triggerBody()?['title']}` 같은 raw 표현식을 텍스트로 붙여넣지 마세요** — Copilot Studio 새 디자이너에서는 "식이 잘못되었습니다" 에러가 뜹니다. 반드시 입력란 옆 **fx 아이콘(식)** 또는 **번개 아이콘(동적 콘텐츠)** 으로 칩을 삽입해야 합니다.
   - 파일 콘텐츠: 2번 단계 (Microsoft Word 템플릿 채우기) 의 출력

   Word 템플릿 채우기 아래 **+** 클릭 → 작업 추가:

   **(23) Word 템플릿 채우기 아래 + 클릭**

   ![Word 템플릿 채우기 아래 + 클릭](../assets/images/s01-3/025_click.png)

   검색 `파일 만` → **비즈니스용 OneDrive → 파일 만들기** 선택:

   **(24) 작업 추가**

   ![작업 추가 — 비즈니스용 OneDrive 파일 만들기](../assets/images/s01-3/026_click.png)

   파일 만들기 매개 변수 — 폴더 경로 입력:

   **(25) 파일 만들기**

   ![파일 만들기 — 폴더 경로](../assets/images/s01-3/027_click.png)

   폴더 경로 `/`, 파일 이름 `템플릿회의록_[utcNow() 칩].docx`, 파일 콘텐츠는 Microsoft Word 템플릿 채우기 출력:

   **(26) 파일 만들기**

   ![파일 만들기 — 폴더 경로/파일 이름/콘텐츠 매핑 완료](../assets/images/s01-3/028_click.png)

4. **OneDrive → 공유 링크 만들기**
   - 파일: 3번 단계 파일 만들기의 **ID**
   - 링크 유형: **Edit** (또는 View)

   파일 만들기 아래 **+** 클릭:

   **(27) 파일 만들기 아래 + 클릭**

   ![파일 만들기 아래 + 클릭](../assets/images/s01-3/029_click.png)

   검색 `링크` → **비즈니스용 OneDrive → 공유 링크 만들기** 선택:

   **(28) 작업 추가**

   ![작업 추가 — 비즈니스용 OneDrive 공유 링크 만들기](../assets/images/s01-3/030_click.png)

   파일 = 3번의 ID, 링크 유형 = Edit:

   **(29) 공유 링크 만들기**

   ![공유 링크 만들기 — 파일=ID, 링크 유형=Edit](../assets/images/s01-3/031_click.png)

5. **Copilot Studio 에 응답**
   - 출력 `파일이름` (텍스트) ← 3번 파일 만들기의 **이름**
   - 출력 `파일링크` (텍스트) ← 4번 공유 링크 만들기의 **웹 URL**

   완성된 흐름 캔버스 (트리거 → JSON 구문 분석 → Word 템플릿 채우기 → 파일 만들기 → 공유 링크 만들기 → 에이전트에게 응답):

   **(30) 완성된 흐름**

   ![완성된 흐름 — 6 개 노드](../assets/images/s01-3/032_click.png)

   에이전트에게 응답 — 출력 두 개 매핑:

   **(31) 에이전트에게 응답**

   ![에이전트에게 응답 — 파일이름=이름, 파일링크=웹 URL](../assets/images/s01-3/033_click.png)

흐름 우측 상단 **게시** 버튼 클릭:

**⑤ 6개 노드 완성 → 우측 상단 `게시`**

![흐름 — 게시 버튼](../assets/images/s01-3/034_click.png)

세부 정보 패널에서 흐름 이름 (예: `meetingnote_template` 또는 `회의록_생성_흐름_Tier3`) 입력 → **저장**:

**⑥ 흐름 이름 `meetingnote_template` 입력 후 `저장`**

![게시 — 흐름 이름 입력 후 저장](../assets/images/s01-3/035_click.png)

> **포인트 1**: 흐름 안에 LLM 이 한 번도 등장하지 않습니다. 흐름은 **JSON 구문분석 → 템플릿 채우기 → 저장 → 링크** 의 단순 plumbing 만 합니다.
>
> **포인트 2**: Parse JSON 의 **스키마가 핵심**입니다. 스키마가 있어야 그 다음 단계에서 동적 콘텐츠 매핑 UI 가 `owner` · `dueDate` · `task` 를 자동 인식합니다.

{: .warning }
> **For each 로 감싸지 말 것**
>
> "배열이니까 For each 로 돌려야 하나?" 하고 자동으로 For each 컨테이너가 추가될 수 있는데, **절대 안 됩니다**. **Microsoft Word 템플릿 채우기** 액션은 반복 섹션 컨트롤이 들어간 템플릿이라면 **배열을 통째로 받아서 행을 자기가 알아서 생성**합니다.
>
> ❌ 잘못 — 액션 아이템 N 개 → Word 문서 N 개 따로 생성됨
> ```
> Parse JSON → For each → Word 템플릿 채우기
> ```
>
> ✅ 올바름 — Word 문서 1 개에 행 N 줄
> ```
> Parse JSON → Word 템플릿 채우기 (ActionItems ← Parse JSON Body 통째로)
> ```

{: .warning }
> **개별 행 입력 모드 ❌ → 전체 배열 입력 모드 ✅**
>
> Word 템플릿 채우기 액션의 매개변수 패널에서 `ActionItems` 박스를 펼치면 기본적으로 **개별 행 입력 모드** 가 켜져 있습니다 — `Actionitems Owner - 1`, `Actionitems Duedate - 1`, `Actionitems Task - 1` 처럼 **한 행씩** 손으로 채우는 형태. 이 상태로 `Body owner` / `Body dueDate` / `Body task` 를 매핑해도 **첫 한 줄만 출력** 됩니다 (Body owner 는 배열 첫 요소만 가리킴).
>
> **해결**: `ActionItems` 박스 **우측 상단** 의 **격자/배열 모양 아이콘** 클릭 → "전체 배열 입력으로 전환 (Switch to input entire array)". 박스가 단일 입력란 하나로 바뀌면 거기에 **JSON 구문 분석** 액션의 **Body** (배열 전체) 를 통째로 넣습니다. 그래야 N 개 행이 모두 출력됩니다.

---

## 백엔드 완성 — 다음 페이지에서 에이전트와 연결

여기까지 Word 템플릿과 Power Automate 흐름(`meetingnote_template`) 백엔드가 완성됐습니다. 흐름 단독 테스트에서 .docx 가 생성되는 것까지 확인했다면 백엔드는 끝.

다음 페이지에서는 이 흐름을 에이전트가 부를 수 있게 지침·도구로 연결하고, 채팅에서 실제 PDF 를 첨부해 동작을 확인합니다.

---

## 다음 페이지

[실습③-B — Tier 3 에이전트 연결 (지침 + 도구 + 테스트)](s01-3b-tier3-agent)

<!-- build-trigger: 2026-05-11 -->
