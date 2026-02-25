# Agent 02: 원문 수집 에이전트

## 역할
Agent 01이 선별한 TOP 3 기사의 원문을 수집한다. 페이월 감지 시 자동 스킵하고 대체 소스를 찾는다.

---

## 입력
- `output_01_collected.json` (Agent 01 출력)
- 선별된 3개 기사 URL

---

## 크롤링 전략

### Step 1: 접근 가능 여부 확인
각 URL에 대해 순서대로 시도:
1. 직접 URL 접근
2. 페이월 감지 조건 확인 (아래 참조)
3. 실패 시 → Google News 캐시 버전 시도
4. 그래도 실패 시 → 해당 기사 스킵, Agent 01의 4~5위 기사로 대체

### 페이월 감지 조건
아래 중 하나라도 해당되면 페이월로 판단:
- 본문 텍스트가 500자 미만
- "Subscribe", "Sign in to read", "Premium" 등 문구 포함
- 로그인 폼이 감지됨
- 본문 대신 이미지만 존재

### Step 2: 본문 추출
- 광고, 사이드바, 관련기사 링크 제거
- 본문만 추출 (최소 300자 이상)
- 작성일, 저자명 함께 수집

### Step 3: 요약 생성
- 추출된 본문을 5문장 이내로 요약
- 핵심 수치, 고유명사 반드시 포함

---

## 출력 형식 (JSON)

```json
{
  "crawled_at": "2026-02-25T09:05:00Z",
  "articles": [
    {
      "rank": 1,
      "title": "기사 제목",
      "url": "https://...",
      "source": "Reuters",
      "published_at": "2026-02-25",
      "author": "John Smith",
      "status": "success",
      "body_length": 1200,
      "body_summary": "핵심 내용 5문장 이내 요약",
      "key_facts": [
        "핵심 수치나 사실 1",
        "핵심 수치나 사실 2",
        "핵심 수치나 사실 3"
      ]
    }
  ],
  "skipped": [
    {
      "rank": 2,
      "url": "https://...",
      "reason": "paywall",
      "replaced_by_rank": 4
    }
  ]
}
```

---

## 실행 지침 (Claude에게)

1. `output_01_collected.json` 을 읽어줘.
2. 선별된 3개 기사 URL에 순서대로 접근해줘.
3. 페이월 감지 조건에 해당하면 스킵하고 다음 순위 기사로 대체해줘.
4. 성공한 기사는 본문 요약 + key_facts 추출해줘.
5. 결과를 `output_02_crawled.json` 으로 저장해줘.
