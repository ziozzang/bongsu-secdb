# 형식 문서: 릴리스 색인, manifest, 정본, SQLite

> 이 문서는 빌더(`bongsu-secdb-build`) 2판이 실제로 만드는 파일의 정확한 형식입니다. README의 표와 이 문서가 같은 이름을 씁니다(서명은 별도 `manifest.sig` 없이 `manifest.json` 한 파일에 서명 봉투로 들어갑니다).

형식 이름: manifest `bongsu-secdb/2`, 정본·SQLite `bsdb-2`. 형식이 바뀌면 이름의 숫자를 올립니다. 1판(`bsdb-1`)은 공개된 적이 없습니다.
(English summary at the end.)

## 1. 릴리스와 파일 이름

릴리스 태그는 `v<순번>-<UTC 날짜시각>`(예: `v42-20261031T060000Z`)입니다. 첨부 파일은 묶음마다 둘이며 데이터가 없는 묶음은 생략됩니다.

묶음: `lang`(언어 생태계 규칙), `malicious`(악성 패키지 규칙), `os-apk` `os-deb` `os-rpm` `os-ubuntu`(배포판 규칙과 그 배포판 주체의 판정), `vuln`(취약점 목록: 대표 공지·별칭, NVD·생태계 주체의 판정, 악용 신호. 규칙은 없음), `desc`(설명 문장과 대표 공지에의 연결. 화면에 보일 때만 쓰므로 따로 뗐다).

| 이름 | 내용 |
|---|---|
| `bsdb-<묶음>-base-<순번>.jsonl.xz` | 정본 JSONL(xz, 단일 스트림) |
| `bsdb-<묶음>-<순번>.sqlite.zst` | 정본에서 만든 SQLite(zstd, 긴 창 27비트) |
| `manifest.json` | 서명 봉투(아래 2) |
| `SHA256SUMS` | `<sha256>␣␣<이름>` 줄, 이름순. 데이터 파일과 `manifest.json` |

git에는 `latest.json`(아래 3)과 문서만 커밋합니다. 데이터는 git 이력에 넣지 않습니다.

## 2. manifest.json (서명됨)

봉수의 서명 봉투(`SignedEnvelope`)입니다: `{envelope_version, purpose:"bundle", key_id, payload_base64, signature_base64}`.
`payload_base64`는 manifest 본문을 **CJ1**(ASCII 객체 키 오름차순·공백 없는 UTF-8·정수만, 절댓값 ≤ 2^53−1)로 직렬화한 바이트입니다. 그래서 모든 수치는 정수입니다(점수도 ×10).
서명은 Ed25519이고 서명 대상은 도메인 접두 `bongsu-envelope-v1\0` 뒤에 (purpose, key_id, payload) 각각을 8바이트 빅엔디언 길이와 함께 이어 붙인 바이트입니다(봉수 `bongsu-wire`의 `signing_bytes`와 같음).
`key_id`는 공개키 32바이트의 SHA-256(소문자 16진)입니다.

본문 필드:

| 필드 | 뜻 |
|---|---|
| `format` | `bongsu-secdb/2` |
| `serial` | 발행 순번(1부터, 한 번에 1씩) |
| `prev_manifest_digest` | 직전 manifest 본문(CJ1)의 SHA-256. 첫 발행은 빈 문자열 |
| `built_at` | 빌드 시각(유닉스 초) |
| `builder` | 빌더 이름·판 |
| `data_license` | `CC-BY-SA-4.0` |
| `bundles[]` | `name`, `claims`(대표 공지), `packages`(조회 키), `ranges`(공유 범위 표), `texts`(설명 문장), `descs`(대표 공지×언어 설명 연결), `judgments`, `signals`, `rules` |
| `files[]` | `name`, `bundle`, `kind`(`jsonl-xz`·`sqlite-zst`), `bytes`, `sha256` |
| `sources[]` | `name`, `license`, `url`, 원문 `sha256`·`bytes`, `etag`, `last_modified`, `fetched_at`, `status`(`fresh`·`not_modified`·`stale`·`missing`) |
| `excluded[]` | 일부러 또는 받지 못해 빠진 원천과 이유(Chainguard·Wolfi는 항상 있음) |
| `quarantine` | 해석하지 못해 격리한 공지·대상의 이유별 개수 |
| `unsupported_schemes` | 범위만 담고 판정은 못 하는 판 체계 |
| `reduction` | 뭉치기 단계별 줄 수·바이트: 원천 레코드 수(`records`) → 대표 공지 수(`claims`, 그중 묶음 공지 `claims_bulletin`, 거부한 간선 `alias_refused`), 조회 키 정규화 전후(`keys_raw`→`keys`), 규칙(`rules_raw`→`rules`), 범위 문자열(`range_bytes_raw`→`range_bytes`), 설명(`desc_candidates`→`desc_kept`→`desc_unique`), 판정, 충돌 해소(`conflicts`) |

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

## 4. 정본 JSONL (`bsdb-2`)

UTF-8, 줄마다 JSON 하나. 첫 줄은 머리말 객체 `{"bundle":"…","format":"bsdb-2"}`이고, 나머지는 **줄 종류 글자(c < g < h < i < j < k < r < s < t) 다음 필드를 왼쪽부터** 정렬하며(숫자는 수 순서, 글은 바이트 순서) 중복이 없습니다. 어떤 줄이 가리키는 대상(대표 공지, 키, 범위, 글)은 항상 앞에 나옵니다.

| 줄 | 필드 |
|---|---|
| `["c", id, modified, withdrawn, aliases, covers]` | **대표 공지**. `id`는 CVE → GHSA → 생태계 공지 순으로 고른 대표 ID. `aliases`는 같은 취약점이라고 묶인 다른 ID(원천 공지 ID 포함, 정렬). `covers`는 **묶음 공지**(CVE를 둘 이상 담은 RHSA·DSA·USN 등)일 때만 덮는 CVE 목록이고, 묶음 공지는 합치지 않고 자기 ID가 대표입니다. `modified` 유닉스 초, `withdrawn` 0/1(대표 공지의 모든 원천 레코드가 철회일 때 1) |
| `["g", rid, range]` | **공유 범위 표**. `rid`는 0부터 이어지는 번호(범위 문자열 순서) |
| `["h", tid, hash, text]` | **설명 문장**(`desc` 묶음만). `tid`는 0부터(해시 순서), `hash`는 공백·대소문자·끝 마침표를 정규화한 문장의 SHA-256 앞 64비트(16진 16자). 문장은 300글자까지 |
| `["i", id, lang, tid]` | 대표 공지의 언어별 대표 설명(`desc` 묶음만. 이 묶음에는 `c` 줄이 없고 `id`는 `vuln` 묶음의 대표 공지를 가리킨다). 대표 선택: NVD → 생태계 공지 → 배포판 공지 |
| `["j", id, actor, dist, label, vector, score10, cwes]` | **판정**. (대표 공지, 판정 주체, 배포판 판)마다 하나, 서로 덮어쓰지 않음. 아래 4.2 |
| `["k", kid, type, ns, dist, chan, name, by_source]` | **조회 키 표**. `kid`는 0부터(키 순서) |
| `["r", kid, id, auth, status, scheme, rid, mask]` | **규칙**. 아래 4.1 |
| `["s", idx, name, license, url]` | 이 묶음에 쓰인 원천. `idx`는 규칙 `mask`의 비트 번호 |
| `["t", id, kev_added, ransomware, epss_ppm, pct_bp]` | 악용 신호(`vuln` 묶음만). `kev_added` KEV 등재일(유닉스 초, 없으면 0), `ransomware` 0/1, `epss_ppm` EPSS 확률×10⁶, `pct_bp` 백분위×10⁴ |

### 4.1 규칙

- 조회 키 `type`(purl 유형: `deb rpm apk pypi npm cargo golang maven nuget gem composer hex pub cran hackage julia opam swift githubactions`), `ns`(네임스페이스: 배포판 이름, npm `@scope`, Maven groupId …), `dist`(`debian-12`, `ubuntu-22.04`, `alpine-3.20`, `rhel-9`·`rhel-9.0`, `almalinux-8`, `rocky-9`, `sles-15.3`, `opensuse-leap-15.5`; 언어는 빈 문자열), `chan`(채널: `baseos`, `rhel_eus/baseos`, `pro` …), `name`(정규화한 이름: PyPI는 PEP 503, crates.io는 소문자와 `_`→`-`, npm·Go·Packagist·NuGet·Swift·GitHub Actions는 소문자, 배포판 패키지는 그대로), `by_source`(0 바이너리 이름, 1 소스 패키지 이름).
- `auth`: `d` 배포판 공지 / `e` 생태계 공지. 패키지 관리자가 설치한 것은 `d`만, 언어 lock에서 나온 것은 `e`만 씁니다.
- `status`: `a` 범위 안이면 영향 / `n` 판과 무관하게 영향 없음(범위는 `i0`).
- `scheme`: `deb rpm apk semver pep440 go maven`은 비교기가 있고, `gem nuget composer cran hackage opam actions`는 범위만 담습니다.
- 범위(`g` 줄): 이벤트를 `|`로 잇고 **정렬해 둡니다** — `i<판>` 들어옴, `f<판>` 고침, `l<판>` 마지막 영향, `m<판>` 한계(`i0`은 처음부터). 예: `i0|f1:3.0.1-47.el9_1`. 구간이 여럿이면 이벤트 순서와 무관하게 "시작 뒤 가장 가까운 끝점"으로 계산합니다. 정확판 목록은 `=판1,판2`(범위가 하나도 없을 때만, 정렬). rpm 판은 `epoch:version-release`이며 설치 판에 epoch가 없으면 판정하지 않고 "모름"입니다. Go 판은 `v` 접두를 붙입니다.
- `mask`: 이 규칙을 말한 원천들의 비트 집합(`s` 줄의 `idx`). 같은 규칙을 여러 원천이 말하면 하나로 합치고 마스크를 OR합니다.
- 같은 (키, 대표 공지)에서 "영향"과 "영향 없음"이 함께 나오면 원천 우선순위가 높은 쪽만 남깁니다: 배포판 자체 추적기(Debian 보안 추적기·Alpine secdb) > 배포판 OSV 변환본 > 언어 생태계 OSV. 같은 순위면 "영향"이 이깁니다. 버려진 쪽은 `manifest.reduction.conflicts`에 개수로 남습니다.

### 4.2 판정

- `actor`: 판정 주체. `nvd`, `ghsa`(GHSA 공지), `pypa`, `go`, `rustsec`, `debian`, `ubuntu`, `redhat`(Red Hat 보안 데이터 API의 CVE별 심각도), `rocky`, `almalinux`, `suse`, `opensuse`, … 배포판 주체의 판정은 그 배포판 묶음에, 그 밖의 주체(`nvd`와 생태계)는 `vuln` 묶음에 있습니다.
- `dist`: 판정이 적용되는 배포판 판(`debian-12`, `rhel-9.0`). 주체 전체에 적용되면 빈 문자열.
- `label`: 주체가 붙인 심각도 이름 **그대로**(소문자): `important`, `moderate`, `low`, `critical` …. 주체마다 말이 다릅니다.
- `vector`: CVSS 벡터(v3.x·v4.0·v2). `score10`: 기본 점수×10(예: 9.8 → 98), 없거나 계산하지 못하면 -1(NVD가 준 점수를 우선, 벡터만 있는 원천은 v3.x·v2를 계산, v4.0은 계산하지 않음). `cwes`: CWE 목록.
- 묶음 공지(`covers`가 있는 대표 공지)의 판정은 그 공지 전체에 대한 것입니다.

### 4.3 조회 규칙(권장)

노드는 대표 공지마다 판정을 하나 고릅니다. (1) 조회 대상 자신의 주체 판정: 배포판 패키지는 그 배포판 주체의 판정을 `distro=` 판 후보 순서(정확한 판 → 주 판 → 판 없음)로, 언어 패키지는 생태계 주체. (2) 없을 때만 `vuln`의 NVD → GHSA → 그 밖 순서로 대신 쓰고 **대신 썼다는 표시와 출처(주체·공지)를 함께 냅니다.** 묶음 공지는 덮는 CVE 가운데 점수가 가장 높은 값을 씁니다. 다른 배포판의 판정은 쓰지 않습니다.

## 5. SQLite

정본에서만 만들어집니다(`bongsu-secdb-build load`로 누구나 재생성). `meta(k,v)`에 `format`·`bundle`이 있습니다. `pid`·`rid`·`tid`는 정본의 `kid`·`rid`·`tid`와 같고, `cid`는 대표 공지 줄 순서의 1부터 번호입니다.

```sql
CREATE TABLE source(sid INTEGER PRIMARY KEY, name TEXT UNIQUE, license TEXT, url TEXT);   -- sid = mask 비트 번호
CREATE TABLE claim(cid INTEGER PRIMARY KEY, id TEXT UNIQUE, modified INT, withdrawn INT, aliases TEXT, covers TEXT);
CREATE TABLE descr(tid INTEGER PRIMARY KEY, hash TEXT, body TEXT);
CREATE TABLE cdesc(claim TEXT, lang TEXT, tid INT, PRIMARY KEY(claim, lang)) WITHOUT ROWID;   -- claim = 대표 공지 ID(설명 묶음에서만 채움)
CREATE TABLE judg(cid INT, actor TEXT, dist TEXT, label TEXT, vector TEXT, score10 INT, cwes TEXT,
                  PRIMARY KEY(cid, actor, dist)) WITHOUT ROWID;
CREATE TABLE pkg(pid INTEGER PRIMARY KEY, type TEXT, ns TEXT, dist TEXT, chan TEXT, name TEXT, by_source INT,
                 UNIQUE(type, ns, dist, name, by_source, chan));
CREATE TABLE rng(rid INTEGER PRIMARY KEY, rng TEXT);
CREATE TABLE rule(pid INT, cid INT, auth INT, status INT, scheme INT, rid INT, mask INT,
                  PRIMARY KEY(pid, cid, rid, status)) WITHOUT ROWID;
CREATE TABLE signal(cid INTEGER PRIMARY KEY, kev INT, ransom INT, epss INT, pct INT);
```

`aliases`·`covers`·`cwes`는 쉼표로 이은 글입니다. 정수 코드: `auth` 0 배포판·1 생태계, `status` 0 영향·1 영향 없음, `scheme` 0 deb·1 rpm·2 apk·3 semver·4 pep440·5 go·6 maven·7 gem·8 nuget·9 composer·10 cran·11 hackage·12 opam·13 actions.

## English summary

Release assets per bundle: `bsdb-<bundle>-base-<n>.jsonl.xz` (canonical sorted JSONL) and `bsdb-<bundle>-<n>.sqlite.zst`, plus a signed `manifest.json` (Bongsu `SignedEnvelope`, purpose `bundle`, CJ1 canonical JSON, Ed25519) and `SHA256SUMS`. Bundles: `lang`, `malicious`, `os-apk`, `os-deb`, `os-rpm`, `os-ubuntu` (rules plus the distro's own judgments) and `vuln` (representative advisories with aliases, NVD/ecosystem judgments, KEV/EPSS signals; no rules) and `desc` (de-duplicated description texts linked to the representative advisories; display-only, so a separate download). IDs of the same vulnerability are merged into one representative advisory (CVE preferred); advisories covering two or more CVEs (RHSA/DSA/USN) are never merged and list the CVEs they cover. Judgments (severity label as the actor wrote it, CVSS vector, score×10, CWE) are stored per (advisory, actor, distro release) and never overwritten; consumers use the target distro's judgment first and fall back to NVD/GHSA, marking the substitution and its source. Version ranges, lookup keys and description texts are shared tables referenced by number. The manifest carries a chained serial (`prev_manifest_digest`), per-file size/SHA-256, per-source license/fetch receipt, the exclusion list and the merge/reduction statistics. `latest.json` is an unsigned pointer; trust comes only from verifying `manifest.json` against a pinned key.
