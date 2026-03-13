---
title: Language Server Protocol (LSP)
created: 2026-03-13
updated: 2026-03-13
type: reference
status: active
tags:
  - LSP
  - IDE
  - 개발도구
  - Protocol
aliases:
  - LSP
  - 언어 서버 프로토콜
---

# Language Server Protocol (LSP)

## 개요

LSP는 Microsoft가 설계한 **오픈 프로토콜**로, 코드 편집기(클라이언트)와 언어 분석 도구(서버) 간의 통신을 표준화한다.
각 언어마다 별도의 플러그인을 만들 필요 없이, LSP를 구현한 언어 서버 하나로 모든 편집기에서 동일한 기능을 사용할 수 있다.

```
편집기 (VS Code, Neovim, Emacs...) ←── JSON-RPC ──→ Language Server (pylsp, gopls...)
```

## 핵심 기능

| 기능 | 설명 |
|------|------|
| **자동완성** | 컨텍스트 기반 코드 제안 |
| **Hover** | 심볼 위에 마우스를 올리면 타입/문서 표시 |
| **Go to Definition** | 정의된 위치로 이동 |
| **Find References** | 해당 심볼이 사용된 모든 위치 탐색 |
| **Rename** | 프로젝트 전체에서 심볼 일괄 이름 변경 |
| **Diagnostics** | 실시간 오류/경고 표시 |
| **Code Actions** | 자동 수정, 리팩토링 제안 |
| **Document Symbols** | 파일 내 심볼 목록 (클래스, 함수 등) |
| **Workspace Symbols** | 프로젝트 전체 심볼 검색 |

## 동작 방식

1. 편집기가 LSP 서버를 **프로세스로 실행**
2. **JSON-RPC** 프로토콜로 요청/응답 교환
3. 서버는 파일 시스템을 분석해 AST(Abstract Syntax Tree) 생성
4. 요청에 따라 정적 분석 결과 반환

## 언어별 서버 목록 (현재 환경 기준)

### JavaScript / TypeScript
- **서버**: `typescript-language-server`
- **확장자**: `.ts`, `.tsx`, `.js`, `.jsx`, `.mts`, `.cts`, `.mjs`, `.cjs`
- **설치**: `npm install -g typescript-language-server typescript`

### Python
- **서버**: `pylsp`
- **확장자**: `.py`, `.pyw`
- **설치**: `pip install python-lsp-server`

### Go
- **서버**: `gopls`
- **확장자**: `.go`
- **설치**: `go install golang.org/x/tools/gopls@latest`

### Rust
- **서버**: `rust-analyzer`
- **확장자**: `.rs`
- **설치**: `rustup component add rust-analyzer`

### Java
- **서버**: `jdtls` (Eclipse JDT Language Server)
- **확장자**: `.java`
- **설치**: [eclipse.jdt.ls](https://github.com/eclipse/eclipse.jdt.ls)

### C / C++
- **서버**: `clangd`
- **확장자**: `.c`, `.h`, `.cpp`, `.cc`, `.cxx`, `.hpp`, `.hxx`
- **설치**: 패키지 매니저 또는 LLVM에서 설치

### Web (JSON / HTML / CSS)
- **서버**: `vscode-langservers-extracted`
- **설치**: `npm install -g vscode-langservers-extracted`

### 기타
| 언어 | 서버 | 설치 |
|------|------|------|
| YAML | `yaml-language-server` | `npm install -g yaml-language-server` |
| PHP | `intelephense` | `npm install -g intelephense` |
| Ruby | `solargraph` | `gem install solargraph` |
| Lua | `lua-language-server` | [LuaLS GitHub](https://github.com/LuaLS/lua-language-server) |
| Kotlin | `kotlin-language-server` | [GitHub](https://github.com/fwcd/kotlin-language-server) |
| Elixir | `elixir-ls` | [GitHub](https://github.com/elixir-lsp/elixir-ls) |
| C# | `omnisharp` | `dotnet tool install -g omnisharp` |
| Dart/Flutter | `dart` | [dart.dev](https://dart.dev/get-dart) |
| Swift | `sourcekit-lsp` | Swift SDK 또는 Xcode |

## LSP vs 전통 방식

| | 전통 방식 | LSP |
|---|---|---|
| 구조 | 언어 × 편집기 수만큼 플러그인 필요 | 언어 서버 1개로 모든 편집기 지원 |
| 유지보수 | M × N 문제 | M + N 문제로 단순화 |
| 일관성 | 편집기마다 기능 차이 | 동일한 분석 결과 |

## 관련 링크

- [공식 스펙](https://microsoft.github.io/language-server-protocol/)
- [지원 서버 목록](https://microsoft.github.io/language-server-protocol/implementors/servers/)
