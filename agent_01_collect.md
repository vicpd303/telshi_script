# Agent 01: 뉴스 수집 + 키워드 선별 에이전트

## 역할
글로벌 뉴스 소스에서 오늘의 TOP 뉴스를 수집하고, 웰시코기 채널 기준으로 스코어링해서 상위 3개를 선별한다.

---

## 뉴스 소스 목록 (페이월 없는 곳만)
- Reuters: https://feeds.reuters.com/reuters/topNews
- AP News: https://rsshub.app/apnews/topics/apf-topnews
- 연합뉴스 영문: https://en.yna.co.kr/RSS/news.xml
- CNBC: https://www.cnbc.com/id/100003114/device/rss/rss.html
- TechCrunch: https://techcrunch.com/feed/
- The Verge: https://www.theverge.com/rss/index.xml
- Nikkei Asia: https://asia.nikkei.com/rss/feed/nar

---

## 키워드 선별 로직

### Step 1: 수집
- 위 소스에서 최신 기사 제목 + 요약 수집 (최대 20개)
- 수집 시각 기록

### Step 2: 키워드 추출
- 제목들에서 겹치는 단어/구문 추출
- 필터 조건 (아래 해당하면 제외 후 다시 추출):
  - 일반 동사: said, says, tells, makes, gets, goes 등
  - 광범위 명사: news, report, update, things, people, world 등
  - 관사/전치사/접속사 전부 제외

### Step 3: 스코어링 (100점 만점)

| 항목 | 배점 |
|------|------|
| 반도체/AI/배터리/빅테크 관련 | +25점 |
| 외교/안보/무역 관련 | +20점 |
| 한국 직접 연관 (Korea/Samsung/Hyundai 등) | +20점 |
| 한국에 간접 영향 (미중관계, 반도체 규제 등) | +15점 |
| 제목에 숫자/고유명사 포함 | +10점 |
| 출처 티어 점수 (아래 참조) | +10점 |

### 출처 티어
- Tier 1 (+10점): Reuters, AP, 연합뉴스
- Tier 2 (+7점): CNBC, Nikkei Asia
- Tier 3 (+5점): TechCrunch, The Verge

### 반복 티어
- 직전 12개 배치 기준으로 같은 키워드가 등장했다면:
  - 1회 반복: +5점
  - 2회 이상 반복: +10점 (지속적으로 중요한 이슈임을 의미)

---

## 출력 형식 (JSON)

```json
{
  "collected_at": "2026-02-25T09:00:00Z",
  "total_collected": 20,
  "selected": [
    {
      "rank": 1,
      "title": "기사 제목",
      "source": "Reuters",
      "source_tier": 1,
      "url": "https://...",
      "keywords": ["반도체", "삼성", "미국"],
      "score": 85,
      "score_breakdown": {
        "topic": 25,
        "korea_direct": 20,
        "specificity": 10,
        "source_tier": 10,
        "repeat_tier": 5,
        "diplomacy": 0,
        "korea_indirect": 15
      },
      "repeat_count": 1,
      "summary": "기사 요약 1-2문장"
    }
  ]
}
```

---

## 실행 지침 (Claude에게)

1. 위 RSS 소스에서 오늘 기사 제목과 요약을 수집해줘.
2. 키워드 필터 조건 적용해서 유효 키워드 추출해줘.
3. 스코어링 기준대로 각 기사에 점수 매겨줘.
4. 상위 3개를 선택해서 위 JSON 형식으로 출력해줘.
5. 결과를 `output_01_collected.json` 으로 저장해줘.
