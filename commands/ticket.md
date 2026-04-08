Jira DEV-team 프로젝트에 작업(Task) 티켓을 생성하고, 현재 git 브랜치의 GitHub 링크를 연결해줘.

## Jira 프로젝트 정보

- Cloud ID: `gameduo-dev.atlassian.net`
- Project Key: `DEV`
- Issue Type: 항상 `작업` (고정, 변경하지 않음)
- GitHub Repo: `https://github.com/kwakeunji/comfyui-gradio-app`

## 실행 절차

### 1. 정보 수집
- 제목: 슬래시 커맨드 인수로 받음. 없으면 사용자에게 질문
- 설명: 선택사항

### 2. GitHub 링크 감지
```bash
git branch --show-current           # 현재 브랜치
gh pr view --json url -q .url 2>/dev/null  # PR URL (있으면)
```
- PR 있으면 → PR URL 사용
- PR 없으면 → `https://github.com/kwakeunji/comfyui-gradio-app/tree/<브랜치명>`
- git repo가 아니면 → GitHub 링크 생략

### 3. 티켓 생성
`mcp__atlassian__createJiraIssue` 호출:
- cloudId: `gameduo-dev.atlassian.net`
- projectKey: `DEV`
- issueType: `작업`
- summary: <제목>
- description: <설명> (있을 때만)

### 4. GitHub 원격 링크 연결
생성된 이슈 키(DEV-XXXX)에 `mcp__atlassian__fetchAtlassian`으로 원격 링크 추가:
- method: POST
- url: `https://gameduo-dev.atlassian.net/rest/api/3/issue/DEV-XXXX/remotelink`
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
✅ DEV-XXXX 생성 완료
🔗 https://gameduo-dev.atlassian.net/browse/DEV-XXXX
🐙 GitHub: <연결된 URL>
```
