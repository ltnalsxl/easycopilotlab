---
title: "S5. MCP"
parent: "📗 심화과정"
nav_order: 6
has_children: true
---

# 도구 — MCP (Model Context Protocol)
{: .no_toc }

| 시간 | 소요 | 수강생 역할 |
|:-----|:-----|:-----------|
| 13:45 | 25분 | 🟢 직접 실습 |

## 목차
{: .no_toc .text-delta }

1. TOC
{:toc}

![S5 MCP — 만능 어댑터 하나로 외부 연결](../assets/images/s08/hero.png)

---

## 이 모듈에서 배우는 것

- **MCP(Model Context Protocol)**가 무엇인지
- 커스텀 커넥터(S4)와의 차이 — 언제 어느 쪽을 쓸지
- **Microsoft Learn MCP** 서버를 에이전트에 연결하는 실습

{: .highlight }
> S4 커스텀 커넥터가 “API 한 개를 도구로 만드는” 길이라면, MCP는 “도구 한 묶음을 통째로 연결”하는 길입니다. 같은 외부 연결이라도 결이 다릅니다.

---

## MCP란

**Model Context Protocol** — AI 모델이 외부 서비스·도구와 통신하기 위한 **표준 규격**입니다.

| 커스텀 커넥터 (S4) | MCP (이번 S5) |
|:--|:--|
| API **한 개**마다 OpenAPI로 등록 | MCP 서버 **하나**가 여러 도구를 한 번에 노출 |
| Microsoft 생태계 친화적 | OpenAI·Anthropic 등 **AI 산업 표준** |
| 인증·스키마를 직접 잡음 | 서버가 자기 도구를 **자동 광고** |
| 사내 시스템·간단한 API에 적합 | 서드파티 서비스(GitHub·Notion·Learn 등)의 **공식 MCP**가 있을 때 적합 |

---

## 어떤 길을 언제 쓰나

```
외부 시스템을 도구로 쓰고 싶다
    │
    ├─ 공식 MCP 서버가 있는가? ── YES ──→ MCP 연결 (S5)
    │                                     · 도구 자동 등록
    │                                     · 인증·스키마 자동
    │
    └─ 없거나 사내 API ────────────────→ 커스텀 커넥터 (S4)
                                          · OpenAPI 작성/import
                                          · 인증 직접 설정
```

---

## 실습으로 가기

{: .important }
> 👉 [실습 — MCP 서버 연결](s05-1-mcp-connect)

이 실습에서는 **Microsoft Learn MCP** 서버를 연결합니다. URL 한 줄만 입력하면 `microsoft_docs_search`, `microsoft_docs_fetch`, `code_sample_search` 같은 도구가 **자동으로 추가**됩니다.

---

## 핵심 정리

1. MCP = 외부 서비스를 에이전트 도구로 연결하는 **표준 프로토콜**
2. 서버 URL 한 줄로 **여러 도구가 한 번에** 등록됨
3. 공식 MCP가 있으면 MCP, 없으면 커스텀 커넥터(S4)
4. MCP 도구도 **Description**과 **지침**이 채택률을 좌우 (S4와 동일)

---

다음 모듈: [S6. Excel 케이스 스터디](s06-excel-case)

<!-- build-trigger: 2026-05-10 -->
