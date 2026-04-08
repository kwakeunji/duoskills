Jira 워크로그 타이머. `/worklog DTP-XXX`로 시작, `/worklog stop`으로 종료 및 기록.

## 사용법

- `/worklog DTP-124` — 타이머 시작
- `/worklog stop` — 타이머 종료 + Jira 워크로그 기록
- `/worklog status` — 현재 타이머 상태 확인
- `/worklog DTP-124 2h` — 타이머 없이 즉시 기록

## 타이머 파일

`~/.claude/.worklog_timer.json`에 시작 시각 저장. 세션 바뀌어도 유지됨.

## 실행 시

1. 인수 파싱: 티켓키만 → 타이머 시작 / stop → 종료 / status → 상태 / 티켓키+시간 → 즉시 기록
2. 타이머 시작: JSON 파일에 issue_key, started_at, started_at_unix 저장
3. 타이머 종료: 경과 시간 계산 → 15분 단위 반올림 → `mcp__atlassian__addWorklogToJiraIssue` 호출 → 파일 삭제
4. Jira 정보: cloudId `gameduo-dev.atlassian.net`
