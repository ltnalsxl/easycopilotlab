---
title: "실습④-A — Tier 4 백엔드: 템플릿 + 프롬프트 + 흐름"
parent: "S1. 문서 업로드 활용 실습"
grand_parent: "📗 심화과정"
nav_order: 5
---

# 실습 ④-A: Tier 4 백엔드 — Word 템플릿 + AI 빌더 프롬프트 + Power Automate 흐름
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 10:20 | 5분 | 🟡 따라보기 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

---

![Tier 4 책임 분리 — 에이전트(파일 위임) → 흐름(AI 빌더 + OneDrive)](../assets/images/s03/11_세션1_Tier4_책임분리.png)

## 핵심 아이디어

AI 빌더 프롬프트의 **"문서 출력 (Document output, 미리 보기)"** 기능을 사용합니다.

## 이 실습에서 완성할 것

| 항목 | 내용 |
|:-----|:-----|
| **완성 산출물** | PDF 파일을 흐름에 넘기면 AI 빌더가 추출과 Word 양식 채우기를 한 번에 처리하는 에이전트 |
| **사용 도구** | Power Automate + AI 빌더 프롬프트 문서 출력 + OneDrive |
| **성공 기준** | 흐름 입력은 파일만 받고, 결과 .docx 는 `Document Output Content Bytes` 로 저장됨 |
| **완성 의미** | 추출 로직이 에이전트 지침이 아니라 재사용 가능한 AI 빌더 프롬프트로 분리됨 |

이 기능의 동작은 다음과 같습니다.

{% raw %}
- **입력**: 텍스트 또는 파일 (PDF · 이미지 등)
- **양식**: 미리 업로드한 Word 양식 (`{{필드명}}` 텍스트 플레이스홀더)
- **동작**: AI 가 입력에서 정보를 추출 + 양식의 `{{}}` 자리를 채움 (한 번의 호출)
- **출력**: `Document Output Content Bytes` (완성된 .docx 바이너리)

Tier 3 에서는 **에이전트가 추출 → 흐름이 채우기** 의 두 단계였는데, Tier 4 에서는 **흐름이 AI 빌더 프롬프트 한 번 호출하면** 추출+채우기가 동시에 일어납니다. 흐름이 더 단순해집니다.

```
[사용자] PDF 첨부 + "표준 회의록으로 만들어줘"
   ↓
[에이전트] 첨부 파일을 흐름에 그대로 넘김 (추출은 안 함)
   ↓
[흐름]  AI 빌더: 프롬프트 실행 (입력=파일, 양식=Word 템플릿)
        → OneDrive: 새 파일 만들기 (콘텐츠 = Document Output Content Bytes)
        → 공유 링크 반환
   ↓
[응답] 다운로드 링크 안내
```

> **참고 문서**: [Generate document output from a prompt — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/generate-document-output-prompt)

---

## Step 4-1. Word 템플릿 — `회의록_템플릿_Tier4.docx`

{: .tip }
> 미리 만들어 둔 템플릿: [회의록_템플릿_Tier4.docx 다운로드](../assets/files/회의록_템플릿_Tier4.docx). Tier 3 와 호환되지 않으니 반드시 이 파일을 사용하세요.

**Tier 3 와 디자인은 같지만 자리표시자 형식이 다릅니다.**

| | Tier 3 (Word Online 채우기) | Tier 4 (AI 빌더 문서 출력) |
|---|---|---|
| 자리표시자 형식 | **콘텐츠 컨트롤** (SDT, 태그 = 변수명) | **`{{필드명}}` 텍스트** |
| 반복 섹션 | 반복 섹션 컨트롤 | 표 셀에 `{{actionItems.owner}}` 형식 |

> 두 양식은 **호환되지 않습니다.** Tier 3 에 Tier 4 양식을 넣으면 빈 .docx 가 떨어지고, 그 반대도 마찬가지. 반드시 두 파일을 따로 둡니다.

직접 만들 때 순서 (Tier 4):

1. Word 데스크톱에서 새 문서 → V3 디자인으로 양식 작성
2. 각 자리에 **그냥 텍스트로** `{{title}}`, `{{meetingDate}}`, ... 입력 (콘텐츠 컨트롤 아님)
3. 액션 아이템 표는 한 행만 두고, 셀에 `{{actionItems.owner}}` `{{actionItems.dueDate}}` `{{actionItems.task}}` 입력
4. 저장 → 20MB 이하 (한도)

**① Word Online 에서 양식 내용 확인 — `{{}}` 가 그냥 텍스트로 박혀 있어야 함**

![Word Online 에서 회의록_템플릿_Tier4.docx 를 연 모습 — {{title}}, {{meetingDate}}, {{location}}, {{attendees}}, {{agenda}} 가 그냥 텍스트로 박혀 있다 (콘텐츠 컨트롤 아님)](../assets/images/s01-4/001_click.png)
{% endraw %}

---

## Step 4-2. AI 빌더 프롬프트 — `회의록_표준화_프롬프트`

1. Power Apps · Power Automate 화면 좌측 **AI 허브 → 프롬프트 → 새 프롬프트**
2. 프롬프트 본문 입력:
   ```
   첨부된 회의록 PDF 에서 정보를 추출해 회의록 양식을 채워주세요.
   - 원본에 없는 정보는 추측하지 말고 빈 칸으로 두세요.
   - 일시는 "YYYY-MM-DD HH:mm" 형식으로 정리.
   - 액션 아이템은 담당자/기한/내용으로 분리.
   ```
3. **입력 추가** → 종류: **파일** → 이름: `meetingFile`
4. **출력 형식** → **문서 (Document)** 선택
5. **양식 업로드** → Step 4-1 의 `회의록_템플릿_Tier4.docx` 업로드
6. **테스트** 탭에서 회의록 PDF 한 번 올려 동작 확인 → 저장

> **제약**: 솔루션 이동 시 양식 재업로드 필요. 5MB 이상 양식은 저장-재오픈 후에야 테스트 가능.

{: .tip }
> 프롬프트는 AI 허브에서 별도로 만들어도 되지만, **흐름의 `프롬프트 실행` 액션 매개 변수 드롭다운에서 `+ 새 사용자 지정 프롬프트`** 를 눌러 그 자리에서 만들어도 됩니다. 아래 캡처는 흐름 안에서 만든 경로입니다.

**① 새 프롬프트 편집기 열기**

![흐름 디자이너의 프롬프트 실행 매개 변수 드롭다운에서 `+ 새 사용자 지정 프롬프트` 를 클릭해 새 프롬프트 편집기 진입](../assets/images/s01-4/008_click.png)

**② 이름·모델 설정**

![새 프롬프트 이름을 `meetingnote_AI` 로 입력하고 모델 드롭다운에서 `Standard GPT-5 chat` 선택 — 문서·이미지 작업에 적합한 품질](../assets/images/s01-4/009_click.png)

**① 빈 편집기 레이아웃 확인**

![모델이 `GPT-5 chat` 으로 바뀐 빈 프롬프트 편집기 — 좌측 지침에 본문 작성, 우측은 모델 응답 미리 보기 영역](../assets/images/s01-4/010_click.png)

**② 추출 규칙 지침 + `문서 입력` 변수 추가**

![지침 본문에 추출 규칙(빈칸 처리·날짜 포맷·액션 아이템 분리)을 적고 `문서 입력` 변수를 추가한 모습](../assets/images/s01-4/011_click.png)

**① 출력 형식 `텍스트` → `문서`**

![우측 상단의 출력 형식을 `텍스트` 에서 `문서` 로 바꾸면 `문서 설정` 버튼이 함께 노출](../assets/images/s01-4/012_click.png)

**② `문서 설정` 패널 열기**

![`문서 설정` 패널의 `문서 레이아웃 업로드` 영역 — Tier 4 양식 .docx 를 끌어서 놓거나 찾아보기로 업로드](../assets/images/s01-4/013_click.png)

**③ Tier 4 양식 업로드 → 필드 자동 인식 확인**

![`회의록_템플릿_Tier4.docx` 업로드 직후 — 10 식별된 필드(title · meetingDate · location · attendees · agenda · decisions · nextMeeting · actionItems 테이블 등)가 자동 인식](../assets/images/s01-4/014_click.png)

**① 샘플 PDF 올리고 `테스트` 실행**

![`문서 입력` 변수의 샘플 데이터로 `회의록_샘플1_줄글형식.pdf` 를 업로드하고 우측 상단 `테스트` 버튼으로 동작 확인](../assets/images/s01-4/015_click.png)

**② 출력 검증 — .docx 다운로드 + 필드 치환 확인**

{% raw %}
![테스트 실행 직후 우측 모델 응답 영역 — `출력.docx` 다운로드 링크와 함께 {{title}}·{{meetingDate}}·{{location}}·{{attendees}}·{{agenda}}·{{decisions}}·{{nextMeeting}}·{{actionItems}} 가 실제 값으로 치환된 요약 표시](../assets/images/s01-4/016_click.png)
{% endraw %}

**③ `저장` 으로 프롬프트 확정**

![응답 하단을 스크롤하면 실행 소요 시간·Copilot 크레딧 표시 — 하단 `저장` 버튼으로 프롬프트 확정](../assets/images/s01-4/017_click.png)

---

## Step 4-3. Power Automate 흐름 — `meetingnote_AI`

흐름은 최종적으로 **5개 노드** 로 구성됩니다: 트리거 → 프롬프트 실행 → OneDrive 파일 만들기 → OneDrive 공유 링크 만들기 → 에이전트에게 응답. 시작·끝 노드 먼저 놓고 중간에 차례로 끼워 넣는 순서로 진행합니다.

### 1) 흐름 생성 · 트리거 선택 · 응답 노드 자리 잡기

Copilot Studio 좌측 메뉴의 에이전트 흐름 목록에서 새 흐름을 시작하고, 트리거는 에이전트 호출에 맞춘 V2 트리거를 고릅니다. 마지막에 놓을 응답 노드(`에이전트에게 응답`)를 먼저 올려두고, 중간의 ⊕ 를 활용해 프롬프트·파일 저장·링크 액션을 차례로 끼워 넣을 계획입니다.

**① 새 에이전트 흐름 만들기**

![Copilot Studio 에이전트 흐름 목록 화면 — 우측 상단 `+ 새 에이전트 흐름` 클릭](../assets/images/s01-4/002_click.png)

**② 트리거 `에이전트가 흐름을 호출할 때` 선택**

![트리거 추가 다이얼로그에서 `에이전트` 검색 → `에이전트가 흐름을 호출할 때` 선택](../assets/images/s01-4/003_click.png)

**③ 빈 캔버스 — ⊕ 로 첫 작업 추가**

![트리거 노드만 놓인 빈 캔버스 — 아래 ⊕ 버튼으로 첫 작업 추가](../assets/images/s01-4/004_click.png)

**④ 마지막 응답 노드(`에이전트에게 응답`) 먼저 자리 잡기**

![작업 추가 패널에서 `에이전트` 검색 → `에이전트에게 응답` 선택 — 마지막 응답 노드 먼저 자리 잡기](../assets/images/s01-4/005_click.png)

### 2) `프롬프트 실행` 액션 끼워 넣기

트리거와 응답 노드 사이의 ⊕ 를 눌러 **AI 기능 → 프롬프트 실행** 을 넣습니다. 이 액션이 Step 4-2 에서 만든 `meetingnote_AI` 프롬프트를 호출해 "추출 + 양식 채우기" 를 **한 번에** 처리합니다.

**① 트리거-응답 사이의 ⊕ 클릭**

![트리거와 `에이전트에게 응답` 사이의 ⊕ 클릭](../assets/images/s01-4/006_click.png)

**② `AI 기능 → 프롬프트 실행` 선택**

![작업 추가 패널의 `AI 기능` 섹션에서 `프롬프트 실행` 선택](../assets/images/s01-4/007_click.png)

### 3) 트리거 입력 `meetingfile` 정의

프롬프트에 넘겨줄 파일을 에이전트로부터 받기 위해 트리거 노드를 다시 열고 **입력 추가** 로 `meetingfile` 파일 매개변수를 만듭니다. 타입은 `파일 또는 이미지`.

**① 트리거 노드 다시 열기**

![5개 노드 구성 직전 상태에서 트리거 노드 다시 클릭 — 매개 변수 패널 열기](../assets/images/s01-4/018_click.png)

**② 입력 `meetingfile` (파일 또는 이미지) 추가**

![트리거 매개 변수 탭에서 `입력 추가` → 이름 `meetingfile`, 타입 `파일 또는 이미지`](../assets/images/s01-4/019_click.png)

### 4) `프롬프트 실행` 매개 변수 연결

프롬프트 실행 노드를 열고 `meetingnote_AI` 선택, `문서 입력` 칸에는 트리거의 `meetingfile contentBytes` 동적 콘텐츠를 넘겨줍니다.

**① 프롬프트 실행 노드 클릭**

![프롬프트 실행 노드 클릭 — 우측 매개 변수 패널이 열림](../assets/images/s01-4/020_click.png)

**② 프롬프트 `meetingnote_AI` + `문서 입력` ← `meetingfile contentBytes`**

![프롬프트 드롭다운 `meetingnote_AI` 선택 후 `문서 입력` 칸에 동적 콘텐츠로 `meetingfile contentBytes` 선택](../assets/images/s01-4/021_click.png)

**③ 매개 변수 연결 완성**

![매개 변수 연결 완성 — `문서 입력` 칸에 `meetingfile cont…` 칩이 들어감](../assets/images/s01-4/022_click.png)

### 5) OneDrive `파일 만들기` 추가

프롬프트의 `Document Output Content Bytes` 가 완성된 .docx 바이너리입니다. 이걸 OneDrive 에 저장하는 게 다음 단계. 프롬프트 실행과 응답 노드 사이 ⊕ 에 `파일 만들기` 액션을 끼워 넣습니다.

**① 프롬프트 실행-응답 사이 ⊕**

![프롬프트 실행과 응답 사이의 ⊕ 클릭 — 이 자리에 OneDrive 파일 저장 끌어 넣을 예정](../assets/images/s01-4/023_click.png)

**② `파일 만들기` (비즈니스용 OneDrive) 선택**

![작업 추가 패널에서 `파일만` 검색 → `비즈니스용 OneDrive` 그룹의 `파일 만들기` 선택](../assets/images/s01-4/024_click.png)

**③ `파일 콘텐츠` ← `Document Output Content Bytes`**

![`파일 콘텐츠` 칸에 동적 콘텐츠로 `프롬프트 실행` 그룹의 `Document Output Content Bytes` 선택](../assets/images/s01-4/025_click.png)

**④ 폴더·파일 이름·콘텐츠 세 필드 완성**

![파일 만들기 세 필드 완성 — 폴더 경로 `/`, 파일 이름 `AI회의록_<utcNow()>.docx`, 파일 콘텐츠 = `Document Output…`](../assets/images/s01-4/026_click.png)

### 6) `공유 링크 만들기` 추가

저장된 파일의 공유 가능한 URL 을 받아서 에이전트 응답에 넘길 차례. 다시 ⊕ 로 다음 액션을 끼워 넣습니다.

**① 파일 만들기-응답 사이 ⊕**

![파일 만들기와 응답 사이의 ⊕ 클릭 — 링크 생성 액션 끼워 넣을 자리](../assets/images/s01-4/027_click.png)

**② `공유 링크 만들기` (비즈니스용 OneDrive) 선택**

![작업 추가 패널에서 `링크` 검색 → `비즈니스용 OneDrive` 그룹의 `공유 링크 만들기` 선택](../assets/images/s01-4/028_click.png)

**③ `파일` ← 파일 만들기 액션의 `ID`**

![`파일` 칸에 동적 콘텐츠로 `파일 만들기` 그룹의 `ID` (파일의 고유 식별자) 선택](../assets/images/s01-4/029_click.png)

**④ 링크 유형 `Edit` · 범위 `Anonymous` 로 완성**

![공유 링크 만들기 완성 — `파일 = ID`, `링크 유형 = Edit`, `링크 범위 = Anonymous`](../assets/images/s01-4/030_click.png)

{: .warning }
> 교육용 테넌트에서는 빠른 테스트를 위해 `Anonymous` 범위를 사용할 수 있지만, 실제 업무 환경에서는 조직 정책에 맞춰 `Organization` 또는 지정 사용자 범위를 우선 검토하세요.

### 7) 응답 노드 출력 연결 · 게시 · 흐름 이름 저장

마지막으로 `에이전트에게 응답` 노드에 출력 2개를 연결합니다 — 파일 이름과 공유 링크. 이걸 연결하기 전에는 응답 노드에 `⚠ 잘못된 매개 변수` 경고가 떠 있습니다.

**① 응답 노드의 `⚠ 잘못된 매개 변수` 경고**

![응답 노드가 `⚠ 잘못된 매개 변수` 경고 상태 — 아직 출력을 연결하지 않아 발생](../assets/images/s01-4/031_click.png)

**② 출력 `파일이름` · `파일링크` 두 개 추가**

![응답 노드의 매개 변수 탭 — `파일이름` (이름 동적) · `파일링크` (웹 URL 동적) 두 출력 추가](../assets/images/s01-4/032_click.png)

경고가 사라지면 5개 노드가 모두 그린 체크 상태로 돌아옵니다. 게시 후 흐름 이름을 `meetingnote_AI` 로 확정합니다.

**③ 5개 노드 완성 → `게시`**

![5개 노드(트리거 → 프롬프트 실행 → 파일 만들기 → 공유 링크 → 응답)가 경고 없이 완성된 흐름 — 우측 상단 `게시` 클릭](../assets/images/s01-4/033_click.png)

**④ 흐름 이름 `meetingnote_AI` 로 저장**

![게시 직후 개요 화면 — `편집` 으로 세부 정보 패널을 열고 흐름 이름을 `meetingnote_AI` 로 지정, 아래 `저장` 클릭](../assets/images/s01-4/034_click.png)

---

## 백엔드 완성 — 다음 페이지에서 에이전트와 연결

Word 템플릿(`{{}}` 자리표시자), AI 빌더 프롬프트(`meetingnote_AI`), Power Automate 흐름(`meetingnote_AI`)까지 — Tier 4 백엔드가 완성됐습니다. 흐름 단독 테스트에서 .docx 가 OneDrive 에 생성되는 것까지 확인했다면 백엔드는 끝.

다음 페이지에서는 이 흐름을 에이전트가 부를 수 있게 도구로 등록하고 지침을 다듬어, 채팅에서 실제 PDF 를 첨부해 동작을 확인합니다.

---

## 다음 페이지

[실습④-B — Tier 4 에이전트 연결 (도구 + 테스트)](s01-4b-tier4-agent)

<!-- build-trigger: 2026-05-11 -->
