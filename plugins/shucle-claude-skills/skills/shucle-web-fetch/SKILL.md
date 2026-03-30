---
name: shucle-web-fetch
description: WebFetch 대안. GitHub URL은 gh api, 일반 웹은 curl로 콘텐츠를 가져와 AI 분석을 제공한다. WebFetch가 차단된 환경에서 사용.
---

# ShucleWebFetch Skill

WebFetch의 대안으로, 네트워크 정책상 WebFetch가 차단된 환경에서 `gh api`와 `curl`을 사용하여 웹 콘텐츠를 가져오고 AI 분석을 제공합니다.

## 개요

이 스킬은 WebFetch처럼 URL에서 콘텐츠를 가져와 사용자 프롬프트에 따라 분석하지만, 내부적으로는:
- **GitHub URL**: `gh api` 사용 (인증 자동, 효율적)
- **일반 웹 URL**: `curl` 사용 (범용 HTTP 클라이언트)
- **AI 분석**: 가져온 콘텐츠를 Claude가 직접 분석하여 프롬프트에 맞게 응답

## 사용법

```bash
/shucle-web-fetch <URL> <prompt>
```

또는 대화 중:
```
사용자: https://github.com/samber/lo/blob/master/slice.go에서 Map 함수 구현 보여줘
Claude: [ShucleWebFetch 스킬 자동 실행]
```

## 주요 기능

1. **자동 URL 타입 감지**
   - GitHub 저장소 URL
   - GitHub Raw URL
   - 일반 웹사이트 URL
   - API 엔드포인트

2. **최적화된 데이터 가져오기**
   - GitHub: `gh api`로 인증 자동 처리
   - 일반 웹: `curl`로 HTML/JSON 가져오기
   - Base64 디코딩 자동 처리

3. **AI 기반 분석**
   - 프롬프트에 따라 필요한 정보만 추출
   - 코드 설명, 요약, 비교 등 다양한 분석
   - 마크다운 형식으로 정리된 응답

4. **캐싱 (선택적)**
   - /tmp에 임시 캐시 저장
   - 동일 URL 재요청 시 빠른 응답

## 실행 프로세스

### 1단계: URL 타입 분석
```
입력 URL을 분석하여 다음 중 하나로 분류:
- GitHub 파일 URL (blob, tree, raw)
- GitHub API URL
- 일반 웹사이트 URL
```

### 2단계: 콘텐츠 가져오기

#### GitHub 파일의 경우
```bash
# URL 파싱
# https://github.com/OWNER/REPO/blob/BRANCH/PATH -> gh api 호출

# 방법 1: gh api (권장)
gh api repos/OWNER/REPO/contents/PATH?ref=BRANCH --jq '.content' | base64 -d

# 방법 2: raw URL (간단한 경우)
curl -s https://raw.githubusercontent.com/OWNER/REPO/BRANCH/PATH
```

#### 일반 웹사이트의 경우
```bash
# curl로 콘텐츠 가져오기
curl -s -L -A "Mozilla/5.0" URL

# HTML이 너무 크면 head로 제한
curl -s -L -A "Mozilla/5.0" URL | head -1000
```

### 3단계: AI 분석 및 응답
```
가져온 콘텐츠를 Claude가 분석:
- 사용자 프롬프트에 맞게 필터링
- 코드 설명, 구조 분석, 예시 추출 등
- 마크다운으로 정리된 응답 생성
```

## 자동 트리거 규칙

다음 패턴 감지 시 자동 실행:

1. **GitHub URL 언급**:
   - "https://github.com/..." 포함 시
   - "에서 코드 보여줘", "분석해줘" 등과 함께 사용

2. **API 문서 요청**:
   - "pkg.go.dev에서...", "문서 확인해줘" 등

3. **웹 페이지 분석 요청**:
   - URL + "요약해줘", "내용 보여줘" 등

## 제약 사항

1. **GitHub 인증**: `gh auth login` 사전 실행 필요
2. **Rate Limiting**: GitHub API 5,000 요청/시간 (인증 시)
3. **콘텐츠 크기**: 너무 큰 파일은 일부만 가져오기
4. **네트워크 정책**: curl도 차단될 수 있음
