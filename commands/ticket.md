Jira DTP(DEV-team PLATFORM) 프로젝트에 티켓을 생성하고, 현재 git 브랜치의 GitHub 링크를 연결해줘.

## 핵심 규칙

- **프로젝트:** DTP (projectKey: DTP)
- **Cloud ID:** `gameduo-dev.atlassian.net`
- **담당자 기본값:** eunji (accountId: `712020:1dc7aff8-a83e-4826-940c-5b2e301a398b`)
- **제목 형식:** `[카테고리] 제목` — 카테고리: AI-IMG, AI-QA, AI-LMK, Infra 중 선택. 없으면 질문
- **이슈 타입:** 기본 `작업`. 맥락에 따라 Bug/Story/Subtask 등 선택
- **GitHub Repo:** `git remote get-url origin`으로 자동 감지

## 설명 템플릿 (작업 타입 메인 티켓)

`작업`(Task) 타입 메인 티켓의 설명은 아래 구조를 따른다. (Subtask, Task(etc)는 간단 작성 가능)

```markdown
## 배경
* 왜 이 작업이 필요한지 (현재 문제점, 기존 방식의 한계)

## 목표
* 이 티켓으로 달성하려는 것 (구체적 결과물)

## 작업 범위
* 포함: 구현할 모듈/기능 상세
* 제외: 이번 범위에서 빠지는 것 (별도 티켓 참조)

## 검증 방법
* API/기능/UX 레벨별 검증 항목

## 참고 링크
* GitHub, PR, 관련 이슈

## planned_size
* 점수: N
* 사유: 범위, 복잡도, 불확실성, 협업/조율, 검증 부담
```

## 실행 절차

### 1. 정보 수집
- 제목: 슬래시 커맨드 인수로 받음. 없으면 사용자에게 질문
- 카테고리가 제목에 없으면 맥락에서 추론하거나 질문
- 설명: 작업 타입이면 위 템플릿으로 작성

### 2. GitHub 링크 감지
```bash
git branch --show-current
git remote get-url origin 2>/dev/null
gh pr view --json url -q .url 2>/dev/null
```
- PR 있으면 → PR URL, 없으면 → 브랜치 URL
- git repo가 아니면 → 생략

### 3. 티켓 생성
`mcp__atlassian__createJiraIssue` 호출:
- cloudId: `gameduo-dev.atlassian.net`
- projectKey: `DTP`
- issueTypeName: `작업` (또는 맥락에 맞는 타입)
- summary: `[카테고리] 제목`
- assignee_account_id: `712020:1dc7aff8-a83e-4826-940c-5b2e301a398b`
- contentFormat: `markdown`
- description: (있을 때만)
- parent: (Subtask인 경우)

### 4. GitHub 원격 링크 연결
생성된 이슈 키(DTP-XXXX)에 `mcp__atlassian__fetchAtlassian`으로 원격 링크 추가:
- method: POST
- url: `https://gameduo-dev.atlassian.net/rest/api/3/issue/DTP-XXXX/remotelink`
- body:
```json
{
  "object": {
    "url": "<GitHub URL>",
    "title": "<PR 제목 또는 브랜치명>",
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
📋 타입: <이슈 타입> | 카테고리: <카테고리>
```
