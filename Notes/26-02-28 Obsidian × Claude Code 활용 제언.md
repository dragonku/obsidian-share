---
created: 2026-02-28
updated: 2026-02-28
type: memo
status: active
tags: [AI/Agent, type/reference]
---

# Obsidian × Claude Code 활용 제언

> 2026-02-28 볼트 분석 기반 개선 제언
> 📅 작성 계기: [[Daily/2026년/02월/26-02-28 Daily Note|2026-02-28]] — 노션→옵시디언 전면 이전 완료 후 Claude Code 활용 전략 수립

> 🧭 **Navigation**: [[Home|🏠 Home]] | [[Dashboard|📊 Dashboard]]

---

## 📊 현재 상태 진단

### 강점
- **Daily Note 습관**: 88개, 86%+ 월별 작성률 → 5개월 연속 유지 중 (최고 수준)
- **AI/Agentic 학습 깊이**: Documents 38개 중 70%가 AI/Claude 관련 → 일관된 학습 방향
- **MOC 구조**: Home → AI-Development-MOC → 개별 문서 계층 잘 설계됨

### 약점

| 문제 | 현황 | 영향 |
|------|------|------|
| Daily Notes 고립 | 100% 내부 링크 없음 | 지식 섬 현상 |
| 태그 활용 부재 | 13% 커버리지 | Dataview 쿼리 효과 저하 |
| Meeting Notes 방치 | 13개 중 대부분 빈 템플릿 | 회의 인사이트 소실 |
| 독서 연결 단절 | 책 ↔ 개념 ↔ 실무 링크 없음 | 지식 활용률 저하 |
| 감상/사유 필드 공백 | Reading notes 95%가 비어있음 | 표면적 기록에 그침 |

---

## 🧠 Obsidian 본질에 맞는 활용 제언

> **Obsidian의 본질: "생각의 네트워크"** — 노트가 아니라 *링크*가 가치를 만든다.

### 1. Daily Notes를 허브로 만들기

지금은 Daily Note가 완전히 고립되어 있음. 매일 쓴 회고에 관련 노트 링크를 3개 이상 추가하면 6개월 후 Daily Notes가 지식 지도의 실 역할을 함.

> ✅ **2026-02-28 첫 실천**: [[Daily/2026년/02월/26-02-28 Daily Note|2026-02-28 Daily Note]]에 링크 3개 추가 완료

```markdown
## 2026-02-28 Daily Note (실천 예시)

### 오늘 한 일
- [[Documents/26-02-27 Claude Code 자동 개발 워크플로우]] 실습 완료
  → [[Memo/Claude Skills 완전 정복]]과 연결되는 인사이트 발견

### 오늘 읽은 것
- [[References/Reading/25-11-04 빅히스토리]] 3장 — [[Documents/26-02-09 AI-Development-MOC]]과 유사한 계층적 사고 패턴

### 오늘의 생각
- #인사이트 #AI/Agent 관련 메모 → [[Documents/26-01-15 아이디어 노트]]
```

### 2. 태그 표준 체계 수립

현재 태그가 일관성 없이 산재해 있음. 권장 체계:

```
#daily-note / #daily-retrospective
#AI/Agent / #AI/NLP / #AI/SDLC
#reading/fiction / #reading/nonfiction / #reading/audiobook
#meeting/weekly / #meeting/strategy
#status/active / #status/completed / #status/on-hold
#type/moc / #type/reference / #type/idea
```

### 3. Reading Notes 깊이 추가

19권의 책을 읽었지만 `thoughts/impressions` 필드가 비어있음. 책 1권당 최소 이것만 추가:

```markdown
## 나의 생각
- 실무에 적용할 것: ...
- 연결되는 개념: [[Documents/...]]
- 다음에 읽을 책: [[References/Reading/...]]
```

---

## 🤖 Claude Code와 Obsidian 연동 제언

### 이미 잘 설정된 것
- `update_obsidian.py` 매일 새벽 5시 자동 실행 ✅
- Claude Code MCP 연동 (`obsidian-claude-code-mcp`) ✅
- 구조화된 YAML frontmatter ✅

### Claude Code로 할 수 있는 것 (지금 당장)

> 🔗 MS To Do 연동도 함께 구성하면 할 일 관리까지 자동화 가능
> → [[Documents/26-02-28 Obsidian × MS To Do 연동]]

**A. 회의록 자동 생성**
```
"오늘 AI 사내 활성화 논의 회의 내용이야: [내용 붙여넣기]
Meeting Notes 템플릿에 맞게 정리하고 관련 노트와 링크해줘"
```

**B. Daily Note 생성 자동화**
```
"오늘 Daily Note 만들어줘.
어제 회고([[Daily/2026년/02월/26-02-27 Daily Note]])와
이번 주 목표([[Dashboard]])를 참고해서"
```

**C. 지식 연결 분석**
```
"[[References/Reading/25-11-04 빅히스토리]]와 [[Documents/26-02-09 AI-Development-MOC]] 사이에
연결할 수 있는 개념이 있는지 분석해줘"
```

**D. 주간 인사이트 리포트**
```
"이번 주 Daily Note 7개를 읽고
반복 패턴, 핵심 인사이트, 다음 주 집중할 것을 정리해줘"
```

---

## 🎯 30일 실행 계획

| 주차 | 액션 |
|------|------|
| **1주차** | Daily Note에 매일 링크 3개 추가하는 습관 만들기 |
| **2주차** | 기존 38개 Documents에 태그 추가 (Claude Code로 일괄 처리) |
| **3주차** | Reading Notes 19권에 `## 나의 생각` 섹션 추가 |
| **4주차** | Meeting Notes 활성화: 다음 회의부터 Claude Code로 생성 |

---

## 💡 가장 임팩트 있는 변화 1가지

> **Daily Note에 `[[링크]]` 3개씩 추가하기**

- 현재: 88개 회고 × 평균 0개 링크 = **0개 연결**
- 목표: 매일 3개 → 30일 후 **90개 신규 연결**

이것 하나만 해도 Obsidian의 Graph View가 살아나고, "내가 무엇을 계속 생각하고 있었는지"가 시각적으로 보이기 시작함. Obsidian의 진짜 가치는 여기서 나온다.

---

*Last updated: 2026-02-28*
