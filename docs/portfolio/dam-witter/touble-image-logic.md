---
sidebar_position: 3
---

# 이미지 업로드 로직 변경

- **문제**: Cloudflare 이미지 등록 시간으로 인해 **게시글 등록 시간이 지연**되는 문제 발생.
- **해결**: 이미지를 선택할 때 **Preview Image**를 먼저 보여주고, 동시에 이미지를 Cloudflare에 업로드하는 로직으로 변경하여 게시글 등록 시간을 단축했습니다.
  - [관련 풀리퀘스트](https://github.com/j2h30728/dam-witter/pull/3)
