# duoskills — 보관됨 (deprecated)

이 저장소는 **더 이상 사용하지 않는다.** 모든 스킬은
[`kwakeunji/eunji-skills`](https://github.com/kwakeunji/eunji-skills) 플러그인 마켓플레이스로 이전됐다.

## 왜 옮겼나

`cp -r`로 `.claude/commands/`와 `.claude/skills/`에 복사하는 방식이라
같은 스킬이 로컬 사본 · duoskills · eunji-skills 세 벌로 갈라졌다.
2026-09-13 기준 `ticket`은 로컬 63줄 / 플러그인 262줄로 내용이 서로 달랐다.
버전이 갈리면 AI가 어느 쪽을 열지 흔들린다. 진입점은 하나여야 한다.

## 이제 이렇게 쓴다

```
/plugin marketplace add kwakeunji/eunji-skills
/plugin install gameduo-jira@eunji-skills
```

갱신은 `/plugin marketplace update eunji-skills`. 복사 붙여넣기 불필요.

## 옛 파일

`commands/`, `skills/` 아래 파일은 이력 보존을 위해 남겨 둔다. 읽지 말고 쓰지도 말 것.
현재 내용은 eunji-skills의 `plugins/gameduo-jira/skills/<name>/SKILL.md`에 있다.
