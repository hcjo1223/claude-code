# 참고 소스 https://ccunpacked.dev/


-----

# Claude Code 전체 소스 코드, npm 소스맵을 통해 유출되다

> **PS:** 이 분석 내용은 [이 블로그](https://kuber.studio/blog/AI/Claude-Code's-Entire-Source-Code-Got-Leaked-via-a-Sourcemap-in-npm,-Let's-Talk-About-it)에서 더 나은 UX와 읽기 환경으로 확인하실 수 있습니다.

> **참고:** 이 저장소는 삭제될 가능성이 있습니다. 나중에 살펴보거나 직접 보관하고 싶다면 **Fork**를 하거나 외부 블로그 링크를 북마크해 두세요\!

-----

## ⚠️ 중요 고지 (Disclaimer)

**본인은 이 파일을 유출하지 않았습니다.** 저는 단순히 연구 목적으로 이 코드베이스에 접근하고 학습할 수 있도록 문서화된 방법을 제공할 뿐입니다. 모든 파일과 정보는 Twitter/X에 공유된 공개된 발견 사항에서 비롯되었습니다. 발견에 대한 모든 공로는 최초 제보자에게 있습니다.

-----

오늘 일찍 (2026년 3월 31일), \*\*Chaofan Shou (@Fried\_rice)\*\*는 Anthropic이 세상에 보여주고 싶지 않았을 법한 사실을 발견했습니다. Anthropic의 공식 AI 코딩 CLI인 **Claude Code의 전체 소스 코드**가 npm 레지스트리에 배포된 패키지 내 소스맵(sourcemap) 파일을 통해 그대로 노출되어 있었습니다.

[트위터 유출 발표 포스트 확인하기](https://x.com/Fried_rice/status/2038894956459290963)

이 저장소는 유출된 소스의 백업본이며, 포함된 내용, 유출 경위, 그리고 공개될 예정이 없었던 내부 시스템에 대한 전체 분석을 제공합니다.

-----

## 🧐 어떻게 이런 일이 발생했나요?

JavaScript/TypeScript 패키지를 npm에 배포할 때, 빌드 도구는 종종 **소스맵 파일**(`.map`)을 생성합니다. 이 파일은 디버깅을 위해 압축된 프로덕션 코드와 원본 소스 코드를 연결해 주는 역할을 합니다.

문제는 **소스맵 파일의 `sourcesContent` 키 안에 원본 소스 코드 전체가 문자열로 포함된다는 점**입니다.

```json
{
  "version": 3,
  "sources": ["../src/main.tsx", "../src/tools/BashTool.ts", "..."],
  "sourcesContent": ["// 각 파일의 '전체' 원본 소스 코드 내용", "..."],
  "mappings": "AAAA,SAAS,OAAO..."
}
```

`.npmignore`에 `*.map`을 추가하는 것을 잊었거나, 프로덕션 빌드에서 소스맵 생성을 비활성화하지 않아(Bun의 기본 동작) 원시 소스 코드 전체가 npm 레지스트리에 업로드된 것입니다.

-----

## 🛠 내부 구조는 어떻게 되어 있나요?

Claude Code는 단순한 CLI가 아닙니다. **785KB에 달하는 `main.tsx`** 진입점을 중심으로 커스텀 React 터미널 렌더러(Ink), 40개 이상의 도구, 복잡한 멀티 에이전트 오케스트레이션(Orchestration)이 포함된 거대한 시스템입니다.

### 🐣 BUDDY - 터미널 다마고치

[`src/buddy/`](https://www.google.com/search?q=./src/buddy/) 내부에는 풀 버전의 **다마고치 스타일 동반자 시스템**이 있습니다.

  - **결정론적 가차(Gacha):** 사용자의 `userId`를 시드로 사용하는 Mulberry32 PRNG를 사용합니다.
  - **18종의 생명체:** 일반 등급(*Pebblecrab*)부터 전설 등급(*Nebulynx*)까지 존재합니다.
  - **스탯 및 영혼:** 모든 버디는 `DEBUGGING`, `CHAOS`, `SNARK`와 같은 스탯을 가지며, Claude가 작성한 "영혼(soul)" 설명을 보유합니다.

### 🕵️‍♂️ 잠입 모드(Undercover Mode) - "정체를 들키지 마세요"

Anthropic 직원들은 공용 레포지토리에 기여할 때 Claude Code를 사용합니다. **잠입 모드**([`src/utils/undercover.ts`](https://www.google.com/search?q=./src/utils/undercover.ts))는 AI가 내부 정보를 유출하는 것을 방지합니다.

  - 내부 모델 코드명(예: *Capybara*, *Tengu*) 노출 차단.
  - 사용자가 AI라는 사실을 숨김.
  - \*\*"Tengu"\*\*가 Claude Code의 내부 코드명임을 확인해 줌.

### 🌙 "드림(Dream)" 시스템

Claude Code는 기억을 통합하기 위해 "꿈"을 꿉니다. **autoDream** 서비스([`src/services/autoDream/`](https://www.google.com/search?q=./src/services/autoDream/))는 백그라운드 서브 에이전트로 실행되어 다음을 수행합니다.

1.  **오리엔트(Orient):** `MEMORY.md`를 읽습니다.
2.  **수집(Gather):** 일일 로그에서 새로운 신호를 찾습니다.
3.  **통합(Consolidate):** 영구 메모리 파일을 업데이트합니다.
4.  **정리(Prune):** 컨텍스트를 효율적으로 유지합니다.

### 🚀 KAIROS & ULTRAPLAN

  - **KAIROS:** 로그를 감시하고 입력 대기 없이 행동하는 "상시 가동" 능동 보조 시스템입니다.
  - **ULTRAPLAN:** 복잡한 작업을 원격 **Opus 4.6** 세션에 오프로드하여 최대 30분 동안 심층 계획을 수립합니다.

-----

## 📂 아키텍처 및 디렉토리 구조

```text
src/
├── main.tsx                 # CLI 진입점 (Commander.js + React/Ink)
├── QueryEngine.ts           # 핵심 LLM 로직
├── Tool.ts                  # 기본 도구 정의
├── tools/                   # 40개 이상의 에이전트 도구 (Bash, Files, LSP, Web)
├── services/                # 백엔드 (MCP, OAuth, 분석, Dreams)
├── coordinator/             # 멀티 에이전트 오케스트레이션 (Swarm)
├── bridge/                  # IDE 통합 레이어
└── buddy/                   # 비밀 다마고치 시스템
```

-----

## ⚙️ 사용 및 탐색 방법

### 📦 사전 요구 사항

  - **[Bun Runtime](https://bun.sh)** (강력 권장) 또는 Node.js v18+
  - 전역 설치된 **TypeScript**

### 🚀 시작하기

1.  **저장소 클론:**

    ```bash
    git clone https://github.com/your-username/claude-leaked.git
    cd claude-leaked
    ```

2.  **의존성 설치:**

    ```bash
    npm install
    ```

3.  **프로젝트 빌드:**

    ```bash
    npm run build
    ```

4.  **CLI 실행:**

    ```bash
    node dist/main.js
    ```

### 🔍 MCP로 탐색하기

이 레포지토리에는 Claude를 사용하여 직접 소스를 탐색할 수 있는 **MCP 서버**가 포함되어 있습니다.

```bash
claude mcp add code-explorer -- npx -y claude-code-explorer-mcp
```

-----

## 📜 크레딧 및 법적 고지

  - **발견:** [Chaofan Shou (@Fried\_rice)](https://x.com/Fried_rice)
  - **원본 포스트:** [Twitter/X 발표 내용](https://x.com/Fried_rice/status/2038894956459290963)
  - **미러 저장소 작성자:** [Yasas Banu](https://www.yasasbanuka.tech)

**면책 조항:** 모든 원본 소스 코드는 **Anthropic PBC**의 독점 자산입니다. 이 저장소는 교육 및 보관 목적으로만 제공됩니다. **본 프로젝트는 Anthropic의 공식 제품이 아닙니다.**
