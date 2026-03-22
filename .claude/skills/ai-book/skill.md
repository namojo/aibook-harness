---
name: ai-book
description: "삼성디스플레이 임직원 대상 AI 교육 도서를 집필하는 오케스트레이터. AI 교육 도서, AI 책, 교육자료 집필."
---

# AI Book Orchestrator

삼성디스플레이 임직원 대상 AI 교육 도서를 리서치 → 집필 → 편집하여 완성하는 통합 스킬.

## 실행 모드: 서브 에이전트

## 에이전트 구성

| 에이전트 | subagent_type | 역할 | 출력 |
|---------|--------------|------|------|
| ai-researcher | general-purpose | AI 역사, 생성형 AI, 최신 트렌드 조사 | `_workspace/research/ai_research.md` |
| industry-researcher | general-purpose | 업종별 사례, 제조 AI, 디스플레이 산업 조사 | `_workspace/research/industry_research.md` |
| book-writer | book-writer | 리서치 기반 챕터 집필 | `_workspace/chapters/ch{NN}_{title}.md` |
| book-editor | book-editor | 편집, 검수, 최종 통합 | `_workspace/final/book_complete.md` |

## 워크플로우

### Phase 1: 준비

1. 사용자 입력 분석 — 추가 주제나 특별 요청사항 파악
2. 작업 디렉토리에 `_workspace/` 하위 폴더 생성:
   ```
   _workspace/
   ├── research/       # 리서치 산출물
   ├── chapters/       # 집필된 챕터
   ├── review/         # 검수 보고서
   └── final/          # 최종 편집본
   ```
3. `references/book-outline.md`를 Read하여 기본 목차 확인
4. 사용자 요청에 따라 목차 조정 (챕터 추가/삭제/순서 변경)
5. 확정된 목차를 `_workspace/00_outline.md`에 저장

### Phase 2: 리서치 (병렬)

단일 메시지에서 2개 Agent 도구를 동시 호출:

| 에이전트 | 프롬프트 요약 | run_in_background |
|---------|-------------|-------------------|
| ai-researcher | 아래 상세 프롬프트 참조 | true |
| industry-researcher | 아래 상세 프롬프트 참조 | true |

**ai-researcher 프롬프트:**
```
당신은 AI 교육 도서의 리서치 담당자입니다. 삼성디스플레이 임직원(비전공자 포함)이 읽을 교육 도서를 위해 다음 주제를 조사하세요.

조사 주제:
1. AI의 역사 (1950년대~현재): 주요 이정표, 인물, 사건
2. 생성형 AI의 원리와 중요성: LLM, 트랜스포머, 기존 AI와의 차이
3. 최신 AI 트렌드: 멀티모달 AI, AI 에이전트, SLM/온디바이스 AI, AI 거버넌스
4. AI와 반도체/디스플레이 산업의 관계

조사 방법:
- WebSearch와 WebFetch를 활용하여 최신 정보 수집
- 각 주제별 핵심 사실, 통계, 사례를 정리
- 출처를 명시 (URL 또는 출처명)

출력:
- `_workspace/research/ai_research.md` 파일에 마크다운으로 작성
- 주제별 섹션으로 구분
- 각 섹션에 핵심 팩트, 통계, 인용 가능한 사례 포함
```

**industry-researcher 프롬프트:**
```
당신은 AI 교육 도서의 산업 사례 리서치 담당자입니다. 삼성디스플레이 임직원이 읽을 교육 도서를 위해 다음 주제를 조사하세요.

조사 주제:
1. 업종별 AI 활용 사례:
   - 금융 (리스크 분석, 사기 탐지, 로보어드바이저)
   - 의료/바이오 (신약 개발, 영상 진단)
   - 유통/물류 (수요 예측, 라스트마일)
   - 미디어/콘텐츠 (생성형 AI 활용)
   - 에너지/환경 (스마트 그리드)
2. 제조 AI:
   - 스마트 팩토리 사례 (품질 검사, 공정 최적화, 예지 보전)
   - 반도체/디스플레이 제조에서의 AI 활용
   - 삼성디스플레이 또는 디스플레이 업계의 AI 도입 사례
3. 기업의 AI 전환 전략:
   - AI 리터러시 교육 사례
   - 프롬프트 엔지니어링 활용
   - 데이터 보안과 AI 윤리 정책

조사 방법:
- WebSearch와 WebFetch를 활용하여 최신 정보 수집
- 구체적 기업명과 수치를 포함한 사례 우선
- 한국 기업 사례를 적극 포함
- 출처를 명시

출력:
- `_workspace/research/industry_research.md` 파일에 마크다운으로 작성
- 업종별/주제별 섹션으로 구분
- 각 사례는 기업명, 적용 분야, 성과(수치), 출처 포함
```

### Phase 3: 집필 (순차)

리서치 완료 후 book-writer 에이전트를 호출하여 챕터를 집필한다.

**book-writer 프롬프트 구성:**
```
당신은 삼성디스플레이 임직원 대상 AI 교육 도서의 집필자입니다.

다음 파일을 Read하여 참고하세요:
- 목차: `_workspace/00_outline.md`
- AI 리서치: `_workspace/research/ai_research.md`
- 산업 사례 리서치: `_workspace/research/industry_research.md`

집필 규칙:
1. 목차의 모든 챕터를 순서대로 집필
2. 각 챕터는 `_workspace/chapters/ch{NN}_{title}.md`에 저장
   (예: ch01_ai_history.md, ch02_generative_ai.md)
3. 챕터 구조: 도입(왜 중요한가) → 본론(핵심 내용) → [정리](핵심 요약)
4. 한 챕터당 3,000~5,000자(한글 기준)
5. 비전공자도 이해할 수 있는 쉬운 설명 + 실무 연결 비유
6. 디스플레이/제조 업종 관련 비유와 사례를 적극 활용
7. 리서치 자료의 통계와 사례를 적절히 인용
8. 서문(preface.md)도 작성

모든 챕터 집필 완료 후, 집필한 파일 목록을 출력하세요.
```

### Phase 4: 편집/검수 (순차)

집필 완료 후 book-editor 에이전트를 호출하여 검수 및 최종 통합한다.

**book-editor 프롬프트 구성:**
```
당신은 삼성디스플레이 임직원 대상 AI 교육 도서의 편집자입니다.

다음 파일을 Read하여 작업하세요:
- 목차: `_workspace/00_outline.md`
- 리서치 자료: `_workspace/research/` 내 파일들
- 집필된 챕터: `_workspace/chapters/` 내 모든 ch*.md 파일

편집 작업:
1. 각 챕터를 검수하고 `_workspace/review/review_ch{NN}.md`에 검수 보고서 작성
   - 판정: PASS / REVISE
   - 정확성, 가독성, 대상 적합성 평가
   - 구체적 수정 사항 나열
2. REVISE 판정 챕터는 직접 수정하여 `_workspace/final/ch{NN}_{title}.md`에 저장
3. PASS 판정 챕터는 그대로 `_workspace/final/`에 복사
4. 모든 챕터 편집 완료 후 통합:
   - `_workspace/final/book_complete.md`에 서문 + 목차 + 전체 챕터를 하나의 파일로 통합
   - 목차에 페이지 대신 섹션 링크 사용
   - 부록(AI 용어 사전) 추가
5. 문체 일관성 최종 점검 (용어 통일, 경어체 통일)

편집이 완료되면 최종 파일 경로와 검수 요약을 출력하세요.
```

### Phase 5: 정리

1. `_workspace/final/book_complete.md`를 Read하여 내용 확인
2. 사용자에게 결과 요약 보고:
   - 최종 도서 파일 경로
   - 총 챕터 수, 예상 분량
   - 검수 결과 요약 (PASS/REVISE 비율)
   - 추가 수정이 필요한 부분이 있으면 안내
3. `_workspace/` 디렉토리 보존 (중간 산출물은 삭제하지 않음)

## 데이터 흐름

```
사용자 입력 + book-outline.md
         ↓
    00_outline.md (확정 목차)
         ↓
    ┌────┴────┐
    ↓         ↓
[ai-researcher]  [industry-researcher]   ← 병렬 실행
    ↓         ↓
ai_research.md  industry_research.md
    └────┬────┘
         ↓
   [book-writer]   ← 순차 실행 (리서치 완료 후)
         ↓
   chapters/ch01~06.md + preface.md
         ↓
   [book-editor]   ← 순차 실행 (집필 완료 후)
         ↓
   final/book_complete.md (최종 산출물)
```

## 에러 핸들링

| 상황 | 전략 |
|------|------|
| 리서처 1개 실패 | 1회 재시도. 재실패 시 나머지 리서치 자료로 진행, 해당 영역 집필 시 `[리서치 미완]` 표시 |
| 리서처 모두 실패 | 사용자에게 알리고 진행 여부 확인. 리서치 없이 집필 가능한지 판단 |
| book-writer 실패 | 1회 재시도. 재실패 시 완성된 챕터만으로 부분 도서 생성 |
| book-editor 실패 | 미편집 챕터를 그대로 통합하여 초고 제공 |
| 챕터 분량 초과 | book-editor가 분할 판단, 리더에게 보고 |

## 테스트 시나리오

### 정상 흐름
1. 사용자가 "AI 교육 도서 써줘"를 요청
2. Phase 1에서 기본 목차 확인, `_workspace/` 생성
3. Phase 2에서 2개 리서처가 병렬 실행, 각각 리서치 산출물 생성
4. Phase 3에서 book-writer가 6개 챕터 + 서문 집필
5. Phase 4에서 book-editor가 검수 → 편집 → `book_complete.md` 생성
6. Phase 5에서 사용자에게 최종 파일 경로와 요약 보고
7. 예상 결과: `_workspace/final/book_complete.md` (서문 + 6개 챕터 + 부록)

### 에러 흐름
1. Phase 2에서 industry-researcher가 WebSearch 실패
2. 1회 재시도 후에도 실패
3. ai-researcher 결과만으로 Phase 3 진행
4. book-writer가 업종별 사례(3장), 제조 AI(4장)을 리서치 없이 자체 지식으로 집필
5. book-editor가 해당 챕터에 `[추가 사례 보강 필요]` 태그 부착
6. 최종 보고서에 "산업 사례 리서치 미완료, 3·4장 사례 보강 권장" 명시
