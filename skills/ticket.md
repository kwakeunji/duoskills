---
name: ticket
description: Use when user runs /ticket to create a Jira issue in the DTP project. Follows team conventions for title format, required fields, issue types, and status transitions.
---

# /ticket — Jira 티켓 자동 발행

## Overview

사용자가 `/ticket`을 실행하면 DTP(DEV-team PLATFORM) 프로젝트에 Jira 이슈를 생성하고, 현재 git 브랜치의 GitHub 링크를 자동으로 연결한다.

## Jira 프로젝트 정보

- **Cloud ID:** `gameduo-dev.atlassian.net`
- **Project Key:** `DTP`
- **담당자 기본값:** eunji (accountId: `712020:1dc7aff8-a83e-4826-940c-5b2e301a398b`)
- **GitHub Repo:** 현재 git remote origin에서 자동 감지

## 제목 규칙

반드시 `[카테고리] 제목` 형식을 따른다.

카테고리는 아래 목록에서 가장 어울리는 것을 선택:
- **AI-IMG** — AI 이미지 관련 작업
- **AI-QA** — AI QA 관련 작업
- **AI-LMK** — AI 번역(LMK) 관련 작업
- **Infra** — 인프라/환경 관련 작업

매칭되는 카테고리가 없으면 사용자에게 질문한다.

## 이슈 타입

사용자가 명시하지 않으면 기본값은 `작업`(Task). 맥락에 따라 적절한 타입을 선택:

| 타입 | ID | 언제 사용 |
|------|-----|----------|
| 작업 (Task) | 10102 | 일반 개발, 기본값 |
| 스토리 (Story) | 10103 | 사용자 관점 기능 |
| 버그 (Bug) | 10104 | 버그 수정 |
| Subtask | 10101 | 부모 티켓 아래 하위 작업 |
| Task(etc) | 10105 | 회의/기타 |
| 에픽 (Epic) | 10100 | 대규모 작업 묶음 |

## 사이즈 필드 (Custom Field IDs)

| 필드 | ID | 옵션 값 (id) |
|------|-----|-------------|
| planned_size | customfield_10248 | 0(10241), 1(10117), 2(10118), 3(10119), 5(10120), 8(10121) |
| planned_size_reason | customfield_10249 | ADF 형식 텍스트 |
| proposed_final_size | customfield_10250 | 0(10242), 1(10122), 2(10123), 3(10124), 5(10125), 8(10126) |
| proposed_size_reason | customfield_10251 | ADF 형식 텍스트 |
| final_size_score | customfield_10252 | 0(10243), 1(10127), 2(10128), 3(10129), 5(10130), 8(10131) |
| size_review_note | customfield_10253 | ADF 형식 텍스트 |
| project_code | customfield_10215 | 라벨 배열 |

> ADF 형식 텍스트 필드는 `contentFormat: "markdown"` 으로 보내면 자동 변환되지 않는 경우가 있다. 실패 시 ADF JSON으로 직접 전달한다.

## 상태 흐름 & 전환 시 필수 필드

```
BACKLOG → TO DO → IN PROGRESS ↔ HOLDING → SIZE REVIEW → DONE
```

| 전환 | 필수 값 | 비고 |
|------|--------|------|
| BACKLOG → TO DO | 없음 | 빠르게 이동 가능 |
| TO DO → IN PROGRESS | Assignee, Start date, Due date, project_code | 계획형 개발: planned_size, planned_size_reason도 필수 |
| TO DO/IN PROGRESS → HOLDING | HOLDING 사유 | 코멘트에 기록 |
| Any → WON'T DO | 안 함 사유 | 코멘트에 기록 |
| IN PROGRESS → SIZE REVIEW | proposed_final_size, proposed_size_reason | 계획 대비 변경점도 정리 |
| SIZE REVIEW → DONE | final_size_score | size_review_note는 필요 시 |

## 점수 산정 기준 (5가지)

planned_size_reason, proposed_size_reason 작성 시 아래 기준으로 판단:
- **범위** — 얼마나 넓은 영역에 영향을 주는가
- **복잡도** — 구현/판단이 얼마나 까다로운가
- **불확실성** — 시작할 때 모르는 것이 얼마나 많은가
- **협업/조율** — 혼자 끝나는지, 여러 사람과 맞춰야 하는지
- **검증 부담** — 끝났는지 확인하는 데 얼마나 손이 드는지

## 버그 0점 규칙

본인 이전 작업에서 놓친 검증 누락으로 발생한 버그는 planned_size = 0.
reason 예시: `0점. {이전 티켓 키} 구현 시 확인했어야 하는 부분으로, 본인 이전 작업에서 놓친 검증 누락으로 발생한 버그.`

## 작업 유형별 처리

| 유형 | 이슈 타입 | 점수 방식 |
|------|----------|----------|
| 일반 개발 | Task / Story | planned → proposed → final |
| 긴급 대응 | Task / Bug | 사후 확정 비중 큼 |
| 기타/운영성 | 주간 기타 parent 아래 Subtask | 부모 티켓 기준 확정 |
| 작은 개발작업 | 주간 기타 parent 아래 Subtask → 커지면 승격 | 부모 또는 승격 후 확정 |

## 설명 템플릿 (작업 타입 메인 티켓)

`작업`(Task) 타입 메인 티켓의 설명은 반드시 아래 구조를 따른다.
(Subtask, Task(etc) 타입은 간단하게 작성해도 무방)

```markdown
## 배경
* 왜 이 작업이 필요한지 (현재 문제점, 기존 방식의 한계)
* 기술적/비즈니스적 맥락

## 목표
* 이 티켓으로 달성하려는 것 (구체적 결과물)
* 사용자/시스템에 제공하는 가치

## 작업 범위
* 포함:
    * 구현할 모듈/기능 상세 (파일명, API 등)
    * 프론트/백엔드 구분하여 기재
* 제외:
    * 이번 범위에서 빠지는 것 (별도 티켓 참조)

## 검증 방법
* API/기능/UX 레벨별 검증 항목
* 성공/실패 케이스 확인 사항

## 참고 링크
* GitHub, PR, 관련 이슈, 문서/슬랙 링크

## planned_size
* 점수: N
* 사유:
    * 범위, 복잡도, 불확실성, 협업/조율, 검증 부담 기준으로 기재
```

## 실행 절차

### 1. 정보 수집

아래를 확인한다 (이미 제공했으면 생략):
- **제목** (필수) — 카테고리 포함 여부 확인. 없으면 맥락에서 추론하거나 질문
- **설명** (선택) — 제공되면 위 템플릿 형식으로 작성
- **이슈 타입** (선택) — 명시 안 하면 `작업` 기본값
- **planned_size** (선택) — 제공되면 함께 설정
- **부모 티켓** (선택) — Subtask인 경우

### 2. GitHub 링크 감지

```bash
# 현재 브랜치 확인
git branch --show-current

# remote origin URL 확인
git remote get-url origin 2>/dev/null

# PR이 존재하는지 확인
gh pr view --json url -q .url 2>/dev/null
```

- PR이 있으면 → PR URL 사용
- PR이 없으면 → 브랜치 URL 사용: `<remote-url>/tree/<branch>`
- git repo가 아니면 → GitHub 링크 생략

### 3. 티켓 생성

`mcp__atlassian__createJiraIssue` 호출:
```
cloudId: gameduo-dev.atlassian.net
projectKey: DTP
issueTypeName: <이슈 타입>
summary: [카테고리] <제목>
description: <설명>
assignee_account_id: 712020:1dc7aff8-a83e-4826-940c-5b2e301a398b
contentFormat: markdown
parent: <부모 티켓 키> (Subtask인 경우)
additional_fields: { planned_size, planned_size_reason 등 해당 시 }
```

### 4. GitHub 링크 연결

생성된 이슈 키(예: DTP-XXXX)에 `mcp__atlassian__fetchAtlassian`으로 원격 링크 추가:

```
method: POST
url: https://gameduo-dev.atlassian.net/rest/api/3/issue/DTP-XXXX/remotelink
body:
{
  "object": {
    "url": "<GitHub URL>",
    "title": "<PR 제목 or 브랜치명>",
    "icon": {
      "url16x16": "https://github.com/favicon.ico",
      "title": "GitHub"
    }
  }
}
```

### 5. 결과 출력

```
✅ DTP-XXXX 생성 완료
🔗 https://gameduo-dev.atlassian.net/browse/DTP-XXXX
🐙 GitHub: <연결된 URL>
📋 타입: <이슈 타입> | 카테고리: <카테고리> | planned_size: <값 or 미설정>
```

## 사용 예시

```
/ticket 스프라이트시트 zoom 파라미터 기능 추가
/ticket 화질높이기 해상도 키 오류 수정
/ticket LMK 번역 프롬프트 API 구현
/ticket
```

인수 없이 실행하면 제목을 물어본다.
