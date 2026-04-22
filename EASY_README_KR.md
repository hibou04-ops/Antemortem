# Antemortem — 쉬운 설명

> 본 README가 어렵게 느껴지는 분들을 위한 압축 버전.
> 원본: [README_KR.md](README_KR.md) · English easy: [EASY_README.md](EASY_README.md)

## 이게 뭔가요?

**Postmortem은 잔해를 설명합니다. Antemortem은 그 잔해를 예방합니다.**

이 repo는 **methodology**입니다 — 패키지가 아님. 7단계 프로토콜 + markdown 템플릿 2개로, 코드를 한 줄 쓰기 전에 LLM과 함께 계획된 변경을 *종이 위에서* 스트레스-테스트하게 만듭니다. 코드 리뷰나 테스트가 못 잡는 부류의 버그를 잡습니다 — PR이 존재하기 *전*에 이미 박혀있는 버그들.

## 이게 고치는 문제

모든 non-trivial 변경은 같은 식으로 시작합니다:
1. 스펙을 씁니다.
2. 리스크를 대충 적습니다.
3. 코딩 시작.
4. 반나절 지나서 발견: **"리스크" 두 개는 상상이었고**, **생각도 못 했던 리스크 하나가 load-bearing**.

Antemortem은 2.5단계 — 비용 비싼 단계 *전에* 발견을 앞당기는 빠진 단계.

## 7단계, 한 화면

1. **스펙 작성** (한 문단).
2. **자기 트랩 리스트업** (관대하게. trap / worry / unknown 라벨).
3. **LLM에 제공** — 스펙 + 변경이 건드릴 파일들.
4. **LLM이 각 트랩 분류** — `REAL` (코드가 확인) / `GHOST` (코드가 반박) / `NEW` (모델이 새로 발견), 각각 `file:line` 인용과 함께.
5. **P(성공) 재추정**.
6. **스펙 수정** — 코딩 시작 전에.
7. **문서 저장** — 미래의 당신이 뭔가 놀랐을 때 다시 읽게.

4단계가 discipline 전부. 인용 없으면 증거 아님. 그게 룰.

> Methodology 자체는 3개 라벨만 formal 정의 (`REAL` / `GHOST` / `NEW`). 자매 도구인 [antemortem-cli](https://github.com/hibou04-ops/antemortem-cli) 는 *"제공된 파일에 증거가 전혀 없음"* 을 위한 4번째 라벨 `UNRESOLVED` 를 추가 — 수동 대신 CLI로 프로토콜 돌릴 때의 유용한 "정직한 무지" 라벨.

## 정직하게 유지하는 가드레일 2개

- **LLM이 코드 보기 *전에* 당신이 트랩을 나열한다.** 안 그러면 모델이 당신 리스크를 frame해서 (anchoring) 모든 것에 동의해버림.
- **모든 분류는 `file:line` 인용 필수.** 안 그러면 LLM이 원하는 답에 증거를 지어냄.

둘 다 없으면: hand-waving 한 종류를 다른 종류로 바꾼 것 뿐. 둘 다 있으면: 15분 짜리 기계적 screening 스텝.

## 이 repo 사용법

Install 없음. 문서입니다.

```
templates/antemortem-template.md            ← 복사해서 채우기
templates/antemortem-template-enhanced.md   ← 고위험 변경용 (prod / 데이터 이주 / 보안)
examples/omega-lock-audit.md                ← 첫 실전 케이스 스터디
docs/methodology.md                         ← 전체 프로토콜
```

문서를 자동 scaffold + 검증하는 도구는? **[antemortem-cli](https://github.com/hibou04-ops/antemortem-cli)** (`pip install antemortem`). 이 repo는 discipline, CLI는 pit crew.

## 실제로 작동하나요?

네 — [`examples/omega-lock-audit.md`](examples/omega-lock-audit.md) 참조. 실제 15분 antemortem에서:

- ghost 트랩 1개 발견 (엔지니어-데이 ~0.5 절약)
- 리스크 3개 downgrade (30–40% → 10–15%)
- 새 스펙 요구사항 1개 수면 위로
- Post-recon P(on-time ship): 55–65% → 70–78%
- 구현은 하루, 신규 테스트 20개 first-run pass

## 쓰면 안 되는 경우

- 사소한 변경 (오타 수정, 한 줄 config).
- 읽을 코드 자체가 없는 greenfield (인용할 게 없음).
- 시간이 없음. Antemortem은 싸지만 공짜 아님. 15분은 15분.

## 한계

Antemortem은 **reading discipline**. 못 잡는 것들:
- Runtime-only 이슈 (race condition, GC, 네트워크 flakiness).
- 플랫폼 quirk (cp949 터미널 인코딩, 파일시스템 case sensitivity).
- 제품 차원 잘못된 방향 ("애초에 이걸 왜 만드나?").
- 코드가 증명도 반박도 못 하는 empirical claim.

전략적 리스크는 pre-mortem. 런타임 행동은 테스트. Antemortem은 *"이미 있는 코드가 내 계획과 정말 일치하나?"*.

## 다음 단계

- 전체 methodology: [docs/methodology.md](docs/methodology.md)
- 3-layer stack (methodology → CLI → first case): [README_KR.md § 3-layer stack](README_KR.md#3-레이어-스택)
- 케이스 스터디: [examples/omega-lock-audit.md](examples/omega-lock-audit.md)

License: Apache 2.0. Copyright (c) 2026 hibou.
