# agent-from-scratch

AI 코딩 에이전트를 TypeScript로 바닥부터 직접 만들어 보는 학습 저장소. API 키로 모델을 직접 부르고, 그 위에 하네스(루프·도구·
승인 게이트·컨텍스트 관리)를 한 단계씩 손으로 쌓는다.

| 단계 | 만드는 것                              | 완료 기준                               |
| ---- | -------------------------------------- | --------------------------------------- |
| 0    | 저장소 + 앱 뼈대                       | 전체 검사가 통과한다                    |
| 1    | 모델 한 번 호출                        | 질문을 치면 답이 스트리밍으로 나온다    |
| 2    | 대화 배열 누적                         | 앞 질문을 기억한다                      |
| 3    | 파일 읽기 도구 + 루프 + 요청·응답 로그 | 파일을 읽고 그 내용으로 답한다          |
| 4    | 가짜 모델로 루프 테스트                | API 호출 없이 테스트가 통과한다         |
| 5    | 승인 게이트 → 셸·파일 수정 도구        | 장난감 저장소의 버그를 스스로 고친다    |
| 6    | 긴 출력 자르기 · 요약 · 캐싱           | 긴 작업에서도 안 터지고 비용이 줄어든다 |

[ts-monorepo-template](https://github.com/sangwook-dev/ts-monorepo-template)에서 만들었다. 지켜야 할 규칙을 문서에 적어 두는 대신
**lint·타입·테스트·git hook이 막게** 해 두었다.

## 무엇이 막히나

- 의존은 바깥 → 안쪽으로만(도메인은 프레임워크·I/O를 모른다) — eslint-plugin-boundaries, 허용 목록만
- 도메인은 명사마다 모듈 하나, 모듈 밖은 그 모듈의 `index.ts`로만 — `src` 아래 폴더가 모듈 목록과 다르면 lint가 시작하지 않는다
- 도메인 모듈 최상위에는 선언만(불러오기만 해서 일어나는 일이 없다, `sideEffects: false`) — `no-restricted-syntax` 허용 목록
- 규칙을 주석으로 끄지 못한다 — `noInlineConfig` + `eslint-comments/no-use`
- lint 설정이 실제로 막는지 — `packages/eslint-config/guards.test.mts`가 위반을 넣어 확인한다(양성 대조 포함)
- 상대 경로 import 금지(`#/` 별칭·패키지 이름만) — `no-restricted-imports`
- 엄격 타입(`noUncheckedIndexedAccess`·`exactOptionalPropertyTypes` 등) — `packages/tsconfig/base.json`
- 결정 기록(ADR) 색인이 AGENTS.md와 맞는다 — 테스트 + 커밋 hook
- 문서·주석에 백틱으로 적은 저장소 경로(`apps/`·`packages/`·`docs/` 등)가 실제로 있다 — 파일을 옮기면 가리키는 글이 낡지 않게, 테스트
- 커밋 전 포매팅, 푸시 전 전체 검사 — lefthook

## 구조

- `apps/agent` — 에이전트 CLI: application(루프·포트) / infrastructure(모델 호출·도구) / interface(터미널) / main(조립)
- `packages/domain` — 엔티티·순수 규칙(zod만 의존). 템플릿의 예시(할 일)가 그대로 있고, 아직 쓰지 않는다
- `packages/eslint-config` — 공통 lint 규칙 + `layers()` 경계 헬퍼 + 가드 테스트
- `packages/tsconfig` — 공통 tsconfig 프리셋
- `packages/docs-index` — `docs/adr` → AGENTS.md 결정 색인 생성·검사, 문서·주석의 저장소 경로 검사
- `docs/adr` — 설계 결정 기록

## 명령

필요한 도구: `.tool-versions`의 Node·pnpm(mise·asdf 등).

- `pnpm agent` — 에이전트 실행(Node 타입 제거로 빌드 없이). API 키는 `apps/agent/.env.example`을 `.env`로 복사해 채운다
- `pnpm --filter @agent-from-scratch/agent debug` — 첫 줄에서 멈춘 채 디버거(WebStorm·Chrome)를 기다린다
- `pnpm turbo run lint check-types test` — 전체 검사(push 전 hook과 CI가 같은 것을 돈다)
- `pnpm docs:index` — ADR 색인을 AGENTS.md에 다시 쓴다
- `pnpm format` — prettier

## 함께 오는 것

- **CI**(`.github/workflows/ci.yml`): lint·타입·테스트·스크립트 테스트·포맷.
- **사람이 볼 것**(`attention.yml`): 매일 아침 자동으로 처리하지 못한 것(멈춘 의존성 PR, 기본 브랜치 실패, TypeScript
  상한을 풀 때)을 이슈 하나에 모으고, **새 항목이 있을 때만** 저장소 주인을 @멘션한다. 저장소 알림을 "참여·멘션만"으로
  두면 사람이 필요한 날에만 메일이 온다. 멘션 대상은 저장소 변수 `ATTENTION_MENTION`으로 바꿀 수 있다.
- **에이전트 지침**: `AGENTS.md`(+ `CLAUDE.md` 링크).

## 결정 기록

왜 이렇게 되어 있는지는 [`docs/adr/`](docs/adr)에 있다.

## 라이선스

[MIT](LICENSE). 템플릿으로 만든 프로젝트의 라이선스는 자유롭게 정하되, 템플릿에서 온 코드의 저작권 표시(`LICENSE`)는
남긴다.
