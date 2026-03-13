---
title: Obsidian Code 플러그인 분석
created: 2026-03-13
updated: 2026-03-13
type: reference
status: active
tags:
  - Obsidian
  - Claude
  - Plugin
  - AI-Agent
  - MCP
aliases:
  - obsidian-code
  - cc-obsidian
source: https://github.com/reallygood83/obsidian-code
---

# Obsidian Code 플러그인 분석

> Claude Code CLI를 Obsidian 사이드바에 내장한 AI 에이전트 플러그인  
> 제작: [@reallygood83](https://github.com/reallygood83) (배움의달인)  
> 버전: `1.4.19` | 언어: TypeScript | 라이선스: MIT | ⭐ 26

---

## 핵심 개념

단순 챗봇이 아닌 **Vault를 직접 관리하는 AI 에이전트**.
Claude Code CLI를 백엔드로 사용해 노트 읽기/쓰기, 터미널 명령 실행까지 가능하다.

```
Obsidian UI (사이드바 채팅)
        ↕ JSON-RPC / subprocess
Claude Code CLI (claude 명령)
        ↕
Anthropic API (Claude 모델)
```

---

## 주요 기능

### 1. 파일 조작 (진짜 일하는 AI)
- 현재 열린 노트 **자동 컨텍스트** 로드
- `@` 입력으로 **특정 파일/폴더** 첨부
- 파일 **직접 생성·수정** (Claude가 실행)
- 터미널 **bash 명령어** 실행 지원

### 2. 스마트 노트 핀 📌
- 파일 칩의 핀 버튼으로 노트 고정
- 다른 노트를 탐색해도 컨텍스트 유지
- `Cmd/Ctrl + P` → "Attach current note to chat"

### 3. 권한 모드 3단계
| 모드 | 설명 | 용도 |
|------|------|------|
| **AUTO** | 파일 수정·명령 자동 실행 | 빠른 작업 |
| **Safe** | 모든 작업에 승인 필요 | 신중한 작업 (권장) |
| **Plan** | 실행 없이 계획만 수립 | 검토 후 실행 |

### 4. Obsidian Skills 시스템
Claude가 Obsidian 문법을 이해하도록 Skills 설치 가능

**기본 제공 Skills:**
- `obsidian-markdown`: `[[wikilinks]]`, `![[embeds]]`, callout, YAML frontmatter, Mermaid, LaTeX
- `json-canvas`: `.canvas` 파일 생성·편집

**Community Skills**: GitHub URL로 외부 Skills 설치 가능

### 5. 기타
- 이미지 붙여넣기 (`Cmd/Ctrl+V`) → Claude가 분석
- 텍스트 선택 후 단축키 → **Inline Edit** (변경 내용 추적 방식)
- `#` 입력 → 시스템 프롬프트(지시사항) 추가
- `/` 입력 → 명령어 템플릿 목록
- **MCP 연동**으로 외부 도구 확장

---

## 기술 구조

```
src/
├── core/       # 핵심 엔진 (Claude CLI 연동)
├── features/   # 기능별 모듈
├── ui/         # 사이드바 UI 컴포넌트
├── utils/      # 유틸리티
├── style/      # CSS 스타일
└── main.ts     # 플러그인 진입점
```

- **빌드**: esbuild (`esbuild.config.mjs`)
- **테스트**: Jest (`jest.config.js`)
- **데스크탑 전용** (`isDesktopOnly: true`)

---

## 설치 방법

### BRAT 사용 (권장)
1. Obsidian 커뮤니티 플러그인에서 **BRAT** 설치
2. `Cmd/Ctrl + P` → `BRAT: Add a beta plugin for testing`
3. URL 입력: `https://github.com/reallygood83/cc-obsidian`
4. 설정 → 커뮤니티 플러그인 → **Obsidian Code** 활성화

### 선행 조건
```bash
# Claude Code CLI 설치 필수
npm install -g @anthropic-ai/claude-code

# 터미널에서 1회 인증
claude
```

---

## 설정 포인트

| 항목 | 내용 |
|------|------|
| 사용자 이름 | Claude가 부를 이름 |
| Obsidian Skills | Install Skills 버튼으로 원클릭 설치 |
| 미디어 폴더 | 이미지 붙여넣기 저장 경로 |
| Safety | `rm -rf` 등 위험 명령 차단 설정 |
| CLI 경로 | CLI 못 찾을 때 `which claude`로 경로 직접 지정 |

---

## 활용 아이디어

- **일일 노트 자동 정리**: "오늘 노트 요약해서 Weekly Note에 추가해줘"
- **노트 간 링크 생성**: "관련 노트들 연결해줘"
- **템플릿 기반 노트 생성**: 회의록, 독서 노트 자동 생성
- **Vault 검색 및 분석**: 특정 태그 노트 모아서 분석
- **MCP 연동**: 외부 API·DB와 연결해 정보 자동 수집

---

## 관련 링크

- [GitHub (obsidian-code)](https://github.com/reallygood83/obsidian-code)
- [YouTube - 배움의달인](https://www.youtube.com/@배움의달인-p5v)
- [Claude Code 공식 문서](https://docs.anthropic.com/claude-code)
