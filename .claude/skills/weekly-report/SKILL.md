---
name: weekly-report
description: 주간/월간 작업 보고서를 Git 커밋 + Claude 세션 기반으로 자동 생성. "6월 보고서", "W28 보고서", "한달 보고서" 등으로 호출 가능.
---

# 작업 보고서 생성

주간 또는 월간 보고서를 자동 생성한다. 호출 시 인자로 기간을 받을 수 있다.

- `/weekly-report` → 지난 주 주간 보고서
- `/weekly-report 6월` 또는 `/weekly-report M06` → 6월 월간 보고서
- `/weekly-report W25` → W25 주간 보고서
- `/weekly-report 3개월` → 최근 3개월 월간 보고서

## Phase 1: 기간 및 모드 결정

1. 인자 파싱:
   - 인자 없음 → **주간 모드**, 지난 주 (ISO 8601 주차)
   - `M{월}` 또는 `{월}월` → **월간 모드**, 해당 월 (1일~말일. 예: 6월 → 6/1~6/30)
   - `W{주차}` → **주간 모드**, 해당 주차
   - `{N}개월` → **월간 모드**, 최근 N개월 (각 월 1일~말일)
2. 사용자에게 기간 확인
3. 사용자 이름 확인 (메모리에 없으면 질문)
4. 파일명은 반드시 **실명** 사용 (GitHub ID 금지)

## Phase 2: Git 활동 수집

1. 사용자의 홈 디렉토리에서 모든 Git 레포 탐색:
   ```bash
   # 홈 디렉토리 하위 1~2레벨에서 .git 폴더 탐색
   find ~/ -maxdepth 2 -name ".git" -type d
   ```
2. 각 레포에서 해당 기간의 커밋 수집:
   ```bash
   git log --oneline --since="{시작일}" --until="{종료일}"
   ```
3. 커밋 메시지에서 작업 유형 자동 분류:
   - `feat:` → 신규 기능
   - `fix:` → 버그 수정
   - `refactor:` → 리팩토링
   - `docs:` → 문서
   - `ci:` → CI/CD
   - 한국어 커밋도 내용 기반 분류
4. 프로젝트별 커밋 수, 주요 변경 사항 정리

## Phase 3: 활동 통계 집계

보고서 상단에 아래 통계를 포함한다:

| 지표 | 산정 방법 |
|------|----------|
| 총 커밋 | 전체 레포 합산 |
| 활성 프로젝트 | 1건 이상 커밋이 있는 레포 수 |
| 신규 프로젝트 | 해당 기간에 initial commit이 있는 레포 |
| 일 평균 커밋 | 총 커밋 / 기간 일수 |
| 하네스 운영 | 활성 Claude 하네스 수 (CLAUDE.md 기반 확인) |

## Phase 4: Claude 사용량 집계

1. `~/.claude/projects/` 디렉토리에서 해당 기간의 세션 로그 집계:
   ```bash
   # 메인 세션 (subagents 제외)
   find ~/.claude/projects/ -maxdepth 3 -name "*.jsonl" ! -path "*/subagents/*" -newermt "{시작일}" ! -newermt "{종료일}" | wc -l
   # 서브에이전트 세션
   find ~/.claude/projects/ -maxdepth 4 -name "*.jsonl" -path "*/subagents/*" -newermt "{시작일}" ! -newermt "{종료일}" | wc -l
   # 일별 분포
   find ~/.claude/projects/ -maxdepth 3 -name "*.jsonl" ! -path "*/subagents/*" -newermt "{시작일}" ! -newermt "{종료일}" -printf "%TY-%Tm-%Td\n" | sort | uniq -c
   # 데이터 크기 (월간만)
   find ~/.claude/projects/ -maxdepth 3 -name "*.jsonl" ! -path "*/subagents/*" -newermt "{시작일}" ! -newermt "{종료일}" -printf "%s\n" | awk '{s+=$1} END {printf "%.1f MB\n", s/1024/1024}'
   ```
2. 보고서에 `Claude 사용량` 섹션으로 포함
3. 월간은 일별 세션 분포 테이블도 추가

## Phase 5: 비코드 작업 확인

사용자에게 질문:
- "코드 외 작업이 있었나요? (분석, 미팅, 기획, 하네스 운영 등)"
- "이슈나 블로커가 있었나요?"
- 주간: "다음 주 계획은?"
- 월간: "다음 달 계획은?"

## Phase 6: 보고서 생성

1. `RULES.md`의 템플릿에 맞춰 보고서 작성
2. **활동 통계 섹션**을 반드시 포함
3. 파일 저장:
   - 주간: `reports/{연도}/W{주차}/{이름}.md`
   - 월간: `reports/{연도}/M{월}/{이름}.md`
4. 사용자에게 내용 확인 요청
5. 확인 후 커밋 & 푸시 (사용자 동의 시)

## 보고서 품질 기준

- 각 프로젝트의 작업 내용은 **구체적**으로 (커밋 메시지 기반 + 맥락 보강)
- 성과는 **정량적**으로 (커밋 수, 기능 수, 배포 여부)
- 상태는 반드시 완료/진행중/보류 중 택 1
- 주간: A4 1~2페이지, 월간: A4 2~3페이지
- 월간 보고서는 프로젝트별 커밋 수를 명시하고 활동량 순으로 정렬

## 취합본 생성 (관리자 전용)

`취합본 생성` 또는 `SUMMARY 만들어줘` 요청 시:
1. 해당 주차/월 폴더의 모든 `*.md` 파일 읽기 (SUMMARY.md 제외)
2. `RULES.md`의 SUMMARY.md 구조에 맞춰 취합본 생성
3. 팀원별 하이라이트 1줄 요약, 공통 이슈, 다음 주/월 포커스 정리
4. `SUMMARY.md`로 저장
