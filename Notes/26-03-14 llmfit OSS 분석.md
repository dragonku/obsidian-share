---
title: llmfit OSS 분석
created: 2026-03-14
updated: 2026-03-14
type: reference
status: active
tags:
  - LLM
  - CLI
  - Rust
  - LocalAI
  - OSS
aliases:
  - llmfit
source: https://github.com/AlexsJones/llmfit
---

# llmfit

> **"Hundreds of models & providers. One command to find what runs on your hardware."**  
> ⭐ 16,426 | Fork 923 | Rust | MIT | 2026-02-15 생성

---

## 한 줄 요약

내 하드웨어(RAM/CPU/GPU)를 자동 감지하고, 수백 개의 LLM 모델 중 **실제로 잘 돌아갈 모델**을 점수 기반으로 추천해주는 터미널 도구.

---

## 핵심 기능

### 하드웨어 자동 감지
- RAM, CPU, GPU 이름, VRAM, 백엔드 자동 탐지
- 멀티 GPU 지원
- MoE(Mixture-of-Experts) 아키텍처 인식 → 실제 활성 파라미터로 메모리 계산

### 모델 스코어링 (4가지 차원)
| 항목 | 설명 |
|------|------|
| **Quality** | 모델 품질 점수 |
| **Speed** | 예상 tok/s |
| **Fit** | 내 하드웨어 적합도 (Perfect / Good / Marginal) |
| **Context** | 지원 컨텍스트 길이 |

### TUI (기본) + CLI 모드
- 기본: 인터랙티브 터미널 UI (Vim 키바인딩)
- `--cli` 플래그로 스크립트 자동화 가능
- `--json` 플래그로 JSON 출력 → `jq`와 조합

---

## 설치

```bash
# macOS / Linux (Homebrew)
brew install llmfit

# 빠른 설치
curl -fsSL https://llmfit.axjns.dev/install.sh | sh

# Windows (Scoop)
scoop install llmfit

# Docker
docker run ghcr.io/alexsjones/llmfit

# 소스 빌드
cargo build --release
```

---

## 주요 사용법

```bash
# TUI 실행 (기본)
llmfit

# CLI 모드 - 추천 모델 5개
llmfit fit --perfect -n 5

# 용도별 추천
llmfit recommend --use-case coding

# JSON 출력
llmfit recommend --json | jq '.models[].name'

# 메모리 수동 지정
llmfit --memory=16G fit -n 10

# 시스템 정보 확인
llmfit --json system

# 원격 Ollama 서버 연결
OLLAMA_HOST="http://192.168.1.100:11434" llmfit
```

---

## TUI 키바인딩

| 키 | 동작 |
|----|------|
| `j` / `k` | 모델 위아래 탐색 |
| `/` | 검색 모드 |
| `f` | Fit 필터 순환 (All → Runnable → Perfect → Good → Marginal) |
| `s` | 정렬 기준 변경 (Score / Params / Mem% / Ctx / Date) |
| `d` | 선택 모델 다운로드 (Ollama) |
| `p` | Plan 모드 - 이 모델에 필요한 하드웨어 역산 |
| `m` / `c` | 모델 마킹 / 비교 뷰 |
| `v` | Visual 모드 (다중 선택 비교) |
| `V` | Select 모드 (컬럼 기반 필터) |
| `i` | 설치된 모델 우선 정렬 |
| `r` | 설치 모델 새로고침 |
| `Enter` | 상세 뷰 토글 |
| `q` | 종료 |

---

## 런타임 프로바이더 연동

| 프로바이더 | 기능 |
|-----------|------|
| **Ollama** | 설치 감지, TUI에서 직접 다운로드, 원격 서버 지원 |
| **llama.cpp** | GGUF 다운로드, 설치 감지 |
| **MLX** | Apple Silicon 지원 |

---

## 플랫폼 지원

| 플랫폼 | GPU 감지 |
|--------|---------|
| Linux | NVIDIA (`nvidia-smi`), AMD (`rocm-smi`), Intel Arc (sysfs), Ascend (`npu-smi`) |
| macOS Apple Silicon | Unified memory (`system_profiler`) - VRAM = 시스템 RAM |
| macOS Intel | `nvidia-smi` 있으면 가능 |
| Windows | NVIDIA (`nvidia-smi`) |
| Android/Termux | GPU 미지원 (수동 `--memory` 지정으로 우회) |

---

## 특이사항

### Plan 모드 (`p`)
일반 Fit 분석의 역방향: "이 모델을 돌리려면 어떤 하드웨어가 필요한가?" 를 계산

### OpenClaw 스킬 연동
- llmfit이 [OpenClaw](https://github.com/openclaw/openclaw) AI 에이전트의 스킬로 내장
- 에이전트가 `llmfit recommend --json` 호출 → 결과 해석 → Ollama/vLLM 자동 설정
- "What local models can I run?" 같은 자연어 질의 처리

### 모델 DB
- HuggingFace 모델명 기반 (`Qwen/Qwen2.5-Coder-14B-Instruct`)
- Ollama 태그와 정확한 매핑 테이블 유지
- MoE 모델 활성 파라미터 기반 메모리 계산 (llm-checker와 차별점)

---

## 활용 시나리오

- **모델 선택 전 사전 검토**: 다운로드 전 내 PC에서 돌아가는지 확인
- **서버 스펙 결정**: Plan 모드로 원하는 모델의 필요 VRAM 역산
- **Ollama 자동화**: JSON 출력 + jq로 스크립트에서 최적 모델 자동 선택
- **원격 GPU 서버**: `OLLAMA_HOST`로 로컬 UI + 원격 GPU 서버 조합

---

## 관련 프로젝트

- [sympozium](https://github.com/AlexsJones/sympozium) - Kubernetes 에이전트 관리 (동일 제작자)
- [llm-checker](https://github.com/Pavelevich/llm-checker) - Node.js, 실제 Ollama 실행 기반 벤치마크 (MoE 미지원)
