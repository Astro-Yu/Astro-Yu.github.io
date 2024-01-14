---
title: "[Flask] RESTful API 구현 <3>"
date: 2024-01-04 21:36:00 +09:00 # 시간
categories: [Flask]
published: true
tags: [python, flask, web, backend, server]
image: /assets/flask.png
---
## 5. 각각 다른 모델 적용시키기

내가 구상한 프로젝트에선 각각 다른 3개의 모델을 제공해야하기 때문에 각각 다른 곳으로 연결할 버튼 3개, 테스트 모델 3개로 늘려서 코드를 다시 작성해보았다.

### 5.1 app.py

변경점만 적어보자면 다음과 같다

- 최초 실행 경로를 home() 으로 정의 후 버튼 3개를 누를 수 있도록 html을 작성했다.
- 버튼 3개마다 다른 경로로 이동할 수 있도록 입력경로3개, 출력경로 3개를 작성했다.
- 각기 다른 모델을 작성했다.

```python
@app.route('/')
def home():
    return render_template('home.html')

@app.route('/food')
def food_input():
    return render_template('food_input.html') # 입력 app.route를 2개 더 정의했다.

@app.route('/food/result', methods = ['POST', 'GET'])
def food_result():
    if request.method == 'POST':
        userInput = request.form['input']
        returnedData = food_model.printData(userInput)
        return "주문하신 " + returnedData['name'] + "은(는) " + str(returnedData['price']) + "원 입니다."
    else:
        return "잘못된 입력입니다." ## 결과 app.route를 2개 더 정의했다.

## --- 이하 생략 --- ##
```

### 5.2 home.html

홈 경로에서 보여줄 버튼 3개를 나열한 html을 작성했다

```html
<html>
   <body>
   
      <button type="button" class="food-button" onclick="location.href='/food'">Food</button>

      <button type="button" class="drink-button" onclick="location.href='/drink'">Drink</button>

      <button type="button" class="dessert-button" onclick="location.href='/dessert'">Dessert</button>
      
   </body>
</html>
```

이전 작성한 `submit`과는 다르게 `button` 타입으로 정의했고, `onclick` 에서 버튼을 누를 시 이동할 경로를 지정해주었다.

---

## 6. 로컬 서버 재실행

![](/assets/flask3_1.png)

홈 경로. 깔끔하게 3개 버튼이 있는 것을 확인 가능하다.

![](/assets/flask3_2.png)

Drink 버튼을 누를 시 이동하는 페이지. 경로가 입력한대로 (`/drink`) 나오는 것을 볼 수 있다..

![](/assets/flask3_3.png)

Coke를 작성하고 제출 시 이동하는 페이지. 경로가 입력한대로(`/drink/result`) 나오는 것이 보이고, 출력 결과도 원하는대로 됐다.

## 7. 개선할만한 점

이 테스트를 하면서 특정 html을 자주 사용했다. 하지만 경로가 달라지기 때문에 각기 다르게 만들어주었다. 

내 생각엔 주소 부분을 변수로 받아서 이 반복작업을 줄일 수 있을것 같은데 이건 다음에 알아보는걸로…