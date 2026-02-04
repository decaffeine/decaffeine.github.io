---
layout: post
author: Hyejin Kim
tags: [openclaw]
---

### Backgrounds
- M1 맥미니 + 16GB RAM 
- openclaw + codex5.2 (클라우드 LLM) 은 무난히 성공

### Problem
- 하지만 로컬 LLM에 연결하려고 하자 (ollama 사용) 매번 openclaw가 먹통이 됨
- 해당 스펙에서 가볍게 돌릴 수 있다는 llama3.2:3b를 설치했음에도 계속 동일 현상 발생
- ollama를 통해 llama3.2에 곧바로 메세지를 보내봤을 때 (예: "hi", "hello 123") 에는 빠르게 응답 제공됨
- openclaw의 문제로 추정

### Cause 
- openclaw가 단순히 메세지 텍스트 그 자체만 보내는 것이 아니라, 시스템 프롬프트, 메모리, 대화 히스토리 등등을 다 합쳐서 보내는 것으로 보임
- openclaw 설정에서 모델별 context Window 크기 확인
   - 기존 16384 -> 8192로 수정함 
      - 직접 openclaw.json 을 수정하거나, 사실 그냥 openclaw에게 특정 모델의 contextWindow를 수정해달라고 하면 됨
   - openclaw gateway restart 로 재기동
- 이제 llama3.2로 모델을 바꿔도 정상적으로 응답이 온다.
- ~~혹은 처음부터 더 큰 RAM의 맥미니를 산다.~~


---
{: data-content="footnotes"}