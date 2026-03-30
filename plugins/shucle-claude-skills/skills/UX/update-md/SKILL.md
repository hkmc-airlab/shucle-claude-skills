---
name: update-md
description: >
  hkmc-airlab MD 레포 업데이트 자동화 skill.
  신규 기획 반영 또는 BTS 이슈 대응 중 워크플로우를 선택하여 MD를 수정하고 PR을 생성한다.
  피그마 URL 또는 스크린샷을 함께 제공하면 화면 흐름과 UI 구성을 참조하여 더 정확한 MD를 작성한다.
  PR merge 후 BTS 이슈 자동 댓글은 GitHub Actions이 처리한다.
---

# MD 업데이트 Skill

## 개요

hkmc-airlab 내 MD 레포 파일을 수정하고 PR을 생성하는 표준 절차.
피그마 URL 또는 스크린샷을 함께 제공하면 화면 흐름과 UI 구성을 참조해 더 정확한 MD를 작성할 수 있다.

**PR 생성 전 반드시 사용자 확인을 받는다. 추가 수정이 있을 수 있으므로 확인 없이 PR을 올리지 않는다.**

## 워크플로우 선택

스킬 실행 시 가장 먼저 사용자에게 워크플로우를 묻는다:

> "신규 기획 반영인가요, BTS 이슈 대응인가요?"

- **신규 기획 반영**: 레포 이슈에 작성된 기획을 MD에 추가/반영하는 작업 → [신규 기획 워크플로우](#신규-기획-워크플로우) 진행
- **BTS 이슈 대응**: 버그/QA 이슈 기반으로 기존 MD를 수정하는 작업 → [BTS 이슈 워크플로우](#bts-이슈-워크플로우) 진행

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

---

## 신규 기획 워크플로우

### Step 1. 기획 이슈 파악

사용자가 제공한 레포 이슈 링크를 읽는다.

```bash
gh issue view [이슈번호] --repo hkmc-airlab/[레포] --comments
```

- 이슈 본문과 코멘트에서 기획 내용, 화면 흐름, 동작 조건 등을 파악한다.
- 불명확한 내용이 있으면 사용자에게 확인 후 진행한다.

### Step 2. 피그마/스크린샷 참조 (선택)

피그마 URL 또는 스크린샷이 제공된 경우 아래를 참조한다. → [Step 0 참조](#step-0-피그마스크린샷-참조-선택)

### Step 3. 관련 MD 파일 파악 및 수정안 제시

위 레포 구조 표를 참고해 MD 레포와 파일 경로를 결정한다.
기획 이슈와 피그마/스크린샷 분석을 종합하여 추가/수정할 MD 내용을 작성한다.

파일 다운로드:

```bash
gh api "repos/hkmc-airlab/[MD레포]/contents/[경로]/[파일명]?ref=[기본브랜치]" \
  --jq '.content' | base64 -d > /tmp/[파일명]
```

수정안을 사용자에게 보여주고 확인을 받는다.

**사용자 확인 없이 다음 단계로 진행하지 않는다.**

### Step 4. 브랜치 생성 → Step 5. 파일 push → Step 6. PR 생성

BTS 이슈 워크플로우의 Step 3~5와 동일하게 진행한다.
PR 본문에 기획 이슈를 `hkmc-airlab/[레포]#[번호]` 형식으로 포함한다.

---

## BTS 이슈 워크플로우

### Step 0. 피그마/스크린샷 참조 (선택)

사용자가 피그마 URL 또는 스크린샷을 제공한 경우 이슈 파악 전에 먼저 분석한다.
화면 흐름과 UI 구성을 파악하여 이후 MD 수정안 작성에 반영한다.

#### 피그마 URL 제공 시

URL에서 파일 키를 추출하여 Figma API로 조회한다.

```bash
# URL 형식: https://www.figma.com/design/{file_key}/... 또는 https://www.figma.com/file/{file_key}/...
# FIGMA_TOKEN 환경변수가 설정되어 있는 경우
curl -s -H "X-Figma-Token: $FIGMA_TOKEN" \
  "https://api.figma.com/v1/files/[file_key]" | jq '{name, lastModified, pages: .document.children[].name}'
```

- `FIGMA_TOKEN`이 없는 경우 사용자에게 요청하거나 스크린샷으로 대체한다.
- 조회 결과에서 페이지 구조, 화면 이름, 컴포넌트 흐름을 파악한다.

#### 스크린샷 제공 시

첨부된 이미지를 직접 분석하여 아래 항목을 파악한다:

- **화면 흐름**: 어떤 조건에서 어떤 화면으로 전환되는지
- **UI 구성**: 버튼, 팝업, 텍스트 등 구성 요소와 문구
- **상태별 동작**: 권한 허용/거부, 성공/실패 등 상태에 따른 UI 변화

#### 분석 결과 활용

파악한 내용을 Step 2의 MD 수정안 작성 시 반영한다.
이슈 내용과 피그마/스크린샷이 다를 경우 사용자에게 확인을 요청한다.

### Step 1. BTS 이슈 파악

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
