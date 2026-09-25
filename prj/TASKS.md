# TASKS

ADR-0001 (Bootstrap 5.3), ADR-0002 (Radix Colors), ADR-0003 (반응형) 은 2026-09-25 승인됐다.

검증은 Playwright 로 폰(360) · 태블릿(820) · 데스크탑(1280) × 라이트/다크 6조합 × 7페이지를 실제 렌더해서 한다. 스크립트는 명암비와 가로 스크롤을 계산하고 스크린샷을 남긴다. **착수 전 358건이던 명암비 미달이 0건이 됐고, 가로 스크롤도 없다.**

## 선행

### T-0001 — 로컬 Ruby 빌드 환경 `DONE`

`.ruby-version` 이 가리키던 3.1 은 Homebrew 에서 2026-05-07 에 비활성화됐다 (upstream EOL). 그래서 **3.3 으로 올렸다** — `nro-site` 도 이미 3.3 으로 올린 전례가 있다. 로컬 `.ruby-version` 과 Actions 워크플로의 `ruby-version` 을 함께 3.3 으로 맞춰 둘이 갈라지지 않게 했다.

`Gemfile.lock` 을 추적 대상으로 돌리고 `x86_64-linux` 플랫폼을 추가했다. 프레임워크를 바꾸는 동안 "로컬에서 본 것 == 배포되는 것"이 성립해야 하기 때문이다.

`bundle exec jekyll build` 로컬 실행 확인함.

관련: OBS-20260925-05

## 이전 (ADR-0001, ADR-0002)

### T-0002 — Bootstrap 5.3 골격 교체 `DONE`

`_layouts/default.html` 의 CDN 링크를 BS 5.3 하나로 줄이고 AdminLTE·jQuery 를 뺀다. navbar / dropdown / tab 마크업을 `data-bs-*` 로 바꾸고, 테마 토글을 Bootstrap 공식 color-mode 스니펫으로 교체한다 (`localStorage` 키와 시스템/라이트/다크 3단 구조는 유지 — 지금 UX 는 문제없다).

AdminLTE·jQuery 제거했고 되돌리기용 `!important` 도 전부 사라졌다. `navbar-expand-md` 로 내려 T-0007 도 같이 처리했다.

측정으로 잡힌 함정 하나: navbar 가 `data-bs-theme="dark"` 를 달고 있어서, 토큰 블록을 `[data-bs-theme="dark"]` 로 쓰면 navbar 안에서 *역할만* 다크로 바뀌고 Radix 스케일(`.dark-theme`, html 에만 있음)은 라이트인 상태가 된다 → 흰 바탕에 흰 글씨. `:root[data-bs-theme="dark"]` 로 한정해서 해결.

### T-0003 — Radix 토큰 도입 `DONE`

`slate` / `blue` 의 라이트·다크 CSS 를 `assets/css/vendor/` 에 받아 커밋하고, `:root` 와 `[data-bs-theme="dark"]` 에서 Bootstrap 시맨틱 변수로 매핑한다 (매핑표는 ADR-0002).

다크 팔레트 블록 3개 제거했다. `light-dark()` 는 쓰지 않았다.

스케일 단계를 고르며 실제로 계산한 값들:

- 솔리드 강조색에 `blue-9` 를 쓰면 흰 글자 기준 **3.26** 이라 작은 글자에서 AA 미달이다. `blue-11` + 흰 글자 = **4.77** 로 바꿨다. 다크에서는 `blue-11` 이 밝은 색(`#70b8ff`)이라 글자를 `slate-1` 로 뒤집는다 (**8.97**).
- 기존 히어로 그라디언트의 밝은 끝 `#0080cc` 는 **흰 글자조차 4.23** 이었다. 손으로 고른 색이라 그렇다. `blue-12 → 브랜드 네이비` 로 바꿔 전 구간 10.78 이상이 됐다.
- navbar 의 `brand-edu` 강조색은 라이트(네이비 위) `blue-7` = 6.03, 다크(slate-2 위) `blue-11` = 8.37. 한 값으로는 양쪽을 못 맞췄다.

**이 계산들이 전부 필요 없어졌다.** Radix *Colors* 는 스케일만 주므로 "몇 번을 어디에"를 사람이 정해야 했고, 위 세 건 모두 그 판단이 틀렸던 사례다. `@radix-ui/themes` 의 `tokens.css` 로 옮기면서 `--accent-contrast` 가 그 판단을 대신한다. `main.css` 의 hex·rgba 는 0개가 됐다. ADR-0004 참고.

### T-0004 — 인라인 색상 제거 `DONE`

HTML 에 박힌 `style="color:…"` / `background` 90곳을 토큰이나 `text-body-secondary` 같은 유틸리티로 치환한다. 파일별 분포:

```
connect/  16   en/connect/  16
about/    11   en/about/    11
policy/    8   en/policy/    8
coverage/  6   en/coverage/  6
index      2   en/index      2
notices/   2   _layouts/notice.html  2
```

90곳 전부 치환했고 위 grep 은 비어 있다. 매핑은 `#444`/`#333` → `text-body`, `#888`/`#666`/`#bbb` → `text-body-secondary`, `var(--edu-light)`/`var(--edu-blue)` → `text-accent`, `#e67e22` → `text-warning-emphasis`, `background:#f8f9fa` → `bg-body-tertiary`.

`--edu-light` 와 `--edu-blue` 는 이전 과정에서 없어진 변수라, 그대로 뒀으면 아이콘 14곳이 색을 잃을 뻔했다.

관련: OBS-20260925-03

### T-0005 — `color-scheme` 을 테마별로 명시 `DONE`

지금은 `html { color-scheme: light dark }` 하나뿐이라, OS 가 다크인데 사용자가 라이트를 고르면 스크롤바·폼 컨트롤·기본 테두리가 다크로 남는다. `[data-bs-theme="light"] { color-scheme: light }` / `[data-bs-theme="dark"] { color-scheme: dark }` 를 넣는다.

## 반응형 (ADR-0003)

### T-0006 — 모바일 브레이크포인트와 한국어 줄바꿈 `DONE`

- `body { word-break: keep-all; overflow-wrap: anywhere; }` — 한국어 어절 중간 줄바꿈 제거. 체감 효과가 가장 큰 한 줄이다.
- `.section-card` 좌우 패딩을 sm 구간에서 32px → 16~20px.
- 히어로·섹션 제목 타이포 스케일을 좁은 폭에서 축소.
- 인증서 SHA-256 지문은 옥텟 단위로만 끊기게 (`:` 뒤 `<wbr>`), 복사 버튼 검토.
- 360px 가로 스크롤: 실제로 렌더해 보니 이전에도 지금도 재현되지 않는다. `.quick-cards` 의 `--bs-gutter-x` 는 음수 마진과 열 패딩이 같은 변수를 써서 자기일관적이었다 — 다른 세션의 진단은 코드 냄새는 맞았지만 오버플로의 원인은 아니었다. 진짜 문제는 미디어 쿼리가 0개였던 것(OBS-20260925-06)이고, 그건 해결됐다.
- 지문 복사 버튼은 하지 않았다. `<wbr>` 로 옥텟 단위 줄바꿈만 넣었고, 복사 버튼은 별도 작업으로 남긴다.

### T-0007 — navbar 접힘 구간 조정 `DONE`

`navbar-expand-lg` 라 992px 미만에서 햄버거로 접힌다. 태블릿 세로(820px)에 메뉴 5개를 펼칠 공간은 충분하다. `expand-md` 로 내리고 md 구간에서는 메뉴 아이콘을 숨긴다.

### T-0008 — 접속 안내 OS 탭을 모바일 우선으로 `TODO`

탭 가로 스크롤은 T-0002 에서 같이 넣었다. 남은 것은 User-Agent 로 현재 OS 탭을 기본 선택하는 부분.

폰에서 Windows 탭이 먼저 열린다. 탭을 가로 스크롤 또는 세그먼트 컨트롤로 바꾸고, User-Agent 로 현재 OS 탭을 기본 선택한다.

## 접근성

### T-0009 — 키보드·스크린리더 대응 `DONE`

- "빠른 도움말" `<li onclick=…>` 8곳(`index.html` 4, `en/index.html` 4)을 `<a>` 로. 지금은 탭 이동도 새 탭 열기도 안 된다.
- 공지 리스트 `li` 전체에 `cursor:pointer` 인데 실제 클릭 영역은 제목 텍스트뿐 — 영역을 일치시킨다.
- 햄버거 버튼과 언어 드롭다운에 `aria-label`.
- 🇰🇷🇺🇸 국기 이모지는 Windows 에서 "KR" / "US" 글자로 나온다. 텍스트 레이블로 교체.

네 항목 모두 처리했다. 드롭다운 토글은 `<a href="#">` 에서 `<button>` 으로 바꿨다 — 링크가 아니라 버튼이므로.

### T-0010 — 실기기 검증 `DONE`

삼성 단말에서 확인 완료 (2026-09-25, 사용자 확인). 정상으로 보인다.

이 개편의 출발점이던 증상이 해소됐다. 브라우저 버전은 기록하지 못했으므로, 나중에 같은 증상이 재발하면 버전부터 확인한다.

## 별도 트랙 (UI 아님)

### T-0011 — 영문 공지 처리 방식 결정 `DONE`

목록 페이지는 만들었다 — `/en/notices/` 404 는 해소됐고 영문 홈의 "View all" 도 살아났다. 목록의 제목은 `title_en` 으로 영문이 나온다.

**본문은 한국어로 유지한다** (2026-09-25 결정). 한국 기관 이용자를 위한 사이트이므로 공지 본문까지 이중화하지 않는다. ADR-0005 참고. `nro-site` 의 `_notices_en/` 방식은 채택하지 않는다.

관련: OBS-20260925-01, OBS-20260925-02

### T-0012 — 접속 안내 내용 오류 `TODO`

다른 세션이 짚은 것으로, 아직 내가 직접 확인하지 않았다. 확인 후 착수한다.

- Android 탭의 "CA 인증서: 시스템 인증서 사용" — 사설 CA 는 시스템 신뢰 저장소에 없으므로 검증에 실패한다. 다운로드한 CA 를 쓰도록 안내해야 한다.
- Windows 3단계 "인증 기관 sandbox.re.kr 선택" — realm 과 CA 이름이 섞였다.
- CAT 링크를 기관 IdP 딥링크로, 모바일은 geteduroam 앱 우선 안내 검토.

기관들이 fork 해서 그대로 복사하는 문서라 틀린 안내가 그대로 퍼진다. 우선순위가 낮지 않다.

## 새로 생긴 것

### T-0013 — 제목바(navbar) 브랜딩 확정 `DOING`

지금 navbar 는 Font Awesome wifi 아이콘 + "기관명 eduroam" 텍스트다. 저장소에 실제 eduroam 로고가 두 벌 들어 있는데(`assets/images/eduroam-logo.svg` 컬러, `eduroam-logo-white.png` 흰색) **아무 데서도 쓰이지 않는다.**

후보는 셋이고, 고르는 기준은 취향보다 **강조색 자유도**다. ADR-0006 참고.
