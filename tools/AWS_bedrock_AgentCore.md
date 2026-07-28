# AWS Bedrock AgentCore 벤치마킹

#### Skill 기반 공통 플랫폼 적용 관점

## 1. **AgentCore 개요**

**AgentCore는 모델·프롬프트·Skill·Tool·Memory를 설정(Config) 기반으로 관리하고 실행 시 조합하는 관리형 Agent 플랫폼이다.** Agent 실행은 Harness를 중심으로 이루어지며, 필요에 따라 Memory·Gateway·Registry·Code Interpreter·Browser를 연결한다. **Harness에 기본 구성을 설정하고, 실행 시 Override로 일부를 변경한다.**

본 문서는 개별 앱을 Skill 기반 공통 플랫폼으로 전환하는 관점에서 Harness(실행·설정), Memory(대화·기억), Gateway(Tool·API 연동), Registry(등록·검색), Code Interpreter/Browser(Artifact 생성)를 분석한다.

---

## 2. Agent · Skill 생성 및 사용

### 2.1 Harness

**AgentCore에서 Agent는 Harness 리소스로 관리**된다. Harness 생성 시 기본 모델, 프롬프트, Tool, Memory를 설정하며, 실행 시에는 호출 단위로 구성을 변경할 수 있다. 이 구조를 사용하면 Agent 코드를 다시 배포하지 않고도 모델, 프롬프트, Skill, Tool 구성을 변경할 수 있다.

| **단계** | **API** | **역할** |
| --- | --- | --- |
| 생성 | CreateHarness | 기본 모델·프롬프트·Tool·Memory 설정 |
| 수정 | UpdateHarness | Harness 기본 설정 변경 |
| 실행 | InvokeHarness | 메시지 전달 및 호출별 Override 적용 |

### 2.2 Skill

**Skill은 에이전트의 업무 수행에 필요한 지침, 스크립트, 참고자료를 묶은 파일 기반 실행 지식 패키지**다.

기본적으로 SKILL.md를 포함하며, 필요에 따라 실행 코드와 참고자료를 함께 구성한다.

```
my-skill/
├── SKILL.md          # 필수: 메타데이터 및 실행 지침
├── scripts/          # 선택: 실행 코드
├── references/       # 선택: 참고 문서
├── assets/           # 선택: 템플릿 및 리소스
└── ...
```

**Skill 등록(소스) 4가지 방식**

Skill은 저장 및 배포 방식에 따라 다음 네 가지 소스로 등록할 수 있다.

| 소스 | 설명 |
| --- | --- |
| AWS Skills | AWS가 제공하는 사전 구축 Skill |
| Git | Git 저장소에서 Skill을 가져옴 |
| Amazon S3 | S3에 저장된 Skill을 실행 시 가져옴 |
| Path | 실행 환경의 파일시스템에 포함된 Skill 사용 |

### 2.3 Tool

**Tool은 에이전트가 실제 작업을 수행하기 위해 호출하는 실행 기능**이다.

Skill이 업무 수행 방법과 지식을 정의한다면, Tool은 API 호출, 코드 실행, 파일 처리 등 실제 동작을 담당한다.

| **Tool 유형** | **역할** |
| --- | --- |
| Remote MCP | 원격 MCP 서버의 Tool 연결 |
| Gateway | API·MCP 연결과 인증·정책 관리 |
| Browser | 웹 탐색 및 웹 애플리케이션 조작 |
| Code Interpreter | 코드 실행 및 파일 처리 |
| Inline Function | 사용자 승인이나 클라이언트 업무 로직 처리 |
| 기본 Tool | Shell 및 파일 조작 |

### 2.4 Gateway

**Gateway는 API, Lambda, MCP 서버, 다른 Agent와 모델을 하나의 보안 Endpoint로 연결하는 계층이다.**

기존 API와 Lambda를 Agent가 호출할 수 있는 MCP Tool 형태로 변환하고, 인증·권한·라우팅·프로토콜 변환을 공통으로 처리한다.

| **기능** | **설명** |
| --- | --- |
| Tool 변환 | 기존 API와 Lambda를 Agent가 호출할 수 있는 MCP Tool로 변환 |
| 통합 연결 | MCP 서버, HTTP 서비스, 다른 Agent, 모델을 단일 Endpoint로 연결 |
| 인증·권한 관리 | Gateway 접근 인증과 외부 서비스 호출 자격증명을 중앙 관리 |
| Semantic Tool Selection | 자연어 질의를 기반으로 현재 업무에 필요한 Tool을 검색 |
| 관리형 실행 | Tool 호출, 라우팅, 로깅과 감사 기능을 공통으로 제공 |

### 2.5 Registry

**Registry는 Agent, MCP 서버, Tool, Skill을 중앙에서 등록·검색·승인·재사용하기 위한 카탈로그**다. Registry를 활용하면 프로젝트별로 흩어진 Skill과 Tool을 공통 자산으로 관리할 수 있다.

> Agent Registry는 조사 시점 기준 Preview 기능이다.
> 

---

## 3. 모델 선택 · 사용

### **3.1 Config 기반 모델 관리**

**Harness 생성 시 기본 모델·프롬프트·Tool을 설정하고, 실행 시 호출 단위로 Override할 수 있다.** 따라서 애플리케이션을 재배포하지 않고도 구성을 변경할 수 있다.

- 모델 · 프롬프트 · Tool · Skill · 추론 설정

모델을 Agent 코드에 직접 고정하지 않고 설정으로 분리하는 구조다.

### 3.2 모델 Provider 연결

Bedrock(Claude·Nova 등), OpenAI(GPT), Google Gemini를 공통 방식으로 연결한다. **LiteLLM / OpenAI 호환 Endpoint**를 활용하면 외부 모델은 물론 사내 모델 서버도 동일 인터페이스로 연결하며, **같은 세션 내 모델 변경 시에도 컨텍스트를 유지**한다.

### 3.3 API 포맷

호출 포맷(Bedrock Converse / OpenAI Responses / Chat Completions)은 Config에서 선택한다. 플랫폼이 모델별 호출 차이를 공통 인터페이스로 감싸므로, Agent는 동일한 방식으로 모델을 사용한다.

### 3.4 **자격증명 및 Guardrail**

API Key·외부 자격증명은 **AgentCore Identity의 Token Vault**에 저장하고, Harness가 실행 시 참조한다(코드에서 원시 Key를 직접 관리하지 않음). Bedrock Guardrails를 Config에 포함해 유해 콘텐츠·금지 주제·입출력 정책·보안 규칙을 제어한다.

---

## 4. 메모리 및 첨부문서

### **4.1 대화 이력**

AgentCore는 대화 이력을 Memory에 자동 저장한다.

동일한 Session으로 호출하면 이전 대화를 자동으로 불러오므로 애플리케이션이 전체 이력을 다시 전달할 필요가 없다.

| 구분 | 종류 | 역할 |
| --- | --- | --- |
| Short-term | — | 현재 세션 대화 유지 |
| Long-term | Semantic / Summarization / User Preference / Episodic | 사실·요약·선호·주요 이벤트를 세션 넘어 저장 |

구성 방식은 **Managed**(자동 관리), **BYO**(기존 인스턴스 연결), **Disabled** 중 선택한다.

### **4.2 첨부문서**

첨부문서는 Memory가 아닌 파일시스템과 영속 Storage에 저장한다.

| **성요소** | **역할** |
| --- | --- |
| MicroVM File System | 세션 내 임시 파일 |
| Session Storage | 세션 유지 |
| Amazon EFS | 여러 Agent 간 공유 |
| Amazon S3 | 영구 저장 |

문서 분석과 변환은 Code Interpreter가 수행하며, 필요한 파일은 실행 전에 Runtime Command로 준비할 수 있다.

#### 첨부문서 저장 및 처리

```
첨부문서
   │
MicroVM File System        (세션 내 임시 파일)
   │
Code Interpreter           (분석·변환)
   │
Session Storage / EFS / S3 (영속 저장·공유)
```

---

## 5. 아티팩트 제공

AgentCore는 텍스트 응답과 파일 아티팩트를 분리해 처리한다. PDF, Excel, 이미지 등의 파일은 Code Interpreter가 생성하고 Storage에 저장한다. 애플리케이션은 반환된 파일 참조 정보와 메타데이터를 이용해 사용자에게 파일을 제공한다.

### **5.1 스트리밍**

InvokeHarness는 텍스트, Tool 결과, 진행 상태, 사용량, 오류를 이벤트 스트림으로 전달한다.

| **이벤트** | **설명** |
| --- | --- |
| Message | 메시지 시작 및 종료 |
| Content | 텍스트, Tool 호출, Tool 결과 등의 증분 데이터 |
| Metadata | 토큰 사용량, 지연 시간 |
| Runtime Error | 실행 오류 |

### **5.2 아티팩트 생성 및 저장**

```
사용자 요청
   ↓
InvokeHarness
   ↓
Code Interpreter
   ↓
PDF / Excel / Image 생성
   ↓
Session Storage / S3 / EFS  # 세션 내 임시 저장 / 장기 저장 및 다운로드 / 여러 Agent 간 파일 공유
   ↓
Artifact URI 및 메타데이터 반환
   ↓
애플리케이션 Viewer 또는 다운로드 제공
```

AgentCore는 파일 생성·저장까지 담당하고, **PDF Viewer 등 최종 UI는 제공하지 않는다**(애플리케이션 구현 몫). Browser는 웹 자동화·정보 수집용이며 Viewer 기능이 아니다.

---

## 6. CLI 데모

AgentCore harness CLI 데모

!AgentCore harness CLI 데모

AgentCore harness CLI 데모

1. agentcore create 실행 → 프로젝트 생성 마법사 시작
2. 프로젝트 이름 입력
3. Harness 구성 (모델·API 형식·실행 환경 등 선택)
4. 실행 환경 선택 (기본 환경 / 컨테이너 이미지 / Dockerfile)
5. cd로 프로젝트 폴더 이동 후 agentcore 재실행
6. 빌드·검증 (CDK 빌드, CloudFormation 합성 등 [done])
7. AWS 배포 (IAM Role·Policy 등 리소스 생성 완료)
8. 완료 후 프롬프트 복귀 → 처음부터 반복 재생

## 7. **벤치마킹 및 시사점**

AgentCore의 핵심은 **모델·Skill·Tool·Memory를 독립 리소스로 관리하고 실행 시 조합**하는 데 있다. Gateway가 외부 연결·인증을, Registry가 등록·검색·승인을 공통화하며, Memory와 첨부문서·아티팩트는 별도 Storage로 분리된다. 즉 완성된 업무 UI라기보다 **여러 Agent 앱이 공유하는 실행·연결·메모리·파일 처리 기반**에 가깝다.

| 과제 | AgentCore 대응 | 벤치마킹 포인트 |
| --- | --- | --- |
| 개별 앱 → 공통 플랫폼 전환 | Harness | 모델·프롬프트·Skill·Tool·Memory를 Config로 관리, 
**생성·실행 분리** 후 실행 시 조합 |
| Agent·Skill 생성·재사용 | Harness Skills + Registry | 표준 파일 번들 등록, 검색·승인·재사용, 버전 관리 |
| Tool·외부 서비스 통합 | Gateway | 공통 Tool 형식 연결, 인증·권한 중앙 관리 |
| 모델 선택·전환 | Model Config | 코드에 고정하지 않고 Provider·호출별 전환, **사내 모델도 동일 인터페이스** |
| 대화 이력·사용자 기억 | Memory | 세션·사용자 단위 이력, **장기 기억은 필요한 정보만 추출·저장** |
| 첨부문서·파일 처리 | File System + Storage + Code Interpreter | Memory와 파일 저장 분리(**수명주기·접근 권한 분리**), 실행 도구가 문서 처리 |
| 아티팩트 제공 | Streaming + Code Interpreter + Storage | 생성·저장·참조 분리, URI·메타데이터 반환, Viewer는 애플리케이션 |
| 자격증명·보안 | Identity + Guardrails | API Key·정책을 코드 외부에서 중앙 관리 |
| 공통 자산 재사용 | Registry | Agent·Skill·Tool 중앙 카탈로그 관리 |
| Agent별 권한 | allowedTools + Identity | Agent마다 사용 가능한 Tool·리소스 범위 제한 |

---

## 7. 참고 문서

- Amazon Bedrock AgentCore
- Amazon Bedrock AgentCore Gateway
- Amazon Bedrock AgentCore Memory
- Amazon Bedrock AgentCore Runtime
- Amazon Bedrock AgentCore Identity
- Amazon Bedrock AgentCore Code Interpreter
- Amazon Bedrock AgentCore Browser
- AWS Agent Registry