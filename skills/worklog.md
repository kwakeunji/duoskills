---
name: worklog
description: Use when user runs /worklog to start/stop a work timer and log time to a Jira ticket. Supports "/worklog DTP-XXX" to start and "/worklog stop" to stop and record.
---

# /worklog — Jira 워크로그 타이머

## Overview

`/worklog <티켓키>`로 타이머를 시작하고, `/worklog stop`으로 종료하면 경과 시간을 계산해서 Jira에 워크로그를 자동 기록한다.

## Jira 정보

- **Cloud ID:** `gameduo-dev.atlassian.net`
- **타이머 파일:** `~/.claude/.worklog_timer.json`

## 사용법

```
/worklog DTP-124          # 타이머 시작
/worklog stop             # 타이머 종료 + Jira 워크로그 기록
/worklog status           # 현재 타이머 상태 확인
/worklog DTP-124 2h       # 타이머 없이 즉시 2시간 기록
/worklog DTP-124 30m      # 타이머 없이 즉시 30분 기록
```

## 실행 절차

### CASE 1: `/worklog <티켓키>` — 타이머 시작

1. 기존 타이머가 실행 중인지 `~/.claude/.worklog_timer.json` 확인
   - 실행 중이면: "⚠️ DTP-XXX 타이머가 실행 중입니다 (Xh Xm 경과). 먼저 `/worklog stop`으로 종료하세요." 출력 후 중단
2. 타이머 파일 생성:
   ```json
   {
     "issue_key": "DTP-124",
     "started_at": "2026-04-08T14:30:00+09:00",
     "started_at_unix": 1775789400
   }
   ```
3. 결과 출력:
   ```
   ⏱️ DTP-124 워크로그 타이머 시작
   🕐 시작: 14:30 (KST)
   작업 끝나면 /worklog stop
   ```

### CASE 2: `/worklog stop` — 타이머 종료 + 기록

1. `~/.claude/.worklog_timer.json` 읽기
   - 파일 없으면: "⚠️ 실행 중인 타이머가 없습니다." 출력 후 중단
2. 경과 시간 계산 (현재 시각 - started_at_unix)
3. Jira 시간 형식으로 변환:
   - 90분 → `1h 30m`
   - 30분 미만은 30분으로 올림 (최소 단위)
4. `mcp__atlassian__addWorklogToJiraIssue` 호출:
   ```
   cloudId: gameduo-dev.atlassian.net
   issueIdOrKey: <issue_key>
   timeSpent: <계산된 시간>
   started: <started_at ISO 형식>
   ```
5. 타이머 파일 삭제
6. 결과 출력:
   ```
   ✅ DTP-124 워크로그 기록 완료
   ⏱️ 14:30 → 17:15 (2h 45m)
   📝 Jira 기록: 2h 45m
   ```

### CASE 3: `/worklog status` — 현재 상태 확인

1. `~/.claude/.worklog_timer.json` 읽기
   - 파일 없으면: "타이머 없음" 출력
   - 있으면: 티켓키, 시작 시각, 현재 경과 시간 출력
   ```
   ⏱️ DTP-124 타이머 실행 중
   🕐 시작: 14:30 (KST)
   ⏳ 경과: 2h 15m
   ```

### CASE 4: `/worklog <티켓키> <시간>` — 즉시 기록

1. 시간 파싱 (`2h`, `30m`, `1h30m`, `1.5h` 등)
2. `mcp__atlassian__addWorklogToJiraIssue` 호출:
   ```
   cloudId: gameduo-dev.atlassian.net
   issueIdOrKey: <티켓키>
   timeSpent: <시간>
   started: <현재 시각 ISO 형식>
   ```
3. 결과 출력:
   ```
   ✅ DTP-124 워크로그 기록 완료
   📝 Jira 기록: 2h
   ```

## 시간 계산 규칙

- 경과 시간은 **15분 단위로 반올림** (예: 37분 → 30m, 53분 → 1h)
- 최소 기록 단위: **15m**
- 15분 미만이면: "⚠️ 경과 시간이 15분 미만입니다. 그래도 15m으로 기록할까요?" 확인

## 타이머 파일

- 경로: `~/.claude/.worklog_timer.json`
- 세션이 바뀌어도 파일이 남아있으므로 타이머 유지됨
- `/worklog stop` 시 파일 삭제
- 수동 삭제로 타이머 리셋 가능

## 사용 예시

```
# 기본 흐름
/worklog DTP-124
(2시간 작업)
/worklog stop

# 즉시 기록
/worklog DTP-124 3h

# 상태 확인
/worklog status
```
