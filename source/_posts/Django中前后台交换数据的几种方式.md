---
title: Django中前后台交换数据的几种方式
date: 2017-04-05 15:14:52
tags:
categories:
description:
---
Django中前台HTML页面如何接收来自后台的数据并显示？

<!--more-->
## 返回HTML页面

这是最简单的后台向前台传送数据的方式：
```
from django.shortcuts import render

def go_index(request):
    return render(request, 'index.html')
```
返回`index.html`的全部内容，实现页面跳转的时候可以使用。


## views >>> HTML

后台传递一些数据给html，直接渲染在网页上，没有复杂的数据处理（如果前台要处理数据，需要传数据给JS处理）

Django后台`views.py`中代码：
```
# Django代码
from django.shortcuts import render

def main_page(request):
    data = [1,2,3,4]
    return render(request, 'index.html', {'data': data})
```

前台HTML中，用``{{ data }}``来获取数据。这里的`data`和`views.py`中`{'data': data}`的单引号内的名称相同。
```vbscript-html
<div>{{ data }}</div>
```

如果传回的是可迭代的数据类型，也可以在前台中进行迭代：
```vbscript-html
{% for item in data%}
<p>{{ item }}</p>
{% endfor %}
```



## views >>> Javascript

传递给JS处理需要注意两点：
- `views.py`中返回的函数中的值要用 `json.dumps()` 处理
- 在网页上要加一个 safe 过滤器。

views.py ↓
```
import json
from django.shortcuts import render
 
def main_page(request):
    list = ['view', 'Json', 'JS']
    return render(request, 'index.html', {'List': json.dumps(list), })
```

JavaScript部分↓
```javascript
var List = {{ List|safe }};
```

## JavaScript Ajax 动态刷新页面

如果想要局部获取并将数据显示出来，可以使用Ajax技术。网页前台使用Ajax发送请求，后台处理数据后返回数据给前台，前台不刷新网页动态加载数据。

views.py ↓
```
def scene_update_view(request):
    if request.method == "POST":
            name = request.POST.get('name')
            status = 0
            result = "Error!"
            return HttpResponse(json.dumps({
                "status": status,
                "result": result
            }))
```

JS 代码：
```javascript
function getSceneId(scece_name, td) {
    var post_data = {
        "name": scece_name,
    };

	$.ajax({
	    url: {% url 'scene_update_url' %},
		    type: "POST",
	        data: post_data,
	        success: function (data) {
		        data = JSON.parse(data);
		        if (data["status"] == 1) {
			        setSceneTd(data["result"], scece_name, td);
		        } else {
				    alert(data["result"]);
			    }
		    }
    });
} 
```

JS 发送ajax请求，后台处理请求并返回`status`, `result`
在` success:` 后面定义回调函数处理返回的数据，需要使用 `JSON.parse(data)`


<!--more-->
