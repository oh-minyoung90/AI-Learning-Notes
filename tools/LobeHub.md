# **LobeHub Agent 플랫폼 벤치마킹**

AI 앱을 Skill 기반 공통 플랫폼으로 전환하기 위한 참고 자료로, **LobeHub의 Agent 중심 챗 플랫폼 구조와 사용자 경험을 조사**함.

## **1. LobeHub 개요**

**LobeHub는 여러 AI 모델과 Agent, Skill, Tool, Knowledge Base, Memory를 하나의 챗 환경에서 사용할 수 있는 오픈소스 AI Agent 플랫폼이다.**

Agent는 역할, 모델, Skill, 지식 등을 미리 설정해 반복적으로 사용하는 업무 단위로 구성된다.

---

## **2. Agent 생성 및 사용**

### **2.1 Agent 생성 및 사용**

**새로운 Agent를 직접 만들거나, Agent Builder를 이용해 자연어로 기본 설정을 생성할 수 있다.** 사용자는 Agent를 선택한 후 일반 채팅처럼 질문하거나 파일을 첨부한다.

Agent는 연결된 모델, Skill, Tool, Knowledge Base를 활용해 응답하며, 사용자는 내부 실행 방식을 알 필요 없이 자연어로 사용할 수 있다.

Agent Profile

#### 설정 항목

```
Agent 생성
→ 이름, 아이콘 설정
→ 역할과 설명 설정 (프롬프트)
→ AI 모델 선택
→ Skill·Tool 추가
→ Knowledge Base·Memory 연결
→ 저장 후 대화 시작
```

### 2-2. Agent Advanced Setting

Agent의 대화 처리·응답 동작을 세밀하게 조정하는 고급 설정 화면이다.

대화 이력 제어 + UX 옵션 + 입력 전처리 + 보조 모델 + 모델별 상세설정

| 설정 | 하는 일 |
| --- | --- |
| **Auto-compress Context** | 대화가 길어지면 컨텍스트를 자동 압축해 토큰 절약 |
| **Limit History Messages (20)** | 현재 대화에서 **이전 메시지 몇 개**를 문맥으로 가져올지 |
| **Auto-scroll During AI Response** | 응답 생성 중 화면 자동 스크롤 (UX) |
| **Enable Streaming Output** | 답변을 한 번에 말고 **실시간 스트리밍**으로 표시 |
| **Follow-up Suggestions** | 답변 후 이어질 질문 **추천** 제공 |
| **User Input Preprocessing** | 입력을 모델에 보내기 전 **템플릿으로 전처리** ({{text}} 치환) |
| **Sub-Agent Model** | 보조 작업(요약·follow-up 등)에 쓸 **별도 모델** 지정 (지금 DeepSeek V4 Flash) |
| 모델 별 설정 |  |
| **Enable Context Caching** | 컨텍스트 캐싱으로 **비용 최대 90%↓, 속도 4배↑** (반복되는 앞부분을 캐시) |
| **Effort** (low/medium/high/xhigh/max) | 답변할 때 **모델이 쓸 토큰량·노력 수준** 조절 (지금 high) |

---

## 3. Skill

**Skill은 Agent에 기능을 더하는 단위**(웹 검색·코드 실행·DB 접근·외부 API 등)이고, **Tool은 Skill이 실제 작업 시 호출하는 실행 수단**이다. MCP는 외부 Tool·서비스를 연결하는 표준 프로토콜이다.

워크스페이스 전체 Skill 관리(설치/제거): Setting-Skills

이 Agent에 사용할 Skill 적용: Agent Profile-Skills

대화 중 Skill 설정: +버튼-Skills

현재 이 대화의 Skill 확인 (Built-in 제외한 Skill 전부)

LobeHub는 Skill 등록·활성화·확인을 여러 진입점(전역 Settings, 채팅 인라인, Agent 프로필, 대화 패널)에서 제공한다. 추가는 Skill Store에서, 활성화는 Agent별 토글로, 확인은 대화 패널에서 이뤄진다.

### 3.1 Skill의 유형

출처에 따라 5가지로 나뉘며, 외부 확장은 대부분 MCP로 연결된다.

| 유형 | 예시 |
| --- | --- |
| Built-in tools | 내장 기능 (Artifacts, Sandbox, GTD, Notebook, Memory) |
| LobeHub Integrations | OAuth 연결형 자체 통합 (Linear, Microsoft 등) |
| Composio | Composio 기반 통합 (Gmail, Notion, Slack 등) |
| Community MCP | 마켓플레이스 MCP 서버 (GitHub, PostgreSQL 등) |
| Custom MCP | 직접 추가하는 MCP 서버 (사내 API 등) |

### 3.2 등록·설치 흐름

#### Skill Store 진입

- Settings → Skills
- Agent 설정 → Skills
- 채팅의 Tools 버튼

### 3.3 실행·호출 방식

- Agent가 요청에 따라 언제 Tool을 호출할지 스스로 판단한다(모델 자동 호출). 사용자는 Tool 사용법을 몰라도 자연어로 요청한다.
- Chain of Thought로 호출 Tool·파라미터를 추적할 수 있고, 권한은 연결 레벨·Tool 레벨로 통제한다.

### 3.4 Skill 포맷

LobeHub의 Skill은 **Agent Skills 오픈 표준(**http://SKILL.md **)** 을 따른다. LobeHub 고유 형식이 아니라 여러 에이전트(Claude Code·Cursor·GitHub Copilot 등)가 공유하는 표준이라, 외부에서 만든 Skill도 그대로 가져올 수 있다.

**기본 구조** — 최소 http://SKILL.md  파일 하나가 든 폴더이며, 폴더명은 frontmatter의 name과 일치해야 한다.

```
my-skill/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
├── assets/           # Optional: templates, resources
└── ...               # Any additional files or directories
```

http://SKILL.md  **구성** — YAML frontmatter(메타데이터) + 마크다운 본문(절차·규칙).

| 필드 | 필수 | 규칙 |
| --- | --- | --- |
| name | 필수 | 소문자·숫자·하이픈, 최대 64자, 폴더명과 일치 |
| description | 필수 | 최대 1,024자. "무엇을 + 언제 쓰는지" 명시
단순 소개가 아니라 자동 라우팅 트리거다. 모델은 시작 시 name·description만 보고 이 Skill을 쓸지 판단하므로, "언제 사용하는지"를 구체적으로 써야 원하는 상황에서 호출된다. |
| license / metadata / allowed-tools | 선택 | 필요 시 추가 |

**예시**

```
name: doc-summarizer
description: 사내 문서를 요약하거나, 문서 근거로 질문에 답하거나, 핵심을 표로 정리할 때 사용한다.

문서 요약 도우미 절차
핵심을 3~5문장으로 요약한다.
항목이 여럿이면 표로 정리한다.
문서에 없는 내용은 "문서에 없음"이라고 답한다.
```

### 3.5 Skill / Tool / MCP 관계

```
filesystem MCP를 붙였다고 하면:

Skill = "filesystem" (파일 다루는 기능 묶음)
Tool = read_file(파일 읽기), list_dir(목록 보기), search(검색) ← 이 Skill이 노출하는 개별 함수들
MCP = 이 filesystem 서버를 LobeHub에 연결한 방식/규격
```

---

## **4. 모델 선택 및 사용**

**여러 AI Provider의 모델을 하나의 UI에서 사용할 수 있다.**

모델 Provider 관리 화면

#### 지원 예시

- OpenAI / Anthropic / Google / AWS Bedrock / Azure OpenAI / Ollama / OpenAI 호환 API

Agent마다 기본 모델을 지정하고, 필요하면 대화 중 변경할 수 있다.

```
Provider 연결
→ 사용 모델 활성화
→ Agent 기본 모델 지정
→ 대화에서 모델 사용 또는 변경
```

---

## **5. 대화 이력 및 Memory**

LobeHub는 **현재 대화의 문맥**과 **여러 대화에서 활용하는 Memory**를 구분한다.

Memory 관리 - Agent Memory

| **구분** | **설명** |
| --- | --- |
| 대화 이력 | 현재 대화의 이전 질문과 답변을 문맥으로 활용해 대화를 이어나감. |
| Agent Memory | 대화 내용을 그대로 저장하지 않고, 지속적으로 활용할 정보(사용자 선호·반복 작업 방식·출력 형식·이전 결정)를 구조화해 저장함. |

---

## **6. 첨부문서, Knowledge Base 및 Resource Library**

**현재 대화에서 사용하는 첨부파일과 반복적으로 활용하는 지식 문서를 구분해 관리한다.**

| **구분** | **설명** |
| --- | --- |
| 대화 첨부파일 | 현재 질문이나 대화에서 분석할 파일 |
| Knowledge Base | Agent가 지속적으로 검색하고 참고하는 문서 모음 (Agent 단위 연결) |
| Resource Library | 업로드 문서와 AI 생성 결과물을 보관하고 재사용하는 자료 공간 |

Knowledge Base

Resource Library

---

## **7. Artifact 및 문서 제공**

LobeHub는 결과물을 성격에 따라 두 갈래로 보여준다

- AI 생성물을 렌더링하는 **Artifact**
- 업로드·원본 파일을 여는 뷰어 **Portal**.

### **7.1 Artifact**

Artifact는 Agent가 대화 과정에서 생성한 별도 결과물이다.

- 이미지 / 코드 / 다이어그램 / 문서 / 오디오 / 기타 시각적 콘텐츠

```
질문
   ↓
Agent 실행
   ↓
Artifact 생성
(PDF, 이미지, 코드, 문서...)
   ↓
필요 시 Resource Library에 저장
   ↓
다음 대화나 Agent에서 재사용
```

### **7.2 Portal**

Portal은 채팅 옆 패널에서 원본 파일을 여는 미리보기 영역으로, PDF·Excel·Word·PPT·TXT 등을 인터페이스 내에서 바로 열람한다. 파일을 클릭하면 Portal에서 열리고, 채팅 창에서 파일 내용과 대화 관련 텍스트 블록을 함께 볼 수 있다.

### **7.3 Pages**

Pages는 긴 문서를 채팅과 분리해 작성하고 편집하는 기능이다. Markdown, 실시간 미리보기, 자동 저장을 지원하며, Agent가 문서 내용을 직접 수정하거나 보완할 수 있다.

---

## **8. 벤치마킹 및 시사점**

**LobeHub는 모델·Skill·Tool·Memory·Knowledge Base를 Agent 단위로 묶어 하나의 챗 UI에서 사용하는 구조를 제공한다. 기능 자체보다 사용자가 Agent와 Skill을 생성·발견·연결해 사용하는 경험(UI/UX)을 참고하기에 적합하다.**

항목별로 우리 플랫폼에 참고할 포인트를 정리하면 다음과 같다.

| **항목** | **벤치마킹 포인트** |
| --- | --- |
| Agent | • 역할·모델·Skill·문서·Memory를 하나의 단위로 통합 관리
• 자연어 생성(Agent Builder)과 직접 생성을 함께 제공
• Agent 목록·선택 UX |
| Skill / Tool | • 유형(내장/통합/MCP)으로 구분하고 확장은 MCP 표준으로 흡수
• 설치↔Agent별 활성화의 2단 구조
• Tool 사용법을 숨기고 자연어로 자동 호출
• 연결/Tool 레벨 2단 권한 제어와 실행 추적(CoT) |
| 모델 | • 여러 Provider를 동일 UI에서 선택
• Agent별 기본 모델 지정
• 대화 중 변경해도 같은 챗 UX 유지 |
| Memory | • 대화 문맥과 장기 Memory를 분리
• 필요한 정보만 추출해 구조화 저장
• Agent별 설정과 사용자 확인·관리 |
| 문서 | • 일회성 첨부파일 / Knowledge Base / Resource Library의 역할 구분
• KB의 Agent 단위 연결
• 생성물까지 한 자료 공간에서 재사용 |
| Artifact / Portal | • 생성 결과물(Artifact)과 원본 뷰어(Portal)를 분리
• 렌더링 대상과 코드 표시를 타입으로 구분
• 채팅·Artifact·Portal·Pages·Resource Library의 연계 구조 |