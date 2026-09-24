# TASKS

ADR-0001 (Bootstrap 5.3), ADR-0002 (Radix Colors), ADR-0003 (반응형) 이 승인되면 아래 순서로 간다. T-0001 이 나머지 전부의 선행 조건이다.

## 선행

### T-0001 — 로컬 Ruby 3.1 빌드 환경 `TODO`

`.ruby-version` 은 3.1.3 인데 이 머신은 시스템 Ruby 2.6.10 뿐이고 rbenv / asdf / Homebrew Ruby 가 없다. `bundle install` 이 `ffi-1.17.4` 에서 멈춘다.

프레임워크를 갈아엎으면서 렌더 결과를 못 보는 건 말이 안 된다. push 해서 Actions 로그로 확인하는 건 검증이 아니다. 이걸 먼저 깐다.

관련: OBS-20260925-05

## 이전 (ADR-0001, ADR-0002)

### T-0002 — Bootstrap 5.3 골격 교체 `TODO`

`_layouts/default.html` 의 CDN 링크를 BS 5.3 하나로 줄이고 AdminLTE·jQuery 를 뺀다. navbar / dropdown / tab 마크업을 `data-bs-*` 로 바꾸고, 테마 토글을 Bootstrap 공식 color-mode 스니펫으로 교체한다 (`localStorage` 키와 시스템/라이트/다크 3단 구조는 유지 — 지금 UX 는 문제없다).

`main.css` 의 AdminLTE 되돌리기용 `!important` 들은 이 단계에서 같이 사라져야 한다. 남아 있으면 이전이 덜 된 것이다.

### T-0003 — Radix Colors 토큰 도입 `TODO`

`slate` / `blue` 의 라이트·다크 CSS 를 `assets/css/vendor/` 에 받아 커밋하고, `:root` 와 `[data-bs-theme="dark"]` 에서 Bootstrap 시맨틱 변수로 매핑한다 (매핑표는 ADR-0002).

`main.css` 의 다크 팔레트 블록 3개(`:35`, `:356`, `:381`)를 제거한다. `light-dark()` 는 쓰지 않는다.

### T-0004 — 인라인 색상 제거 `TODO`

HTML 에 박힌 `style="color:…"` / `background` 90곳을 토큰이나 `text-body-secondary` 같은 유틸리티로 치환한다. 파일별 분포:

```
connect/  16   en/connect/  16
about/    11   en/about/    11
policy/    8   en/policy/    8
coverage/  6   en/coverage/  6
index      2   en/index      2
notices/   2   _layouts/notice.html  2
```

한/영 쌍을 같은 커밋에서 고친다 (CLAUDE.md). 끝나면 `grep -rE 'style="[^"]*color' --include='*.html' .` 가 비어야 한다.

관련: OBS-20260925-03

### T-0005 — `color-scheme` 을 테마별로 명시 `TODO`

지금은 `html { color-scheme: light dark }` 하나뿐이라, OS 가 다크인데 사용자가 라이트를 고르면 스크롤바·폼 컨트롤·기본 테두리가 다크로 남는다. `[data-bs-theme="light"] { color-scheme: light }` / `[data-bs-theme="dark"] { color-scheme: dark }` 를 넣는다.

## 반응형 (ADR-0003)

### T-0006 — 모바일 브레이크포인트와 한국어 줄바꿈 `TODO`

- `body { word-break: keep-all; overflow-wrap: anywhere; }` — 한국어 어절 중간 줄바꿈 제거. 체감 효과가 가장 큰 한 줄이다.
- `.section-card` 좌우 패딩을 sm 구간에서 32px → 16~20px.
- 히어로·섹션 제목 타이포 스케일을 좁은 폭에서 축소.
- 인증서 SHA-256 지문은 옥텟 단위로만 끊기게 (`:` 뒤 `<wbr>`), 복사 버튼 검토.
- 360px 에서 가로 스크롤이 실제로 생기는지 확인하고 원인을 찾는다. 다른 세션이 `.quick-cards` 의 `--bs-gutter-x` 를 원인으로 지목했는데, 코드를 읽어보면 음수 마진과 열 패딩이 같은 변수를 써서 자기일관적이다 — 원인이 다른 데 있을 수 있으니 T-0001 이후 실제 렌더로 재현부터 한다.

### T-0007 — navbar 접힘 구간 조정 `TODO`

`navbar-expand-lg` 라 992px 미만에서 햄버거로 접힌다. 태블릿 세로(820px)에 메뉴 5개를 펼칠 공간은 충분하다. `expand-md` 로 내리고 md 구간에서는 메뉴 아이콘을 숨긴다.

### T-0008 — 접속 안내 OS 탭을 모바일 우선으로 `TODO`

폰에서 Windows 탭이 먼저 열린다. 탭을 가로 스크롤 또는 세그먼트 컨트롤로 바꾸고, User-Agent 로 현재 OS 탭을 기본 선택한다.

## 접근성

### T-0009 — 키보드·스크린리더 대응 `TODO`

- "빠른 도움말" `<li onclick=…>` 8곳(`index.html` 4, `en/index.html` 4)을 `<a>` 로. 지금은 탭 이동도 새 탭 열기도 안 된다.
- 공지 리스트 `li` 전체에 `cursor:pointer` 인데 실제 클릭 영역은 제목 텍스트뿐 — 영역을 일치시킨다.
- 햄버거 버튼과 언어 드롭다운에 `aria-label`.
- 🇰🇷🇺🇸 국기 이모지는 Windows 에서 "KR" / "US" 글자로 나온다. 텍스트 레이블로 교체.

### T-0010 — 실기기 검증 `TODO`

T-0004 까지 끝난 뒤 삼성 단말 + Samsung Internet 에서 절전모드를 켜고 확인한다. 강제 다크가 건너뛰어지는지, 건너뛰어진 뒤 화면이 정상인지 둘 다 본다. 이 개편의 출발점이므로 이걸 확인하기 전에는 완료로 치지 않는다.

Samsung Internet 버전도 같이 기록한다 — 버전별로 동작이 다르다.

## 별도 트랙 (UI 아님)

### T-0011 — 영문 공지 처리 방식 결정 `TODO`

`/en/notices/` 가 없어 404 이고, 공지 상세는 영문에서도 한국어가 나온다. ADR 로 방식을 먼저 정한다.

관련: OBS-20260925-01, OBS-20260925-02

### T-0012 — 접속 안내 내용 오류 `TODO`

다른 세션이 짚은 것으로, 아직 내가 직접 확인하지 않았다. 확인 후 착수한다.

- Android 탭의 "CA 인증서: 시스템 인증서 사용" — 사설 CA 는 시스템 신뢰 저장소에 없으므로 검증에 실패한다. 다운로드한 CA 를 쓰도록 안내해야 한다.
- Windows 3단계 "인증 기관 sandbox.re.kr 선택" — realm 과 CA 이름이 섞였다.
- CAT 링크를 기관 IdP 딥링크로, 모바일은 geteduroam 앱 우선 안내 검토.

기관들이 fork 해서 그대로 복사하는 문서라 틀린 안내가 그대로 퍼진다. 우선순위가 낮지 않다.
