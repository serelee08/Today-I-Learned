# 2026-07-01 — RAG/LangChain 기본 개념 & AppointmentVO JOIN 필드 확장

## 1. RAG / LangChain — 기본 틀 학습

오늘은 개념 수준에서 전체 틀을 익혔다. 상세 실습이 아니라 "무엇이 왜 필요한가"를 정리한 단계.

### RAG (Retrieval-Augmented Generation) 기본 흐름

```
문서 → Chunking(분할) → Embedding(벡터화) → Vector DB 저장
                                                    ↓
사용자 질문 → Embedding → 유사도 검색(Retrieval) → 관련 chunk 추출
                                                    ↓
                                    프롬프트에 chunk 삽입 → LLM 답변 생성
```

- **왜 필요한가:** LLM은 학습 시점 이후 정보나 특정 도메인/사내 데이터를 모른다. RAG는 매번 모델을 재학습시키는 대신, 질문 시점에 관련 문서를 검색해서 프롬프트에 끼워 넣어 LLM이 근거 있는 답을 하도록 만드는 구조다.
- **Chunking:** 문서를 통째로 넣으면 컨텍스트 낭비 + 검색 정확도 하락. 의미 단위로 잘라서 저장해야 필요한 부분만 정확히 검색된다.
- **Embedding:** 텍스트를 벡터(숫자 배열)로 바꿔서 "의미적으로 가까운지"를 계산 가능하게 만드는 것. 단어가 달라도 의미가 비슷하면 벡터 거리가 가깝다.
- **Vector DB:** 이 임베딩 벡터들을 저장하고, 질문 벡터와 가장 가까운 것들을 빠르게 찾아주는 저장소(Pinecone, Chroma, FAISS 등).

### LangChain 기본 개념

- **Chain:** 여러 단계(프롬프트 생성 → LLM 호출 → 출력 파싱 등)를 하나의 파이프라인으로 엮는 단위. "입력 → 처리 → 출력"을 조립식으로 연결한다.
- **PromptTemplate:** 프롬프트를 하드코딩하지 않고 변수를 채워 넣는 템플릿 구조로 관리. 재사용성과 유지보수성을 높인다.
- **Retriever:** Vector DB에서 관련 문서를 가져오는 역할을 추상화한 컴포넌트. RAG 파이프라인에서 "검색" 단계를 담당.
- **Memory:** 대화형 앱에서 이전 대화 맥락을 유지하기 위한 컴포넌트. 매번 전체 히스토리를 수동으로 관리하지 않아도 되게 해준다.
- **Agent:** LLM이 스스로 어떤 도구(tool)를 쓸지 판단하고 순차적으로 실행하는 구조. 단순 Chain과 다르게 "다음에 뭘 할지"를 LLM이 결정한다.

### 오늘 정리한 핵심 인사이트

RAG와 LangChain은 별개가 아니라, **LangChain은 RAG 파이프라인을 포함한 LLM 애플리케이션을 조립하기 위한 프레임워크**라는 관계다. RAG는 "왜/어떤 흐름으로 검색+생성을 결합하는가"라는 아키텍처 개념이고, LangChain은 그 아키텍처를 코드로 구현할 때 Chain/Retriever/Memory 같은 재사용 가능한 부품을 제공하는 도구다.

**다음 할 일:** 개념을 실제 코드로 연결 — 간단한 텍스트 파일을 chunking → embedding → 로컬 vector store(Chroma 등) 저장 → 질문 시 retrieval까지 되는 최소 파이프라인을 직접 구현해보기. 지금은 개념만 아는 단계라 "설명은 가능하나 구현 경험은 없음"이 정확한 현재 위치다.

---

## 2. 병원예약시스템 — AppointmentVO에 JOIN 결과 필드 추가

`AppointmentVO.java`에 목록 화면 표시용 필드 3개를 추가했다.

```java
public class AppointmentVO {
    private String status;
    private String memo;
    private LocalDateTime createdAt;

    // JOIN 결과 담는 표시용 필드
    private String patientName;
    private String doctorName;
    private String departmentName;
}
```

Appointment 목록에서 `patient_no`, `doctor_no` FK 숫자만 보여줄 수 없어서, MyBatis 매퍼에서 patient/doctor/department를 JOIN해 이름까지 함께 내려주고 그 결과를 담을 자리로 이 필드들을 추가했다. React 쪽에서는 예약 목록/폼에 `doctorName`을 표시하도록 연결했다.

**기술적 배움:** 이 방식은 VO 하나가 "테이블 매핑용 컬럼"과 "JOIN 조합 결과"라는 두 가지 책임을 동시에 갖게 만든다. 정석은 `AppointmentVO`(순수 컬럼 매핑, INSERT/UPDATE용)와 `AppointmentResponseDTO`(VO 상속 + JOIN 필드, 조회 전용)를 분리하는 것이다. 지금은 화면을 빠르게 띄우기 위한 임시 처리이고, `doctor.specialty` 중복 컬럼 이슈와 같은 계열의 문제 — JOIN으로 얻을 수 있는 정보를 별도 자리에 중복 저장하면서 source of truth가 흐려지는 패턴이다.

**다음 할 일:** `AppointmentResponseDTO` 분리, resultMap 조회용/쓰기용 나누기.
