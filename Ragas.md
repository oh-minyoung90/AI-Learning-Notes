# Ragas - LLM/RAG 평가 프레임워크 
## 목적

Ragas를 사용해서 LLM / RAG 응답의 품질을 수치로 측정한다.

## 프레임워크 비교

| 이름 | 설명 & 특징 | 측정 지표 | 동작 방식 |
| --- | --- | --- | --- |
| [**Ragas**](https://docs.ragas.io/en/stable/) | RAG 파이프라인의 각 단계를 독립적으로 정량 평가하는 RAG 특화 지표 패키지<br>• OSS (완전 무료)<br>• LLM-as-judge 방식으로 지표 계산 배치 평가<br>• 응답 속도 측정 없음 (별도 추가 필요)<br>• 데이터 외부 전송 없음 | • context_recall (Recall@K)<br>• context_precision<br>• answer_correctness<br>• faithfulness (환각 탐지)<br>• answer_relevancy | 입력: question + ground_truth + answer + contexts (청크 목록)<br>↓<br>LLM 평가자(gpt-4o 등)가 각 지표를 0~1 점수로 계산<br>↓<br>출력: 지표별 점수 + 전체 평균 |
| [**DeepEval**](https://deepeval.com/) | pytest 기반으로 LLM 앱을 단위 테스트처럼 검증하는 프레임워크<br>• pytest 완전 통합 → CI/CD 연결 쉬움<br>• threshold 기반 pass/fail 판정<br>• LatencyMetric 내장 (속도 기준 설정 가능)<br>• admin/qna 테스트 파일 분리 자연스러움 | • ContextualRecallMetric (Recall@K)<br>• ContextualPrecisionMetric<br>• AnswerRelevancyMetric<br>• FaithfulnessMetric<br>• HallucinationMetric<br>• LatencyMetric | 입력: question + ground_truth + answer + contexts + 응답 시간<br>↓<br>LLM 평가자가 각 지표를 0~1 점수로 계산<br>↓<br>threshold(기준값) 초과 여부로 PASS / FAIL 판정<br>↓<br>출력: 테스트 결과 리포트 (pytest 형식) |
| [**TruLens**](https://www.trulens.org/) | 앱 실행 중 실시간으로 품질·속도·비용을 자동 기록하는 모니터링 툴<br>• 기존 앱을 래핑하는 방식 (코드 수정 최소화)<br>• latency·token·비용 자동 기록<br>• 로컬 Streamlit 대시보드 제공<br>• LangGraph 직접 지원 없음<br>• Recall@K 지표 취약 | • context_relevance<br>• answer_relevance<br>• groundedness (환각 탐지)<br>• latency (자동)<br>• token / 비용 (자동) | 기존 앱을 TruLens로 감싸기 (래핑)<br>↓<br>API 호출 시 응답 시간·토큰 자동 측정<br>↓<br>LLM 평가자가 품질 지표를 0~1 점수로 계산<br>↓<br>출력: 로컬 대시보드에서 호출별 점수·속도·비용 시각화 |
| [**LangSmith**](https://smith.langchain.com/) | LangChain/LangGraph 앱의 전체 실행 흐름을 trace하고 단계별로 평가하는 관측 플랫폼<br>• LangGraph 공식 지원 (노드별 latency 분리 확인 가능)<br>• 환경변수 설정만으로 자동 trace<br>• 팀 협업·공유 가능한 웹 UI<br>• 데이터가 외부 서버로 전송됨 (보안 정책 확인 필요)<br>• Recall@K 등 RAG 특화 지표는 커스텀 구현 필요 | • trace latency (노드 단계별)<br>• token 사용량<br>• correctness (내장)<br>• 커스텀 평가자 | 환경변수 설정만으로 자동 trace 활성화<br>↓<br>graph.invoke() 실행 시 노드별 흐름 자동 기록(loader_start → qna_prep → qna_retriever → qna_llm)<br>↓<br>각 노드의 소요 시간·토큰 사용량 자동 측정<br>↓<br>출력: 웹 UI에서 단계별 latency·비용·품질 점수 확인 |

## Ragas 설치

### 세팅 구조

```
rag_eval/
├── rag.py          ← 샘플 RAG 시스템 
├── evals.py        ← 평가 실행 핵심 파일
└── evals/
    ├── datasets/   ← 테스트 케이스 CSV 저장 위치
    ├── experiments/← 평가 결과 CSV 저장 위치
    └── logs/       ← 실행 로그
```

### Ragas 동작 흐름

```
evals.py 실행
    ↓
load_dataset() → 테스트 질문 + 채점 기준 목록 로드
    ↓
run_experiment() → 각 질문을 RAG에 전달 → 응답 수집
    ↓
DiscreteMetric → LLM이 응답을 채점 기준과 비교 → pass / fail 판정
    ↓
결과를 evals/experiments/ 에 CSV로 저장
```

## Ragas 실행 방법

### Step1. Test Case 작성

evals.py 안의 load_dataset() 함수를 수정한다.

```
from ragas import Dataset

def load_dataset():
    """Load test dataset for evaluation."""
    dataset = Dataset(
        name="test_dataset",
        backend="local/csv",
        root_dir=".",
    )

    data_samples = [
        {
            "question": "What is Ragas?",
            "grading_notes": "Ragas is an evaluation framework for LLM applications",
        },
        {
            "question": "How do metrics work?",
            "grading_notes": "Metrics evaluate the quality and performance of LLM responses",
        },
        # Add more test cases here
    ]

    for sample in data_samples:
        dataset.append(sample)

    dataset.save()
    return dataset
```

| **항목** | **설명** | **예시** |
| --- | --- | --- |
| question | 테스트할 질문 | 복지 포인트가 얼마야? |
| grading_notes | 정답에 반드시 포함되어야 할 내용(채점 기준) | 연간 50만원 |

### Step2. 실제 프로젝트 API 연결

evals.py에서 RAG 호출 부분을 실제 프로젝트 API로 사용한다.

```
rag_client = default_rag_client(llm_client=openai_client, logdir="evals/logs")

@experiment()
async def run_experiment(row):
    response = rag_client.query(row["question"])   # ← 샘플 RAG 호출
```

### Step.3 평가 실행

```
cd rag_eval

uv run python evals.py
```

### Step4. 결과 확인 방법

#### 방법 1 - CSV 파일 직접 열기

```
rag_eval/evals/experiments/run_experiment_날짜시간.csv
```

#### 방법 2 - 코드로 분석

```
# evals.py 하단에 추가
import pandas as pd

csv_path = Path(".") / "evals" / "experiments" / f"{experiment_results.name}.csv"
df = pd.read_csv(csv_path)

# 전체 통과율
total   = len(df)
passed  = len(df[df["score"] == "pass"])
print(f"통과율: {passed}/{total} ({passed/total*100:.1f}%)")

# 실패 케이스만 출력
failed = df[df["score"] == "fail"]
print("\n--- 실패 케이스 ---")
print(failed[["question", "response", "latency_sec"]])

# 평균 응답 속도
print(f"\n평균 응답 시간: {df['latency_sec'].mean():.2f}초")
print(f"최대 응답 시간: {df['latency_sec'].max():.2f}초")
```

#### 그 외 방법

```
# 최신 실험 결과 보기
cd rag_eval
uv run python report.py

# 특정 파일 보기
uv run python report.py evals/experiments/flamboyant_musk.csv
```

---

## 한 프로젝트 내에 다중 테스트 환경 

### 데이터셋 분리

| **함수** | **데이터셋 이름** | **저장 파일** |
| --- | --- | --- |
| load_qna_dataset() | qna_dataset | evals/experiments/qna_dataset_*.csv |
| load_admin_dataset() | admin_dataset | evals/experiments/admin_dataset_*.csv |

### 실행 방법

```
# 1.루트 이동
cd rag_eval

# 2-1.QNA만 실행 (기본값)
uv run python evals.py --mode qna

# 2-2.Admin만 실행
uv run python evals.py --mode admin

# 2-3.둘 다 순서대로 실행
uv run python evals.py --mode all

# 3.실행 후 최신 실험 결과 보기
uv run python report.py

# 특정 파일 보기
uv run python report.py evals/experiments/flamboyant_musk.csv
```

---

## 참고

### 용어

> **pytest**
> 
> 
> Python 코드가 올바르게 동작하는지 자동으로 검증해주는 테스트 프레임워크
> 

> **Recall@K(재현율)**
> 
> 
> 정답 문서 중에서, 상위 K개 검색 결과 안에 몇 개나 포함됐는가
> 
> 예) Recall@3 → 상위 3개 결과 안에 정답이 있는가?
> 

### 관련 문서

https://docs.ragas.io/en/stable/getstarted/quickstart/