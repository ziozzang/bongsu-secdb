# bongsu-secdb

[한국어](#한국어) · [English](#english)

---

## 한국어

**봉수(Bongsu) 자산 점검 도구에 맞춰 만든 취약점 데이터베이스**입니다.
공개된 취약점 자료를 모아, 봉수가 실제로 판정에 쓰는 정보만 남기고 압축해 배포합니다.

> 상태: **준비 중.** 빌드 도구와 첫 공개판을 만들고 있습니다. 아직 내려받을 수 있는 릴리스가 없습니다.

### 무엇이 다른가

같은 이름이라도 대상이 다르면 다른 취약점이 붙습니다. 이 데이터베이스는 그 차이를 구분합니다.

- **이름이 같아도 다른 대상**: `curl`이 PyPI 패키지인지, 배포판 패키지(rpm/deb)인지, 디스크의 실행 파일인지 구분합니다. 조회 키는 생태계·유형, 배포판·버전, 패키지 이름, 실행 파일이면 경로와 소속 패키지입니다.
- **배포판 백포트**: RHEL 같은 배포판은 원래 버전 번호를 그대로 두고 보안 수정만 가져옵니다. 그래서 원래 프로그램 버전이 아니라 배포판 보안 공지의 "고쳐진 릴리스 번호(`epoch:version-release`)"와 "영향 없음" 판정으로 비교합니다.
- **크기**: 공개 원본보다 훨씬 작습니다(시제품 측정: 대표 8개 생태계 396 MB → 7~24 MB).

### 데이터 출처

| 원천 | 내용 | 라이선스 |
|---|---|---|
| [OSV](https://osv.dev) — npm, PyPI, Go, crates.io, Maven, NuGet, RubyGems, Packagist, Hex, Pub | 언어 생태계 취약점 | CC BY 4.0 (GHSA·PyPA 등), RustSec CC0 1.0, OpenSSF 악성 패키지 Apache-2.0 |
| [GitHub Advisory Database](https://github.com/advisories) | 공지 원본 | CC BY 4.0 |
| Ubuntu (OSV) | 배포판 공지 | **CC BY-SA 4.0** |
| Alpine (OSV, secdb) | 배포판 공지 | **CC BY-SA 4.0** |
| Red Hat (OSV, VEX) | 배포판 공지·영향 판정 | CC BY 4.0 |
| SUSE / openSUSE (OSV, CSAF) | 배포판 공지 | CC BY 4.0 |
| AlmaLinux (OSV) | 배포판 공지 | MIT |
| Rocky Linux (OSV) | 배포판 공지 | BSD |
| Debian (OSV, 보안 추적기) | 배포판 공지·영향 없음 판정 | 라이선스 확인 중 — 확인되기 전에는 공개판에 넣지 않습니다 |
| [NVD](https://nvd.nist.gov) | CVE 세부 정보 | 미국 정부 자료 (아래 고지 참고) |
| [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog) | 실제 악용 확인 목록 | CC0 1.0 |
| [EPSS](https://www.first.org/epss/) | 악용 가능성 점수 | 무료, 출처 표기 |

**넣지 않는 원천**: Chainguard·Wolfi(CC BY-NC-ND 4.0, 변형물 재배포 금지).

원천별 자세한 출처와 표기 문구는 [NOTICE.md](NOTICE.md)에 있습니다.

### 갱신 주기

- **6시간마다** 자동으로 받기 → 정규화 → 빌드 → 서명 → 검증 → 게시합니다.
- 어느 단계든 실패하면 새 판을 게시하지 않고 **이전 판을 그대로 둡니다.**
- 판마다 만든 시각(UTC)과 원천별 기준 시점이 manifest에 적힙니다.

### 받는 것

릴리스 자산으로 배포합니다(git 이력에는 작은 색인과 문서만 둡니다).

| 파일 | 설명 |
|---|---|
| `bsdb-<날짜>.jsonl.xz` | 정본. 정렬된 JSONL, 차이 비교와 재현이 쉬운 형식 |
| `bsdb-<날짜>.sqlite.zst` | 바로 조회할 수 있는 SQLite |
| `manifest.json` / `manifest.sig` | 파일별 해시, 원천 기준 시점, 봉수 서명 |

### 검증

봉수 서명 형식(정규 JSON CJ1 + Ed25519 + SHA-256)으로 서명합니다. 봉수 서버는 처음 한 번 등록한 공개 키로 서명을 확인한 뒤에만 받아들입니다. 공개 키 지문은 첫 릴리스 때 이 문서에 적습니다.

### 고지

- 이 자료는 **있는 그대로** 제공되며 정확성·완전성을 보증하지 않습니다. 보안 판단의 유일한 근거로 쓰지 마십시오.
- 이 제품은 NVD API를 사용하지만 NVD가 보증하거나 인증하지 않았습니다. NVD 내용은 봉수 형식으로 **유도·수정**했으므로 NVD 원본으로 표시하지 않습니다.
- CISA KEV 자료를 쓰지만 CISA가 이 제품을 보증하지 않습니다.
- 각 원천의 상표와 권리는 해당 소유자에게 있습니다.

### 라이선스

- **데이터**(릴리스 자산): [CC BY-SA 4.0](LICENSE-DATA). Ubuntu·Alpine 자료(CC BY-SA 4.0)를 포함하므로 묶음 전체가 같은 조건을 따릅니다. 원천별 표기는 [NOTICE.md](NOTICE.md).
- **코드**(빌드 도구·스크립트): [AGPL-3.0-only](LICENSE-CODE).

Copyright Jioh Jung <ziozzang@gmail.com>

---

## English

A **vulnerability database tailored to the Bongsu asset-inspection tool.** It collects public advisory data, keeps only what Bongsu actually uses for matching, and ships it compressed.

> Status: **in preparation.** The builder and first public release are being made. No downloadable release yet.

### What is different

- **Same name, different target**: distinguishes `curl` the PyPI package, the distro package (rpm/deb), and the binary on disk. Lookup key = ecosystem/type, distro + release, package name, and (for binaries) path and owning package.
- **Distro backports**: compares against the distro advisory's fixed `epoch:version-release` and "not affected" verdicts, not the upstream version.
- **Size**: much smaller than the raw sources (prototype: 8 ecosystems 396 MB → 7–24 MB).

### Sources

See the table above and [NOTICE.md](NOTICE.md). Chainguard and Wolfi (CC BY-NC-ND 4.0) are **excluded**. Debian is included only after its license is confirmed.

### Update schedule

Built automatically **every 6 hours**: fetch → normalize → build → sign → verify → publish. If any step fails, nothing is published and the previous release stays.

### Files

Release assets: canonical sorted JSONL (`.jsonl.xz`), ready-to-query SQLite (`.sqlite.zst`), and a signed `manifest.json`. Signed with Bongsu's format (canonical JSON CJ1 + Ed25519 + SHA-256); the public-key fingerprint will be listed here with the first release.

### Disclaimer

Provided **as is**, without warranty of accuracy or completeness. This product uses the NVD API but is not endorsed or certified by the NVD; NVD content has been derived and modified. CISA does not endorse this product.

### License

- **Data** (release assets): [CC BY-SA 4.0](LICENSE-DATA) — includes CC BY-SA 4.0 sources (Ubuntu, Alpine). Per-source attribution in [NOTICE.md](NOTICE.md).
- **Code** (builder, scripts): [AGPL-3.0-only](LICENSE-CODE).

Copyright Jioh Jung <ziozzang@gmail.com>
