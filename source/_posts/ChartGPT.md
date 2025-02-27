---
title: 使用自然语言配合GPT生成图表
mathjax: true
tags:
- Echarts
- ChatGPT
- Prompts
categories:
- 摸鱼记录
---

<p align="center">
    <img src="https://s2.loli.net/2023/05/10/LDUpFzo7VJiOshd.png" alt="ChartGPT logo" width=200 height=200 />
</p>

使用自然语言描述生成ECharts图表。使用了Flask作为后端，并使用了Ajax进行数据的前后端交互，不涉及数据库，是Flask和前端js的一个简单学习。该摸鱼产品受启发于ChartGenie，并在此基础上对常用的Web小产品开发进行了实践，也算是这一段时间做的比较有意思的东西。

<!-- more -->

## 原理

Flask充当后端，前端使用Ajax和ECharts刷新页面。整个流程是这样的：在前端页面中我按下一个按钮，这个按钮按下之后，首先通过js的onclick函数绑定事件，然后使用ajax请求某个路由页面，比如说`/ajax`：

```javascript
$("button").click(function () {
            $.ajax({
                url:"/ajax",
                type:"post",
                data:{"name":"李四","score":99},
                success:function (d) {
                    // 你要实现的功能 
                },
                error:function () {
                    alert("发送ajax请求失败")
                }
            })
        })
```

那么我们现在要做的事情就是把数据放到这个路由页面进行请求：在Python中我使用一个函数，比如说`Get_reponse()`这个通过GPT通信返回响应回答的数据，我们把得到的结果放在`ajax路由页面`：

```python
@app.route('/ajax', methods=["get","post"])
def Get_reponse(message):
    ...
    return response
```

还剩下最后一个问题：怎么从前端获取message消息然后将其作为参数传入Python函数？我的做法是在js中获取对应的元素，然后用结合上面的`$.ajax`方法把它post到路由，比如说post到`\ajax`：

```javascript
$("#generate").click(function () {
  var Text = document.getElementById('user-input').value;
  var input = {
      'user-input': `${Text}`,
  };

    $.ajax({
          url: '/ajax',
          type: 'POST',
          data: JSON.stringify(input),
          dataType: "json",
          contentType: "application/json",
          success:function (d) {
              console.log("数据提交成功")
          },
          error:function (d) {
              console.log(d)
              alert("提交数据失败，请检查代理设置或余额。")
          }
      })
```

然后Python的Flask端写入处理函数：

```python
@app.route("/ajax",methods=['POST','GET'])
def ajax():
    global user_input
    user_input = request.json
    print(user_input['user-input'])
    return user_input
```

以及用于f返回GPT回复的函数：

```python
@app.route("/json",methods=['GET', 'POST'])
def response():
    global response_
    response = Bot.send(user_input['user-input'])
    
    response_ = Bot.parse_reply(response)
    print(response_)
    return response_
```

此时前端的Javascript需要通过GET方法获取返回的json数据：

```javascript
  $.ajax({
      url:"/json",
      type:"get",
      async: true,
      timeout: 60000,
      beforeSend:function(){
        document.getElementById('user-input').disabled = true
        document.getElementById('generate').disabled = true
        document.getElementById('loader').style.display = 'flex'
    },
      success:function (data) {
          document.getElementById('user-input').disabled = false
          document.getElementById('generate').disabled = false
          document.getElementById('loader').style.display = 'none'
          chart.clear()
          chart.setOption(optool)
          chart.setOption($.parseJSON(data));
      },
      error:function (data) {
          alert("获取返回json失败，console已记录返回数据")
          console.log(data)
      },
      complete:function(){
        document.getElementById('user-input').disabled = false;
        document.getElementById('generate').disabled = false
        document.getElementById('loader').style.display = 'none'
      }
  })
```

最关键的是beforeSend的部分，它用于显示一个等待动画，并且同时等待python后端处理完数据后再关闭掉动画，并设置Echarts样式。

## UI设计

初版UI设计成这样：

![image-20230515154217612](https://s2.loli.net/2023/05/15/zPNB1HucS6qy2Fv.png)

后来懒了，最下面那三行全都去掉了。

## 成品

![image-20230515154756433](https://s2.loli.net/2023/05/15/J6vqwbrpC5RYAzB.png)

完整代码在Github上：[Dafeigy/ChartsGPT: 使用GPT生成ECharts图表 (github.com)](https://github.com/Dafeigy/ChartsGPT)
