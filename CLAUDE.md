# Weekly Reports

주간 작업 보고서 공유 레포. 규칙과 템플릿은 `RULES.md` 참조.

## 트리거

- `주간 보고서 생성`, `이번 주 보고서`, `weekly report` 요청 시 `weekly-report` 스킬 사용
- `취합본 생성`, `SUMMARY 만들어줘` 요청 시 해당 주차 폴더의 모든 보고서를 읽어서 SUMMARY.md 생성

## 규칙

- 보고서 파일은 `reports/{연도}/W{주차}/{이름}.md` 형식
- 템플릿은 `RULES.md`의 보고서 템플릿 섹션을 따름
- ISO 8601 주차 기준 (월요일~일요일)
- 커밋 메시지는 한국어로 작성
