---
title: "[AI] LLM 기반 챗봇 0 : 소개"
date: 2024-03-14 04:01:00 +09:00
categories: [AI]
published: true
tags: [python, ai, llm, chatbot, langchain]
image: /assets/langchain.webp
use_math: true
---
## 1. 개요

개인 프로젝트로 진행할 챗봇 프로그램이다. 평소에 AI 관련 프로그램을 작성해보고 싶었고, 최근 이쪽 분야가 굉장히 활발히 개발되는 것으로 알고 있어서 해봐서 나쁠건 없을 것 같다.

기본적으로 오픈소스 LLM을 활용해 내가 따로 준비한 데이터를 학습시키는 RAG(Retrieval-Augmented Generation)을 시도해볼 계획이다.

## 2. 학습 계획

유뷰트에 있는 **모두의 AI**님의 Langchian 소개 영상을 기본으로 학습할 계획이다.

기본적으로 LLM Chatbot을 구현하기 위한 기본 지식에는

- Langchain
- LLM
- Document Loaders
- Retrieval Text Splitters
- Retrieval Text Embeddings
- Vectorstores
- Retrievers

등이 있고, 좀 많긴 하지만 알고 코드를 작성하는것과 모르고 코드를 작성하는데는 하늘과 땅 만큼의 차이가 있으니까 일단 모두 학습하면서 코드를 작성할 계획이다.