# site-template-2 — 작업 규약

eduroamKR NRO 사이트 개편 작업 저장소. Jekyll 정적 사이트이고, `main` 에 push 하면 GitHub Actions 가 빌드해 GitHub Pages 로 올린다. 사이트가 무엇인지는 [README.md](README.md) 에 있다. 이 파일은 **어떻게 작업할지**만 다룬다.

## 이 저장소의 위치

세 저장소의 관계를 먼저 알고 시작한다.

- **`eduroam-kr/site-template`** — 원본. 기관이 fork 해서 자기 eduroam 안내 사이트로 쓰라고 만든 템플릿. 이 저장소는 그 사본에서 출발했다.
- **`eduroam-kr/nro-site`** — 같은 템플릿에서 갈라져 나간 NRO 판. 인턴이 이어 작업했다. 검색 자동완성, eduroam 맵(Leaflet), 기관 목록 `_data/*.json`, breadcrumb, `_notices_en/` 분리, vendor 정적 자산 커밋 등이 여기에 있다.
- **이 저장소 (`site-template-2`)** — 개편 작업본. 개발 도메인은 `eduroam-site-dev.kreonet.net`.

`nro-site` 에서 가져올 것과 버릴 것을 고르는 게 개편의 큰 축이다. 가져올 때는 통째로 복사하지 말고 무엇을 왜 가져왔는지 커밋 하나로 남긴다.

**아직 정해지지 않은 것**: 이 저장소가 계속 "기관이 fork 하는 템플릿" 성격을 유지하는지, 아니면 NRO 전용 사이트로 갈라서는지. 둘은 요구가 다르다 — 템플릿이면 모든 기관별 값이 `_config.yml` 한 곳에 모여 있어야 하고, NRO 전용이면 그 제약이 필요 없다. 여기에 영향받는 변경을 하기 전에 `prj/ADR.md` 에 결정을 먼저 적는다.

## 프로젝트 운영

`prj/` 5파일 시스템으로 운영한다. 규칙 전문은 [prj/PROCESS.md](prj/PROCESS.md) 에 있고, 요약하면:

- [prj/PRD.md](prj/PRD.md) — 무엇을 왜, 성공 기준.
- [prj/TASKS.md](prj/TASKS.md) — `T-0001` 번호가 붙은 실행 작업. 상태는 `TODO` / `DOING` / `BLOCKED` / `DONE` / `DROP`.
- [prj/ADR.md](prj/ADR.md) — `ADR-0001` 번호가 붙은 아키텍처 결정. 상태는 `Proposed` / `Accepted` / `Superseded`.
- [prj/NOTES.md](prj/NOTES.md) — 어디에도 안 들어가는 관찰. `OBS-YYYYMMDD-NN`.

작업을 시작하기 전에 `prj/TASKS.md` 를 보고, 되돌리기 어려운 구조 결정을 했으면 `prj/ADR.md` 에 남긴다. 결정을 코드에만 남기지 않는다 — 몇 주 뒤에 같은 논쟁을 다시 하게 된다.

## 커밋

[prj/PROCESS.md](prj/PROCESS.md) 의 Git 로그 규칙을 따른다. 그중 실제로 자주 어기게 되는 것들:

- **한 줄.** 제목만, 본문 없음. 이유가 길면 `prj/` 나 `docs/` 로 간다.
- **주제별로 쪼갠다.** 세션 하나에 커밋 하나가 아니다. 테마 수정과 공지 페이지 추가와 문서 갱신은 같은 파일을 건드려도 세 커밋이다.
- 제목에 "무슨 파일을 바꿨는지"가 아니라 "왜 바꿨는지"를 쓴다.
- 어느 커밋도 그 시점에 존재하지 않는 경로를 참조하지 않는다.
- `Co-Authored-By: Claude <noreply@anthropic.com>` 는 붙이고, `Claude-Session` 링크는 붙이지 않는다 (public 저장소).

## 한국어 / 영어 이중 트리

가장 자주 깨지는 부분이니 먼저 읽는다.

- **경로가 언어를 정한다.** `_config.yml` 의 `defaults:` 가 `/` 아래는 `lang: ko`, `/en` 아래는 `lang: en` 을 자동으로 넣는다. 페이지 front matter 에 `lang` 을 직접 쓰지 않는다.
- **문자열은 `site.t[page.lang]` 에서 꺼낸다.** 레이아웃에서 `{% assign t = site.t[page.lang] %}` 로 잡아 `{{ t.nav_about }}` 처럼 쓴다. 내비게이션·푸터 문구를 페이지에 하드코딩하지 않는다. 새 UI 문자열이 필요하면 `_config.yml` 의 `t.ko` 와 `t.en` **양쪽에** 키를 추가한다 — 한쪽만 넣으면 다른 언어에서 빈 문자열이 렌더된다.
- **언어 전환은 URL 문자열 조작이다.** `_layouts/default.html` 이 `/en` 접두사를 붙이거나 떼서 상대 URL 을 만든다. 따라서 **ko 트리와 en 트리의 경로가 1:1 로 대응해야 한다.** 한쪽에만 페이지를 만들면 그 페이지의 언어 전환 버튼이 404 로 간다. 페이지를 추가할 때는 `x/index.html` 과 `en/x/index.html` 을 같은 커밋에서 만든다.
- **양쪽을 고치거나 둘 다 안 고친다.** 한국어 페이지만 손보고 영문을 나중으로 미루면 그 "나중"은 오지 않는다.

## 테마 (라이트 / 다크)

Bootstrap 5.3 의 color modes + Radix **Themes** 토큰. 결정 배경은 [prj/ADR.md](prj/ADR.md) 의 ADR-0001, ADR-0004.

**`assets/css/main.css` 에 hex 나 rgba 가 하나도 없다. 이 상태를 유지한다.** 색이 필요하면 `assets/css/vendor/radix-themes-tokens.css` 의 토큰을 참조한다. 그 파일은 업스트림 그대로이고 손으로 고치지 않는다 (재생성 명령이 파일 머리에 있다).

자주 쓰는 토큰:

| 토큰 | 뜻 |
|---|---|
| `--color-background` | 페이지 배경 |
| `--color-panel-solid` | 카드·패널 면 |
| `--accent-9` | 솔리드 강조 면 |
| `--accent-contrast` | **그 면 위에 올려도 되는 글자색** |
| `--accent-11` | 배경 위 강조 텍스트 |
| `--gray-12` / `--gray-11` | 본문 / 보조 텍스트 |
| `--gray-6` / `--gray-4` | 테두리 / 옅은 구분선 |
| `--shadow-1..6`, `--radius-1..6` | 그림자, 모서리 |

`main.css` 의 `:root` 블록은 이 토큰들에 역할 이름을 붙인 것뿐이다 (`--surface-card: var(--color-panel-solid)`). 다크용 재정의는 `color-scheme` 외에 거의 없다 — Radix 가 `.dark-theme` 에서 이미 전부 바꾼다.

지켜야 할 것들:

- **솔리드 면 위 글자색을 직접 고르지 않는다.** `--accent-9` 배경에는 `--accent-contrast` 를 쓴다. 이 쌍만 보장된다. 배경을 `--accent-10` 으로 옮기는 순간 보장이 깨진다 — 실제로 다크에서 4.28 이 나왔다.
- **강조색을 바꾸려면 `_config.yml` 의 `theme.accent` 를 바꾼다.** CSS 를 열지 않는다. 26종 중에서 고른다. `theme.gray` 는 gray / mauve / olive / sage / sand / slate.
- **테마 적용은 두 가지를 같이 건다.** `<html>` 의 `data-bs-theme` 속성(Bootstrap 컴포넌트용)과 `.dark-theme` 클래스(Radix 토큰용). `_layouts/default.html` 의 `<head>` 인라인 스크립트와 본문 끝 토글 스크립트가 같은 규칙으로 처리한다. 한쪽만 바꾸면 테마가 반쪽만 바뀐다. 인라인 스크립트는 첫 페인트 전에 돌아야 하므로 `<head>` 밖으로 내리지 않는다. `<html>` 의 `radix-themes` 클래스도 지운다 — 시맨틱 토큰이 `:where(.radix-themes)` 아래 있어서, 없으면 `--color-background` 부터 사라진다.
- **`light-dark()` 를 쓰지 않는다.** Chromium 123 / Safari 17.5 이상이라 구버전 Samsung Internet 에서 동작하지 않는다. 이 개편의 출발점이 삼성 단말 다크 이슈인데 정작 삼성 브라우저에서 안 먹는 문법을 쓰면 본말전도다.
- **HTML 에 색을 인라인으로 박지 않는다.** 한 번 90곳을 걷어냈다. 색이 필요하면 `text-body` / `text-body-secondary` / `text-accent` 를 쓴다.
- **그라디언트를 새로 만들지 않는다.** 자동 검사가 그라디언트 위 텍스트의 명암비를 계산하지 못해서 사각지대가 된다. 지금은 사각지대가 0종이다.

**검증은 렌더해서 한다.** Playwright 로 폰(360) · 태블릿(820) · 데스크탑(1280) × 라이트/다크 × 7페이지를 돌며 계산된 색으로 명암비와 가로 스크롤을 잰다. 색을 건드렸으면 돌리고 0건인지 본다. 짐작하지 않는다.

## 반응형

`main.css` 에 한때 화면 폭 미디어 쿼리가 하나도 없었다. 그리드 클래스는 열 배치만 바꾸므로 여백과 타이포는 아무도 안 보고 있었다. 지금은 `max-width: 767.98px` 구간에서 패딩과 폰트를 줄인다.

- **`word-break: keep-all` 을 지운다면 이유를 적는다.** 한국어에서 이게 없으면 "프로파/일", "비/밀번호" 처럼 어절 중간에서 끊긴다. 좁은 화면 완성도 차이가 가장 큰 한 줄이다.
- 인증서 지문처럼 끊기면 안 되는 문자열은 `.fingerprint` 를 쓰고 `<wbr>` 로 끊을 자리를 지정한다 (`| replace: ":", ":<wbr>"`). `word-break: break-all` 을 쓰지 않는다 — 옥텟 한가운데서 끊긴다.

## 사이트 설정은 `_config.yml` 한 곳에

기관명, realm, 연락처, 인증서 지문, UI 문자열 — 바뀔 수 있는 값은 전부 `_config.yml` 에 있고 페이지는 거기서 읽는다. 페이지에 값을 직접 쓰면 fork 한 기관이 그 값을 찾지 못한다.

**`url:` 은 CNAME 과 같게 유지한다.** 도메인을 바꿀 때 `CNAME` 과 `_config.yml` 의 `url:` 을 같은 커밋에서 바꾼다. 에셋 링크는 전부 `relative_url` 이라 예전처럼 CSS 가 통째로 다른 도메인에서 로드되는 일은 없지만, `url:` 은 sitemap·feed·SEO 태그의 절대 주소를 만든다.

## eduroam 참여기관 메타데이터 (`general/institution.xml`)

이건 사람이 읽는 페이지가 아니라 **기계가 harvest 하는 데이터**다. 깨져도 사이트는 멀쩡해 보이므로 눈으로 알아채지 못한다.

- eduroam database specification **v2.0.1** 을 따른다. 스펙에 없는 필드를 추가하지 않는다.
- `ROid` 는 대한민국 기관 전부 `kr01` 고정이다.
- `lang` 속성이 있는 필드(`inst_name`, `address`, `info_URL`, `policy_URL`, `loc_name`)는 **`en` 항목이 반드시 하나 있어야 한다.** 한국어만 있으면 검증에서 떨어진다.
- `ts` 는 마지막 수정 시각(ISO 8601 UTC). 내용을 고쳤으면 같이 고친다.
- 고쳤으면 push 전에 검증한다. 눈으로 보고 넘어가지 않는다.

  ```
  https://monitor.eduroam.org/eduroam-database/v2/scripts/xml_validation_test.php?url=<서빙 URL>
  ```

- `_config.yml` 의 `keep_files: [general]` 과 `include: [general]` 이 이 디렉터리를 빌드 산출물에 남긴다. 둘 중 하나라도 빼면 `/general/institution.xml` 이 404 가 되고, NRO 수집기는 조용히 실패한다.

## 인증서

`assets/certs/*.pem` 과 `_config.yml` 의 `cert.*` 블록(CN, SHA-256 지문, 유효기간)은 **한 쌍이다.** 갱신할 때 둘 다 고친다.

지문은 추정하거나 옮겨 적지 말고 실제 파일에서 계산한다.

```sh
openssl x509 -in assets/certs/sandbox_ca.pem -noout -fingerprint -sha256 -dates
```

사이트에 적힌 지문이 실제로 배포되는 인증서와 다르면, 수동 설정하는 사용자가 MITM 을 정상으로 받아들이게 된다. 이 파일들은 장식이 아니다.

## 공지사항

`_notices/` 의 마크다운 파일 하나가 게시물 하나다.

- 파일명은 `YYYY-MM-DD-short-slug.md`.
- Front matter 에 `title_ko` 와 `title_en` 을 **둘 다** 넣는다. 영문 홈이 `title_en` 을 읽는다.
- `is_new: true` 는 NEW 배지다. 지난 공지는 `false` 로 내린다.
- 홈에는 `date` 내림차순 최근 5건이 뜨고, `/notices/` 에는 전체가 뜬다.

현재 공지 **본문**은 언어 구분 없이 한국어 하나뿐이고 `/en/notices/` 자체가 없다. `nro-site` 는 `_notices_en/` + `_layouts/notice_en.html` 로 풀었다. 개편에서 어느 쪽으로 갈지는 결정 사항이다 — `prj/NOTES.md` 의 OBS-20260925-01, 02 참고.

## 로컬 개발

Ruby 3.3 (`.ruby-version`, Actions 워크플로와 같은 값) 에서 빌드한다. Homebrew 로 깔았고 keg-only 라 PATH 를 잡아야 한다.

```sh
export PATH="/usr/local/opt/ruby@3.3/bin:$PATH"
bundle install
bundle exec jekyll serve --port 4321
```

Ruby 3.1 로 되돌리지 않는다 — Homebrew 에서 2026-05-07 에 비활성화됐다 (upstream EOL).

`Gemfile.lock` 은 커밋되어 있고 `x86_64-linux` 플랫폼이 들어 있다. "로컬에서 본 것 == 배포되는 것"이 성립해야 하므로, gem 을 올렸으면 lock 도 같이 커밋한다.

`general/institution.xml` 은 빌드와 무관하게 따로 검증한다.

```sh
python3 -c 'import xml.dom.minidom,sys; xml.dom.minidom.parse(sys.argv[1])' general/institution.xml
```

## 배포

- `main` push → `.github/workflows/pages.yml` → Pages. 브랜치는 `main` 하나.
- 워크플로가 `--baseurl "${{ steps.pages.outputs.base_path }}"` 를 넘긴다. 커스텀 도메인에서는 빈 문자열이라 `_config.yml` 의 `baseurl: ""` 와 일치한다. `baseurl` 을 직접 채우지 않는다 — 워크플로 인자와 충돌한다.
- `CNAME` 파일과 GitHub Pages 설정의 custom domain 은 같이 움직인다. 파일만 고치고 Pages 설정을 안 고치면(또는 그 반대) HTTPS 인증서 발급이 멈춘다.
- 커스텀 도메인은 DNS 에 `eduroam-kr.github.io` 로 가는 CNAME 레코드가 먼저 있어야 검증된다.

## 외부 의존성

런타임에 불러오는 것은 셋뿐이다 — Bootstrap 5.3.8 과 Font Awesome 6.5.0 은 cdnjs 에서, Noto Sans KR 은 Google Fonts 에서. jQuery 와 AdminLTE 는 걷어냈다. Radix Themes 토큰은 CDN 이 아니라 `assets/css/vendor/` 에 커밋되어 있다.

**새 CDN 의존을 추가하지 않는다.** CDN 이 죽으면 레이아웃이 무너진다. `nro-site` 는 vendor 정적 자산을 커밋하는 쪽으로 이미 해결했고, 이 저장소도 그렇게 갈지는 아직 결정 사항이다 — 결정하면 `prj/ADR.md` 에 적는다.

버전은 URL 에 정확히 박아 쓴다. `latest` 나 범위 지정을 쓰지 않는다.

## 문서

- **하드 랩 금지.** 문단·목록 항목·인용은 각각 한 줄로 쓰고 줄바꿈은 렌더러에 맡긴다. 하드 랩이 있으면 단어 하나 고쳤을 때 이후 모든 줄이 리플로우되어 diff 가 못 쓰게 된다. 코드 블록과 표는 예외다.
- README 는 짧게 유지한다. fork 하는 기관이 처음 읽는 문서라서, 길어지면 안 읽힌다.
- 숫자는 측정해서 쓴다. 인증서 지문, 유효기간, 기관 수 같은 값을 기억이나 짐작으로 적지 않는다.
