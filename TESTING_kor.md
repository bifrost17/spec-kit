# 테스트 가이드

이 문서는 [`CONTRIBUTING.md`](./CONTRIBUTING.md)의 상세 테스트 보조 문서입니다.

세 가지 용도로 사용하세요:

1. 수동 테스트 전 빠른 자동화 검사 실행,
2. AI 에이전트를 통한 영향받는 슬래시 명령어 수동 테스트, 그리고
3. PR에 적합한 형식으로 결과 기록.

슬래시 명령어의 동작에 영향을 미치는 모든 변경사항은 AI 에이전트를 통해 해당 명령어를 수동으로 테스트하고, 결과를 PR과 함께 제출해야 합니다.

## 권장 순서

1. **환경 동기화** — 프로젝트 및 테스트 의존성을 설치합니다.
2. **집중 자동화 검사 실행** — 특히 패키징, 스캐폴딩, 에이전트 설정, 생성된 파일 변경사항을 확인합니다.
3. **수동 에이전트 테스트 실행** — 영향받는 슬래시 명령어에 대해 실행합니다.
4. **PR에 결과 붙여넣기** — 명령어 선택 근거와 수동 테스트 결과를 모두 포함합니다.

## 빠른 자동화 검사

변경사항이 패키징, 스캐폴딩, 템플릿, 릴리스 아티팩트 또는 에이전트 배선에 영향을 미칠 때 수동 테스트 전에 실행하세요.

### 환경 설정

```bash
cd <spec-kit-repo>
uv sync --extra test
source .venv/bin/activate  # Windows (CMD): .venv\Scripts\activate  |  (PowerShell): .venv\Scripts\Activate.ps1
```

### 생성된 패키지 구조 및 내용

```bash
uv run python -m pytest tests/test_core_pack_scaffold.py -q
```

이것은 CI 스타일 패키징이 의존하는 생성된 파일을 검증합니다. 디렉토리 레이아웃, 파일 이름, 프론트매터/TOML 유효성, 플레이스홀더 대체, `.specify/` 경로 재작성, `create-release-packages.sh`와의 동등성을 포함합니다.

### 에이전트 설정 및 릴리스 배선 일관성

```bash
uv run python -m pytest tests/test_agent_config_consistency.py -q
```

에이전트 메타데이터, 릴리스 스크립트, 컨텍스트 업데이트 스크립트 또는 아티팩트 이름 지정을 변경할 때 실행하세요.

### 선택적 단일 에이전트 패키징 스팟 체크

```bash
AGENTS=copilot SCRIPTS=sh ./.github/workflows/scripts/create-release-packages.sh v1.0.0
```

하나의 에이전트/스크립트 조합에 대해 정확한 패키지 출력을 검토하고 싶을 때 `.genreleases/sdd-copilot-package-sh/`와 `.genreleases/`의 해당 ZIP을 검사하세요.

## 수동 테스트 프로세스

1. **영향받는 명령어 식별** — 아래의 [프롬프트](#실행할-테스트-결정하기)를 사용하여 에이전트가 변경된 파일을 분석하고 테스트가 필요한 명령어를 결정하도록 합니다.
2. **테스트 프로젝트 설정** — 로컬 브랜치에서 스캐폴딩합니다 ([설정](#설정) 참조).
3. **각 영향받는 명령어 실행** — 에이전트에서 호출하고, 성공적으로 완료되는지 확인하며, 예상 출력(생성된 파일, 실행된 스크립트, 채워진 아티팩트)을 확인합니다.
4. **전제 조건을 먼저 실행** — 이전 명령어에 의존하는 명령어(예: `/speckit.tasks`는 `/speckit.plan`이 필요하고, 이는 `/speckit.specify`가 필요함)는 순서대로 실행해야 합니다.
5. **결과 보고** — PR에 각 테스트된 명령어의 합격/불합격과 함께 [보고 템플릿](#결과-보고)을 붙여넣습니다.

## 설정

```bash
# 로컬 브랜치에서 프로젝트 및 테스트 의존성 설치
cd <spec-kit-repo>
uv sync --extra test
source .venv/bin/activate  # Windows (CMD): .venv\Scripts\activate  |  (PowerShell): .venv\Scripts\Activate.ps1
uv pip install -e .
# 이 환경의 `specify` 바이너리가 작업 트리를 가리키도록 하여 에이전트가 테스트 중인 브랜치를 실행하게 합니다.

# 로컬 변경사항을 사용하여 테스트 프로젝트 초기화
uv run specify init /tmp/speckit-test --ai <agent> --offline
cd /tmp/speckit-test

# 에이전트에서 열기
```

라이브 소스 트리가 아닌 패키지된 출력을 테스트하는 경우, [`CONTRIBUTING.md`](./CONTRIBUTING.md)에 설명된 대로 먼저 로컬 릴리스 패키지를 생성하세요.

## 결과 보고

PR에 이것을 붙여넣으세요:

~~~markdown
## 수동 테스트 결과

**에이전트**: [예: VS Code의 GitHub Copilot]  |  **OS/셸**: [예: macOS/zsh]

| 테스트한 명령어 | 비고 |
|----------------|------|
| `/speckit.command` | |
~~~

## 실행할 테스트 결정하기

이 프롬프트를 에이전트에 복사하세요. 에이전트의 응답(선택된 테스트와 매핑에 대한 간략한 설명)을 PR에 포함하세요.

~~~text
Read TESTING.md, then run `git diff --name-only main` to get my changed files.
For each changed file, determine which slash commands it affects by reading
the command templates in templates/commands/ to understand what each command
invokes. Use these mapping rules:

- templates/commands/X.md → the command it defines
- scripts/bash/Y.sh or scripts/powershell/Y.ps1 → every command that invokes that script (grep templates/commands/ for the script name). Also check transitive dependencies: if the changed script is sourced by other scripts (e.g., common.sh is sourced by create-new-feature.sh, check-prerequisites.sh, setup-plan.sh, update-agent-context.sh), then every command invoking those downstream scripts is also affected
- templates/Z-template.md → every command that consumes that template during execution
- src/specify_cli/*.py → CLI commands (`specify init`, `specify check`, `specify extension *`, `specify preset *`); test the affected CLI command and, for init/scaffolding changes, at minimum test /speckit.specify
- extensions/X/commands/* → the extension command it defines
- extensions/X/scripts/* → every extension command that invokes that script
- extensions/X/extension.yml or config-template.yml → every command in that extension. Also check if the manifest defines hooks (look for `hooks:` entries like `before_specify`, `after_implement`, etc.) — if so, the core commands those hooks attach to are also affected
- presets/*/* → test preset scaffolding via `specify init` with the preset
- pyproject.toml → packaging/bundling; test `specify init` and verify bundled assets

Include prerequisite tests (e.g., T5 requires T3 requires T1).

Output in this format:

### Test selection reasoning

| Changed file | Affects | Test | Why |
|---|---|---|---|
| (path) | (command) | T# | (reason) |

### Required tests

Number each test sequentially (T1, T2, ...). List prerequisite tests first.

- T1: /speckit.command — (reason)
- T2: /speckit.command — (reason)
~~~
