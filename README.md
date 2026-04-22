# eduroam Site Web 📡

KREONET이 운영하는 eduroam 서비스 안내 사이트 템플릿입니다.
참여기관에서 fork 하여 기관용 eduroam 안내 사이트로 사용할 수 있으며,
GitHub Pages를 통한 정적 호스팅이 가능합니다.

데모 사이트: **[eduroam.demo.kreonet.net](https://eduroam.demo.kreonet.net)**

---

## 배포

[Jekyll](https://jekyllrb.com/) 기반 정적 사이트로, GitHub Pages가 push 시점에
자동으로 빌드·배포합니다.

1. 이 저장소를 fork 하거나 clone 하여 사용하실 레포로 옮깁니다. (저장소는 public)
2. `CNAME` 파일을 기관의 eduroam 안내 사이트 도메인으로 수정합니다.
3. DNS에 해당 도메인의 CNAME 레코드를 추가합니다.

    ```
    eduroam.demo.kreonet.net.    IN CNAME    kreonet.github.io.
    ```

4. GitHub 저장소의 Settings → Pages에서 Pages 빌드가 활성화되어 있는지 확인합니다.

DNS 전파가 완료되면 **https://eduroam.<기관도메인>** 으로 접속 가능합니다.

---

## 사이트 설정

수정이 필요한 모든 값은 `_config.yml`에 모여 있습니다.

| 키 | 설명 |
|-----|-----|
| `realm` | 기관의 RADIUS realm (예: `sandbox.re.kr`) |
| `cert.ca.*` | CA 인증서의 CN, 지문(fingerprint), 유효기간 |
| `cert.server.*` | RADIUS 서버 인증서의 CN, 지문, 유효기간 |
| `support.email` | 헬프데스크 이메일 |
| `support.tel` | 헬프데스크 전화번호 |
| `t.ko` / `t.en` | UI 문자열 (한국어 / 영어) |

---

## 다국어 지원

| 경로 | 언어 |
|------|------|
| `/`, `/connect/`, `/coverage/` 등 | 한국어 |
| `/en/`, `/en/connect/`, `/en/coverage/` 등 | 영어 |

페이지의 언어는 `_config.yml`의 `defaults:` 섹션에서 경로별로 자동 지정됩니다.
사용자는 내비게이션 바의 언어 전환 스위치로 두 버전을 오갈 수 있습니다.

---

## eduroam 참여기관 메타데이터

각 참여기관은 [eduroam database specification v2.0.1](https://monitor.eduroam.org/fact_eduroam_db.php)에
따라 자기 기관의 메타데이터(기관명·주소·좌표·연락처·hotspot 정보 등)를 NRO(KREONET)에 제공해야 합니다.

본 사이트 템플릿은 이 메타데이터를 사이트의 일부로 함께 서빙하도록 설계되어 있어,
별도의 업로드 창구 없이 GitHub에 커밋하는 것만으로 NRO 제출이 완료됩니다.

**작성 방법**

1. `general/institution.xml` 을 기관 정보에 맞게 수정합니다.
   (파일 상단 주석에 각 필드 작성 요령이 안내되어 있습니다.)
2. 커밋·push 하면 다음 경로로 자동 게시됩니다.

    ```
    https://eduroam.<기관도메인>/general/institution.xml
    ```

3. KREONET NRO 수집기가 주기적으로 이 URL을 harvest 하여
   eduroam.kr NRO 파일로 머지하고, GÉANT eduroam monitor에 제출합니다.

**검증**

제출 전 아래 URL로 스펙 준수 여부를 확인할 수 있습니다.

```
https://monitor.eduroam.org/eduroam-database/v2/scripts/xml_validation_test.php?url=<your-url>
```

---

## 인증서 파일

수동 설정이 필요한 사용자를 위해 기관의 CA/RADIUS 서버 인증서를 정적으로 제공합니다.

| 파일 | 설명 |
|------|------|
| `assets/certs/sandbox_ca.pem` | 루트 CA 인증서 |
| `assets/certs/sandbox_radius.pem` | RADIUS 서버 인증서 |

인증서 갱신 시 두 가지를 함께 업데이트해주세요.

- `assets/certs/` 아래의 PEM 파일
- `_config.yml`의 `cert.*` 블록 (지문 및 유효기간)

---

## 공지사항 추가·수정

공지사항은 `_notices/` 디렉터리의 마크다운 파일 하나가 게시물 하나에 대응합니다.

**파일명 규칙**

```
YYYY-MM-DD-short-slug.md
```

**Front matter 예시**

```yaml
---
title_ko: "공지 제목 (한국어)"
title_en: "Notice title (English)"
date: 2025-06-10
is_new: true        # NEW 배지 표시 여부; 지난 공지는 false
---
```

홈 페이지에는 `date` 기준 내림차순으로 최근 5건이 표시됩니다.
공지를 내리려면 해당 파일을 삭제하거나 `is_new: false`로 변경하세요.
