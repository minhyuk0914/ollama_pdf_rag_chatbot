# Ollama PDF Bot

로컬 LLM(Ollama)과 RAG(Retrieval-Augmented Generation)를 활용하여 PDF 문서를 기반으로 질문에 답변하는 챗봇입니다.


## 프로젝트 개요

PDF 문서를 업로드하면 해당 문서의 내용만을 바탕으로 질문에 답변합니다. 외부 API 없이 완전히 로컬 환경에서 실행되며, LG AI Research의 `exaone3.5:2.4b` 모델을 사용합니다.


## 주요 기능

- PDF 파일을 마크다운으로 변환 후 텍스트 청크 분할
- 다국어 임베딩 모델(`multilingual-e5-small`)을 이용한 벡터화
- ChromaDB를 이용한 벡터 저장 및 코사인 유사도 기반 검색
- 대화 히스토리 관리 (최대 10턴)
- 스트리밍 방식의 실시간 답변 출력
- 대화 로그 저장 및 DataFrame 출력


## 기술 스택

| 구분 | 사용 기술 |
|------|-----------|
| LLM | Ollama (`exaone3.5:2.4b`) |
| PDF 파싱 | pymupdf4llm |
| 텍스트 분할 | LangChain `RecursiveCharacterTextSplitter` |
| 임베딩 | SentenceTransformer (`intfloat/multilingual-e5-small`) |
| 벡터 DB | ChromaDB (PersistentClient) |
| 로그 분석 | pandas |


## 설치 방법

### 1. Ollama 설치 및 모델 다운로드

[Ollama 공식 사이트](https://ollama.com)에서 Ollama를 설치한 후, 아래 명령어로 모델을 다운로드합니다.

```bash
ollama pull exaone3.5:2.4b
```

### 2. Python 패키지 설치

```bash
pip install pymupdf4llm sentence-transformers langchain chromadb ollama pandas
```

> Python 3.9 이상 환경을 권장합니다.


## 디렉토리 구조

```
project/
├── ollama_pdf_bot.ipynb   # 메인 노트북
├── pdf_folder/            # 분석할 PDF 파일 저장 경로
│   └── your_document.pdf
└── chroma_db/             # ChromaDB 벡터 저장소 (자동 생성)
```


## 사용 방법

1. `pdf_folder/` 디렉토리에 분석할 PDF 파일을 넣습니다.
2. 노트북의 `file_path` 변수를 해당 PDF 파일 경로로 수정합니다.
3. 노트북을 순서대로 실행합니다.
4. `chat_loop()` 셀을 실행하면 대화형 챗봇이 시작됩니다.
5. 질문을 입력하고 Enter를 누르면 PDF 내용을 기반으로 답변을 생성합니다.
6. 종료하려면 `exit`을 입력합니다.

```
RAG 챗봇 시작! 질문 입력 (종료하려면 'exit' 입력):
> 구직단념 청년을 위한 정책이 무엇이 있어?

답변: 구직단념 청년을 위해 시행되는 주요 정책들은 다음과 같습니다:
1. **청년 일자리 첫걸음 플랫폼**:
   - 미취업 대학 졸업자와 군 복무 후 취업하지 않은 청년들을 대상으로 고용보험 데이터베이스와 연동된 플랫폼을 구축합니다.
   - 개인정보 제공 동의를 기반으로 장기 미취업 위험군을 선제적으로 발굴하고 맞춤형 취업 지원을 제공합니다.
   - 비대면 참여를 허용하며, 밀착형 멘토링 등 다양한 참여 형태를 제공하여 청년들이 점진적으로 취업에 적응하고 경력을 형성할 수 있도록 돕습니다.

2. **'쉬고 있는 청년' 특화 일경험 프로그램**:
   - 사회연대경제와 공공부문을 통해 쉬고 있는 청년들에게 점진적 적응을 유도하고 경력 형성을 지원합니다.
   - 유연한 참여 방식을 제공하며, 맞춤형 멘토링을 포함하여 비대면 옵션도 포함하고 있습니다.

3. **청년도전지원사업**:
   - 구직단념 청년들을 대상으로 밀착 상담, 역량 강화, 구직 의욕 고취 등 맞춤형 프로그램을 제공합니다.
   - 참여자들에게 참여수당이 지급되며, 지원 금액은 50만원에서 250만원 사이입니다.
```


## 파라미터 설정

| 파라미터 | 기본값 | 설명 |
|----------|--------|------|
| `chunk_size` | 1500 | 텍스트 분할 단위 (문자 수) |
| `chunk_overlap` | 150 | 청크 간 중복 문자 수 |
| `top_k` | 5 | 질문당 검색할 관련 문서 수 |
| `maxlen` (deque) | 10 | 유지할 최대 대화 턴 수 |
| `MODEL_NAME` | `exaone3.5:2.4b` | 사용할 Ollama 모델명 |


## 시스템 프롬프트 정책

챗봇은 아래 규칙에 따라 답변을 생성합니다.

- 반드시 제공된 문서 내용만을 기반으로 답변
- 문서에 없는 정보는 외부 지식을 활용하지 않음
- 해당 내용이 문서에 없을 경우 명확히 "찾을 수 없습니다"로 안내
- 답변 시 참고한 문서의 장·절 또는 핵심 키워드를 가급적 명시


## 실행 환경

- OS: Windows (Anaconda 가상환경 `pdf_bot`)
- Python: 3.9.21
- Ollama 모델: `exaone3.5:2.4b` (2.7B, Q4_K_M, 약 1.6GB)


## 주의사항

- Ollama 서버가 로컬에서 실행 중이어야 합니다 (`ollama serve`).
- 처음 실행 시 PDF 로드 및 임베딩에 수십 초가 소요될 수 있습니다.
- `chroma_db/` 폴더는 최초 실행 후 자동으로 생성되며, 재실행 시 기존 벡터를 재사용합니다.