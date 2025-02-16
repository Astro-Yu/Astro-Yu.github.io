---
title: "[Flask] RESTful API 구현 <2>"
date: 2024-01-03 21:27:00 +09:00 # 시간
categories: [Flask]
published: true
tags: [python, flask, web, backend, server]
image: /assets/flask.png
---
## 4. 코드 작성

Flask 설치나 기본 설정 같은건 다른 블로그에 많으니 여기서는 다루지 않겠다.

처음 기본적으로 구상해본건 사용자 입력을 받을 input과 간단한 테스트 Model, 그리고 결과를 보여줄 output.

기능 분리는 아직 하지말고 일단 기능이 돌아가는지 작성해보자

### 4.1 app.py

만약 Flask를 아무런 설정 없이 시작한다면 `app.py` 라는 이름의 파일을 제일 먼저 찾아서 실행시킨다고 한다. 그렇기에 input과 output에 대한 코드를 서버 실행시 가장 먼저 실행되게 이 곳에 작성했다.

```python
from flask import Flask, request, render_template
from Model import model

app = Flask(__name__)

@app.route('/')
def input():
    return render_template('input.html')

@app.route('/result', methods = ['POST','GET'])
def result():
    if request.method == 'POST':
        userInput = request.form['input']
        returnedData = model.printData(userInput) # json으로 받은 데이터
        return "주문하신 " + returnedData['name'] + "은(는) " + str(returnedData['price']) + "원 입니다."
    else:
        return "잘못된 입력입니다."

if __name__ == '__main__':
    app.run(host="127.0.0.1", port="8080")
```

`input` 은 사용자 입력을 받는다. return으로 특정 html을 불러오는데 이 곳에서 사용자 입력을 받는다.

경로는 최초 실행 페이지에서 볼 수 있게 `‘/'`로 지정했다.

`result`는 반환되는 데이터를 출력한다. 이 코드를 자세히 살펴보면

`/result` 경로에서 아래 함수가 실행되고, 이 페이지에서 처리 가능한 요청(?)은 `POST`와 `GET`이다.

만약 요청이 `POST` 인 경우, if문 아래를 실행한다.

`userInput`은 요청 form 에서 key가 input인 것을 받아온다. 이것은 사용자 입력 데이터이다.

사용자 입력을 모델에 넣고 반환되는 값을 적절히 수정해 반환한다.

### 4.2 input.html

```html
<html>
   <body>
   
      <form action = "http://localhost:8080/result" method = "POST">
         <p>Input <input type = "text" name = "input" /></p>

         <p><input type = "submit" value = "submit" /></p>
      </form>
      
   </body>
</html>
```

html은 잘 모르기 때문에 대강 추측하자면

`action = "http://localhost:8080/result" method = "POST"`

는 이 페이지에서 해당 action을 취했을 시 넘어가는 URI를 지정해주는 것 같다.

`<p>Input <input type = "text" name = "input" /></p>`

는 사용자 입력을 받는 곳이다. 사용자 입력의 type은 text, form 에서의 key는 ‘input’ 이다.

`<p><input type = "submit" value = "submit" /></p>`

는 제출 버튼이다. 이 버튼을 누르면 위의 action의 URI로 이동하는 듯 하다.

### 4.3 model.py

이번 테스트에서 사용할 간단한 model이다.

```python
menuList = [
    {"id" : 1, "name" : "Pasta", "price" : 10000},
    {"id" : 2, "name" : "Gukbab", "price" : 8000},
    {"id" : 3, "name" : "Hamburger", "price" : 5000},
    {"id" : 4, "name" : "Sushi", "price" : 50000},
    {"id" : 5, "name" : "Coke", "price" : 1000},
    ]

def printData(userInput):
    for idx in range(len(menuList)):
        if (menuList[idx]['name'] == userInput):
            data = menuList[idx]

    return data
```

따로 db는 없기 때문에 model에 데이터를 json 형식으로 정의했다.

유저 인풋을 받아 함수 사용시 json 데이터를 반환한다.

## 5. 로컬 서버 실행

터미널에서 로컬 서버를 실행해보자.   

![](/assets/flask2_1.png)

따로 설정을 하지 않았다면 app.py를 실행하는 것으로 서버 실행이 가능하다.   

홈 경로( / ) 에 입력 칸과 제출 버튼이 보인다.

![](/assets/flask2_2.png)

이 곳에 Sushi를 입력해보자.

![](/assets/flask2_3.png)

경로가 이동되고( /result ) 원하는 답이 나왔다.

다음번 글에선 제출 버튼을 여러개 만들고 각각 다른 모델을 적용해 다른 경로로 이동되게 만들어보겠다.