# AGENTS.md

## Spec Kit과 Specify에 대하여

**GitHub Spec Kit**은 명세 주도 개발(Spec-Driven Development, SDD)을 구현하기 위한 종합 툴킷입니다. SDD는 구현 전에 명확한 명세를 작성하는 것을 강조하는 방법론입니다. 이 툴킷에는 개발 팀이 소프트웨어를 구조적으로 구축할 수 있도록 안내하는 템플릿, 스크립트, 워크플로우가 포함되어 있습니다.

**Specify CLI**는 Spec Kit 프레임워크로 프로젝트를 초기화하는 명령줄 인터페이스입니다. 명세 주도 개발 워크플로우를 지원하기 위해 필요한 디렉토리 구조, 템플릿, AI 에이전트 통합을 설정합니다.

이 툴킷은 여러 AI 코딩 어시스턴트를 지원하므로, 팀은 일관된 프로젝트 구조와 개발 방식을 유지하면서 선호하는 도구를 사용할 수 있습니다.

---

## 새 에이전트 지원 추가

이 섹션에서는 Specify CLI에 새 AI 에이전트/어시스턴트 지원을 추가하는 방법을 설명합니다. 새 AI 도구를 명세 주도 개발 워크플로우에 통합할 때 이 가이드를 참고하세요.

### 개요

Specify는 프로젝트 초기화 시 에이전트별 명령 파일과 디렉토리 구조를 생성하여 여러 AI 에이전트를 지원합니다. 각 에이전트는 다음 항목에 대한 고유한 규칙을 가집니다:

- **명령 파일 형식** (Markdown, TOML 등)
- **디렉토리 구조** (`.claude/commands/`, `.windsurf/workflows/` 등)
- **명령 호출 패턴** (슬래시 명령, CLI 도구 등)
- **인수 전달 규칙** (`$ARGUMENTS`, `{{args}}` 등)

### 현재 지원되는 에이전트

| 에이전트                   | 디렉토리               | 형식     | CLI 도구        | 설명                        |
| -------------------------- | ---------------------- | -------- | --------------- | --------------------------- |
| **Claude Code**            | `.claude/commands/`    | Markdown | `claude`        | Anthropic's Claude Code CLI |
| **Gemini CLI**             | `.gemini/commands/`    | TOML     | `gemini`        | Google's Gemini CLI         |
| **GitHub Copilot**         | `.github/agents/`      | Markdown | N/A (IDE-based) | GitHub Copilot in VS Code   |
| **Cursor**                 | `.cursor/commands/`    | Markdown | N/A (IDE-based) | Cursor IDE (`--ai cursor-agent`) |
| **Qwen Code**              | `.qwen/commands/`      | Markdown | `qwen`          | Alibaba's Qwen Code CLI     |
| **opencode**               | `.opencode/command/`   | Markdown | `opencode`      | opencode CLI                |
| **Codex CLI**              | `.agents/skills/`      | Markdown | `codex`         | Codex CLI (`--ai codex --ai-skills`) |
| **Windsurf**               | `.windsurf/workflows/` | Markdown | N/A (IDE-based) | Windsurf IDE workflows      |
| **Junie**                  | `.junie/commands/`     | Markdown | `junie`         | Junie by JetBrains          |
| **Kilo Code**              | `.kilocode/workflows/` | Markdown | N/A (IDE-based) | Kilo Code IDE               |
| **Auggie CLI**             | `.augment/commands/`   | Markdown | `auggie`        | Auggie CLI                  |
| **Roo Code**               | `.roo/commands/`       | Markdown | N/A (IDE-based) | Roo Code IDE                |
| **CodeBuddy CLI**          | `.codebuddy/commands/` | Markdown | `codebuddy`     | CodeBuddy CLI               |
| **Qoder CLI**              | `.qoder/commands/`     | Markdown | `qodercli`      | Qoder CLI                   |
| **Kiro CLI**               | `.kiro/prompts/`       | Markdown | `kiro-cli`      | Kiro CLI                    |
| **Amp**                    | `.agents/commands/`    | Markdown | `amp`           | Amp CLI                     |
| **SHAI**                   | `.shai/commands/`      | Markdown | `shai`          | SHAI CLI                    |
| **Tabnine CLI**            | `.tabnine/agent/commands/` | TOML | `tabnine`       | Tabnine CLI                 |
| **Kimi Code**              | `.kimi/skills/`        | Markdown | `kimi`          | Kimi Code CLI (Moonshot AI) |
| **Pi Coding Agent**        | `.pi/prompts/`         | Markdown | `pi`            | Pi terminal coding agent    |
| **iFlow CLI**              | `.iflow/commands/`     | Markdown | `iflow`         | iFlow CLI (iflow-ai)        |
| **Forge**                  | `.forge/commands/`     | Markdown | `forge`         | Forge CLI (forgecode.dev)   |
| **IBM Bob**                | `.bob/commands/`       | Markdown | N/A (IDE-based) | IBM Bob IDE                 |
| **Trae**                   | `.trae/rules/`         | Markdown | N/A (IDE-based) | Trae IDE                    |
| **Antigravity**            | `.agent/commands/`     | Markdown | N/A (IDE-based) | Antigravity IDE (`--ai agy --ai-skills`) |
| **Mistral Vibe**           | `.vibe/prompts/`       | Markdown | `vibe`          | Mistral Vibe CLI            |
| **Generic**                | `--ai-commands-dir`로 사용자 지정 | Markdown | N/A | 사용자 정의 에이전트        |

### 단계별 통합 가이드

새 에이전트를 추가하려면 다음 단계를 따르세요 (가상의 새 에이전트를 예시로 사용):

#### 1. AGENT_CONFIG에 추가

**중요**: 딕셔너리 키로 단축 이름이 아닌 실제 CLI 도구 이름을 사용하세요.

`src/specify_cli/__init__.py`의 `AGENT_CONFIG` 딕셔너리에 새 에이전트를 추가합니다. 이것이 모든 에이전트 메타데이터의 **단일 출처(single source of truth)**입니다:

```python
AGENT_CONFIG = {
    # ... existing agents ...
    "new-agent-cli": {  # Use the ACTUAL CLI tool name (what users type in terminal)
        "name": "New Agent Display Name",
        "folder": ".newagent/",  # Directory for agent files
        "commands_subdir": "commands",  # Subdirectory name for command files (default: "commands")
        "install_url": "https://example.com/install",  # URL for installation docs (or None if IDE-based)
        "requires_cli": True,  # True if CLI tool required, False for IDE-based agents
    },
}
```

**핵심 설계 원칙**: 딕셔너리 키는 사용자가 설치하는 실제 실행 파일 이름과 일치해야 합니다. 예:

- ✅ CLI 도구가 실제로 `cursor-agent`라는 이름이므로 `"cursor-agent"`를 사용
- ❌ 도구 이름이 `cursor-agent`인데 `"cursor"`처럼 단축 이름을 사용하지 말 것

이 방식으로 코드베이스 전체에 걸친 특수 케이스 매핑의 필요성을 없앨 수 있습니다.

**필드 설명**:

- `name`: 사용자에게 표시되는 사람이 읽을 수 있는 표시 이름
- `folder`: 에이전트별 파일이 저장되는 디렉토리 (프로젝트 루트 기준 상대 경로)
- `commands_subdir`: 명령/프롬프트 파일이 저장되는 에이전트 폴더 내 하위 디렉토리 이름 (기본값: `"commands"`)
  - 대부분의 에이전트는 `"commands"`를 사용 (예: `.claude/commands/`)
  - 일부 에이전트는 다른 이름 사용: `"agents"` (copilot), `"workflows"` (windsurf, kilocode), `"prompts"` (codex, kiro-cli, pi), `"command"` (opencode - 단수형)
  - 이 필드를 통해 `--ai-skills`가 스킬 생성 시 올바른 명령 템플릿을 찾을 수 있음
- `install_url`: 설치 문서 URL (IDE 기반 에이전트의 경우 `None`으로 설정)
- `requires_cli`: 초기화 중 CLI 도구 확인이 필요한지 여부

#### 2. CLI 도움말 텍스트 업데이트

`init()` 명령의 `--ai` 파라미터 도움말 텍스트를 업데이트하여 새 에이전트를 포함시킵니다:

```python
ai_assistant: str = typer.Option(None, "--ai", help="AI assistant to use: claude, gemini, copilot, cursor-agent, qwen, opencode, codex, windsurf, kilocode, auggie, codebuddy, new-agent-cli, or kiro-cli"),
```

사용 가능한 에이전트를 나열하는 함수 독스트링, 예시, 오류 메시지도 함께 업데이트하세요.

#### 3. README 문서 업데이트

`README.md`의 **지원되는 AI 에이전트** 섹션을 업데이트하여 새 에이전트를 포함시킵니다:

- 적절한 지원 수준(Full/Partial)과 함께 새 에이전트를 표에 추가
- 에이전트의 공식 웹사이트 링크 포함
- 에이전트 구현에 관련된 주요 사항 추가
- 표 서식이 정렬되고 일관되게 유지되도록 확인

#### 4. 릴리스 패키지 스크립트 업데이트

`.github/workflows/scripts/create-release-packages.sh`를 수정합니다:

##### ALL_AGENTS 배열에 추가

```bash
ALL_AGENTS=(claude gemini copilot cursor-agent qwen opencode windsurf kiro-cli)
```

##### 디렉토리 구조를 위한 case 구문 추가

```bash
case $agent in
  # ... existing cases ...
  windsurf)
    mkdir -p "$base_dir/.windsurf/workflows"
    generate_commands windsurf md "\$ARGUMENTS" "$base_dir/.windsurf/workflows" "$script" ;;
esac
```

#### 4. GitHub 릴리스 스크립트 업데이트

`.github/workflows/scripts/create-github-release.sh`를 수정하여 새 에이전트의 패키지를 포함시킵니다:

```bash
gh release create "$VERSION" \
  # ... existing packages ...
  .genreleases/spec-kit-template-windsurf-sh-"$VERSION".zip \
  .genreleases/spec-kit-template-windsurf-ps-"$VERSION".zip \
  # Add new agent packages here
```

#### 5. 에이전트 컨텍스트 스크립트 업데이트

##### Bash 스크립트 (`scripts/bash/update-agent-context.sh`)

파일 변수 추가:

```bash
WINDSURF_FILE="$REPO_ROOT/.windsurf/rules/specify-rules.md"
```

case 구문에 추가:

```bash
case "$AGENT_TYPE" in
  # ... existing cases ...
  windsurf) update_agent_file "$WINDSURF_FILE" "Windsurf" ;;
  "")
    # ... existing checks ...
    [ -f "$WINDSURF_FILE" ] && update_agent_file "$WINDSURF_FILE" "Windsurf";
    # Update default creation condition
    ;;
esac
```

##### PowerShell 스크립트 (`scripts/powershell/update-agent-context.ps1`)

파일 변수 추가:

```powershell
$windsurfFile = Join-Path $repoRoot '.windsurf/rules/specify-rules.md'
```

switch 구문에 추가:

```powershell
switch ($AgentType) {
    # ... existing cases ...
    'windsurf' { Update-AgentFile $windsurfFile 'Windsurf' }
    '' {
        foreach ($pair in @(
            # ... existing pairs ...
            @{file=$windsurfFile; name='Windsurf'}
        )) {
            if (Test-Path $pair.file) { Update-AgentFile $pair.file $pair.name }
        }
        # Update default creation condition
    }
}
```

#### 6. CLI 도구 확인 업데이트 (선택 사항)

CLI 도구가 필요한 에이전트의 경우, `check()` 명령과 에이전트 유효성 검사에 확인 코드를 추가합니다:

```python
# In check() command
tracker.add("windsurf", "Windsurf IDE (optional)")
windsurf_ok = check_tool_for_tracker("windsurf", "https://windsurf.com/", tracker)

# In init validation (only if CLI tool required)
elif selected_ai == "windsurf":
    if not check_tool("windsurf", "Install from: https://windsurf.com/"):
        console.print("[red]Error:[/red] Windsurf CLI is required for Windsurf projects")
        agent_tool_missing = True
```

**참고**: CLI 도구 확인은 이제 AGENT_CONFIG의 `requires_cli` 필드를 기반으로 자동으로 처리됩니다. `check()` 또는 `init()` 명령에 별도의 코드 변경이 필요하지 않습니다. 이 명령들은 AGENT_CONFIG를 자동으로 순회하여 필요에 따라 도구를 확인합니다.

## 중요한 설계 결정 사항

### 실제 CLI 도구 이름을 키로 사용

**중요**: AGENT_CONFIG에 새 에이전트를 추가할 때, 단축 이름이나 편의상의 이름이 아닌 **실제 실행 파일 이름**을 딕셔너리 키로 항상 사용하세요.

**이것이 중요한 이유:**

- `check_tool()` 함수는 `shutil.which(tool)`을 사용하여 시스템 PATH에서 실행 파일을 찾습니다
- 키가 실제 CLI 도구 이름과 일치하지 않으면 코드베이스 전체에 특수 케이스 매핑이 필요하게 됩니다
- 이는 불필요한 복잡성과 유지보수 부담을 초래합니다

**예시 - Cursor의 교훈:**

❌ **잘못된 방법** (특수 케이스 매핑 필요):

```python
AGENT_CONFIG = {
    "cursor": {  # Shorthand that doesn't match the actual tool
        "name": "Cursor",
        # ...
    }
}

# Then you need special cases everywhere:
cli_tool = agent_key
if agent_key == "cursor":
    cli_tool = "cursor-agent"  # Map to the real tool name
```

✅ **올바른 방법** (매핑 불필요):

```python
AGENT_CONFIG = {
    "cursor-agent": {  # Matches the actual executable name
        "name": "Cursor",
        # ...
    }
}

# No special cases needed - just use agent_key directly!
```

**이 방법의 장점:**

- 코드베이스 전체에 분산된 특수 케이스 로직 제거
- 코드의 유지보수성과 가독성 향상
- 새 에이전트 추가 시 버그 발생 가능성 감소
- 추가 매핑 없이 도구 확인이 자동으로 동작

#### 7. Devcontainer 파일 업데이트 (선택 사항)

VS Code 확장 프로그램이 있거나 CLI 설치가 필요한 에이전트의 경우, devcontainer 설정 파일을 업데이트합니다:

##### VS Code 확장 프로그램 기반 에이전트

VS Code 확장 프로그램으로 제공되는 에이전트는 `.devcontainer/devcontainer.json`에 추가합니다:

```json
{
  "customizations": {
    "vscode": {
      "extensions": [
        // ... existing extensions ...
        // [New Agent Name]
        "[New Agent Extension ID]"
      ]
    }
  }
}
```

##### CLI 기반 에이전트

CLI 도구가 필요한 에이전트는 `.devcontainer/post-create.sh`에 설치 명령을 추가합니다:

```bash
#!/bin/bash

# Existing installations...

echo -e "\n🤖 Installing [New Agent Name] CLI..."
# run_command "npm install -g [agent-cli-package]@latest" # Example for node-based CLI
# or other installation instructions (must be non-interactive and compatible with Linux Debian "Trixie" or later)...
echo "✅ Done"

```

**빠른 팁:**

- **확장 프로그램 기반 에이전트**: `devcontainer.json`의 `extensions` 배열에 추가
- **CLI 기반 에이전트**: `post-create.sh`에 설치 스크립트 추가
- **하이브리드 에이전트**: 확장 프로그램과 CLI 설치 모두 필요할 수 있음
- **충분한 테스트**: devcontainer 환경에서 설치가 정상 동작하는지 확인

## 에이전트 분류

### CLI 기반 에이전트

명령줄 도구 설치가 필요한 에이전트:

- **Claude Code**: `claude` CLI
- **Gemini CLI**: `gemini` CLI
- **Qwen Code**: `qwen` CLI
- **opencode**: `opencode` CLI
- **Codex CLI**: `codex` CLI (`--ai-skills` 필요)
- **Junie**: `junie` CLI
- **Auggie CLI**: `auggie` CLI
- **CodeBuddy CLI**: `codebuddy` CLI
- **Qoder CLI**: `qodercli` CLI
- **Kiro CLI**: `kiro-cli` CLI
- **Amp**: `amp` CLI
- **SHAI**: `shai` CLI
- **Tabnine CLI**: `tabnine` CLI
- **Kimi Code**: `kimi` CLI
- **Mistral Vibe**: `vibe` CLI
- **Pi Coding Agent**: `pi` CLI
- **iFlow CLI**: `iflow` CLI
- **Forge**: `forge` CLI

### IDE 기반 에이전트

통합 개발 환경(IDE) 내에서 동작하는 에이전트:

- **GitHub Copilot**: VS Code/호환 편집기에 내장
- **Cursor**: Cursor IDE에 내장 (`--ai cursor-agent`)
- **Windsurf**: Windsurf IDE에 내장
- **Kilo Code**: Kilo Code IDE에 내장
- **Roo Code**: Roo Code IDE에 내장
- **IBM Bob**: IBM Bob IDE에 내장
- **Trae**: Trae IDE에 내장
- **Antigravity**: Antigravity IDE에 내장 (`--ai agy --ai-skills`)

## 명령 파일 형식

### Markdown 형식

사용 에이전트: Claude, Cursor, GitHub Copilot, opencode, Windsurf, Junie, Kiro CLI, Amp, SHAI, IBM Bob, Kimi Code, Qwen, Pi, Codex, Auggie, CodeBuddy, Qoder, Roo Code, Kilo Code, Trae, Antigravity, Mistral Vibe, iFlow, Forge

**표준 형식:**

```markdown
---
description: "Command description"
---

Command content with {SCRIPT} and $ARGUMENTS placeholders.
```

**GitHub Copilot Chat Mode 형식:**

```markdown
---
description: "Command description"
mode: speckit.command-name
---

Command content with {SCRIPT} and $ARGUMENTS placeholders.
```

### TOML 형식

사용 에이전트: Gemini, Tabnine

```toml
description = "Command description"

prompt = """
Command content with {SCRIPT} and {{args}} placeholders.
"""
```

## 디렉토리 규칙

- **CLI 에이전트**: 보통 `.<agent-name>/commands/`
- **단수형 명령 예외**:
  - opencode: `.opencode/command/` (복수형 `commands`가 아닌 단수형 `command`)
- **중첩 경로 예외**:
  - Tabnine: `.tabnine/agent/commands/` (추가 `agent/` 세그먼트 포함)
- **공유 `.agents/` 폴더**:
  - Amp: `.agents/commands/` (`.amp/`가 아닌 공유 폴더)
  - Codex: `.agents/skills/` (공유 폴더; `--ai-skills` 필요; `$speckit-<command>`로 호출)
- **스킬 기반 예외**:
  - Kimi Code: `.kimi/skills/` (스킬 방식; `/skill:speckit-<command>`로 호출)
- **프롬프트 기반 예외**:
  - Kiro CLI: `.kiro/prompts/`
  - Pi: `.pi/prompts/`
  - Mistral Vibe: `.vibe/prompts/`
- **규칙 기반 예외**:
  - Trae: `.trae/rules/`
- **IDE 에이전트**: IDE별 패턴을 따름:
  - Copilot: `.github/agents/`
  - Cursor: `.cursor/commands/`
  - Windsurf: `.windsurf/workflows/`
  - Kilo Code: `.kilocode/workflows/`
  - Roo Code: `.roo/commands/`
  - IBM Bob: `.bob/commands/`
  - Antigravity: `.agent/skills/` (`--ai-skills` 필요; `.agent/commands/`는 더 이상 사용되지 않음)

## 인수 패턴

에이전트마다 서로 다른 인수 플레이스홀더를 사용합니다:

- **Markdown/프롬프트 기반**: `$ARGUMENTS`
- **TOML 기반**: `{{args}}`
- **Forge 전용**: `{{parameters}}` (사용자 정의 파라미터 문법 사용)
- **스크립트 플레이스홀더**: `{SCRIPT}` (실제 스크립트 경로로 교체됨)
- **에이전트 플레이스홀더**: `__AGENT__` (에이전트 이름으로 교체됨)

## 특수 처리 요구 사항

일부 에이전트는 표준 템플릿 변환 외에 추가적인 커스텀 처리가 필요합니다:

### Copilot 통합

GitHub Copilot은 고유한 요구 사항이 있습니다:
- 명령 파일에 `.md` 대신 `.agent.md` 확장자 사용
- 각 명령마다 `.github/prompts/`에 동반 `.prompt.md` 파일 생성
- 프롬프트 파일 추천을 위한 `.vscode/settings.json` 설치
- 컨텍스트 파일 위치: `.github/copilot-instructions.md`

구현: 다음 작업을 수행하는 커스텀 `setup()` 메서드로 `IntegrationBase`를 확장:
1. `process_template()`으로 템플릿 처리
2. 동반 `.prompt.md` 파일 생성
3. VS Code 설정 병합

### Forge 통합

Forge는 특수한 프론트매터 및 인수 요구 사항이 있습니다:
- `$ARGUMENTS` 대신 `{{parameters}}` 사용
- `handoffs` 프론트매터 키 제거 (Forge 전용 협업 기능)
- 프론트매터에 `name` 필드가 없을 경우 주입

구현: 다음 작업을 수행하는 커스텀 `setup()` 메서드로 `MarkdownIntegration`을 확장:
1. `MarkdownIntegration`으로부터 표준 템플릿 처리 상속
2. 템플릿 처리 후 추가적인 `$ARGUMENTS` → `{{parameters}}` 교체 수행
3. `_apply_forge_transformations()`를 통해 Forge 전용 변환 적용
4. `handoffs` 프론트매터 키 제거
5. 누락된 `name` 필드 주입
6. 공유 `update-agent-context.*` 스크립트에 `forge` case가 포함되어 컨텍스트 업데이트가 `AGENTS.md`에 매핑되도록 보장 (`opencode`/`codex`/`pi`와 유사) 하고, 사용법/도움말 텍스트에 `forge` 목록 포함

### 표준 Markdown 에이전트

대부분의 에이전트(Bob, Claude, Windsurf 등)는 `MarkdownIntegration`을 사용합니다:
- `key`, `config`, `registrar_config`만 설정하는 단순 서브클래스
- `MarkdownIntegration.setup()`으로부터 표준 처리 상속
- 커스텀 처리 불필요

## 새 에이전트 통합 테스트

1. **빌드 테스트**: 로컬에서 패키지 생성 스크립트 실행
2. **CLI 테스트**: `specify init --ai <agent>` 명령 테스트
3. **파일 생성 확인**: 올바른 디렉토리 구조와 파일 생성 여부 확인
4. **명령 유효성 검사**: 생성된 명령이 해당 에이전트에서 정상 동작하는지 확인
5. **컨텍스트 업데이트**: 에이전트 컨텍스트 업데이트 스크립트 테스트

## 일반적인 실수

1. **실제 CLI 도구 이름 대신 단축 키 사용**: AGENT_CONFIG 키로 항상 실제 실행 파일 이름을 사용하세요 (예: `"cursor"`가 아닌 `"cursor-agent"`). 이렇게 하면 코드베이스 전체에 특수 케이스 매핑이 필요하지 않습니다.
2. **업데이트 스크립트 누락**: 새 에이전트 추가 시 Bash와 PowerShell 스크립트 모두 업데이트해야 합니다.
3. **잘못된 `requires_cli` 값**: 실제로 확인할 CLI 도구가 있는 에이전트에만 `True`로 설정하고, IDE 기반 에이전트는 `False`로 설정하세요.
4. **잘못된 인수 형식**: 각 에이전트 유형에 맞는 올바른 플레이스홀더 형식을 사용하세요 (Markdown은 `$ARGUMENTS`, TOML은 `{{args}}`).
5. **디렉토리 명명**: 에이전트별 규칙을 정확하게 따르세요 (기존 에이전트에서 패턴 확인).
6. **도움말 텍스트 불일치**: 사용자에게 표시되는 모든 텍스트를 일관되게 업데이트하세요 (도움말 문자열, 독스트링, README, 오류 메시지).

## 향후 고려 사항

새 에이전트를 추가할 때:

- 에이전트의 기본 명령/워크플로우 패턴을 고려하세요
- 명세 주도 개발 프로세스와의 호환성을 확인하세요
- 특수 요구 사항이나 제한 사항을 문서화하세요
- 얻은 교훈을 반영하여 이 가이드를 업데이트하세요
- AGENT_CONFIG에 추가하기 전에 실제 CLI 도구 이름을 확인하세요

---

*이 문서는 정확성과 완전성을 유지하기 위해 새 에이전트가 추가될 때마다 업데이트되어야 합니다.*
