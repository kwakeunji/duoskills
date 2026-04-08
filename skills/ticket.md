---
name: ticket
description: Use when user runs /ticket to create a Jira issue in the DEV-team project. Creates a 작업-type ticket and links the current GitHub branch or PR.
---

# /ticket — Jira 티켓 자동 발행

## Overview

사용자가 `/ticket`을 실행하면 Jira DEV-team 프로젝트에 **작업** 이슈를 생성하고, 현재 git 브랜치의 GitHub 링크를 자동으로 연결한다.

## Jira 프로젝트 정보

- **Cloud ID:** `gameduo-dev.atlassian.net`
- **Project Key:** `DEV`
- **Issue Type:** 항상 `작업` (고정)
- **GitHub Repo:** `https://github.com/kwakeunji/comfyui-gradio-app`

## 실행 절차

### 1. 정보 수집

아래를 확인한다 (이미 제공했으면 생략):
- 제목 (필수)
- 설명 (선택)

### 2. GitHub 링크 감지

```bash
# 현재 브랜치 확인
git branch --show-current

# PR이 존재하는지 확인 (gh CLI 있을 경우)
gh pr view --json url -q .url 2>/dev/null
```

- PR이 있으면 → PR URL 사용
- PR이 없으면 → 브랜치 URL 사용: `https://github.com/kwakeunji/comfyui-gradio-app/tree/<branch>`
- git repo가 아니면 → GitHub 링크 생략

### 3. 티켓 생성

`mcp__atlassian__createJiraIssue` 호출:
```
cloudId: gameduo-dev.atlassian.net
projectKey: DEV
issueType: 작업
summary: <제목>
description: <설명 or 생략>
```

### 4. GitHub 링크 연결

생성된 이슈 키(예: DEV-XXXX)에 `mcp__atlassian__fetchAtlassian`으로 원격 링크 추가:

```
method: POST
url: https://gameduo-dev.atlassian.net/rest/api/3/issue/DEV-XXXX/remotelink
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
✅ DEV-XXXX 생성 완료
🔗 https://gameduo-dev.atlassian.net/browse/DEV-XXXX
🐙 GitHub: <연결된 URL>
```

## 사용 예시

```
/ticket 스프라이트시트 zoom 파라미터 기능 추가
/ticket 화질높이기 해상도 키 오류 수정
/ticket
```

인수 없이 실행하면 제목을 물어본다.
