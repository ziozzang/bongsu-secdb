# bongsu-secdb — 봉수 공개 취약점 자료

봉수 자산 점검 도구가 사용하는 취약점 데이터베이스입니다. 공개 원천의 공지·배포판별 판정·악용 신호를 모아 정본과 SQLite로 제공합니다.

첫 운영 공개판: [data-20261105-12](https://github.com/ziozzang/bongsu-secdb/releases/tag/data-20261105-12). 원천 54개, 대표 공지 737,010건, 판정 규칙 7,577,196개입니다. 원천 목록·건수·받은 시각·빠진 원천은 릴리스 설명과 서명 manifest에 있습니다.

공급사 공지를 일반 패키지에 붙이는 오류를 피하려고 Root·TuxCare·Seal·Red Hat Lightwell은 수정 노드 배포 전까지 공개판에서 뺐습니다. 기존 로컬판 전용 원천도 공개하지 않습니다. Debian을 포함한 나머지 공개 원천을 사용합니다.

| 자산 | 내용 |
|---|---|
| `bsdb-1.jsonl.xz` | 정렬된 정본 |
| `bsdb-1.sqlite.zst` | 같은 내용의 조회용 SQLite |
| `bsdb-1-appendix.*` | 전체 설명·완화·조치의 선택 부록 |
| `manifest.json` | 원천·건수·파일 지문과 Ed25519 서명 봉투 |
| `SHA256SUMS` | 자료 파일과 manifest의 SHA-256 |
| `bongsu-secdb.pub` | 운영 공개키 |

공개키와 고정 방법은 [trust/](trust/README.md)에 있습니다. 공개키 32바이트 SHA-256은 `6f94eeff71eef8a9fc1440299f010b139230786aa5fd89094b0a39789d3fbf28`입니다. 저장소 밖 경로로도 대조해 키를 고정하세요. `test-*`는 시험 키로 만든 과거 시험판입니다.

```sh
bongsu-secdb-build verify --dir ./자료폴더 --pubkey "$(cat ./자료폴더/bongsu-secdb.pub)" --cases
```

정기 실행 설정은 하루 두 번(KST 05:30·17:30)이며 운영 자동화 PR 검토 중입니다. 실제 설치 전에는 자동 갱신이 보장되지 않습니다. 검증에 실패하면 새 자료를 게시하지 않습니다.

데이터: [CC BY-SA 4.0](LICENSE-DATA). 빌더 코드: [AGPL-3.0-only](LICENSE-CODE). [원천 출처와 표기](NOTICE.md). 각 원천의 자료를 봉수 형식으로 변환했으며 원천 제공자의 보증·인증을 뜻하지 않습니다.

Copyright Jioh L. Jung <jung@jioh.net>
