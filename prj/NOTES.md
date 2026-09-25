# NOTES

`site-template` 사본을 훑으면서 확인한 것들. 각 항목은 실제 파일을 읽고 확인했으며, 로컬에 Ruby 3.1 이 없어 빌드로는 재현하지 못했다.

## OBS-20260925-01 — 영문 공지 목록 페이지가 없다

`notices/index.html` 은 있지만 `en/notices/` 가 없다. 결과가 두 가지다.

- `en/index.html:58` 의 "View all" 링크가 `href="#"` 로 죽어 있다. 한국어 홈은 같은 자리가 `/notices/` 로 간다.
- `/notices/` 에서 언어 전환을 누르면 `/en/notices/` 로 가는데 그 경로가 없어 404 다. 언어 전환이 URL 접두사 조작으로만 동작하기 때문에 생기는 문제다.

## OBS-20260925-02 — 공지 본문은 한국어 하나뿐이다

`_notices/` 컬렉션 하나를 두 언어가 공유한다. 목록에서는 `title_ko` / `title_en` 로 갈리지만, 상세로 들어가면 `_layouts/notice.html` 이 `page.title_ko` 와 한국어 본문을 그대로 보여준다. 영문 사용자가 공지를 열면 한국어 페이지가 나온다.

`nro-site` 는 `_notices_en/` 컬렉션과 `_layouts/notice_en.html` 을 따로 두는 방식으로 풀었다. 선택지는 (a) `nro-site` 방식, (b) front matter 에 `body_en` 을 넣고 레이아웃에서 분기, (c) 영문 공지를 포기하고 목록만 영문 제목으로 노출. 결정하면 ADR 로 남긴다.

## OBS-20260925-03 — 인라인 색이 다크 테마를 뚫는다

테마는 CSS 변수로 하는데, HTML 에 색이 직접 박힌 자리가 남아 있다.

- `_layouts/notice.html` — 본문 `color:#333`, 목록 링크 `color:#888`.
- `index.html` / `en/index.html` — 공지 카드 헤더의 "전체보기" 링크 `color:#888`.
- `notices/index.html` — 빈 목록 안내문 `color:#888`.

다크에서 배경이 `#0b1220` 으로 바뀌어도 이 글자들은 그대로라 대비가 무너진다. 변수로 옮기면 된다 (`--card-text`, `--card-subtext`).

## OBS-20260925-04 — `url:` 과 `CNAME` 이 어긋나면 CSS 가 안 뜬다

`_layouts/default.html:25` 이 `main.css` 를 `relative_url` 이 아니라 `absolute_url` 로 건다. `absolute_url` 은 `_config.yml` 의 `url:` 을 앞에 붙이므로, `url:` 이 실제 서빙 도메인과 다르면 스타일시트를 다른 도메인에서 불러온다. 사본을 만들 때 `CNAME` 만 고치고 `url:` 을 안 고치면 바로 걸린다.

이 한 줄만 `absolute_url` 이고 나머지 에셋은 전부 `relative_url` 이다. 의도된 차이인지 실수인지 원저자 확인이 필요하다. 실수라면 `relative_url` 로 통일하는 쪽이 도메인 변경에 강하다.

## OBS-20260925-05 — 로컬 빌드 환경이 없다

이 머신의 Ruby 는 시스템 기본 2.6.10 이고, `Gemfile` 의 jekyll 4.3 은 Ruby 3.0 이상을 요구한다 (`bundle install` 이 `ffi-1.17.4` 에서 실패). rbenv / asdf / Homebrew Ruby 모두 없다. `.ruby-version` 은 3.1.3 을 가리킨다.

즉 현재는 push 해서 Actions 로그를 보는 것 외에 렌더 결과를 확인할 방법이 없다. 개편을 본격적으로 하기 전에 Ruby 3.1 설치가 선행되어야 한다.

## OBS-20260925-06 — `main.css` 에 반응형 미디어 쿼리가 하나도 없다

`grep -n '@media' assets/css/main.css` 결과가 `prefers-color-scheme` 두 줄뿐이다. 화면 폭 대응은 전적으로 마크업의 `col-6 col-md-3` 같은 Bootstrap 그리드 클래스에 의존한다.

그리드는 열 배치만 바꾸므로, 폭이 좁아져도 `.section-card` 의 좌우 패딩 32px, 히어로 폰트 크기, 섹션 여백이 데스크탑 값 그대로 남는다. 360px 화면에서 본문 가용 폭이 270px 남짓이 되는 직접적 원인이다.

"모바일에서 레이아웃은 무너지지 않지만 답답하다"는 인상의 출처가 여기다. ADR-0003 의 근거.

## OBS-20260925-07 — 인라인 색상 90곳

`grep -roE 'style="[^"]*(color|background)[^"]*"' --include='*.html'` 기준 90곳이다. `connect/` 와 `en/connect/` 가 각 16 으로 가장 많고, `policy/`·`about/`·`coverage/` 가 뒤를 잇는다. OBS-20260925-03 이 레이아웃 파일 몇 군데 얘기였다면 이건 전수 조사 결과다.

CSS 변수로 테마를 전환해도 이 90곳은 반응하지 않는다. 다크가 절반쯤 깨져 보이는 원인은 다크 팔레트가 아니라 여기다.

다른 세션은 같은 것을 73곳으로 셌다. 세는 기준(선택자 단위 / 선언 단위)이 다를 뿐 같은 문제를 가리킨다.

## OBS-20260925-08 — `eduroam.<기관도메인>` 전용 사이트는 사실상 없다

"다른 나라 기관 중 잘 만든 `eduroam.instdomain` 사이트"를 찾아봤는데, **그 범주 자체가 거의 존재하지 않는다.** 검색에 걸리는 기관들은 전부 기존 IT 서비스 포털·지식베이스 안의 한 페이지로 다룬다.

- Oxford — `help.it.ox.ac.uk/wireless-access-university`, `ox.ac.uk/staff/it/services/wifi/eduroam`
- ETH Zürich — `unlimited.ethz.ch/display/itkb/Wi-Fi`
- Buffalo, Cornell Weill, Wisconsin, Nebraska, Maryland — 전부 KB 아티클

Buffalo 것을 열어 구조를 봤다. **한 페이지**에 연결 마법사 → 단계별 안내 → 네트워크 설명 → 커버리지 맵 → FAQ → 관련 뉴스 순이고, 기기별 설정만 별도 페이지로 링크한다. realm·인증서 같은 기술 정보는 표면에 내지 않는다.

즉 기관 단위에서 사이트를 "짓는" 사례가 드물고, 짓더라도 한 페이지다.

## OBS-20260925-09 — `eduroam.kaist.ac.kr` 의 구조가 참고가 된다

5년 전 급하게 만들어진 사이트인데 구조는 지금 논의에 그대로 쓸 만하다. (curl 은 403, 실제 브라우저로는 열린다)

- **한 페이지, 본문 2,458자.** 상단에 앵커 목차, 본문은 `#1-eduroam-service` 같은 인페이지 링크로 이동.
- 구성: 1 서비스 안내 / 2 사용방법 (2-1 방문객, 2-2 구성원, 2-3 서비스 지역) / 3 보안정책.
- **한/영을 같은 페이지에 나란히** 둔다. `/en/` 별도 트리가 없다.
- 공지는 자기가 갖지 않고 **KAIST IT 공지 게시판으로 링크**한다.
- 서비스 지역은 **eduroam.kreonet.net(NRO)으로 링크**한다.

기관이 실제로 쥐고 있어야 할 것(사용 방법, 보안 정책)만 남기고 나머지를 이미 존재하는 곳으로 넘긴 형태다. `site-template` 이 기관용으로 남는다면 이 모양이 기준점이다.
