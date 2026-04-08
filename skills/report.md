---
name: report
description: Use when user runs /report to generate a daily standup report. Fetches assigned Jira DEV backlog issues for interactive selection as today's plan, and uses yesterday's git commits for previous day's work.
---

# /report — 일일 업무 보고 자동 생성

## Overview

사용자가 `/report`를 실행하면 전날 git 커밋 내역과 현재 변경사항을 분석해 데일리 스탠드업 보고서를 생성한다.

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

### 2. Jira 백로그 이슈 조회

`mcp__atlassian__atlassianUserInfo`로 현재 사용자 계정 ID를 확인한 뒤,
`mcp__atlassian__searchJiraIssuesUsingJql`로 아래 JQL을 실행한다:

```
project = DEV AND assignee = currentUser() AND status != Done ORDER BY priority DESC
```

- cloudId: `gameduo-dev.atlassian.net`
- 최대 20개까지 조회

### 3. 이슈 목록 출력 & 선택 요청

조회된 이슈를 아래 형식으로 출력하고 사용자에게 번호 입력을 요청한다:

```
📋 백로그 이슈 (DEV 프로젝트)
  1. DEV-101 로그인 버그 수정
  2. DEV-102 이미지 업로드 기능 추가
  3. DEV-103 성능 최적화

오늘 할 이슈 번호 입력 (예: 1,3):
```

**예외 처리:**
- 이슈가 없으면: "백로그 이슈가 없습니다. 오늘 작업 계획을 직접 입력해주세요." 출력 후 사용자 입력 대기
- MCP 오류 발생 시: Jira 단계를 건너뛰고 기존 방식(git diff 기반)으로 오늘 작업 계획 채움

### 4. 변경사항 분석

- **전일 진행 업무**: 어제 커밋된 내용 → 커밋 메시지를 보고 실제 구현 내용으로 해석
- **오늘 작업 계획**: 사용자가 선택한 Jira 이슈 (DEV-XXX 키 + 이슈 제목)
- 파일명/함수명 그대로 나열하지 말고, 기능 단위로 요약

### 5. 보고서 출력

슬랙 mrkdwn 형식으로 아래 템플릿에 맞춰 작성한다.
- 굵게: `*텍스트*`
- 기울임: `_텍스트_`
- 인라인 코드: `` `코드` ``
- 링크: `<URL|텍스트>`
- 들여쓰기는 스페이스 4칸 기준

```
1. *전일 진행 업무*
    - *<기능 카테고리>* _(<소요시간 추정>h)_ — <Jira URL|티켓키>
        - <구현 내용 1>
        - <구현 내용 2>

2. *오늘 작업 계획*
    - *<기능 카테고리>* — <Jira URL|티켓키>
        - <작업 항목 1>
        - <작업 항목 2>

3. *블로커*
    - 없음

4. *이번 주 목표*
    - *<이번 주 브랜치/작업 테마 기반으로 요약>* — <Jira URL|티켓키>
        - <세부 목표 1>
        - <세부 목표 2>
```

**Jira 티켓 연동 규칙:**
- 전일 진행 업무: 어제 업데이트된 Jira 이슈(`updated >= "어제날짜"`)를 조회해 git 커밋 내용과 매핑
- 오늘 작업 계획: 사용자가 선택한 Jira 이슈 URL을 링크로 연결
- 이슈가 여러 개일 경우 카테고리별로 묶고 가장 관련성 높은 티켓 1개를 대표로 연결

## 작성 규칙

- 구현 내용은 **짧은 명사구/키워드** 스타일로 작성 (완전한 문장 금지)
  - ✅ `연동/소스 UI 분리 + 동기화 상태 관리 + S3 파일 업로드`
  - ✅ `리포트 정확성 개선 6건 + --include AST 크래시 수정`
  - ❌ `Gemini Vision API를 활용해 화면을 인식하고 버튼 위치를 AI가 자동 탐색·탭`
- 세부 항목은 한 줄에 관련 내용을 `+`로 묶어서 최대한 압축
- 기능 카테고리명은 간결하게 (프로젝트명/모듈명 수준)
- 소요시간은 변경된 코드량과 커밋 수를 기반으로 합리적으로 추정
- 코드가 없는 환경(git repo 아님)이면 사용자에게 수동 입력 요청
- 블로커와 이번 주 목표는 사용자가 별도로 언급한 경우에만 내용을 채움, 아니면 기본값 유지
- 출력은 항상 코드블록(```) 안에 감싸서 복사하기 편하게 제공

## 사용 예시

```
/report
/report 블로커: 배포 환경 미설정
```

추가 인수를 넘기면 해당 내용을 블로커나 특이사항에 반영한다.
