# trust — 서명 공개키

릴리스의 `manifest.json`은 봉수 서명(Ed25519)으로 서명됩니다. 이 폴더에는 서명 공개키(`bongsu-secdb.pub`, 소문자 16진 64자 한 줄)와 그 지문(공개키 32바이트의 SHA-256, 소문자 16진)을 **첫 릴리스와 함께** 올립니다. 아직 올라와 있지 않으면 서명키가 만들어지기 전입니다.

받는 쪽은 이 파일을 믿지 말고 **지문을 저장소 밖의 경로로 한 번 대조해 고정**한 뒤, 고정한 키로만 `manifest.json`의 서명을 검증하십시오. 키를 바꿀 때는 새 키를 이전 키로 서명한 안내를 이 폴더와 README에 함께 게시합니다.

The release `manifest.json` is signed with Ed25519 (Bongsu envelope). The public key and its fingerprint (SHA-256 of the 32 key bytes) will be published here with the first release. Pin the fingerprint out of band and verify `manifest.json` only with the pinned key.
