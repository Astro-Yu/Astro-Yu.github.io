---
title: "[AI] LLM 기반 챗봇 1 : LangChain"
date: 2024-03-14 19:09:00 +09:00
categories: [AI]
published: true
tags: [python, ai, llm, chatbot, langchain]
image: /assets/langchain.webp
use_math: true
---

*[Caution] 해당 게시글은 유튜브 **모두의 AI** 님의 영상 정리와 제 사견이 들어있습니다.*

## 0. 이전 글

[0 : 소개](https://astro-yu.github.io/posts/LLM-Chatbot1/)

## 1. LangChain이란?

랭체인(LangChain) 은 대형 언어 모델(LLM)을 사용해 애플리케이션을 쉽게 개발하고, 지원하는 프레임워크이다. 최근 ChatGPT는 물론, LLaMA나 Alpaca 같은 LLM이 굉장히 핫해지면서 동시에 이 언어 모델들을 사용할 수 있는 LangChain을 활용해 많은 어플리케이션들이 만들어지고 있다.

즉 LangChain은 언어모델을 더 잘 활용할 수 있게끔 하는 도구이다.

## 2. LangChain을 사용해야 하는 이유?

1. 인터넷 정보 검색이 제한되어 있다.
    
    ChatGPT의 경우 2021년 까지의 데이터만으로 학습되어있고, 스스로 인터넷 검색을 하지 않기 때문에 그 이후에 나온 정보에 대한 지식은 없다. 일례로 LangChain 자체를 ChatGPT에 물어보면 답변해주지 않는다.
    
2. 토큰 제한
    
    한번에 입력할 수 있는 양이 제한되어 있다. 즉 거대한 데이터를 입력하는데 문제가 있다는 뜻이다.
    
3. 환각 현상
    
    ChatGPT를 한번이라도 써 봤으면 가끔 답변 중 거짓말을 하거나, 애매하게 이상하게 대답하는 경우를 봤을 것이다. 만약 정보를 제공하는 챗봇을 만든다면 해당 현상은 치명적이다.
    

이러한 한계들을 LangChain을 활용해 LLM을 개량하면서 해결할 수 있다.

***In-context Learning***을 통해 LLM에게 특정 문맥(데이터)를 제공해, 이 문맥을 기반으로 LLM이 답변할 수 있도록 만들 수 있다.

## 3. LangChain의 구조

- LLM - Large Language Model
    
    초거대 언어 모델로, 생성 모델의 엔진과 같은 역할을 하는 핵심 구성요소다.
    
    GPT 3.5, PALM-2, LLAMA, StableVicuna 등이 존재한다.
    
- Prompts
    
    LLM에게 지시하는 명령문
    
    ex) Prompt Templates, Chat Prompt Template, Output Parsers…
    
- Index
    
    LLM이 문서를 쉽게 탐색할 수 있도록 구조화 하는 모듈
    
    ex) Document Loaders, Text Splitter, Vectorstores, Retrievers …
    

- Memory
    
    채팅 이력을 기억하도록 하여, 이를 기반으로 대화가 가능하도록 하는 모듈
    
- Chain
    
    LLM 사슬을 형성하여, 연속적인 LLM 호출이 가능하도록 하는 핵심 구성 요소
    
    ex) Question Answering, Summarization …
    
- Agents
    
     LLM이 기존 Prompt Template 로 수행할 수 없는 작업을 가능하게 하는 모듈.
    
    ex) 웹 검색, SQL query 등을 작성
    

## 4. PDF 챗봇 구축의 예시

1. 문서 업로드
    
    **Document Loader**를 통해 context가 될 PDF를 업로드한다.
    

1. 문서 분할
    
    GPT의 토큰 제한을 피하기 위해 **Text Splitter**를 활용해 문서를 분할한다.
    

1. 문서 임베딩
    
    LLM이 이해할 수 있게 문서를 숫자로 만든다. 다차원 벡터의 형식으로 만들어 **vectorstore**에 저장한다.
    
2. 임베딩 검색
    
    사용자가 입력한 prompt와 가장 유사한 벡터값을 찾아 ( = 가장 연관성이 높은) 그 문장이 들어있는 chunk를 가져온다.
    
    (여기서 chunk는 LLM의 토큰 제한에 걸리지 않게 Text Splitter를 통해 나눈 문단이다.)
    
    다차원 공간에서 해당 벡터값과 거리가 가장 가까운 벡터를 찾는 과정이다.
    

1. 답변 생성
    
    가져온 데이터를 바탕으로 LLM Chain을 형성해 다중으로 가공해 답변하도록 수정한다.