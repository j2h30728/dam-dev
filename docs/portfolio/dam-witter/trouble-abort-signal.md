---
sidebar_position: 3
---

# 불필요한 이미지 등록

- **문제**: 사용자가 연속적으로 이미지를 변경할 때 **불필요한 이미지가 Cloudflare에 등록**되는 문제가 발생.
- **해결**: **Abort Signal**을 사용하여 이전 이미지 요청을 취소함으로써 불필요한 이미지 등록을 방지했습니다.
  - [관련 풀리퀘스트](https://github.com/j2h30728/dam-witter/pull/74)
