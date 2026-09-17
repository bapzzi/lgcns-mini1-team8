# 같이 일하는 규칙

노션 8조 페이지의 「커밋 및 PR, 브랜치 사용 규칙(GitHub)」(매니저님 배포)을 따릅니다. 여기에는 매일 쓰는 부분만 옮겼습니다. 둘이 다르면 노션 원문을 따릅니다.

## 브랜치

```
main      최종 제출 · 발표용 안정 버전
develop   기능이 모이는 통합 브랜치 (기본 브랜치)
feat/*    기능 하나당 브랜치 하나. 예) feat/login, feat/job-list
fix/* · docs/* · style/* · refactor/* · chore/*
```

- `main` · `develop`에는 직접 push하지 않습니다. 저장소 설정으로 막혀 있습니다
- 작업 시작 전에 항상 `develop`을 최신으로 받습니다

```bash
git checkout develop
git pull origin develop
git checkout -b feat/login
```

## 커밋 메시지

```
<type>(<scope>): <설명>
```

- type: `feat` · `fix` · `docs` · `style` · `refactor` · `chore` · `test` (소문자, 콜론 뒤 한 칸, 끝에 마침표 없음)
- 다른 사람 코드가 깨지는 변경은 콜론 앞에 `!`. 예) `refactor(auth)!: 토큰 전달 방식 변경`
- 임시 저장은 `wip: ...`. **PR 보내기 전에 하나로 합칩니다**
- 분류가 애매하면 `chore`

## PR

- `feat/*` → `develop`으로 보냅니다
- 제목은 커밋 메시지 형식과 같습니다
- 본문은 PR 템플릿 4칸(작업 내용 · 변경 이유 · 확인한 내용 · 참고 사항)을 채웁니다
- 승인 1명 이상이어야 병합됩니다(저장소 설정)
- 병합되면 브랜치는 자동으로 지워집니다

| 정할 것 | 값 |
|---|---|
| 리뷰어 | `[9/18 회의에서 정한다]` |
| 리뷰 대기 시간 | `[9/18 회의에서 정한다]` |
| wip 정리를 「Squash and merge」로 대신해도 되는지 | `[매니저님께 확인]` |

## 충돌이 났을 때

혼자 지우지 않고, 그 파일을 고친 사람과 같이 봅니다.

```bash
git checkout develop && git pull origin develop
git checkout feat/login
git merge develop
# 정리 후
git commit -m "fix(merge): develop 충돌 해결"
```

## 규격을 바꿔야 할 때

`docs/api.md` · `docs/data.md`는 혼자 고치지 않습니다. 영향받는 담당과 먼저 이야기하고, 합의한 뒤 파일을 고쳐 PR로 올립니다.

## 올리면 안 되는 것

- `.env`: 값이 든 파일. `.gitignore`가 막고 있지만 강제로 올리지 않습니다
- API 키 · 비밀번호 · 토큰을 코드에 직접 쓰지 않습니다
- 이 저장소는 **공개**입니다. 올린 것은 누구나 봅니다
