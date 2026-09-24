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
