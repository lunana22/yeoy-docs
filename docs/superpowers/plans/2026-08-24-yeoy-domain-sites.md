# yeoy.cc 도메인 사이트 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 앱 심사용 문서 허브(docs.yeoy.cc)를 기존 정적 레포 재편으로 오늘 열고, 포트폴리오(yeoy.cc)를 새 Next.js 한 페이지로 띄운다.

**Architecture:** 두 사이트 분리 — docs는 기존 `lunana22/subspocket`(GitHub Pages, Actions workflow) 재편 + 커스텀 도메인, 포트폴리오는 luna-init(next 모듈만) → Cloudflare Pages. 정책 URL의 배포 축을 포트폴리오 리디자인과 분리한다.

**Tech Stack:** 정적 HTML(GitHub Pages Actions), Next.js(luna 템플릿), Cloudflare(DNS/Pages), gh CLI.

**Spec:** `docs/superpowers/specs/2026-08-23-yeoy-domain-sites-design.md` (이 레포)

## Global Constraints

- GitHub 원격은 `github-personal` SSH 별칭 사용 (`git@github-personal:lunana22/...`)
- 옛 URL 3개는 절대 죽으면 안 됨: `lunana22.github.io/subspocket/privacy{,-en,-ja}` (스토어 리스팅에 등록됨)
- `presets.json`은 루트 유지 — 경로/내용 변경 금지 (출시 Android 빌드가 fetch)
- 정책 문서 본문(법적 텍스트)은 한 글자도 수정하지 않는다 — 파일 이동만
- luna 프로젝트 spacing은 named 토큰만 (`px-md` 등, 숫자 클래스 금지)
- 커밋에 Claude 공동저자 트레일러 넣지 않기
- Actions workflow 기반 Pages라 CNAME 파일은 무시됨 — 커스텀 도메인은 repo 설정(API)으로 (스펙의 "CNAME 파일 커밋"은 이 방식으로 대체)

---

### Task 1: docs 레포 재편 (제품 폴더 + 스텁 + 새 index)

**Files:**
- 작업 디렉토리: `/Users/yy/Documents/project/release/subspocket-site`
- Move: `privacy{,-en,-ja}.{html,md}` → `subspocket/` (git mv)
- Create: `privacy.html`, `privacy-en.html`, `privacy-ja.html` (루트, 리다이렉트 스텁)
- Modify: `index.html`, `index.md`
- 그대로: `presets.json`, `.github/workflows/`, `.nojekyll`

**Interfaces:**
- Produces: 최종 URL 체계 `docs.yeoy.cc/subspocket/privacy{,-en,-ja}` — Task 2~5가 이 경로를 전제

- [ ] **Step 1: 파일 이동**

```bash
cd /Users/yy/Documents/project/release/subspocket-site
mkdir -p subspocket
git mv privacy.html privacy.md privacy-en.html privacy-en.md privacy-ja.html privacy-ja.md subspocket/
```

- [ ] **Step 2: 루트 스텁 3개 생성** — 파일당 아래 내용(경로·언어만 치환: privacy→ko `한국어`, privacy-en→en `English`, privacy-ja→ja `日本語`)

```html
<!-- privacy.html -->
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta http-equiv="refresh" content="0; url=./subspocket/privacy">
<link rel="canonical" href="https://docs.yeoy.cc/subspocket/privacy">
<title>이동됨 · Moved</title>
</head>
<body>
<p>페이지가 이동했습니다: <a href="./subspocket/privacy">docs.yeoy.cc/subspocket/privacy</a></p>
</body>
</html>
```

`privacy-en.html`: `url=./subspocket/privacy-en`, canonical `.../subspocket/privacy-en`, 본문 "This page has moved:".
`privacy-ja.html`: `url=./subspocket/privacy-ja`, canonical `.../subspocket/privacy-ja`, 본문 "ページが移動しました:".

- [ ] **Step 3: index.html 교체** — 기존 스타일 블록(다크모드 포함)은 그대로 복사하고 본문만 아래로

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>yeoy docs — 제품 문서 · Product Docs</title>
<style> [기존 index.html의 <style> 블록 그대로] </style>
</head>
<body>
<h1>yeoy docs</h1>
<p>yeoy 제품들의 개인정보처리방침·지원 문서 모음입니다. Policies and support docs for yeoy products.</p>
<h2>구독정리 · Subs Pocket</h2>
<p>로그인 없이 가볍게 쓰는 구독 관리 앱 — 이번 달 구독비와 다가오는 결제일을 한눈에.</p>
<ul>
<li>개인정보처리방침: <a href="./subspocket/privacy">한국어</a> · <a href="./subspocket/privacy-en">English</a> · <a href="./subspocket/privacy-ja">日本語</a></li>
</ul>
<h2>지원 · Support</h2>
<p>문의사항이나 버그 제보는 이메일로 보내주세요. For questions or bug reports, contact:</p>
<ul>
<li><strong>Email</strong>: contact@yeoy.cc</li>
</ul>
</body>
</html>
```

`index.md`는 같은 내용의 md 버전으로 갱신(소스 역할 유지).

- [ ] **Step 4: 로컬 검증** — 링크 무결성

```bash
cd /Users/yy/Documents/project/release/subspocket-site
ls subspocket/ | sort   # 6개 파일 확인
grep -o 'href="[^"]*"' index.html          # ./subspocket/privacy* 3개 + mailto 없음 확인
grep -l 'http-equiv="refresh"' privacy.html privacy-en.html privacy-ja.html
```

기대: subspocket/에 html 3 + md 3, 스텁 3개에 refresh 존재.

- [ ] **Step 5: 커밋 & 푸시 & 배포 확인**

```bash
git add -A && git commit -m "refactor: 제품별 폴더 구조로 재편 (subspocket/) + 옛 경로 스텁 + 허브 index"
git push origin main
gh run watch --repo lunana22/subspocket --exit-status $(gh run list --repo lunana22/subspocket -L1 --json databaseId -q '.[0].databaseId')
curl -s -o /dev/null -w '%{http_code}\n' https://lunana22.github.io/subspocket/subspocket/privacy   # 200
curl -s https://lunana22.github.io/subspocket/privacy | grep -c refresh                              # ≥1
curl -s -o /dev/null -w '%{http_code}\n' https://lunana22.github.io/subspocket/presets.json          # 200
```

---

### Task 2: 레포·로컬 디렉토리 이름 변경 (subspocket → yeoy-docs)

**Files:**
- GitHub: `lunana22/subspocket` → `lunana22/yeoy-docs`
- Local: `release/subspocket-site` → `release/yeoy-docs` (**move-project 스킬 사용** — 그냥 mv 하면 Claude 세션/메모리 경로가 깨진다)

**Interfaces:**
- Produces: 원격 `git@github-personal:lunana22/yeoy-docs.git`, Pages URL `lunana22.github.io/yeoy-docs/` (옛 URL은 GitHub가 301)

- [ ] **Step 1: GitHub 레포 이름 변경**

```bash
gh repo rename yeoy-docs -R lunana22/subspocket --yes
git -C /Users/yy/Documents/project/release/subspocket-site remote set-url origin git@github-personal:lunana22/yeoy-docs.git
git -C /Users/yy/Documents/project/release/subspocket-site fetch origin   # 인증·리다이렉트 확인
```

- [ ] **Step 2: 옛 github.io URL 리다이렉트 확인**

```bash
curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' https://lunana22.github.io/subspocket/privacy
```

기대: `301 https://lunana22.github.io/yeoy-docs/privacy` (그 끝은 스텁 → subspocket/privacy).

- [ ] **Step 3: 로컬 디렉토리 이동** — `move-project` 스킬 호출로 `release/subspocket-site` → `release/yeoy-docs`

---

### Task 3: 커스텀 도메인 docs.yeoy.cc 연결

**Files:** 없음 (Cloudflare DNS + GitHub Pages 설정)

**Interfaces:**
- Consumes: Task 2의 레포 이름 `yeoy-docs`
- Produces: `https://docs.yeoy.cc/*` — Task 4·5·7이 이 도메인을 URL에 사용

- [ ] **Step 1: Cloudflare DNS 레코드** — wrangler 미인증 상태라 둘 중 하나:
  - (a) 사용자에게 `! wrangler login` 요청 후 API로 생성, 또는
  - (b) 사용자가 대시보드에서 직접: yeoy.cc zone → DNS → `CNAME docs → lunana22.github.io`, **Proxy 끔(DNS only)** — GitHub TLS 발급 때문
  - 확인: `dig +short docs.yeoy.cc` → `lunana22.github.io.`

- [ ] **Step 2: GitHub Pages 커스텀 도메인 설정 (API)**

```bash
gh api -X PUT repos/lunana22/yeoy-docs/pages -f cname=docs.yeoy.cc
sleep 60
gh api repos/lunana22/yeoy-docs/pages -q '{status: .status, cname: .cname, https: .https_enforced, state: .protected_domain_state}'
```

인증서 발급이 몇 분 걸릴 수 있음 — `https_enforced`가 false면 발급 완료 후:

```bash
gh api -X PUT repos/lunana22/yeoy-docs/pages -F https_enforced=true
```

- [ ] **Step 3: end-to-end 검증**

```bash
for p in "" subspocket/privacy subspocket/privacy-en subspocket/privacy-ja presets.json; do
  curl -s -o /dev/null -w "%{http_code} https://docs.yeoy.cc/$p\n" "https://docs.yeoy.cc/$p"
done   # 전부 200
curl -sL https://lunana22.github.io/subspocket/privacy -o /dev/null -w '%{url_effective}\n'  # 최종 docs.yeoy.cc 도착 확인
```

---

### Task 4: subscribeNote 쪽 URL 갱신

**Files:**
- Modify: `release/subscribeNote/apps/native/eas.json` (production env `EXPO_PUBLIC_PRESETS_URL`)
- Modify: `release/subscribeNote/store-assets/listing/{ko,en,ja}.md` (Privacy/Support URL 2줄씩)
- Modify: `release/subscribeNote/docs/privacy-policy.md` (URL 표 + 지원 페이지)

**Interfaces:**
- Consumes: Task 3의 `https://docs.yeoy.cc/*`
- Produces: 다음 `store-release`/빌드 18에 새 URL 반영

- [ ] **Step 1: 값 교체**

| 파일 | 옛 값 | 새 값 |
|---|---|---|
| eas.json | `https://lunana22.github.io/subspocket/presets.json` | `https://docs.yeoy.cc/presets.json` |
| listing/ko.md | `.../subspocket/privacy` | `https://docs.yeoy.cc/subspocket/privacy` |
| listing/en.md | `.../subspocket/privacy-en` | `https://docs.yeoy.cc/subspocket/privacy-en` |
| listing/ja.md | `.../subspocket/privacy-ja` | `https://docs.yeoy.cc/subspocket/privacy-ja` |
| listing/*.md Support | `https://lunana22.github.io/subspocket/` | `https://docs.yeoy.cc/` |
| docs/privacy-policy.md | 표의 URL 3개 + 지원 페이지 | 위와 동일 체계 |

- [ ] **Step 2: 검증 & 커밋**

```bash
cd /Users/yy/Documents/project/release/subscribeNote
grep -rn "lunana22.github.io" apps/native/eas.json store-assets/listing docs/privacy-policy.md   # 0건
for u in $(grep -rho 'https://docs.yeoy.cc[^ )"]*' apps/native/eas.json store-assets/listing docs/privacy-policy.md | sort -u); do
  curl -s -o /dev/null -w "%{http_code} $u\n" "$u"; done   # 전부 200
git add apps/native/eas.json store-assets/listing docs/privacy-policy.md
git commit -m "chore: 정책·프리셋 URL을 docs.yeoy.cc로 교체"
```

(ASC/Play 콘솔의 실제 값 교체는 다음 store-release 때 — 이 계획 범위 밖.)

---

### Task 5: macmini-hosting 우산 도메인 문서 갱신

**Files:**
- Modify: `macmini-hosting/README.md` (`yeoyeong.com` → `yeoy.cc`, 진행 상태 체크)
- Modify: `macmini-hosting/docs/2026-07-27-macmini-public-hosting-design.md` (동일 치환)
- Modify: 메모리 `macmini-hosting-project.md` ("도메인 yeoyeong.com 등록 대기" → yeoy.cc 확보됨)

- [ ] **Step 1: 치환 & 확인**

```bash
cd /Users/yy/Documents/project/macmini-hosting
grep -rn "yeoyeong" README.md docs/ | wc -l    # 치환 전 개수 파악
# sed로 yeoyeong.com → yeoy.cc 치환 후:
grep -rn "yeoyeong" README.md docs/            # 0건
```

README 진행 상태의 "[ ] 도메인 `yeoyeong.com` 등록"을 "[x] 도메인 `yeoy.cc` 확보 (2026-08-24, docs.yeoy.cc 가동)"으로.

- [ ] **Step 2: 커밋** (git 레포면) + 메모리 파일 갱신 + MEMORY.md 한 줄 갱신

---

### Task 6: yeoy-site 프로젝트 생성 + DESIGN.md/토큰

**Files:**
- Create: `release/yeoy-site/` — **luna-init 스킬** 호출, **next 모듈만** (server/native/storybook 없음, variant 없음, DB 없음)
- Modify: `release/yeoy-site/DESIGN.md` → 토큰 채우기 (**design-md-tokens 스킬** 경로)

**Interfaces:**
- Produces: `apps/web` Next.js 앱 — Task 7·8의 작업 대상

- [ ] **Step 1: luna-init 스킬로 생성** (이름 `yeoy-site`, 위치 `release/`)
- [ ] **Step 2: `pnpm install && pnpm --filter web build` 통과 확인**
- [ ] **Step 3: 초기 커밋** (`git init` + 원격 `git@github-personal:lunana22/yeoy-site.git` 생성·푸시)

---

### Task 7: 시안 (톤 2~3개) → 사용자 선택 → 구현

**Files:**
- Create: 시안 HTML 2~3개 (`docs/design-refs/` 또는 scratchpad)
- Modify: `apps/web` 랜딩 페이지 (선택된 시안 기준)

**Interfaces:**
- Consumes: Task 6의 앱 골격 + 디자인 토큰
- Produces: 한 페이지 — 섹션: ① 소개 한 문단+이메일 ② 제품 카드(구독정리: App Store/Play 링크 + `https://docs.yeoy.cc/subspocket/privacy`; chodae: 서비스 링크) ③ 연락 `mailto:contact@yeoy.cc`

- [ ] **Step 1: 정적 HTML 시안 2~3개 작성** (디자인 워크플로: DESIGN.md→토큰→HTML 시안) — 톤 예: 미니멀 타이포 중심 / 카드·컬러 중심
- [ ] **Step 2: 🛑 CHECKPOINT — 사용자가 톤 선택** (로컬 파일 경로로 안내; 아티팩트는 물어보고만)
- [ ] **Step 3: 선택 시안대로 구현** — 섹션 3개를 각각 컴포넌트로(나중에 외주 섹션 추가가 컴포넌트 하나 추가가 되게). spacing은 named 토큰만
- [ ] **Step 4: 검증** — `pnpm --filter web build` 통과, 모바일 폭(375px) 확인, title/description/OG 메타 존재
- [ ] **Step 5: 커밋**

---

### Task 8: yeoy.cc 배포 (Cloudflare Pages)

**Files:** 없음 (web-deploy 스킬 + Cloudflare 설정)

**Interfaces:**
- Consumes: Task 7의 빌드 통과한 `apps/web`
- Produces: `https://yeoy.cc` 라이브

- [ ] **Step 1: web-deploy 스킬로 Cloudflare Pages 셋업·배포** (wrangler 로그인 필요 시 사용자에게 `! wrangler login`)
- [ ] **Step 2: 커스텀 도메인 `yeoy.cc` + `www.yeoy.cc` 리다이렉트 연결** (Pages 대시보드/스킬 경로)
- [ ] **Step 3: 검증**

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://yeoy.cc          # 200
curl -s -o /dev/null -w '%{http_code} %{redirect_url}\n' https://www.yeoy.cc   # 30x → yeoy.cc
curl -s https://yeoy.cc | grep -o '<title>[^<]*</title>'
```

- [ ] **Step 4: 마무리** — 각 레포 `docs/CONTEXT.md` 갱신(컨벤션), 메모리에 yeoy-site 프로젝트 한 줄 추가, work-log
