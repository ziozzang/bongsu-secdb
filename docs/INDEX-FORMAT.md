# 형식 문서: 릴리스 색인, manifest, 정본, SQLite

> 이 문서는 빌더(`bongsu-secdb-build`) 1판이 실제로 만드는 파일의 정확한 형식입니다. README의 표(`bsdb-<날짜>…`, `manifest.sig`)는 개요이며, 정확한 이름과 구성은 이 문서가 정본입니다(서명은 별도 `manifest.sig` 없이 `manifest.json` 한 파일에 서명 봉투로 들어갑니다).

형식 이름: manifest `bongsu-secdb/1`, 정본·SQLite `bsdb-1`. 형식이 바뀌면 이름의 숫자를 올립니다.
(English summary at the end.)

## 1. 릴리스와 파일 이름

릴리스 태그는 `v<순번>-<UTC 날짜시각>`(예: `v42-20261031T060000Z`)입니다. 첨부 파일은 묶음(`lang`, `malicious`, `os-apk`, `os-deb`, `os-rpm`, `os-ubuntu`)마다 둘이며 데이터가 없는 묶음은 생략됩니다.

| 이름 | 내용 |
|---|---|
| `bsdb-<묶음>-base-<순번>.jsonl.xz` | 정본 JSONL(xz, 단일 스트림) |
| `bsdb-<묶음>-<순번>.sqlite.zst` | 정본에서 만든 SQLite(zstd, 긴 창 27비트) |
| `manifest.json` | 서명 봉투(아래 2) |
| `SHA256SUMS` | `<sha256>␣␣<이름>` 줄, 이름순. 데이터 파일과 `manifest.json` |

git에는 `latest.json`(아래 3)과 문서만 커밋합니다. 데이터는 git 이력에 넣지 않습니다.

## 2. manifest.json (서명됨)

봉수의 서명 봉투(`SignedEnvelope`)입니다: `{envelope_version, purpose:"bundle", key_id, payload_base64, signature_base64}`.
`payload_base64`는 manifest 본문을 **CJ1**(ASCII 객체 키 오름차순·공백 없는 UTF-8·정수만, 절댓값 ≤ 2^53−1)로 직렬화한 바이트입니다.
서명은 Ed25519이고 서명 대상은 도메인 접두 `bongsu-envelope-v1\0` 뒤에 (purpose, key_id, payload) 각각을 8바이트 빅엔디언 길이와 함께 이어 붙인 바이트입니다(봉수 `bongsu-wire`의 `signing_bytes`와 같음).
`key_id`는 공개키 32바이트의 SHA-256(소문자 16진)입니다.

본문 필드:

| 필드 | 뜻 |
|---|---|
| `format` | `bongsu-secdb/1` |
| `serial` | 발행 순번(1부터, 한 번에 1씩) |
| `prev_manifest_digest` | 직전 manifest 본문(CJ1)의 SHA-256. 첫 발행은 빈 문자열 |
| `built_at` | 빌드 시각(유닉스 초) |
| `builder` | 빌더 이름·판 |
| `data_license` | `CC-BY-SA-4.0` |
| `bundles[]` | `name`, `advisories`, `packages`(조회 키 수), `rules` |
| `files[]` | `name`, `bundle`, `kind`(`jsonl-xz`·`sqlite-zst`), `bytes`, `sha256` |
| `sources[]` | `name`, `license`, `url`, 원문 `sha256`·`bytes`, `etag`, `last_modified`, `fetched_at`, `status`(`fresh`·`not_modified`·`stale`·`missing`) |
| `excluded[]` | 일부러 또는 받지 못해 빠진 원천과 이유(Chainguard·Wolfi는 항상 있음) |
| `quarantine` | 해석하지 못해 격리한 공지·대상의 이유별 개수 |
| `unsupported_schemes` | 범위만 담고 판정은 못 하는 판 체계 |

검증하는 쪽은 (1) 고정한 공개키로 서명을, (2) `serial`이 직전+1이고 `prev_manifest_digest`가 직전 본문 지문인지를, (3) 모든 원천이 허용 목록·라이선스 표와 같고 금지 원천이 없는지를, (4) 파일 크기·SHA-256을 확인해야 합니다.

## 3. latest.json (서명 없음, 편의용 포인터)

```json
{
  "format": "bongsu-secdb-latest/1",
  "serial": 42,
  "tag": "v42-20261031T060000Z",
  "built_at": "2026-10-31T06:00:00Z",
  "signing_key_id": "<공개키 SHA-256 16진>",
  "manifest": {"name": "manifest.json", "sha256": "…", "url": "https://github.com/ziozzang/bongsu-secdb/releases/download/v42-20261031T060000Z/manifest.json"}
}
```

`latest.json` 자체는 신뢰의 근거가 아닙니다. 가리키는 `manifest.json`의 서명을 검증해야 합니다.

## 4. 정본 JSONL (`bsdb-1`)

UTF-8, 줄마다 JSON 하나. 첫 줄은 머리말 객체 `{"bundle":"…","format":"bsdb-1"}`이고, 나머지는 **줄 종류(a < r < s) 다음 필드를 왼쪽부터 바이트 순서로** 정렬하며 중복이 없습니다.

| 줄 | 필드 |
|---|---|
| `["a", id, source, modified, withdrawn, cves]` | 공지. `modified` 유닉스 초(없으면 0), `withdrawn` 0/1, `cves`는 정렬된 목록 |
| `["r", type, ns, dist, chan, name, by_source, auth, status, scheme, range, advisory_id]` | 규칙. 아래 표 |
| `["s", name, license, url]` | 이 묶음에 쓰인 원천 |

규칙 필드:

- 조회 키 `type`(purl 유형: `deb rpm apk pypi npm cargo golang maven nuget gem composer hex pub cran hackage julia opam swift githubactions`), `ns`(네임스페이스: 배포판 이름, npm `@scope`, Maven groupId …), `dist`(`debian-12`, `ubuntu-22.04`, `alpine-3.20`, `rhel-9`·`rhel-9.0`, `almalinux-8`, `rocky-9`, `sles-15.3`, `opensuse-leap-15.5`; 언어는 빈 문자열), `chan`(채널: `baseos`, `rhel_eus/baseos`, `pro` …), `name`(정규화한 이름: PyPI는 PEP 503, npm·Go·Packagist·NuGet은 소문자), `by_source`(0 바이너리 이름, 1 소스 패키지 이름).
- `auth`: `d` 배포판 공지 / `e` 생태계 공지. 패키지 관리자가 설치한 것은 `d`만, 언어 lock에서 나온 것은 `e`만 씁니다.
- `status`: `a` 범위 안이면 영향 / `n` 판과 무관하게 영향 없음(범위는 `i0`).
- `scheme`: `deb rpm apk semver pep440 go maven`은 비교기가 있고, `gem nuget composer cran hackage opam actions`는 범위만 담습니다.
- `range`: 이벤트를 `|`로 잇습니다 — `i<판>` 들어옴, `f<판>` 고침, `l<판>` 마지막 영향, `m<판>` 한계(`i0`은 처음부터). 예: `i0|f1:3.0.1-47.el9_1`. 구간이 여럿이면 이벤트 순서와 무관하게 "시작 뒤 가장 가까운 끝점"으로 계산합니다. 정확판 목록은 `=판1,판2`(범위가 하나도 없을 때만). rpm 판은 `epoch:version-release`이며 설치 판에 epoch가 없으면 판정하지 않고 "모름"입니다. Go 판은 `v` 접두를 붙입니다.

## 5. SQLite

정본에서만 만들어집니다(`bongsu-secdb-build load`로 누구나 재생성). `meta(k,v)`에 `format`·`bundle`이 있습니다.

```sql
CREATE TABLE source(sid INTEGER PRIMARY KEY, name TEXT UNIQUE, license TEXT, url TEXT);
CREATE TABLE adv(aid INTEGER PRIMARY KEY, id TEXT UNIQUE, sid INT, modified INT, withdrawn INT, cves TEXT);
CREATE TABLE pkg(pid INTEGER PRIMARY KEY, type TEXT, ns TEXT, dist TEXT, chan TEXT, name TEXT, by_source INT,
                 UNIQUE(type, ns, dist, name, by_source, chan));
CREATE TABLE rule(pid INT, aid INT, auth INT, status INT, scheme INT, rng TEXT,
                  PRIMARY KEY(pid, aid, rng, status)) WITHOUT ROWID;
```

정수 코드: `auth` 0 배포판·1 생태계, `status` 0 영향·1 영향 없음, `scheme` 0 deb·1 rpm·2 apk·3 semver·4 pep440·5 go·6 maven·7 gem·8 nuget·9 composer·10 cran·11 hackage·12 opam·13 actions.

## English summary

Release assets per bundle: `bsdb-<bundle>-base-<n>.jsonl.xz` (canonical sorted JSONL) and `bsdb-<bundle>-<n>.sqlite.zst`, plus a signed `manifest.json` (Bongsu `SignedEnvelope`, purpose `bundle`, CJ1 canonical JSON, Ed25519) and `SHA256SUMS`. The manifest carries a chained serial (`prev_manifest_digest`), per-file size/SHA-256, per-source license/fetch receipt, and the exclusion list. `latest.json` is an unsigned pointer; trust comes only from verifying `manifest.json` against a pinned key. The JSONL has a header line then lines `a` (advisory), `r` (rule) and `s` (source) sorted by tag then fields; the SQLite file is derived solely from the JSONL. Rule lookup keys are `(type, ns, dist, chan, name, by_source)`; verdicts use the distro's own fixed `epoch:version-release` or a version-independent "not affected" status.
