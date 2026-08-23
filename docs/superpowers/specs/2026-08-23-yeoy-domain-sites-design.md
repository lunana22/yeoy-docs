# yeoy.cc 도메인 사이트 설계 (docs.yeoy.cc + yeoy.cc)

> 2026-08-23. 상태: 승인됨(채팅). 구현 계획은 writing-plans로 별도 작성.

## 목적

- **지금**: 앱스토어/플레이 심사에 필요한 신뢰 요소(개발자 사이트·개인정보처리방침·지원 연락처)를 내 도메인 아래로.
- **나중**: yeoy.cc를 포트폴리오·외주 컨택 사이트로 확장. 디자인에 투자.

## 결정: 두 사이트로 분리

| | yeoy.cc | docs.yeoy.cc |
|---|---|---|
| 역할 | 포트폴리오·외주 컨택·제품 쇼케이스 | 전 제품 개인정보처리방침·이용약관·지원 |
| 변경 빈도 | 자주(리디자인) | 거의 없음 |
| 구현 | 새 Next.js 앱(luna-init, next 모듈만) → Cloudflare Pages | 기존 `lunana22/subspocket` 정적 레포 그대로 + 커스텀 도메인 |

**분리 이유**: 스토어에 등록한 정책 URL은 앱 수명 내내 404가 나면 안 된다. 포트폴리오 리디자인/배포 사고와 배포 축을 분리해 그 위험을 0으로. 제품이 늘면 docs 아래 **폴더 추가**(서브도메인 추가 없음).

**기각한 안**: 한 Next.js 앱에 `yeoy.cc/docs/...` 경로 — 디자인 통일은 좋지만 위 위험을 떠안고 하루짜리 일이 며칠이 됨. 제품별 서브도메인(`subspocket.yeoy.cc`) — 제품마다 DNS·사이트가 늘어 관리 부담.

## 1. docs.yeoy.cc (기존 레포 재편)

현재: `lunana22/subspocket` → GitHub Pages(Actions workflow) → `lunana22.github.io/subspocket/`. 파일: `index.html`, `privacy{,-en,-ja}.{md,html}`, `presets.json`(앱이 fetch).

변경:
- 레포 이름 `subspocket` → `yeoy-docs` (GitHub가 옛 이름 리다이렉트). 로컬 `release/subspocket-site` → `release/yeoy-docs`.
- 제품별 폴더: `subspocket/privacy.html`, `subspocket/privacy-en.html`, `subspocket/privacy-ja.html` (+ 원본 md 동반 이동). 각 정책 페이지의 상대 링크·제목은 그대로.
- 루트 `index.html`: 제품 목록(현재 구독정리 하나 → ko/en/ja 정책 링크) + 지원 이메일. 기존 톤(단순 HTML, 다크모드 CSS) 유지.
- **호환 스텁**: 루트 `privacy.html`, `privacy-en.html`, `privacy-ja.html`에 `<meta http-equiv="refresh">` + 링크로 `/subspocket/...` 이동. 스토어 리스팅(`store-assets/listing/*.md`)에 적힌 옛 URL이 계속 살게.
- `presets.json`은 **루트에 그대로** (문서가 아니라 앱 자산). subscribeNote `apps/native/eas.json`의 `EXPO_PUBLIC_PRESETS_URL`을 `https://docs.yeoy.cc/presets.json`으로 교체(빌드 18에 포함). fetch 실패 시 번들 프리셋 폴백이 있어 전환 중 위험 없음.
- `CNAME` 파일(`docs.yeoy.cc`) 커밋 + Cloudflare DNS `docs` CNAME → `lunana22.github.io` (DNS only, 프록시 끔 — GitHub TLS 발급 위해). GitHub Pages 설정에서 커스텀 도메인 + HTTPS 강제.
- subscribeNote `store-assets/listing/*.md`, `docs/privacy-policy.md`의 URL을 새 주소로 갱신(ASC/Play 콘솔의 실제 값은 다음 `store-release` 때 반영).

검증: `curl -I https://docs.yeoy.cc/subspocket/privacy` 200, 옛 github.io 경로 3개가 최종적으로 새 페이지에 도착, `https://docs.yeoy.cc/presets.json` 200 + JSON.

## 2. yeoy.cc (새 사이트)

- `luna-init` → `release/yeoy-site`, **next 모듈만**(server/native/storybook 없음, variant 없음). DB 없음.
- v1 = 한 페이지, 섹션 3개:
  1. 소개 한 문단 (이름/무엇을 만드는지) + 이메일
  2. 제품 카드: 구독정리(스토어 링크, 정책 → docs.yeoy.cc), chodae(사이트 링크)
  3. 연락: `contact@yeoy.cc` mailto
  - 외주/포트폴리오는 v2 — 섹션 컴포넌트를 추가하는 구조로 두되 v1엔 만들지 않음(YAGNI).
- 디자인: DESIGN.md → 토큰 → HTML 시안 → 구현. 시안 단계에서 톤 2~3개 보여주고 선택.
- 배포: `web-deploy` 스킬 → Cloudflare Pages, 커스텀 도메인 `yeoy.cc`(+ `www` 리다이렉트).
- 메타: title/description/OG, `robots` 허용. 분석 도구 없음.

검증: 빌드 통과, Lighthouse 접근성·SEO 90+, 모바일 폭 확인, `https://yeoy.cc` 200.

## 3. 문서 정리

- `macmini-hosting/README.md` + 설계서: 우산 도메인 `yeoyeong.com` → `yeoy.cc` (`<svc>.yeoy.cc`). "도메인 등록 대기" 항목 완료 처리.
- 메모리 `macmini-hosting-project.md` 갱신.

## 순서

1 → 3 → 2. (1·3은 오늘 끝나는 작업, 2는 디자인 시간을 들임.)

## 전제 (확인됨 2026-08-23)

- yeoy.cc 네임서버 Cloudflare(aiden/addyson), Email Routing MX 설정됨, `contact@yeoy.cc` 수신됨.
- 루트·docs 레코드는 아직 없음.
