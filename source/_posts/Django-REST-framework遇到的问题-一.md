---
title: Django REST framework遇到的问题(一)
date: 2017-09-13 17:12:18
tags: Django Rest Framework
categories: Django Rest Framework
description:
---

第一次用django rest framework做项目，记录中间遇到的问题与解决办法。

<!--more-->

### 1、rest_framewor认证问题（如何去掉认证）

#### 问题
需要登录（获取cookie）之后才能访问rest api。

#### 问题原因
配置中设置了需要权限认证

#### 解决办法
在settings.py中注释掉即可
```python
# REST_FRAMEWORK = {
#     # 'DEFAULT_PERMISSION_CLASSES': [
#     #     'rest_framework.permissions.IsAdminUser',
#     # ],
# }
```


### 2、跨域请求问题

#### 问题
Ajax跨域请求错误：
```xml
XMLHttpRequest cannot load http://192.168.8.51:8888/hubs/?format=json. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:63342' is therefore not allowed access
```

#### 问题原因

返回的数据必须要是CORS的数据，也就是返回头部要有Access-Control-Allow-Origin这个信息。否则只能同源调用。

整个CORS通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，CORS通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。

#### 解决办法

安装django-cors-middleware，并在settings.py中作一些配置即可。

http://www.jianshu.com/p/1fd744512d83
用CORS 解决vue.js django跨域调用



### 3、admin界面无法进入

#### 错误信息
```xml
Site matching query does not exist.
```

#### 错误原因
不明

#### 解决办法

把settings.py中 INSTALLED_APPS中sites去掉
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # 'django.contrib.sites',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]

```


### 4、动态条件如何进行查询

#### 问题信息

比方说有3个条件，name/age/birthday，现在我需要通过其中任意一个或多个条件进行查询，如何最简单的实现？

#### 解决办法
```python
q = Q()
for kw in query_params:
	q.add(Q(**{kw + '__icontains': query_params[kw]}), Q.AND)   # icontains: case-insensitive

print(query_params)
print(q)

queryset = queryset.filter(q)
```
这里icontains是大小写不敏感，如果需要大小写敏感，可以用contains代替。
query_params和q打印出来分别是：
```python
# query_params
<QueryDict: {'address': ['上海'], 'sn': ['123456']}>
# q
(AND: ('address__icontains', '浦东'), ('sn__icontains', 'FB'))
```

#### 如果有逗号查询的解决办法
```python
Q_sn = Q()
Q_others = Q()
for kw in query_params:
    if kw == 'sn' and ',' in query_params[kw]:
        # sn逗号查询，如['20160805,20170911']，进行or联结
        for sn in query_params[kw].split(','):
            Q_sn.add(Q(**{kw + '__icontains': sn}), Q.OR)     # icontains: case-insensitive
            
    else:
        Q_others.add(Q(**{kw + '__icontains': query_params[kw]}), Q.AND)

queryset = queryset.filter(Q_sn, Q_others)
```


### 5、继承AbstractUser后创建User表失败

#### 错误信息
```
ERRORS:
auth.User.groups: (fields.E304) Reverse accessor for 'User.groups' clashes with reverse accessor for 'User.groups'.
        HINT: Add or change a related_name argument to the definition for 'User.groups' or 'User.groups'.
auth.User.user_permissions: (fields.E304) Reverse accessor for 'User.user_permissions' clashes with reverse accessor for 'User.user_permissions'.
        HINT: Add or change a related_name argument to the definition for 'User.user_permissions' or 'User.user_permissions'.
manage_user.User.groups: (fields.E304) Reverse accessor for 'User.groups' clashes with reverse accessor for 'User.groups'.
        HINT: Add or change a related_name argument to the definition for 'User.groups' or 'User.groups'.
manage_user.User.user_permissions: (fields.E304) Reverse accessor for 'User.user_permissions' clashes with reverse accessor for 'User.user_permissions'.
        HINT: Add or change a related_name argument to the definition for 'User.user_permissions' or 'User.user_permissions'.

```

#### 解决办法
在settings.py中重载AUTH_USER_MODEL
```
...
AUTH_USER_MODEL = 'manage_user.User'
...
```
manage_user是app的名字，User是models.py中定义的表的名称。

### 6、migrate失败：InconsistentMigrationHistory


#### 错误信息
```
django.db.migrations.exceptions.InconsistentMigrationHistory: Migration admin.0001_initial is applied before its dependency manage_user.0001_initial on database 'default'.

```

#### 解决办法
删除所有数据表，重新migrate。


### 7、直接在admin中创建用户无法登陆的问题(未解决)

#### 错误信息
在admin管理页面中直接add一个user后，进行登录，失败：

![login error](http://oh2r1lczr.bkt.clouddn.com/%E7%99%BB%E5%BD%95%E5%A4%B1%E8%B4%A5.png)











#### 错误原因

需要重写create方法，在创建时给密码加密。用set_password(raw_password)。


### 8、修改kwargs中的默认键值'pk'

#### 解决办法

在自己继承的viewset类中重载lookup_field这个属性即可：
```python
class HubLampViewSet(viewsets.ModelViewSet):

    lookup_field = 'hub_sn'    # 重载lookup_field
    queryset = Lamp.objects.all().filter(is_deleted=False)
    serializer_class = LampSerializer

    def get_queryset(self):
        hub_sn = self.kwargs['hub_sn']
        print(hub_sn)
        return Lamp.objects.filter(hub_sn__icontains=hub_sn)
```


### 9、实现/hubs/{id}/lamps/接口(无外键)

获取hubs[id]下的所有lamps。这里我使用hub_sn进行查询的，不是id。因为没有外键约束，是通过hub_sn进行关联。

#### 实现方法
在lamps/view.py中继承viewset：
```python
class HubLampViewSet(LampViewSet):   # 这里LampViewSet已经继承了viewset
    queryset = Lamp.objects.all().filter(is_deleted=False)
    serializer_class = LampSerializer

    def list(self, request, *args, **kwargs):
        """
        Get list of lamps by hub_sn.
        """

        hub_sn = self.kwargs['hub_sn']
        queryset = Lamp.objects.filter(hub_sn=hub_sn)
        queryset = self._query(request, queryset=queryset)

        if isinstance(queryset, QuerySet):
            queryset = queryset.all()

        serializer = self.get_serializer(queryset, many=True)

        return Response(serializer.data)
```

在project/urls.py中配置：
```
from manage_lamp import views as lamp_views

hub_lamps_view = lamp_views.HubLampViewSet.as_view({
    'get': 'list',          # 路由到HubLampViewSet类中的list方法
})


urlpatterns = [
    ...
    url(r'^hubs/(?P<hub_sn>[0-9]+)/lamps/', hub_lamps_view, name='hub-lamps-list'), 
    ...
]
```

<!--more-->