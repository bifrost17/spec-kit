# Spec Kit에 기여하기

안녕하세요! Spec Kit에 기여하고자 하시는 것을 환영합니다. 이 프로젝트에 대한 기여는 [프로젝트의 오픈 소스 라이선스](LICENSE)에 따라 공개적으로 [배포](https://help.github.com/articles/github-terms-of-service/#6-contributions-under-repository-license)됩니다.

이 프로젝트는 [기여자 행동 강령](CODE_OF_CONDUCT.md)과 함께 배포됩니다. 이 프로젝트에 참여함으로써 해당 조건을 준수하는 것에 동의하게 됩니다.

## 코드 실행 및 테스트를 위한 사전 요구 사항

아래 항목들은 PR 제출 과정의 일환으로 로컬에서 변경 사항을 테스트하기 위해 한 번만 설치하면 되는 필수 항목입니다.

1. [Python 3.11+](https://www.python.org/downloads/) 설치
1. 패키지 관리를 위한 [uv](https://docs.astral.sh/uv/) 설치
1. [Git](https://git-scm.com/downloads) 설치
1. [AI 코딩 에이전트 사용 가능](README.md#-supported-ai-agents)

<details>
<summary><b>💡 <code>VSCode</code> 또는 <code>GitHub Codespaces</code>를 IDE로 사용하는 경우 힌트</b></summary>

<br>

머신에 [Docker](https://docker.com)가 설치되어 있다면, 이 [VSCode 확장](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)을 통해 [Dev Containers](https://containers.dev)를 활용하여 개발 환경을 손쉽게 설정할 수 있습니다. 프로젝트 루트에 위치한 `.devcontainer/devcontainer.json` 파일 덕분에 앞서 언급한 도구들이 이미 설치 및 구성되어 있습니다.

이를 위해 다음과 같이 진행하세요:

- 저장소를 체크아웃합니다
- VSCode로 엽니다
- [명령 팔레트](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette)를 열고 "Dev Containers: Open Folder in Container..."를 선택합니다

[GitHub Codespaces](https://github.com/features/codespaces)에서는 codespace를 열 때 `.devcontainer/devcontainer.json`이 자동으로 적용되므로 더욱 간단합니다.

</details>

## Pull Request 제출하기

> [!NOTE]
> Pull request가 CLI의 동작이나 저장소 전반에 실질적인 영향을 미치는 대규모 변경 사항을 도입하는 경우(예: 새로운 템플릿, 인수 추가 또는 그 밖의 주요 변경), 프로젝트 유지 관리자들과 **충분히 논의하고 합의가 이루어졌는지** 확인하세요. 사전 논의 및 합의 없이 대규모 변경을 포함한 pull request는 닫힐 수 있습니다.

1. 저장소를 fork하고 clone합니다
1. 의존성을 구성하고 설치합니다: `uv sync --extra test`
1. 머신에서 CLI가 올바르게 작동하는지 확인합니다: `uv run specify --help`
1. 새 브랜치를 만듭니다: `git checkout -b my-branch-name`
1. 변경 사항을 적용하고, 테스트를 추가하고, 모든 것이 여전히 올바르게 작동하는지 확인합니다
1. 관련이 있는 경우 샘플 프로젝트로 CLI 기능을 테스트합니다
1. fork에 push하고 pull request를 제출합니다
1. Pull request가 검토되고 병합될 때까지 기다립니다.

자세한 테스트 워크플로, 명령 선택 프롬프트, PR 보고 템플릿은 [`TESTING.md`](./TESTING.md)를 참고하세요.
프로젝트 가상 환경을 활성화([ `TESTING.md`](./TESTING.md)의 Setup 블록 참고)한 후, 작업 트리에서 CLI를 설치하거나(`uv sync --extra test` 이후 `uv pip install -e .`) 아래에 설명된 수동 slash-command 테스트를 실행하기 전에 셸이 로컬 `specify` 바이너리를 사용하도록 설정하세요.

Pull request가 승인될 가능성을 높이기 위해 할 수 있는 몇 가지 사항이 있습니다:

- 프로젝트의 코딩 규칙을 따르세요.
- 새로운 기능에 대한 테스트를 작성하세요.
- 변경 사항이 사용자 대면 기능에 영향을 미치는 경우 문서(`README.md`, `spec-driven.md`)를 업데이트하세요.
- 변경 사항을 최대한 집중적으로 유지하세요. 서로 의존하지 않는 여러 변경 사항이 있다면 별도의 pull request로 제출하는 것을 고려하세요.
- [좋은 커밋 메시지](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)를 작성하세요.
- Spec-Driven Development 워크플로를 통해 변경 사항을 테스트하여 호환성을 확인하세요.

## 개발 워크플로

spec-kit 작업 시:

1. 선호하는 코딩 에이전트에서 `specify` CLI 명령(`/speckit.specify`, `/speckit.plan`, `/speckit.tasks`)으로 변경 사항을 테스트합니다
2. `templates/` 디렉터리에서 템플릿이 올바르게 작동하는지 확인합니다
3. `scripts/` 디렉터리에서 스크립트 기능을 테스트합니다
4. 주요 프로세스 변경 사항이 있는 경우 메모리 파일(`memory/constitution.md`)이 업데이트되었는지 확인합니다

### 권장 검증 순서

가장 원활한 리뷰 경험을 위해 다음 순서로 변경 사항을 검증하세요:

1. **먼저 집중적인 자동화 검사를 실행하세요** — [`TESTING.md`](./TESTING.md)의 빠른 검증 명령을 사용하여 패키징, 스캐폴딩, 구성 회귀를 조기에 발견하세요.
2. **두 번째로 수동 워크플로 테스트를 실행하세요** — 변경 사항이 slash command나 개발자 워크플로에 영향을 미치는 경우, [`TESTING.md`](./TESTING.md)를 따라 적절한 명령을 선택하고 에이전트에서 실행한 후 PR을 위한 결과를 캡처하세요.
3. **패키지 출력 디버깅 시 로컬 릴리스 패키지를 사용하세요** — CI 방식의 패키징이 생성하는 정확한 파일을 검사해야 할 경우, 아래에 설명된 방법으로 로컬 릴리스 패키지를 생성하세요.

### 로컬에서 템플릿 및 명령 변경 사항 테스트하기

`uv run specify init`을 실행하면 릴리스된 패키지를 가져오므로 로컬 변경 사항이 포함되지 않습니다.  
템플릿, 명령 및 기타 변경 사항을 로컬에서 테스트하려면 다음 단계를 따르세요:

1. **릴리스 패키지 생성**

   다음 명령을 실행하여 로컬 패키지를 생성합니다:

   ```bash
   ./.github/workflows/scripts/create-release-packages.sh v1.0.0
   ```

2. **관련 패키지를 테스트 프로젝트에 복사**

   ```bash
   cp -r .genreleases/sdd-copilot-package-sh/. <path-to-test-project>/
   ```

3. **에이전트 열기 및 테스트**

   테스트 프로젝트 폴더로 이동한 후 에이전트를 열어 구현이 올바른지 확인합니다.

수동 에이전트 테스트 전에 생성된 파일 구조와 내용만 검증하면 된다면, [`TESTING.md`](./TESTING.md)의 집중 자동화 검사부터 시작하세요. 로컬에서 정확한 패키지 출력을 검사해야 하는 경우에만 이 섹션을 참고하세요.

## Spec Kit에서의 AI 기여

> [!IMPORTANT]
>
> Spec Kit에 기여할 때 **어떤 형태의 AI 지원을 사용하더라도**
> pull request 또는 이슈에 반드시 공개해야 합니다.

Spec Kit을 개선하는 데 도움이 되는 AI 도구의 사용을 환영하고 장려합니다! 많은 가치 있는 기여들이 코드 생성, 문제 감지, 기능 정의를 위한 AI 지원을 통해 향상되었습니다.

그렇긴 하지만, Spec Kit에 기여하는 동안 어떤 형태의 AI 지원(예: 에이전트, ChatGPT)을 사용하고 있다면,
**이를 pull request 또는 이슈에 반드시 공개해야 하며**, AI 지원이 사용된 범위(예: 문서 주석 대 코드 생성)도 함께 명시해야 합니다.

PR 응답이나 댓글이 AI에 의해 생성되고 있다면, 그것도 공개하세요.

예외적으로, 사소한 공백이나 오타 수정은 변경 사항이 코드의 일부 또는 짧은 문구로 제한되는 한 공개할 필요가 없습니다.

공개 예시:

> This PR was written primarily by GitHub Copilot.

또는 더 상세한 공개:

> I consulted ChatGPT to understand the codebase but the solution
> was fully authored manually by myself.

이를 공개하지 않는 것은 무엇보다 pull request의 다른 쪽에 있는 사람에게 무례한 일이며, 기여에 얼마나 많은 검토를 적용해야 하는지 판단하기 어렵게 만듭니다.

이상적인 세상에서라면 AI 지원은 어떤 사람보다 동등하거나 더 높은 품질의 작업물을 만들어낼 것입니다. 하지만 우리가 사는 세상은 그렇지 않으며, 인간의 감독이나 전문성이 개입되지 않는 대부분의 경우, 합리적으로 유지하거나 발전시키기 어려운 코드가 생성됩니다.

### 우리가 기대하는 것

AI 지원 기여를 제출할 때 다음 사항이 포함되어 있는지 확인하세요:

- **AI 사용에 대한 명확한 공개** - AI 사용 여부와 기여에 AI를 어느 정도 활용했는지 투명하게 밝히세요
- **인간의 이해와 테스트** - 변경 사항을 직접 테스트하고 그 내용을 이해하고 있어야 합니다
- **명확한 근거** - 변경이 필요한 이유와 Spec Kit의 목표에 어떻게 부합하는지 설명할 수 있어야 합니다
- **구체적인 근거** - 개선 사항을 보여주는 테스트 케이스, 시나리오 또는 예시를 포함하세요
- **본인의 분석** - 전체 개발자 경험에 대한 본인의 생각을 공유하세요

### 닫힐 수 있는 기여

다음과 같이 보이는 기여는 닫을 권리를 유보합니다:

- 검증 없이 제출된 테스트되지 않은 변경 사항
- Spec Kit의 특정 요구 사항을 다루지 않는 일반적인 제안
- 인간의 검토나 이해가 없는 일괄 제출

### 성공을 위한 지침

핵심은 제안한 변경 사항을 이해하고 검증했음을 입증하는 것입니다. 유지 관리자가 기여가 인간의 입력이나 테스트 없이 AI에 의해 완전히 생성되었음을 쉽게 알아볼 수 있다면, 제출 전에 더 많은 작업이 필요할 가능성이 높습니다.

일관되게 낮은 노력의 AI 생성 변경 사항을 제출하는 기여자는 유지 관리자의 재량에 따라 추가 기여가 제한될 수 있습니다.

유지 관리자를 존중하고 AI 지원을 공개해 주세요.

## 리소스

- [Spec-Driven Development 방법론](./spec-driven.md)
- [오픈 소스에 기여하는 방법](https://opensource.guide/how-to-contribute/)
- [Pull Request 사용하기](https://help.github.com/articles/about-pull-requests/)
- [GitHub 도움말](https://help.github.com)
