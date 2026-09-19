## 최초 세팅 (레포 클론 후 1회)

```bash
brew install pre-commit
pre-commit install --install-hooks
```

이 한 줄로 커밋 메시지 검사(commit-msg)와 파일 검사(pre-commit) 훅이 모두 설치됩니다.
(`default_install_hook_types`가 `.pre-commit-config.yaml`에 이미 지정되어 있어서
`--hook-type`을 따로 지정할 필요가 없습니다.)

## 커밋 타입
feat, fix, chore, docs, refactor (필요시 팀 합의 후 추가)

스코프는 선택사항입니다. 필요하면 `feat(api): ...`, `feat(entity): ...`처럼 붙여도 되고,
없어도 통과합니다.

## 커밋 예시
```
feat(api): 상점 아이템 조회 API 추가
fix(entity): 연관관계 매핑 오류 수정
chore: gitignore 업데이트
```

## 자동으로 걸리는 검사
- 커밋 메시지가 위 타입 형식을 따르는지 (commit-msg 시점)
- 줄 끝 공백, 파일 끝 개행, 머지 충돌 마커 잔여, 5MB 넘는 파일 실수 커밋 (pre-commit 시점)
