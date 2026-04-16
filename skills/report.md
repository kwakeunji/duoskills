---
name: report
description: 전날 git 커밋 내역과 Jira 이슈를 분석해 데일리 스탠드업 보고서를 Notion 데이터베이스에 작성한다.
model: haiku
---

# /report — 일일 업무 보고 자동 생성 → Notion 저장

## Overview

사용자가 `/report`를 실행하면 전날 git 커밋 내역과 Jira 이슈를 분석해 데일리 스탠드업 보고서를 작성하고, Notion 데이터베이스에 페이지로 저장한다.

## 실행 절차

### 1. git 데이터 수집

아래 명령을 실행해 데이터를 수집한다:

```bash
# 어제 날짜 범위 커밋 (로컬 타임존 기준)
git log --oneline --after="yesterday 00:00" --before="today 00:00"

# 어제 커밋이 없으면 최근 5개
git log --oneline -5

# 현재 브랜치
git branch --show-current
```

### 2. Jira 이슈 조회

`mcp__atlassian-rovo__getAccessibleAtlassianResources`로 cloudId를 확인한 뒤,
`mcp__atlassian-rovo__searchJiraIssuesUsingJql`로 아래 JQL을 실행한다:

**메인 이슈 + 하위작업 조회 (2회 실행):**

1. 메인 이슈 조회:
```
project = DTP AND assignee = currentUser() AND status != Done AND updated >= -7d ORDER BY updated DESC
```

2. 메인 이슈의 하위작업 조회 (메인 이슈 키를 기반으로):
```
project = DTP AND parent in (DTP-XXX, DTP-YYY, ...) ORDER BY updated DESC
```
- 1번에서 조회된 `작업` 타입 이슈의 키를 parent 조건에 넣는다
- 이렇게 하면 본인에게 할당되지 않은 하위작업도 포함하여 전체 작업 현황을 파악할 수 있다

- cloudId: `getAccessibleAtlassianResources`로 조회한 값 사용
- 각 쿼리 최대 20개까지 조회

### 3. 변경사항 분석

- **전일 진행 업무**: 어제 커밋 메시지 + diff를 분석, Jira 메인 이슈 및 하위작업과 매핑하여 기능 단위로 해석. 각 메인 티켓 아래 관련 하위작업도 함께 표기
- **오늘 작업 계획**: Jira에서 Backlog/해야 할 일 상태인 하위작업 + 진행 중 이슈 기반으로 작성
- 파일명/함수명 그대로 나열하지 말고, 기능 단위로 요약

### 4. Notion 데이터베이스에 페이지 생성

`mcp__notion__notion-create-pages` 도구로 아래 데이터베이스에 페이지를 생성한다:

- **데이터베이스 ID**: `3442bb52fd43801bbf91fb873ae809c0`
- **데이터소스 URL**: `collection://3442bb52-fd43-8053-96ee-000bb3a7fa04`
- **프로퍼티**:
  - `이름` (title): `YYYY-MM-DD 데일리 리포트` (예: `2026-04-16 데일리 리포트`)
  - `날짜` (date): 오늘 날짜 (ISO-8601, 예: `2026-04-16`)

**페이지 본문**은 아래 템플릿의 Notion enhanced markdown으로 작성한다:

```markdown
## 전일 진행 업무

- **<기능 카테고리>** _(<소요시간 추정>h)_ — [티켓키](Jira URL)
    - <구현 내용 1>
    - <구현 내용 2>

## 오늘 작업 계획

- **<기능 카테고리>** — [티켓키](Jira URL)
    - <작업 항목 1>
    - <작업 항목 2>

## 블로커

- 없음

## 이번 주 목표

- **<이번 주 브랜치/작업 테마 기반으로 요약>** — [티켓키](Jira URL)
    - <세부 목표 1>
    - <세부 목표 2>
```

### 5. CLI 출력

Notion 페이지 생성 후, 생성된 페이지 URL을 출력하고 보고서 내용도 CLI에 간략히 표시한다.

## 작성 규칙

- 구현 내용은 **짧은 명사구/키워드** 스타일로 작성 (완전한 문장 금지)
  - ✅ `연동/소스 UI 분리 + 동기화 상태 관리 + S3 파일 업로드`
  - ✅ `리포트 정확성 개선 6건 + --include AST 크래시 수정`
  - ❌ `Gemini Vision API를 활용해 화면을 인식하고 버튼 위치를 AI가 자동 탐색·탭`
- 세부 항목은 한 줄에 관련 내용을 `+`로 묶어서 최대한 압축
- 기능 카테고리명은 간결하게 (프로젝트명/모듈명 수준)
- 소요시간은 변경된 코드량과 커밋 수를 기반으로 합리적으로 추정
- Jira 티켓 연동: 이슈가 여러 개일 경우 카테고리별로 묶고 대표 티켓 1개를 링크로 연결
- 하위작업(Subtask)도 포함하여 관련 작업을 빠짐없이 반영
- 블로커와 이번 주 목표는 사용자가 별도로 언급한 경우에만 내용을 채움, 아니면 기본값 유지
- 같은 날짜의 페이지가 이미 존재하면 새로 만들지 말고 기존 페이지를 업데이트

## 사용 예시

```
/report
/report 블로커: 배포 환경 미설정
```

추가 인수를 넘기면 해당 내용을 블로커나 특이사항에 반영한다.
