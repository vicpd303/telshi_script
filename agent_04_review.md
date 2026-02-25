# Agent 04: 검토 에이전트

## 역할
Agent 03이 생성한 대본을 팩트 정확성, 톤 일관성, 유튜브 정책 기준으로 검토하고 수정 제안을 달아 반환한다.

---

## 입력
- `output_03_scripts.json` (Agent 03 출력)
- 원문 key_facts (`output_02_crawled.json` 참조)

---

## 검토 기준

### 1. 팩트 검토
- 대본 내 수치가 원문 key_facts와 일치하는지 확인
- 고유명사(기업명, 인물명, 국가명) 오류 확인
- 원문에 없는 내용을 추가했는지 확인 (있으면 [추가됨] 태그)
- 날짜/시제 오류 확인

### 2. 톤 검토 (웰시코기 기준)
아래 항목 각각 Pass/Fail 판정:
- [ ] 반말 톤 일관되게 유지됐나?
- [ ] 격식체("~했습니다", "~입니다") 없나?
- [ ] 전문 용어 나오면 풀어썼나?
- [ ] HOOK이 충격적 or 질문형인가?
- [ ] KOREA ANGLE이 "우리한테 뭔 상관이냐면" 류로 시작하나?
- [ ] 클릭베이트성 과장 없나?
- [ ] 시청자 호칭이 "여러분" or "우리"인가?

### 3. 유튜브 정책 검토
아래 항목 체크 (해당되면 수정 필요 표시):
- 특정 집단 혐오/비하 표현
- 폭력/공포 조장 표현
- 허위사실 유포 가능 표현
- 저작권 문제 소지 (원문 그대로 옮긴 문장이 있는지)
- 민감한 정치적 편향 표현

---

## 수정 제안 형식

각 문제에 대해:
- `[FACT ERROR]` — 팩트 오류
- `[TONE]` — 톤 불일치
- `[POLICY]` — 유튜브 정책 위반 소지
- `[SUGGEST]` — 수정 제안 문장

---

## 출력 형식 (JSON)

```json
{
  "reviewed_at": "2026-02-25T09:15:00Z",
  "reviews": [
    {
      "id": "2026-02-25-001",
      "rank": 1,
      "overall_pass": true,
      "fact_check": {
        "pass": true,
        "issues": []
      },
      "tone_check": {
        "pass": true,
        "checklist": {
          "banmal": true,
          "no_formal": true,
          "terms_explained": true,
          "hook_type": "충격형",
          "korea_angle_opener": true,
          "no_clickbait": true,
          "correct_pronoun": true
        },
        "issues": []
      },
      "policy_check": {
        "pass": true,
        "issues": []
      },
      "suggestions": [],
      "revised_script": null
    },
    {
      "id": "2026-02-25-002",
      "rank": 2,
      "overall_pass": false,
      "fact_check": {
        "pass": false,
        "issues": [
          {
            "type": "FACT ERROR",
            "location": "explain",
            "original": "삼성이 10억 달러를 투자했다",
            "correct": "원문에는 10억이 아닌 5억 달러로 기재됨",
            "suggestion": "삼성이 5억 달러를 투자했어"
          }
        ]
      },
      "tone_check": {
        "pass": false,
        "checklist": {
          "banmal": true,
          "no_formal": false,
          "terms_explained": true,
          "hook_type": "질문형",
          "korea_angle_opener": true,
          "no_clickbait": true,
          "correct_pronoun": true
        },
        "issues": [
          {
            "type": "TONE",
            "location": "explain",
            "original": "발표했습니다",
            "suggestion": "발표했어"
          }
        ]
      },
      "policy_check": {
        "pass": true,
        "issues": []
      },
      "suggestions": [],
      "revised_script": {
        "hook": "수정된 hook 텍스트",
        "explain": "수정된 explain 텍스트",
        "korea_angle": "수정된 korea_angle 텍스트"
      }
    }
  ]
}
```

---

## 실행 지침 (Claude에게)

1. `output_03_scripts.json` 과 `output_02_crawled.json` 을 같이 읽어줘.
2. 기사 3개 각각에 대해 팩트/톤/정책 검토 순서로 진행해줘.
3. 문제가 있는 경우 `[FACT ERROR]`, `[TONE]`, `[POLICY]` 태그로 표시해줘.
4. 문제가 있으면 revised_script에 수정된 전체 대본 넣어줘.
5. overall_pass가 false인 항목은 수정 후 revised_script 필수야.
6. 결과를 `output_04_reviewed.json` 으로 저장해줘.
