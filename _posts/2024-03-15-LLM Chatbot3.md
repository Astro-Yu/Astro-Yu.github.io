---
title: "[AI] LLM 기반 챗봇 2 : LLM"
date: 2024-03-15 20:35:00 +09:00
categories: [AI]
published: true
tags: [python, ai, llm, chatbot, langchain]
image: /assets/langchain.webp
use_math: true
---

*[Caution] 해당 게시글은 유튜브 **모두의 AI** 님의 영상 정리와 제 사견이 들어있습니다.*

## 0. 이전 글

[0 : 소개](https://astro-yu.github.io/posts/LLM-Chatbot1/)

[1 : Langchain](https://astro-yu.github.io/posts/LLM-Chatbot2/)

## 1. LLM이란?

LLM은 Large Language Model 의 약자이다. LLM은 다양한 자연어 처리(NLP) 작업을 수행할 수 있는 딥 러닝 알고리즘이다.

현재 많은 자연어 처리 모델은 대부분 Transformer 아키텍처를 기반으로 하고 있다.

### 1.1 Transformer 아키텍쳐

트랜스포머 아키텍처는 Encoder 와 Decoder로 이루어져 있고, 

Encoder의 경우 **기계에게 자연어를 잘 이해시키기 위한** 역할을 한다.

Decoder는 **언어를 의도대로 잘 출력하기 위한** 역할을 한다.

![](/assets/llm.webp)

[사진 출처](https://www.interconnects.ai/p/llm-development-paths)

왼쪽 분홍색 트리는 Encoder로만 만든 모델이다.

오른쪽 청색 트리는 Decoder로만 만든 모델이다. 우리에게 친숙한 GPT가 이쪽에 있다.

가운데 녹색 트리는 두 가지를 모두 사용해 만든 모델들이다.

트리의 크기를 보면 알겠지만 Decoder 중심으로 많은 모델들이 빠르게 발전하고 있다.

의문점: 단순하게 생각해보면 Encoder과 Decoder를 둘 다 활용해 만든 LLM이 뛰어날 것 같은데 현실은 Decoder 기반으로 발전이 이루어졌다. 이게 뜻하는 의미가 뭔지? 

1. LLM이 사실 별 다른 처리 없이 자연어를 잘 받아들인다는 의미?
2. 그게 아니면 LLM이 이해하기 쉽게 정보를 가공하는 과정이 의외로 쉽다는 의미?
3. 빅테크조차 encoder와 decoder를 둘 다 활용하기에는 버겁다는 의미?

## 2. Closed vs Open

빅테크에서 만들어 내놓은 LLM들은 크게 2가지 종류가 존재한다. 표로 정리하면 다음과 같다.

| 종류 | Closed Source | Open Source |
| --- | --- | --- |
| 예시 | GPT 시리즈, PALM, Bard | LLaMa, Alpaca, MPT-7B |
| 장점 | 뛰어난 성능, API 호출 방식으로 인한 편리함 | 크게 뒤떨어지지 않는 성능, 높은 보안, 낮은 비용 |
| 단점 | 보안의 취약함. API 호출 비용 | 사용 난이도 높음, GPU 를 탑재한 서버 필요 |

본인이 쓸만한 GPU를 갖고있거나, 자금의 여유가 없거나, 데이터 보안이 중요하거나 ,스스로 코드를 뜯어서 고칠 수 있는 실력이 된다면 Open Source를 사용하면 좋다.

(실력이 된다면 목적에 맞는 파인튜닝도 가능하다.)

하지만 자금에 여유가 있고, 빠르게 개발을 하고 싶다면 Closed Source를 사용하면 좋을 것 같다.

나의 경우 gpu를 가진 서버가 없기 때문에(개인 pc를 24시간 켜둘 순 없으니까…) Chat-GPT 3 혹은 3.5를 사용할 예정이다.