# NOTICE — 데이터 출처와 표기 / Data sources and attribution

이 저장소의 릴리스 자산(데이터)은 아래 공개 자료에서 유도·수정한 것입니다. 봉수 형식으로 정규화하고 필드를 줄였으므로 **원본 그대로가 아닙니다.**
Release assets are derived from and modified versions of the sources below; they are **not** the original data.

데이터 묶음 전체 라이선스: **CC BY-SA 4.0** (LICENSE-DATA). 아래 원천의 조건도 함께 따릅니다.

| 원천 / Source | 출처 / URL | 라이선스 / License | 표기 / Attribution |
|---|---|---|---|
| OSV.dev (집계) | https://osv.dev | 원천별 라이선스를 따름 | "Data from OSV.dev and its upstream sources" |
| GitHub Advisory Database | https://github.com/github/advisory-database | CC BY 4.0 | "GitHub Advisory Database, CC BY 4.0" |
| PyPA Advisory Database | https://github.com/pypa/advisory-database | CC BY 4.0 | "PyPA Advisory Database, CC BY 4.0" |
| Go Vulnerability Database | https://vuln.go.dev | CC BY 4.0 | "Go Vulnerability Database, CC BY 4.0" |
| RustSec Advisory Database | https://rustsec.org | CC0 1.0 | (표기 의무 없음, 감사 표시) |
| OpenSSF Malicious Packages | https://github.com/ossf/malicious-packages | Apache-2.0 | "OpenSSF Malicious Packages, Apache-2.0" |
| EEF CNA (Hex) | https://cna.erlef.org | CC BY 4.0 | "EEF CNA, CC BY 4.0" |
| Ubuntu Security Notices / OSV | https://ubuntu.com/security | CC BY-SA 4.0 | "Canonical Ubuntu security data, CC BY-SA 4.0" |
| Alpine secdb / OSV | https://secdb.alpinelinux.org | CC BY-SA 4.0 | "Alpine Linux secdb, CC BY-SA 4.0" |
| Red Hat Security Data (OSV, VEX) | https://security.access.redhat.com/data/ | CC BY 4.0 | "Red Hat security data, CC BY 4.0" |
| SUSE / openSUSE (OSV, CSAF) | https://ftp.suse.com/pub/projects/security/ | CC BY 4.0 (2021-05-12 이후 자료) | "SUSE security data, CC BY 4.0" |
| AlmaLinux (OSV) | https://errata.almalinux.org | MIT | "AlmaLinux OS errata, MIT" |
| Rocky Linux (OSV) | https://errata.rockylinux.org | BSD | "Rocky Linux errata, BSD" |
| Debian Security Tracker / OSV | https://security-tracker.debian.org | 확인 중 / under review | 확인 전에는 공개판에서 제외 / excluded until confirmed |
| NVD | https://nvd.nist.gov | 미국 정부 저작물 / U.S. Government work | 아래 NVD 고지 |
| CISA KEV | https://www.cisa.gov/known-exploited-vulnerabilities-catalog | CC0 1.0 | CISA 로고 미사용, 보증 암시 없음 |
| FIRST EPSS | https://www.first.org/epss/ | 무료 사용, 출처 표기 요청 | "EPSS scores by FIRST.org" |

## 제외한 원천 / Excluded

- **Chainguard**, **Wolfi** — CC BY-NC-ND 4.0. 변형물 배포와 영리 사용이 금지되어 넣지 않습니다. / Excluded: derivative redistribution not permitted.

## NVD 고지 / NVD notice

This product uses the NVD API but is not endorsed or certified by the NVD.
NVD 내용은 봉수 형식으로 유도·수정했으며, NVD 원본으로 표시하지 않습니다.

## CISA 고지 / CISA notice

CISA KEV 자료를 사용하지만 CISA가 이 제품을 보증하지 않습니다. / CISA does not endorse this product.

## 보증 부인 / No warranty

자료는 있는 그대로 제공되며, 정확성·완전성·특정 목적 적합성을 보증하지 않습니다.
Provided "as is", without warranty of any kind.

## 빌더 1판이 실제로 쓰는 원천과 수정 내용 (2026-10-30 덧붙임) / Sources actually used by builder v1 (addendum)

위 표 중 빌더 1판이 받는 것은 **OSV(아래 생태계), Debian 보안 추적기, Alpine secdb**뿐입니다. NVD·CISA KEV·EPSS는 아직 받지 않으므로 이 자료에 들어 있지 않습니다.
Builder v1 fetches only OSV (ecosystems below), the Debian security tracker and Alpine secdb. NVD, CISA KEV and EPSS are not ingested yet and are not part of the data.

| OSV 생태계 / ecosystem | 상류 / upstream | 라이선스 / License |
|---|---|---|
| npm, Maven, NuGet, RubyGems, Packagist, Pub, SwiftURL, GitHub Actions | GitHub Advisory Database | CC BY 4.0 |
| npm·PyPI 악성 패키지 / malicious packages | OpenSSF Malicious Packages | Apache-2.0 |
| PyPI / Go / crates.io | PyPA / Go vulndb / RustSec (+GHSA) | CC BY 4.0 / CC BY 4.0 / CC0 1.0 |
| Hex | EEF CNA | CC BY 4.0 |
| CRAN | R Consortium Advisory Database https://github.com/RConsortium/r-advisory-database | Apache-2.0 |
| Hackage | Haskell Security Advisories https://github.com/haskell/security-advisories | CC0 1.0 |
| Julia | Julia Security Advisories https://github.com/JuliaLang/SecurityAdvisories.jl | CC BY 4.0 |
| opam | OCaml security-advisories https://github.com/ocaml/security-advisories | CC0 1.0 |

- **SUSE·openSUSE**: 2021-05-12 이후에 공개된 공지만 넣습니다(그 전 공지는 CC BY-NC라 재배포할 수 없음). Only advisories published on/after 2021-05-12.
- **Debian**: 라이선스 확인 전이므로 빌더는 기본으로 제외하고, 사람이 `--include-unverified`를 줄 때만 포함합니다. Excluded by default until the license is confirmed.
- **수정 내용 / Modifications**: 조회 키(생태계·유형, 배포판·버전, 이름, 바이너리/소스)로 다시 정리하고, 판 범위·고친 판·영향 없음·철회·CVE 연결만 남겼습니다(본문·링크·점수·GIT 커밋 범위 제외). 이름 정규화(PyPI는 PEP 503, npm·Go 등은 소문자), 정렬·중복 제거 뒤 JSONL·SQLite로 다시 담았습니다. Re-keyed, reduced to version ranges / fixed versions / not-affected / withdrawn / CVE links, names normalised, sorted and de-duplicated, re-serialised as JSONL and SQLite.
- 원천 파일마다 받은 시각·SHA-256·ETag가 서명된 `manifest.json`에 기록됩니다. Per-source fetch time, SHA-256 and ETag are recorded in the signed `manifest.json`.
