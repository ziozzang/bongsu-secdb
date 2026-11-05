# 봉수 공개 자료 서명키

운영 공개키는 [bongsu-secdb.pub](bongsu-secdb.pub)입니다. 시험판 `test-*`의 키와 다릅니다.

공개키 32바이트의 SHA-256 지문:

```
6f94eeff71eef8a9fc1440299f010b139230786aa5fd89094b0a39789d3fbf28
```

이 지문을 저장소 밖 경로로도 확인한 뒤 키를 고정하세요. 릴리스 `manifest.json`은 정규 JSON(CJ1)과 Ed25519를 쓰는 봉수 서명 봉투입니다. 서명은 manifest 안에 있으므로 별도 `manifest.sig`는 없습니다. SHA256SUMS만으로는 서명 검증을 대신하지 못합니다.

공개키를 바꿀 때에는 이전 키로 서명한 안내를 함께 제공합니다.

Copyright Jioh L. Jung <jung@jioh.net>
