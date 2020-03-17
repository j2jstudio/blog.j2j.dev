---
title: "Ce Pass Centos8"
date: 2019-12-11T14:04:13+08:00
draft: true
---



open_paas

appengine: 192.168.2.105:7001
paas: 192.168.2.105:7002
esb: 192.168.2.105:7003
login: 192.168.2.105:7004

<!--more-->

# domain
PAAS_DOMAIN = '192.168.2.105:7002'
# 跳转到本地开发使用的login服务, 仅本地开发用. 注意生产环境使用nginx反向代理不需要配置LOGIN_DOMAIN变量(删除即可)
LOGIN_DOMAIN = '192.168.2.105:7004'

# inner domain, use consul domain,  for api
PAAS_INNER_DOMAIN = ''
HTTP_SCHEMA = 'http'


# cookie 名称
BK_COOKIE_NAME = 'bk_token'
# cookie有效期
BK_COOKIE_AGE = 60 * 60 * 24
# cookie访问域
BK_COOKIE_DOMAIN = '.bking.com'

# 控制台地址
ENGINE_HOST = "http://192.168.2.105:7001"
# 登陆服务地址
LOGIN_HOST = "http://192.168.2.105:7004"

# paas host
PAAS_HOST = 'http://192.168.2.105:7002'

# host for bk login
HOST_BK_LOGIN = 'http://192.168.2.105:7004'