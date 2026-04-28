# Antemortem (한국어)

> **처음이신가요?** 먼저 보세요: [EASY_README_KR.md](EASY_README_KR.md) (한국어) · [EASY_README.md](EASY_README.md) (English). 아래 본 문서가 어렵게 느껴지는 분들을 위한 압축된 쉬운 소개.

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache--2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Version](https://img.shields.io/badge/version-0.1.1-blue)](CHANGELOG.md)
[![Status](https://img.shields.io/badge/status-stable-brightgreen)](#상태)
[![Tooling](https://img.shields.io/badge/tooling-antemortem--cli-blueviolet)](https://github.com/hibou04-ops/antemortem-cli)

> **소프트웨어 변경을 위한 AI-assisted 사전 정찰 (Pre-implementation reconnaissance).**
> Postmortem 은 뭔가 깨지고 나서 쓰는 것. Antemortem 은 빌드하기 *전에* 하는 것 — 그리고 그것을 정직하게 유지하는 discipline.

Postmortem 은 잔해를 설명합니다. Antemortem 은 "리스크" 중 하나는 애초에 없었고 상상도 못 했던 게 load-bearing 이었다는 걸 발견하느라 태우는 반나절의 구현을 방지합니다. 계획된 변경을 *종이에서* 스트레스 테스트하고, 능력 있는 LLM 에 spec 과 관련 파일을 넘기고, 모든 가설 리스크를 primary-source `file:line` 인용과 함께 `REAL` / `GHOST` / `NEW` / `UNRESOLVED` 로 classify 하고, 한 줄도 쓰기 *전에* spec 을 수정합니다.

이 repo 는 methodology — 7-step 프로토콜, 두 개의 템플릿, 첫 case study. 컴패니언 도구 [`antemortem-cli`](https://github.com/hibou04-ops/antemortem-cli) 가 스캐폴딩, classification pass, schema lint (Pydantic enforced, citations 가 disk 에서 재검증) 를 automate 합니다. 함께 3-layer stack 형성: methodology 는 여기, tooling 은 저기, 첫 shipped artifact 는 [`omega-lock`](https://github.com/hibou04-ops/omega-lock) 에.

English README: [README.md](README.md)

## 목차

- [이 discipline 이 왜 존재하나](#이-discipline-이-왜-존재하나)
- [7 단계](#7-단계)
- [작동하나?](#작동하나)
- [인접한 실천법들과의 차이](#인접한-실천법들과의-차이)
- [3-layer stack](#3-layer-stack)
- [한계](#한계)
- [이 repo 사용법](#이-repo-사용법)
- [상태](#상태)
- [FAQ](#faq)
- [기여](#기여)
- [이 작업 인용](#이-작업-인용)
- [라이선스](#라이선스)
- [Colophon](#colophon)

## 이 discipline 이 왜 존재하나

LLM 능력이 옛날에는 비쌌던 두 가지를 싸게 만들었습니다:

1. **Codebase 읽기.** 능력 있는 reasoning 모델이 단일 자릿수 분 안에 열 몇 개 파일을 스캔하고, call pattern 을 따라가고, 누가 뭘 호출하는지 정확히 식별할 수 있습니다.
2. **실패 모드 enumerate.** 모델은 사람 혼자보다 빠르고 철저하게 plausible trap 을 나열할 수 있습니다.

비싼 채로 남은 것: **정확한 코드 쓰기.** AI 가 이해와 리스크 enumeration 을 싸게 front-load 할 수 있다면, 합리적 수순은 그것을 비싼 단계 *전에* 하는 것이지 *도중에* 하는 것이 아닙니다. Antemortem 은 이 능력을 반복 가능한 프로토콜로 바꾸는 discipline 입니다 — LLM 에게 plan 을 리뷰해달라고 하는 실패 모드 (anchoring, vibes-based agreement, 할루시네이트된 증거) 에 대한 가드레일과 함께.

두 가드레일이 discipline 을 정직하게 유지합니다:

1. **모델이 어떤 코드를 보기 전에 본인이 trap 을 enumerate.** 모델 framing 에 anchoring 방지.
2. **모든 classification 이 `file:line` 인용.** *"코드가 X 를 보여준다"* 는 증거가 아닙니다; *"`walk_forward.py` 의 82번째 줄이 `evaluate()` 를 params 당 한 번 호출, 주변 loop 없음"* 이 증거입니다. 이 없이는 다른 형태의 hand-waving 으로 교환한 것뿐.

## 7 단계

1. **변경 진술.** 한 문단. 추가/제거/리팩토링할 것.
2. **Trap enumerate.** 모든 가설 리스크 나열. 관대하게. 각각 `trap` / `worry` / `unknown` 라벨.
3. **Plan + repo 를 LLM 에 넘김.** 변경이 touch 할 파일을 읽어달라고 요청.
4. **Primary-source 인용으로 findings 를 classify.** 각 trap 에 대해: `REAL` (코드가 확인) / `GHOST` (코드가 반박) / `NEW` (모델이 surface) / `UNRESOLVED` (어느 쪽 증거도 없음). UNRESOLVED 가 아닌 모든 라벨에 `file:line` 인용.
5. **확률 수정.** 새 trap count 로 `P(success)` 업데이트.
6. **Spec 업데이트.** load-bearing 으로 느껴지기 전, 지금 spec 을 변경.
7. **문서 저장.** 가치는 recon 자체만이 아닙니다 — 구현이 나중에 당신을 놀라게 했을 때, 미래의 당신이 다시 읽고 어떤 가정이 깨졌는지 볼 수 있다는 것.

4단계는 건너뛰지 마세요. Citation 이 discipline 입니다.

Rubric 과 실패 모드를 포함한 전체 methodology 는 [`docs/methodology.md`](docs/methodology.md) 에. 복사 준비된 템플릿은 [`templates/antemortem-template.md`](templates/antemortem-template.md) 에.

고-stakes 변경 (prod 배포, 데이터 마이그레이션, 보안 경계) 을 위해선 [enhanced 템플릿](templates/antemortem-template-enhanced.md) — calibration 차원 (evidence strength, blast radius, reversibility), 세분화된 classification subtype, 명시적 skeptic pass, decision-first output 구조 추가. Rationale 은 [`docs/methodology.md § Enhanced protocol`](docs/methodology.md#enhanced-protocol-optional).

## 작동하나?

첫 case study 는 [`examples/omega-lock-audit.md`](examples/omega-lock-audit.md). Python calibration framework 에 새 audit 서브모듈을 추가하기 전에 ~15분 antemortem 실행 내용.

그 antemortem 이 잡은 것의 짧은 버전:

- **Ghost trap 하나.** `WalkForward` 가 내부적으로 fold 한다고 확신했음. 코드는 그렇지 않음을 보여줌 — 각 호출은 하나의 clean 한 평가. Primary-source 증거, 7 개 계획된 리스크 중 하나 제거. ≈0.5 engineer-day 절약.
- **리스크 3개 하향.** JSON serialization, memory blow-up, iterative-round bookkeeping 모두 codebase 에 기존 완화 존재. 각각 30–40% 에서 10–15% 로.
- **새 요구사항 하나.** 모델이 target-모양 객체가 동시에 세 역할로 존재함을 눈치챘음. 구현 전 spec 에 `target_role` 필드 추가.

Recon 후 `P(full spec ships on time)` 이 55–65% 에서 70–78% 로. 구현 1 engineer-day 소요; 신규 20 tests 가 first run 에 통과.

같은 문서는 recon 이 *놓친* 것에 대한 정직한 post-implementation 노트 포함 (Windows cp949 em-dash 터미널 인코딩 이슈가 런타임에 surface). Antemortem 은 플랫폼 인코딩 이슈를 못 잡음 — [한계](#한계) 참조.

## 인접한 실천법들과의 차이

| 실천법 | 타이밍 | 범위 | 전형적 소요 | 산출물 |
|---|---|---|---|---|
| Code review | 변경이 쓰이고 난 후 | Diff-local | PR 당 수 분 | 승인 / 변경 요청 |
| **Antemortem** | 변경이 쓰이기 **전** | Change-local | 15–30분, 솔로 + LLM | `file:line` 인용 붙은 리스크 classified 문서 |
| Pre-mortem (Klein, 2007) | 프로젝트 commit 전 | Project-level | 30–60분, 팀 연습 | 랭크된 실패 시나리오 |
| Postmortem | 뭔가 깨진 후 | Incident-local | 수시간, 팀 | Root cause + action item |

Code review 는 PR 에 있는 것을 잡습니다. Antemortem 은 *아직 PR 에 없는* 것을 잡습니다 — 작성자가 아직 쓰지 않았기 때문. Pre-mortem 은 전략적 (*해야 하는가?*); antemortem 은 전술적 (*할 것이니, 코드가 이 방식의 리스크에 대해 이미 무엇을 말하는가?*). 더 긴 구분은 [`docs/methodology.md § Antemortem vs pre-mortem`](docs/methodology.md#antemortem-vs-pre-mortem) 참조.

## 3-layer stack

Antemortem 은 고립돼 존재하지 않습니다. Layered discipline 의 중간 tier 입니다:

```
 Layer 3 │  antemortem-cli  (tooling)      스캐폴드, classify, lint
         │  v0.2.0 on PyPI                 https://github.com/hibou04-ops/antemortem-cli
         │      operationalizes
 Layer 2 │  Antemortem  (이 repo)          Methodology, 템플릿, case studies
         │  v0.1.1                         https://github.com/hibou04-ops/Antemortem
         │      demonstrated by
 Layer 1 │  omega-lock  (첫 case)          Python calibration audit framework
         │  v0.1.4 on PyPI                 https://github.com/hibou04-ops/omega-lock
```

- **Layer 1 — [omega-lock](https://github.com/hibou04-ops/omega-lock)** 은 Antemortem 이 *처음 실제로 practice 된* Python calibration framework. `omega_lock.audit` 서브모듈이 위에 기록된 ghost trap 을 잡은 15분짜리 antemortem recon 을 사용해 빌드됐습니다.
- **Layer 2 — Antemortem** (이 repo) 은 그 빌드에서 추출된 methodology: 7-step 프로토콜, basic 과 enhanced 템플릿, 첫 case study. Docs-only by design — 뭔가 실행하지 않아도 discipline 이 legible 해야 하므로.
- **Layer 3 — [antemortem-cli](https://github.com/hibou04-ops/antemortem-cli)** 는 마찰을 제거하는 도구: `antemortem init`, `antemortem run`, `antemortem lint`. Pydantic-enforced 스키마, disk-verified citation, CI-ready lint gate.

Layering 은 정확성에 중요합니다: methodology 는 CLI 빌드 *전에* 실제 shipped artifact 로 검증됐으므로, 도구는 이미 작동하는 걸로 알려진 프로토콜을 automate — 도구와 나란히 발명된 프로토콜이 아님.

## 한계

Antemortem 은 **reading discipline** 입니다. 잡는 것:

- Disproof 가 코드 자체에 있는 ghost trap.
- Recon 이 surface 하는 missing 또는 잘못된 spec 필드.
- 잊고 있던, 소스에서 볼 수 있는 숨은 완화책.
- 당신의 mental model 과 다른 call pattern.

잡지 **못하는** 것:

- 런타임-전용 이슈 (race condition, GC timing, 네트워크 동작).
- 모델이 파일-수준 증거를 못 가진 플랫폼-특정 이슈 (텍스트 인코딩, shell quirk, OS-특정 path handling).
- 제품-수준 잘못된 방향 — spec 이 내부적으로 일관되면서도 여전히 틀린 것을 빌드할 수 있음.
- 경험적 측정을 요하는 주장 (성능, 수치적 안정성, 실제 데이터 정확도).
- 이미 걱정하는 리스크에 대해서만 모델에 물으면 — 본인의 blind spot.

Antemortem 이 아는 세계는 파일 안의 세계입니다. 그 밖의 것에 대해선, 방치하면 misplaced confidence 로 말합니다. 전체 논의 [`docs/methodology.md § Limits`](docs/methodology.md#limits).

## 이 repo 사용법

이 repo 는 **docs 와 templates 만** ship 합니다. Clone, read, 본인 프로젝트에 템플릿 복사, 선택한 LLM 으로 프로토콜 실행.

프로그램 레벨 automation — 스캐폴딩, prompt caching 붙은 LLM invocation, Pydantic-enforced classification, disk-verified citation lint — 을 위해선 컴패니언 CLI 설치:

```bash
pip install antemortem
```

전체 tooling 은 [`antemortem-cli`](https://github.com/hibou04-ops/antemortem-cli).

## 상태

- **v0.1.1 (현재)** — methodology + basic 템플릿 + enhanced 템플릿 + 첫 case study (omega-lock). Stable.
- **v0.2 — [`antemortem-cli`](https://github.com/hibou04-ops/antemortem-cli) 로 shipped.** CLI + PyPI + Pydantic 스키마 + disk-verified citation lint. 2026-04-22 릴리스.
- **v0.3 (계획)** — 팀이 PR 간 diff 할 수 있도록 antemortem 문서용 structured schema. 실험 트랙.

전체 changelog: [`CHANGELOG.md`](CHANGELOG.md).

## FAQ

**이건 그냥 pre-mortem 을 새 이름으로?**
아닙니다. Gary Klein 의 pre-mortem (*HBR*, 2007) 은 팀-수준 전략적 연습. Antemortem 은 change-수준, 솔로, 소스-코드-기반, 15–30 분에 discharged. [`docs/methodology.md § Antemortem vs pre-mortem`](docs/methodology.md#antemortem-vs-pre-mortem) 참조.

**어떤 LLM 이 필요한가요?**
Multi-file call chain 을 따라갈 수 있는 reasoning 모델이면 됨. Discipline 은 model-agnostic; 엔진은 교체 가능. 이 repo 의 첫 case study 와 `antemortem-cli` tooling 둘 다 재현성을 위해 특정 Claude 모델을 pin 하지만, 프로토콜 자체는 충분히 능력 있는 reasoner 에 adapt 가능.

**"LLM 에게 내 plan 리뷰해줘" 와 어떻게 다른가?**
두 제약. 2단계는 LLM 이 보기 전 본인 리스크를 enumerate 강제 (framing anchoring 방지). 4단계는 `file:line` 인용 강제 (vibes 수용 방지). 없으면 다른 형태의 hand-waving.

**1–2일짜리 변경에 15분이 현실적?**
모델이 parallel-read 할 수 있는 4–7개 파일 touch 하는 변경이면 yes. 더 큰 변경은 여러 antemortem 으로 분할. Recon 이 1시간 초과하면 변경 자체가 너무 큼 — 먼저 분할.

**LLM 의 classification 이 틀리면?**
Citation 이 check. *"코드가 X 를 보여준다"* 는 증거 아님; *"`walk_forward.py` 82번째 줄이 `evaluate()` 를 params 당 한 번 호출, loop 없음"* 이 증거. 각 citation 을 실제 파일에 대조 검증. Citation 이 틀리면 classification 도 틀림. `antemortem-cli lint` 가 이 검증을 automate.

**비공개/private 코드에 antemortem 돌릴 수 있나?**
가능. LLM 은 당신이 주는 걸 읽음; public repo 필요 없음. 유일 제약은 공개 case study 의 citation 은 repo 없이도 readers 가 verify 할 수 있도록 inline context 를 충분히 quote 해야 함.

## 기여

[`CONTRIBUTING.md`](CONTRIBUTING.md) 참조. 짧게: case study 는 `examples/` 아래 PR, methodology 수정은 `docs/methodology.md` 대상, primary-source citation 이 bar.

Release note 는 [`CHANGELOG.md`](CHANGELOG.md). 커뮤니티 기대치는 [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md).

## 이 작업 인용

논문, 블로그 포스트, 사내 문서에서 이 methodology 를 참조하신다면:

```
Antemortem v0.1.1 — AI-assisted pre-implementation reconnaissance for software changes.
https://github.com/hibou04-ops/Antemortem, 2026.
```

첫 case study 는 별도 인용 가능:

```
Antemortem case study: omega_lock.audit (2026-04-18).
https://github.com/hibou04-ops/Antemortem/blob/main/examples/omega-lock-audit.md
```

컴패니언 tooling:

```
antemortem-cli v0.2.0 — tooling for the Antemortem pre-implementation reconnaissance discipline.
https://github.com/hibou04-ops/antemortem-cli, 2026.
```

## 라이선스

Apache License 2.0. [LICENSE](LICENSE) 및 [NOTICE](NOTICE) 참조.

Copyright (c) 2026 hibou.

**라이선스 히스토리.** `v0.1` 과 `v0.1.1` 태그는 MIT License 로 릴리스되었습니다. 2026-04-22 (commit `8bf43b4`) Apache 2.0 으로 재라이선싱되었고, 이후 모든 commit 및 태그는 Apache 2.0 입니다. `v0.1` / `v0.1.1` 시점에 clone 또는 vendoring 한 사용자는 그 snapshot 에 대해 MIT 라이선스를 보유합니다 — 라이선스 변경은 소급 적용되지 않습니다.

## Colophon

Antemortem 은 2026-04-18 `omega_lock.audit` 개발 중 named practice 로 결정화됨. 첫 case study 는 `omega_lock.audit` 서브모듈이 빌드되기 전 [`omega-lock`](https://github.com/hibou04-ops/omega-lock) codebase 에서 ghost trap 을 잡은 recon. Recon 자체는 Claude Code 안에서 Claude 로 수행; 이 repo 는 그것의 synthesis.
