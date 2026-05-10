---
title: "실습③ — Tier 3: 책임 분리"
parent: "S1. 문서 업로드 활용 실습"
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

   ![Word 템플릿 — 콘텐츠 컨트롤 배치 (개발 도구 탭, 디자인 모드)](../assets/images/s01-3/001_click.png)

5. 저장 → OneDrive 의 적당한 폴더 (예: `/myTemplates/회의록_템플릿_Tier3.docx`) 에 업로드

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

![Copilot Studio — 새 에이전트 흐름](../assets/images/s01-3/003_click.png)

빈 흐름은 **에이전트가 흐름을 호출할 때** (트리거) 와 **에이전트에게 응답** 두 노드로 시작합니다.

![새 흐름 — 트리거 + 응답 기본 노드](../assets/images/s01-3/004_click.png)

트리거를 클릭 → **매개 변수 → + 입력 추가** 로 8 개 입력을 모두 **텍스트** 타입으로 추가합니다.

![트리거 — 입력 추가](../assets/images/s01-3/005_click.png)

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

   ![트리거 아래 + 클릭](../assets/images/s01-3/007_click.png)

   검색창에 `json` → **JSON 구문 분석** 선택:

   ![작업 추가 — JSON 구문 분석](../assets/images/s01-3/008_click.png)

   Content 입력란 옆 동적 콘텐츠 픽커에서 트리거 입력 더 보기 (9):

   ![Content — 동적 콘텐츠 더 보기](../assets/images/s01-3/009_click.png)

   트리거 입력 목록에서 **`actionItemsJson`** 클릭:

   ![트리거 입력 — actionItemsJson 선택](../assets/images/s01-3/010_click.png)

   Content 박스에 `actionItemsJson` 칩이 들어갑니다:

   ![Content 에 actionItemsJson 칩 삽입 완료](../assets/images/s01-3/011_click.png)

   Schema 박스 아래 **샘플 페이로드를 사용하여 스키마 생성** 링크 클릭:

   ![샘플 페이로드를 사용하여 스키마 생성](../assets/images/s01-3/012_click.png)

   대화상자에 위 샘플 JSON 한 줄 붙여넣고 **완료**:

   ![샘플 JSON 페이로드 붙여넣기 → 완료](../assets/images/s01-3/013_click.png)

   자동 생성된 스키마 (array → items → object → owner/dueDate/task):

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

   ![JSON 구문 분석 아래 + 클릭](../assets/images/s01-3/015_click.png)

   검색창에 `word` → **Microsoft Word 템플릿 채우기** 선택:

   ![작업 추가 — Microsoft Word 템플릿 채우기](../assets/images/s01-3/016_click.png)

   매개 변수 패널이 비어 있는 초기 상태:

   ![Word 템플릿 채우기 — 위치/문서 라이브러리/파일 비어 있음](../assets/images/s01-3/017_click.png)

   위치·라이브러리·파일 선택 (캡쳐에서는 `/myTemplates/회의록_템플릿_Tier3.docx`):

   ![위치=OneDrive for Business / 라이브러리=OneDrive / 파일=회의록_템플릿_Tier3.docx](../assets/images/s01-3/018_click.png)

   파일을 선택하면 **고급 매개 변수** 영역이 0/8 로 나타납니다 → **모두 보기** 클릭해 8 개를 펼침:

   ![고급 매개 변수 — 모두 보기](../assets/images/s01-3/019_click.png)

   평면 7 개 입력 (Title/MeetingDate/NextMeeting/Agenda/Decisions/Attendees/Location) 에 트리거 입력 동명을 매핑. **ActionItems 는 아직 손대지 않은 상태** (다음 단계에서 전체 배열 모드로 전환):

   ![Title~Location 7 개 + ActionItems 박스 (개별 행 모드 — 새 항목 추가)](../assets/images/s01-3/020_click.png)

3. **OneDrive → 파일 만들기**
   - 폴더: `/Apps/회의록/생성결과/`
   - 파일 이름: 평문 `회의록_` + 동적 콘텐츠 픽커로 트리거 입력 **`title`** 삽입 + 평문 `.docx`
     - 즉 입력란이 이렇게 보여야 함: `회의록_[title 칩].docx`
     - {: .warning } **`@{triggerBody()?['title']}` 같은 raw 표현식을 텍스트로 붙여넣지 마세요** — Copilot Studio 새 디자이너에서는 "식이 잘못되었습니다" 에러가 뜹니다. 반드시 입력란 옆 **번개 아이콘(동적 콘텐츠)** 으로 트리거 입력 칩을 삽입해야 합니다.
     - {: .note } 굳이 날짜를 파일명에 넣고 싶다면 `meetingDate` 가 콜론(`:`) 을 포함할 수 있으므로 OneDrive 가 거부합니다. 강의 시연용으로는 `title` 만 쓰는 것이 가장 실패 없습니다 (동명 충돌 시 OneDrive 가 자동으로 `(1)` 부여).
   - 콘텐츠: 2번 단계의 출력 (Microsoft Word 문서)
4. **OneDrive → 항목 링크 가져오기** (또는 공유 링크 만들기)
5. **Copilot Studio 에 응답**
   - 출력: `다운로드링크` (텍스트), `파일이름` (텍스트)

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
   - actionItemsJson (액션 아이템. 아래 형식의 **JSON 문자열**)
     ```
     [{"owner":"홍길동","dueDate":"2026-05-15","task":"초안 작성"}, ...]
     ```
     — 반드시 대괄호로 시작하는 유효한 JSON. 이스케이프되지 않은 큰따옴표 사용. 액션 아이템이 없으면 `[]`.

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
