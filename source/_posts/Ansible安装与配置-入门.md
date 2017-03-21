---
title: Ansible安装与配置(入门)
date: 2017-03-21 17:04:35
tags: Ansible
categories:
description:
---

ansible是新出现的自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。
ansible是基于模块工作的，本身没有批量部署的能力。**真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架**。主要包括：

因为现在在做运维，Ansible是必须会的。记录一下第一次使用Asible的过程。

<!--more-->

*注：以下环境为CentOS 6.5。*

## Ansible基础安装

####(1) python2.7安装
https://www.python.org/ftp/python/2.7.8/Python-2.7.8.tgz
```
# tar xvzf Python-2.7.8.tgz
# cd Python-2.7.8
# ./configure --prefix=/usr/local
# make --jobs=`grep processor/proc/cpuinfo | wc -l`
# make install
```


```
// 备份旧版本的python，并符号链接新版本的python

# cd /usr/bin
# mv python python2.6
# ln -s /usr/local/bin/python
```



#### (2) setuptools模块安装
https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz
```
# tar xvzf setuptools-7.0.tar.gz
# cd setuptools-7.0
# python setup.py install
```

#### (3) pycrypto模块安装
https://pypi.python.org/packages/source/p/pycrypto/pycrypto-2.6.1.tar.gz
```
# tar xvzf pycrypto-2.6.1.tar.gz
# cd pycrypto-2.6.1
# python setup.py install
```

#### (4) PyYAML模块安装
http://pyyaml.org/download/libyaml/yaml-0.1.5.tar.gz
```
# tar xvzf yaml-0.1.5.tar.gz
# cd yaml-0.1.5
# ./configure --prefix=/usr/local
# make --jobs=`grep processor/proc/cpuinfo | wc -l`
# make install
```

https://pypi.python.org/packages/source/P/PyYAML/PyYAML-3.11.tar.gz
```
# tar xvzf PyYAML-3.11.tar.gz
# cd PyYAML-3.11
# python setup.py install
```

#### (5) Jinja2模块安装
https://pypi.python.org/packages/source/M/MarkupSafe/MarkupSafe-0.9.3.tar.gz
```
# tar xvzf MarkupSafe-0.9.3.tar.gz
# cd MarkupSafe-0.9.3
# python setup.py install
```

https://pypi.python.org/packages/source/J/Jinja2/Jinja2-2.7.3.tar.gz
```
# tar xvzf Jinja2-2.7.3.tar.gz
# cd Jinja2-2.7.3
# python setup.py install
```

#### (6) paramiko模块安装
https://pypi.python.org/packages/source/e/ecdsa/ecdsa-0.11.tar.gz
```
# tar xvzf ecdsa-0.11.tar.gz
# cd ecdsa-0.11
# python setup.py install
```

https://pypi.python.org/packages/source/p/paramiko/paramiko-1.15.1.tar.gz
```
# tar xvzf paramiko-1.15.1.tar.gz
# cd paramiko-1.15.1
# python setup.py install
```

#### (7) simplejson模块安装
https://pypi.python.org/packages/source/s/simplejson/simplejson-3.6.5.tar.gz
```
# tar xvzf simplejson-3.6.5.tar.gz
# cd simplejson-3.6.5
# python setup.py install
```

#### (8) ansible安装
https://github.com/ansible/ansible/archive/v1.7.2.tar.gz
```
# tar xvzf ansible-1.7.2.tar.gz
# cd ansible-1.7.2
# python setup.py install
```

## Ansible配置

#### (1) SSH免密钥登录设置

① 生成公钥/私钥
```
ssh-keygen -t rsa -P ''
```

② 分发到其他服务器

将`/root/.ssh/id_rsa.pub`文件分发到目标服务器上。

③ 写入信任文件

在目标服务器上执行：
```powershell
cat /root/.ssh/id_rsa.pub >> /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

#### (2) ansible配置
修改配置文件`ansible.cfg`
```
$ mkdir -p /etc/ansible

$ vim /etc/ansible/ansible.cfg

……

remote_port = 22  //ssh端口

private_key_file = /root/.ssh/id_rsa

……

```

主机组定义
```
# vim /etc/ansible/hosts

[test_cluster]
10.39.117.10
10.39.117.11
```

#### (3) 测试
```powershell
[root@caojiayu bin] ansible test_cluster -m command -a 'uptime'
10.39.117.10 | SUCCESS | rc=0 >>
 17:00:07 up  2:56,  2 users,  load average: 0.00, 0.00, 0.0
```
注意，第一次运行时，需要输入一下“yes”【进行公钥验证】，后续无需再次输入。

<!--more-->