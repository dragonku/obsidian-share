---
created: 2026-02-28
updated: 2026-02-28
type: reference
status: active
tags: [AI/Agent, type/reference, productivity/todo]
---

# Obsidian × MS To Do 연동

> Obsidian Tasks 플러그인 + Microsoft Graph API 기반 양방향 동기화

📅 작성 계기: [[Daily/2026년/02월/26-02-28 Daily Note|2026-02-28]] — Claude Code + 옵시디언 본격 활용 시작
🧭 **Navigation**: [[Home|🏠 Home]] | [[Documents/26-02-09 AI-Development-MOC|AI Dev MOC]]
📎 동기화 스크립트: [[../Scripts/sync_ms_todo.py|sync_ms_todo.py]]

---

## 전체 아키텍처

```
Obsidian Vault                  MS To Do
─────────────────               ─────────────────
Daily Note                      "Obsidian" 목록
  - [ ] 할 일 #ms-todo  ──────▶  ☐ 할 일
  - [x] 완료 #ms-todo   ◀──────  ✓ 완료 (체크 시)

Weekly Note                     Microsoft Graph API
  - [ ] 주간 목표 #ms-todo        /me/todo/lists/{id}/tasks
```

### 데이터 흐름

```
[Obsidian 작성]
   - [ ] 회의 준비 📅 2026-03-01 #ms-todo
          │
          ▼  sync_ms_todo.py --push
[MS To Do 생성]
   ☐ 회의 준비  (기한: 3/1)
          │
          │  모바일/데스크톱에서 완료 체크
          ▼  sync_ms_todo.py --pull
[Obsidian 자동 갱신]
   - [x] 회의 준비 📅 2026-03-01 ✅ 2026-03-01 #ms-todo
```

---

## 설치 방법

### 1단계: Obsidian Tasks 플러그인 설치

```
Obsidian → 설정 → 커뮤니티 플러그인 → 탐색
→ "Tasks" 검색 → 설치 → 활성화
```

### 2단계: Azure App 등록 (5분 소요)

```
1. portal.azure.com 접속
2. Azure Active Directory → 앱 등록 → 새 등록
3. 이름: "Obsidian Sync"
   지원 계정 유형: 개인 Microsoft 계정
   리디렉션 URI: 없음 (Device Code Flow 사용)
4. 등록 완료 후 → 응용 프로그램(클라이언트) ID 복사
5. API 사용 권한 → 추가 → Microsoft Graph → 위임된 권한
   → Tasks.ReadWrite 추가 → 동의 부여
```

### 3단계: 환경변수 설정

볼트 루트의 `.env.local` 파일:

```env
MS_TODO_CLIENT_ID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
MS_TODO_TENANT_ID=consumers
MS_TODO_LIST_NAME=Obsidian
OBSIDIAN_VAULT_PATH=/mnt/d/dragonku
```

### 4단계: 의존성 설치

```bash
pip install msal requests python-dotenv
```

### 5단계: 최초 인증

```bash
cd /mnt/d/dragonku/Scripts
python sync_ms_todo.py --auth
# 브라우저에서 코드 입력 → 로그인 → 완료
# 이후 토큰 자동 갱신 (재인증 불필요)
```

---

## 사용법

### Obsidian에서 할 일 작성

Tasks 플러그인 단축키 또는 직접 입력:

```markdown
- [ ] 회의 준비 📅 2026-03-01 #ms-todo
- [ ] CS224N Lecture 03 학습 #ms-todo
- [ ] 독서 기록 갱신 📅 2026-03-07 #ms-todo
```

> **규칙**: `#ms-todo` 태그가 있는 항목만 동기화됩니다

### 동기화 실행

```bash
# 양방향 동기화 (기본)
python /mnt/d/dragonku/Scripts/sync_ms_todo.py

# Obsidian → MS To Do만 (새 할 일 밀어넣기)
python sync_ms_todo.py --push

# MS To Do → Obsidian만 (완료 상태 가져오기)
python sync_ms_todo.py --pull
```

---

## 자동화 설정

### update_obsidian.py에 통합 (매일 새벽 5시)

기존 `update_obsidian.py`에 아래 라인 추가:

```python
import subprocess

# 기존 코드 하단에 추가
subprocess.run(
    ["python", "/mnt/d/dragonku/Scripts/sync_ms_todo.py"],
    capture_output=True
)
```

### Windows 작업 스케줄러 (15분마다)

```
작업 스케줄러 → 기본 작업 만들기
프로그램: python
인수: /mnt/d/dragonku/Scripts/sync_ms_todo.py
트리거: 매 15분마다
```

---

## Daily Note 템플릿 연동

Daily Note에서 MS To Do 연동 태스크 작성 예시:

```markdown
## ✅ 오늘 할 일

- [ ] 오전 스탠드업 준비 📅 {{date}} #ms-todo
- [ ] [[Documents/26-02-27 Claude Code 자동 개발 워크플로우]] 실습 #ms-todo
- [ ] 저녁 운동 #ms-todo
```

---

## 동기화 대상 폴더

| 폴더 | 동기화 여부 | 비고 |
|------|-----------|------|
| `Daily/` | ✅ | 매일 할 일 |
| `Weekly Notes/` | ✅ | 주간 목표 |
| `Documents/` | ✅ | 프로젝트 태스크 |
| `Memo/` | ❌ | 제외 (메모 성격) |
| `Reading/` | ❌ | 제외 |
| `Private/` | ❌ | 제외 |

---

## 상태 파일 구조

`Scripts/.todo_sync_state.json` — 동기화 매핑 추적:

```json
{
  "obsidian_to_todo": {
    "a3f9b2c1d4e5f6a7": "AAMkAGE1...",
    "로컬_task_id_16자": "MS_To_Do_task_id"
  }
}
```

---

## 관련 노트

- [[Memo/26-02-28 Obsidian × Claude Code 활용 제언]] — 볼트 전체 활용 전략
- [[Documents/26-02-27 Claude Code 자동 개발 워크플로우]] — /feature 워크플로우

*Last updated: 2026-02-28*
