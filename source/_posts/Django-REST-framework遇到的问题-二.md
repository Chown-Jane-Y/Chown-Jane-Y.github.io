---
title: Django REST framework遇到的问题(二)
date: 2017-09-13 17:16:51
tags: Django Rest Framework
categories: Django Rest Framework
description:
---


问题第二波。

<!--more-->

### 1、各种viewset和mixin有什么区别？

#### 解答

viewset是继承结合了各种mixin后的集大成者。
如果只需要其中一两个接口的话，可以自己继承需要的mixin和genericview。

### 2、如何自定义django-filter？

#### 解决办法

安装django-filter库：
```
pip install django-filter
```
settings.py中配置：
```
INSTALLED_APPS = [
	...
    'django_filters',
    ...
]
```

view.py中添加：
```
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import OrderingFilter

from .models import Hub
from .serializers import HubSerializer
from .filters import HubFilter

class HubViewSet(viewsets.ModelViewSet):

    queryset = Hub.objects.all().filter(is_deleted=False)
    serializer_class = HubSerializer
    filter_backends = (DjangoFilterBackend, OrderingFilter)
    filter_class = HubFilter
    ordering_fields = ('id', 'sn', 'registered_time', 'status', 'rf_band', 'channel', 'address')

```

新建filters.py：
```
# coding: utf-8
from django.db.models import Q
from django_filters import rest_framework as filters

from .models import Hub


class HubFilter(filters.FilterSet):
    """
    Filter of hubs.
    """

    sn = filters.CharFilter(name='sn', method='sn_filter', lookup_expr='icontains')
    address = filters.CharFilter(name='address', lookup_expr='icontains')
    memo = filters.CharFilter(name='memo', lookup_expr='icontains')
    start_time = filters.DateFilter(name='registered_time', lookup_expr='gte')
    end_time = filters.DateFilter(name='registered_time', lookup_expr='lte')

    class Meta:
        model = Hub
        fields = (
            'id',
            'sn',           # fuzzy query
            'status',
            'rf_band',
            'channel',
            'address',      # fuzzy query
            'start_time',
            'end_time',     # registered time in range(start_time, end_time)
            'longitude',
            'latitude',
            'memo'          # fuzzy query
        )

    @staticmethod
    def sn_filter(queryset, name, value):
        """
        自定义过滤方法，sn可以包含逗号，如'2016,2017'，进行or查询
        查询sn中包含'2016'或'2017'的所有集控
        
        允许'2016,2017' , '2016,' , '2016, 2017' , '2016 ,2017'等形式
        """

        value = value.replace(' ', '')

        Q_sn = Q()
        for sn in value.split(','):
            if sn:
                Q_sn.add(Q(**{'sn__icontains': sn}), Q.OR)

        queryset = queryset.filter(Q_sn)
        return queryset

```


### 3、如何实现token认证

#### 解决办法

在urls.py中添加一条：
```python
from rest_framework.authtoken import views as authtoken_views

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    ...
    ...
    url(r'api-auth-token/', authtoken_views.obtain_auth_token),   # 添加此条
]
```

settings.py中添加INSTALLED_APPS：
```python
INSTALLED_APPS = [
	...
    'rest_framework',
    'rest_framework.authtoken',      # 添加此app
    ...
]
```
REST_FRAMWORK中添加：
```python
REST_FRAMEWORK = {
	...
	...
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',     # 主要是这个
    ),
}
```

然后重新生成数据表：
```
python manage.py makemigrations
python manage.py migrate
```
这是数据库中会多一张表，用来存放token，与用户ID相关联。
![token table](http://oh2r1lczr.bkt.clouddn.com/token%E6%95%B0%E6%8D%AE%E8%A1%A8.png)

POST方法请求该URL：http://127.0.0.1:8000/api-auth-token/
需要带上body参数，参数是用户名和密码：
```python
{
	"username": caojiayu,
	"password": admin123
}

# 还需要头部信息：
{
	"Content-Type": "application/json"
}
```
如果用户名密码正确，服务器会返回一条token：
（如果该token不存在，会自动创建一个新的token。如果存在，则返回数据库里原有的token。）
```
{
    "token": "9cf6adbd6372b7162e48f82b322d6526b6877b76"
}
```
拿到这个token后，在其他请求的headers中带上此参数：
```
{
	"Authorization"： "Token 9cf6adbd6372b7162e48f82b322d6526b6877b76"
}
```
如果token正确的话，后端request参数中就可以拿到用户，用request.user就可以拿到caojiayu。

如果token错误的话，就会返回：
```
{
    "detail": "Invalid token."
}
```


### 4、默认token是全局的，如何实现局部？

#### 解决办法

一是让前端去做，需要token的时候再加token，不推荐这种。

二是让后端做，在类中添加authentication_classes，来进行token限制。

在settings.py中把全局的去掉：
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        # 'rest_framework.authentication.TokenAuthentication',  # 去掉该行配置
    ),
}
```
在需要添加token验证的viewset中，加上一个属性：
```python
from rest_framework.authentication import TokenAuthentication   # 添加该行

class HubViewSet(viewsets.ModelViewSet):
    """
    集控列表页，可进行过滤与排序。
    """

    queryset = Hub.objects.all().filter(is_deleted=False)
    serializer_class = HubSerializer
    authentication_classes = (TokenAuthentication, )       # 添加该行
```

这样一来HubViewSet中的方法都需要进行token验证，其他没有配置的则不需要验证。

### 5、实现JWT用户认证

#### 解决办法
settings.py中加入一行配置：
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        # 'rest_framework.authentication.TokenAuthentication',
        'rest_framework_jwt.authentication.JSONWebTokenAuthentication',  # 加入此行
    ),
}
```
urls.py中配置：
```
urlpatterns = [
	...
    # url(r'api-auth-token/', authtoken_views.obtain_auth_token),  # drf自带的token认证
    url(r'login/', jwt_authtoken_views.obtain_jwt_token),       # 加此行，jwt认证
]
```
然后POST请求http://192.168.8.51:8888/jwt-auth/，body中输入用户名和密码，得到token：
```
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImNhb2ppYXl1IiwiZXhwIjoxNTA0NzU5ODQ3LCJlbWFpbCI6ImNhb2ppYXl1QDE2My5jb20ifQ.JM8K_md1yshU-7Z2slqhlEpe7u99AAnfvRdT0TFqiCw"
}
```

拿到token后，请求其他url时，在headers中加上该token：
```
"Authorization": "JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImNhb2ppYXl1IiwiZXhwIjoxNTA0NzU5ODQ3LCJlbWFpbCI6ImNhb2ppYXl1QDE2My5jb20ifQ.JM8K_md1yshU-7Z2slqhlEpe7u99AAnfvRdT0TFqiCw"
```
注意这里是JWT开头，和drf自带的Token开头有区别。


### 6、如何在serializers.py中定义非models字段？

#### 解决办法
这是permission的serialiser：
```
class PermissionSerializer(serializers.ModelSerializer):
    class Meta:
        model = Permission
        fields = ('id', 'user_id', 'hub_sn')

    user_id = serializers.IntegerField(required=True)
    hub_sn = serializers.CharField(required=True)
```

比如现在我想在user的serializer中需要添加permission这个字段，但permission不在user表中：
```
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('id', 'username', 'password', 'is_superuser', 'mobile', 'email', 'is_active', 'permission')

	...
	...
    permission = serializers.SerializerMethodField() # 使用SerializerMethodField()类型
```
可以使用参数method='xxx'，也可以不用，默认是get_加上字段名，比如这里就是get_permission。

然后在下面实现get_permission：
```
def get_permission(self, instance):
    """
    获取用户权限，与用户信息一同返回
    """
    return PermissionSerializer(instance.permission, many=True).data
```
就可以把permission同user一起返回了。

显示如下：
```
{
  "id": 1,
  "username": "caojiayu",
  "is_superuser": true,
  "mobile": "18605237059",
  "email": "caojiayu@163.com",
  "is_active": true,
  "permission": [
    {
      "id": 1,
      "user_id": 1,
      "hub_sn": "AA201611050954"
    },
    {
      "id": 2,
      "user_id": 1,
      "hub_sn": "201554124564"
    }
  ]
}
```



### 7、如何在serializer中自定义validator(验证字段有效性)

#### 解决办法

在serializers.py中自定义一个validate_xxx(self, value)的方法，xxx是需要验证的字段名：
```python
class LampSerializer(serializers.ModelSerializer):

    class Meta:
        model = Lamp
        fields = ('sn', 'status', 'type', 'hub_sn')

    hub_sn = serializers.CharField(max_length=16, required=True)

    def validate_hub_sn(self, value):
        """
        验证hub_sn是否有效
        """
        hub_sn = value
        hub_sn_exist = False
        for hub in Hub.objects.all().filter(is_deleted=False):
            if hub_sn == hub.sn:
                hub_sn_exist = True

        if not hub_sn_exist:
            raise serializers.ValidationError("The provided hub sn is invalid.")

        return value
```

<!--more-->