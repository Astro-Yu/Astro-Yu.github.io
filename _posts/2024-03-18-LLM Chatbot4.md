---
title: "[AI] LLM 기반 챗봇 4 : Document Loaders"
date: 2024-03-18 20:29:00 +09:00
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

[2 : LLM](https://astro-yu.github.io/posts/LLM-Chatbot3/)

## 1. Document Loader란?

Document Lodar를 설명하기 이전에 이게 왜 RAG 에서 필요한지를 설명하는게 먼저이다.

### 1.1 RAG

RAG는 **Retrieval Augmented Generation**의 약자이다. **검색 증강 생성**이라는 말 그대로 외부 데이터를 참조해 LLM이 답변할 수 있도록 하는 프레임워크이다. 즉 사람이 외부 데이터 저장소를 따로 구축하고, QnA 시스템이 유사한 문장을 저장소에서 검색한 후 LLM이 가공해 답변을 한다.

그 외부 데이터 저장소를 구축하는 가장 첫 번째 단계가 Document Loader가 하는 일이다. 우리가 갖고 있는 데이터의 형태는 다양하다. PDF, docx, Web, csv, 영상 등 …

이런 다양한 요소들을 Document Loader로 가져와 Langchain에서 활용할 수 있는 요소로 가공할 수 있다.

## 2. Document Loader 구성

Document Loader를 통해 데이터를 불러오면 document 객체로 되어있어 다음과 같이 구성되어 있다.

- Page_content : 문서의 내용
- Metadata : 내용을 가져온 위치, 제목, 페이지 넘버

우리가 LLM을 통해 답변받은 내용을 아직 100% 확신할 수 없기 때문에 이런 출처를 알기 위해 Metadata가 꼭 필요하다.

document에 담긴 내용을 활용하려면 `.page_content` 를 사용하면 된다.

### 2.1 Document Loader 종류

`langchain.document_loaders`를 통해 사용할 수 있다.

1. `WebBaseLoader`
    
    웹 URL에서 내용을 얻는 방식이다. 즉 크롤링이다. 한 가지 URL 뿐 아니라 여러 URL을 지정해서 불러올 수도 있다.
    
    ```python
    from langchain_community.document_loaders import WebBaseLoader
    
    url = "https://namu.wiki/w/amazarashi"
    loader = WebBaseLoader(url)
    
    data = loader.load()
    print(data[0].page_content)
    ```
    
    `data` 의 필요한 내용 page_content만 가져오도록 출력한다.
    
    *[주의] 사용시 bs4가 설치되어 있어야 한다.*
    
2. `PyPDFLoader`
    
    pdf를 기반으로 내용을 얻어온다.
    
    ```python
    from langchain_community.document_loaders import PyPDFLoader
    
    data = PyPDFLoader("/history_of_seoul.pdf")
    
    pages = data.load_and_split()
    
    print(pages)
    ```
    
    PyPDFLoader를 통해 PDF가 위치한 경로를 가져온다. 이후 `load_and_split` 을 사용한다면 pdf 페이지 별로 구분해 리스트로 가져온다.
    
    pages[0] → 1번 페이지, pages[1] → 2번 페이지 와 같이 가져올 수 있다. 이를 원하지 않는다면 pages 단독으로 출력한다.
    
3. `Docx2txtLoader` 
    
    docx 문서를 기반으로 내용을 얻어온다.
    
    ```python
    from langchain.document_loaders import Docx2txtLoader
    
    data = Docx2txtLoader("antinomy.docx")
    
    pages = data.load_and_split()
    
    print(pages)
    ```
    
    *[주의] 사용시 docx2txt가 설치되어 있어야 한다.*
    
4. `CSVLoader`
    
    csv를 기반으로 내용을 얻어온다
    
    ```python
    from langchain_community.document_loaders import CSVLoader
    
    loader = CSVLoader(file_path = '2024-02-20_19-13-59_finalSearchList.csv', csv_args={
                'delimiter' : ',',
                'quotechar' :'"',
                'fieldnames' : ['title', 'tags']
    })
    
    data = loader.load()
    print(data)
    ```
    
    위에서 소개한 다른 Loader 들과 다르게 CSV는 `csv_args` 가 포함된다.
    
    대부분의 csv는 구분자가 `,` 이기 때문에 `delimiter`에 `,`를 적어준다.
    
    이후 `filednames`는 csv 파일의 열 이름을 적어주면 된다.
    

이외에도 공식 페이지에선 `Markdown`, `HTML`, `JSON` 도 소개하고 있다.