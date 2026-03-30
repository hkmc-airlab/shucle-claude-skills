# shucle-claude-skills

hkmc-airlab Shucle 팀의 Claude Code 공용 스킬 마켓플레이스입니다.

## 스킬 목록

| 스킬 | 설명 |
|---|---|
| `update-md` | BTS 이슈 기반 MD 레포 업데이트 자동화 |
| `shucle-web-fetch` | WebFetch 대안 (gh api / curl 사용) |

## 설치

```bash
# 1. 마켓플레이스 등록
claude plugin marketplace add hkmc-airlab/shucle-claude-skills

# 2. 플러그인 설치
claude plugin install shucle-claude-skills@shucle-claude-skills
```

## 사용법

설치 후 Claude Code에서:
- `/update-md` — BTS 이슈 MD 업데이트
- `/shucle-web-fetch` — 웹 콘텐츠 조회
