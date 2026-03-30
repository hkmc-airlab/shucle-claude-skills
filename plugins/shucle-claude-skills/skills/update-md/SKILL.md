---
name: update-md
description: >
  hkmc-airlab MD 레포 업데이트 자동화 skill.
  BTS 이슈 링크를 받아 관련 MD 수정안을 제시하고, 사용자 확인 후 브랜치 생성 → PR 생성까지 수행한다.
  PR merge 후 BTS 이슈 자동 댓글은 GitHub Actions이 처리한다.
---

# MD 업데이트 Skill

## 개요

BTS 이슈 기반으로 hkmc-airlab 내 MD 레포 파일을 수정하고 PR을 생성하는 표준 절차.

**PR 생성 전 반드시 사용자 확인을 받는다. 추가 수정이 있을 수 있으므로 확인 없이 PR을 올리지 않는다.**

## 레포 구조

| BTS 이슈 레포 | MD 수정 레포 | MD 경로 | 기본 브랜치 |
|---|---|---|---|
| `shucle-bts-driver` | `shucle-DriverVehicle-product` | `Driver App/` | master |
| `shucle-rider` | `shucle-rider` | `01_라이더앱/`, `02_화이트라벨/` 등 | master |
| `shucle-taxidriver-product` | `shucle-taxidriver-product` | `Voucher Taxi Driver App/` | master |
| `shucle-CallAgent-product` | `shucle-CallAgent-product` | 루트 또는 하위 디렉토리 | master |
| `shucle-kiosk-product` | `shucle-kiosk-product` | `01_키오스크웹앱/` | master |
| `shucle-operation-tool-doc` | `shucle-operation-tool-doc` | `DRT 운영/`, `공통/` 등 | main |

> 이슈 레포와 MD 레포가 같은 경우(예: shucle-rider), 같은 레포 내에서 브랜치 생성 후 PR 작성.

## 절차

### Step 1. 이슈 파악

사용자가 BTS 이슈 링크를 제공하면 각 이슈 내용을 읽는다.

```bash
gh issue view [이슈번호] --repo hkmc-airlab/[이슈레포] --comments
```

- 이슈 내용 중 MD 수정 방향 파악 (담당자 코멘트 기준)
- 다국어 key는 사용자가 알려주지 않으면 반드시 확인 후 진행

### Step 2. 관련 MD 파일 파악 및 수정안 제시

위 레포 구조 표를 참고해 MD 레포와 파일 경로를 결정한다.
모르는 경우 레포 디렉토리 구조를 확인한다.

```bash
gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]" --jq '.[].name'
```

파일 다운로드:

```bash
gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/[파일명]?ref=[기본브랜치]" \
  --jq '.content' | base64 -d > /tmp/[파일명]
```

수정안을 사용자에게 보여주고 확인을 받는다.
추가 수정 요청이 있으면 반영한 후 다시 확인을 받는다.

**사용자 확인 없이 다음 단계로 진행하지 않는다.**

### Step 3. 브랜치 생성

사용자가 수정안을 확인한 후 진행한다.
브랜치 네이밍: `M/DD-[버전]-[내용]` (날짜 기반, 예: `3/23-4.10-update`)

```bash
SHA=$(gh api "repos/hkmc-airlab/[MD레포]/branches/[기본브랜치]" --jq '.commit.sha')

gh api "repos/hkmc-airlab/[MD레포]/git/refs" \
  --method POST \
  --field ref="refs/heads/[브랜치명]" \
  --field sha="$SHA"
```

### Step 4. 파일 push

파일명에 공백이 있으면 URL 인코딩 필요 (예: `07. 운행종료.md` → `07.%20운행종료.md`)

#### MD 파일 push

```bash
FILE_SHA=$(gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/[파일명]?ref=[브랜치]" --jq '.sha')

CONTENT=$(base64 -i /tmp/[파일명] | tr -d '\n')
gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/[파일명]" \
  --method PUT \
  --field message="[커밋 메시지]" \
  --field content="$CONTENT" \
  --field sha="$FILE_SHA" \
  --field branch="[브랜치명]"
```

#### 이미지 파일 push

이미지 수정/추가가 필요한 경우 동일한 방식으로 push한다. (PNG, JPG 등 바이너리 파일 모두 가능)

- **새 이미지 추가**: SHA 없이 PUT 요청
- **기존 이미지 교체**: 기존 파일 SHA 조회 후 PUT 요청 (파일명이 같으면 MD 링크 유지됨)
- 이미지는 보통 `images/` 디렉토리에 위치

```bash
# 새 이미지 추가
CONTENT=$(base64 -i /tmp/[이미지파일명] | tr -d '\n')
gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/images/[이미지파일명]" \
  --method PUT \
  --field message="[커밋 메시지]" \
  --field content="$CONTENT" \
  --field branch="[브랜치명]"

# 기존 이미지 교체
IMG_SHA=$(gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/images/[이미지파일명]?ref=[브랜치]" --jq '.sha')
CONTENT=$(base64 -i /tmp/[이미지파일명] | tr -d '\n')
gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/images/[이미지파일명]" \
  --method PUT \
  --field message="[커밋 메시지]" \
  --field content="$CONTENT" \
  --field sha="$IMG_SHA" \
  --field branch="[브랜치명]"
```

### Step 5. PR 생성

**중요**: PR 본문에 BTS 이슈를 반드시 `hkmc-airlab/[이슈레포]#[번호]` 형식으로 포함해야 GitHub Actions이 자동 댓글을 달 수 있다.

```bash
gh pr create \
  --repo hkmc-airlab/[MD레포] \
  --head "[브랜치명]" \
  --base "[기본브랜치]" \
  --title "[제목]" \
  --body "$(cat <<'EOF'
## 변경 내용

### 1. [변경 내용 제목]
- **파일**: `[경로]/[파일명]`
- **관련 BTS**: hkmc-airlab/[이슈레포]#[번호]

[변경 설명]
EOF
)"
```

## PR merge 후 자동 댓글

PR이 기본 브랜치(master/main)에 merge되면 GitHub Actions(`bts_comment_on_merge.yml`)이 자동으로 BTS 이슈에 댓글을 달아준다.

- 댓글 형식: `- md 업데이트 완료 ([PR URL])`
- PR 본문의 `hkmc-airlab/[레포]#[번호]` 패턴을 모두 파싱하여 처리
- 댓글은 **`md-edit-bot`** (GitHub App, App ID: 3189527) 프로필로 달림
- 별도 작업 불필요

### 봇 설정 정보

- **GitHub App**: `md-edit-bot` (App ID: `3189527`)
- **시크릿**: 6개 MD 레포 각각에 `BOT_PRIVATE_KEY` 등록됨
- 새 MD 레포 추가 시: `BOT_PRIVATE_KEY` 시크릿 등록 + `bts_comment_on_merge.yml` 추가 필요
