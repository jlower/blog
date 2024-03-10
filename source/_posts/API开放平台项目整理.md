---
title: API开放平台项目整理
date: 2024-03-02 03:00:00
categories: Java项目
---

## 项目部署

### 前端 Ant Design Pro 使用 Nginx 部署

> nginx.conf

```conf
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       8002;        # 自定义端口
        server_name  localhost; # 这是可配置的服务代理，若要配置则需要和 antd pro 里的 proxy 对应起来才能进行网页的访问。配置成 localhost 则可使用 ip: 端口 进行访问；
        
        # gzip config 优化vue打包发布chunk-vendors过大问题
        gzip on;
        gzip_min_length 1k;
        gzip_comp_level 6;
        gzip_types text/plain text/css text/javascript application/json application/javascript application/x-javascript application/xml;
        gzip_vary on;
        gzip_disable "MSIE [1-6]\.";

        include /etc/nginx/mime.types;


        #charset koi8-r;

        #access_log  logs/host.access.log  main;
  
        root   /var/www/html/lowo-api-frontend-dist;       # 实际写成绝对路径 ; 这个地方需要配置我们刚打包的 dist 路径，包括 root 路径、index 文件名 和 try_files;

        location / {
            # index  index.html index.htm;
            # 用于配合 browserHistory 使用
            try_files $uri $uri/ /index.html;
        }
  
        # location /api {
        #     rewrite ^/api/(.*) /$1 break;
        #     # 后台服务地址
        #     proxy_pass http://127.0.0.1:7529;
        #     proxy_set_header   X-Forwarded-Proto $scheme;
        #     proxy_set_header   Host              $http_host;
        #     proxy_set_header   X-Real-IP         $remote_addr;
        # }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
}
```

### 后端打包后使用 jdk11 部署

> 除了jar包外还有, mysql, redis, nacos

## 项目整理

### 项目是你自己做的吗？你为什么做这样的一个项目？你做这个项目的背景(初衷)是什么？为什么你要使用网关？

> 注：因为我个人已经将项目上线，并能够提供一些真实的接口服务。有条件的同学尽量将项目上线。此外有两场的面试官想要**查看数据库
**，我开了屏幕共享给他们看，所以要**对数据库的表结构和设计**有一定的了解。

本来想做一个自己用的API接口整合项目，比如天气服务(目前使用高德的免费API，如果哪天要收费了就用Python爬虫自己实现一个接口)、用Docker部署的网易云音乐API、随机头像、一言......  只做前端展示 ; 后来准备分享出去，为了保护自己的接口不被滥用，就用网关拦截请求隐藏真正的接口的地址，并做了用户系统限制使用次数，统计API接口使用次数，后来慢慢扩展成这样的，算是个练手项目

### 数据库的表结构怎么设计的？

1. ```lowo_api_platform.user``` 表中
```gender``` ```userRole``` ```status``` ```isDelete``` 字段都可以统一为 ```tinyint``` 类型(还没有改)
使用逻辑删除的原因是：即使误删还有机会恢复反悔，可以设定定时任务自动删除被标记删除的行
1. ```lowo_api_platform.daily_check_in``` 表中
```description``` 字段可以废弃(浪费空间)，描述能用固定的模板消息与查出的 ```addPoints``` 字段值组合自动生成描述
1. ```lowo_api_platform.interface_info``` 表中
使用逻辑删除
```requestHeader``` ```responseHeader``` 前端暂时没展示
```totalInvokes``` 用 Redis 存接口的总调用次数后此字段可以废弃(，或通过定时任务刷写到 MySQL 数据库中，Redis 宕机后可以查 MySQL 的数据)
1. ```lowo_api_platform.user_interface_invoke``` 表中
给每个用户维护其调用各个接口的历史记录(最新调用时间、总次数......)，使用逻辑删除，
```totalInvokes``` 也可以用 Redis 存当前用户调用某接口的次数，之后此字段可以废弃(，或通过定时任务刷写到 MySQL 数据库中，Redis 宕机后可以查 MySQL 的数据)
1. ```lowo_api_platform.product_info``` ```lowo_api_platform.product_order``` ```lowo_api_platform.payment_info``` ```lowo_api_platform.recharge_activity``` 支付相关的功能还没开发完(借鉴别人的设计)
```lowo_api_platform.recharge_activity``` 大致显示用户的订单(不查详细信息，可以再增加一个订单是否完成的状态，省的查两次才能显示是否成功)，要具体信息时可以通过其中的id查找对应 ```lowo_api_platform.product_order``` 的订单信息和 ```lowo_api_platform.payment_info``` 的支付信息

### 为什么使用 Spring Session + Redis 而不用 JWT？单点登录（SSO）？

1. JWT 的有效性无法主动撤销，只能等待 token 过期，更改密码或点击退出后之前的 token 依然有效 (设置黑名单/Redis 保存 JWT 的状态可以撤销但与 JWT 不在服务端存储状态的理论相违背，变成类似session的东西了)，而 Spring Session + Redis 可以通过修改 Redis 中的 session 数据来主动撤销用户的登录状态 (单点退出后其他服务也一并退出，SSO)方便后续做单点登录
单点登录（SSO）适合在多个应用系统中，只需登录一次就可以访问其他相互信任的应用系统时使用。这种方法提高了用户体验，减少了重复登录的繁琐，特别适用于企业内部网或外部网、学生门户网站、公有云服务等环境。通过单点登录，用户可以在一次登录后访问多个相关应用和服务，无需重复输入凭证，节省时间并提高便利性。然而，需要注意凭证泄露可能带来的安全风险，因此建议结合多因子身份验证（MFA）以增强安全性
1. 适合使用jwt的场景：
有效期短
只希望被使用一次
比如，用户注册后发一封邮件让其激活账户，通常邮件中需要有一个链接，这个链接需要具备以下的特性：能够标识用户，该链接具有时效性（通常只允许几小时之内激活），不能被篡改以激活其他可能的账户，一次性的。这种场景就适合使用jwt。或者下载文件
1. 而由于jwt具有一次性的特性。单点登录和会话管理非常不适合用jwt，如果在服务端部署额外的逻辑存储jwt的状态，那还不如使用session。基于session有很多成熟的框架可以开箱即用，但是用jwt还要自己实现逻辑。
1. 使用session与redis结合，因为单纯的使用session缓存数据的话，当一次会话结束后，session就会消失，如果结合redis使用则不会出现数据失效的情况。
1. JWT (类似身份证)无需在服务端存储 session 信息，不用数据库的查询(仅需存储解密密钥，使用密钥校验)，对跨域友好，可以在不同应用间共享用户信息
1. 用 OAuth 2.0 集成第三方登录 QQ、支付宝、钉钉、GitHub......
1. 使用 SpringSesion 的思路是：除了网关服务，每个服务都引入 SpringSession，使用 redis 实现 session 同步。获取信息时，根据 sessionId 从服务器 session 取，如果没有，连接 redis 服务器获取。
1. 使用 jwt 的话，我们在卡号服务里生成一个 jwtToken，放在 cookie 里，由网关服务解密后放在请求头里，这样的话，所有服务都可以从请求头里获取卡号信息。
1. Spring Session 默认使用 cookie 保存sessionid (对浏览器方便)，如果客户端是 手机app/小程序 等非浏览器，没有cookie实现保存不了cookie就不能通过cookie来访问session对象，需要在登录成功后，将sessionid添加到响应数据中，保存在客户端，下次用户访问，就在请求头当中带上sessionid ; 用 Spring Session 中的 ```HeaderHttpSessionIdResolver``` 实现
1. JWT 只能传输非敏感的人员数据，base64解码后就可以得到json，每次请求都需要在header中携带token信息，增大了带宽的压力
1. JWT (Json Web Token) 分为三部分，header、payload(载荷)、signature(签名)。前后端分离开发模式下，token加密后前端请求服务端获取授权，完成登陆校验，其中我们可以拿到payload中的内容，来回显到前端展示界面
1. session是存在服务端的，他是后端的产物，在前后端不分离场景下，单体应用、分布式架构下都能很好的担任他的角色，特别是有SpringSession这一完美解决方案，将session的令牌存在浏览器cookie中，数据存储在redis；但是在前后端分离流行的当下，特别是微服务架构下，JWT以简洁、易用更能胜任校验状态的角色，JWT更是最流行的跨域认证解决方案，通过加密，载荷也相对安全的存储了用户的信息，前端也更容易拿到这一信息，并把信息存储在localStorge或cookie中

### 签名和加密有啥区别？MD5签名认证怎么实现的？怎么防止请求头被篡改？怎么防止重放攻击？登录时怎么保护POST传入的账号密码，其与使用access_key和secret_key签名验证的网关接口服务有啥区别？

1. 签名是不可逆的，不存储原数据，只能做签名验证 ; 加密是可逆的，能够解密
1. 为防止重放攻击，加上了timestamp字段
1. 保存在数据库里的 **登录密码** 不用明文，而保存用MD5生成的签名，网页表单Post提交用户输入的密码，将提交的密码用MD5签名，把生成的签名与保存在数据库里的签名对比，相同则登录成功
**登录密码** 通过网络传输了，而 **secret_key** 不通过网络传输，所以在数据库中是否保存明文、校验合法的方法 **登录密码** 和 **secret_key** 使用的设计不一样
1. MD5签名没有信息所以传输时参数要包含原数据(Base64编码包裹传输)，例如access_key(方便服务器通过access_key查到secret_key)、timestamp......(请求头字段，但密钥secret_key别放在请求头明文传输)，secret_key包含在生成并传递过来的MD5签名中了 ; 服务器通过access_key查到secret_key用secret_key再加上**请求头的JSON**作为参数进行MD5签名与传递来的MD5签名对比，相同则签名验证通过(secret_key正确)
    - 为防止请求头被篡改：使用**请求头的JSON**加上**secret_key**一起签名，保证请求头没被篡改
1. MD5秒传(提取文件签名，对比签名，相同 {认为文件一致} 则秒传)
1. RSA也可用于签名，MD5不够安全
1. Spring Boot使用HTTPS保证POST请求中传递的密码安全 ; 签名验证的网关接口有防重放保护，请求时secret_key不在网络上传递，无论使用的是 HTTP 还是 HTTPS 都能保证安全
1. Spring Boot配置SSL证书开启HTTPS请求，并将HTTP请求转换成HTTPS请求。用acme.sh脚本自动生成泛域名SSL证书，并自动续期【生成证书前必须将认证服务设置为letsencrypt（Let's Encrypt）】

### [RPC与MQ的区别以及MQ的使用场景？](https://zhuanlan.zhihu.com/p/97841943)

1. RPC是远程过程调用，MQ是消息队列，RPC通常是同步的 请求/响应 调用，MQ是异步的流处理
1. N个不同系统相互之间都有RPC调用，依赖程度深，引入MQ降低耦合度
1. MQ实现RPC会造成更大通讯开销，不要强行替代
1. MQ异步有自动重传重试，与http同步调用相比能提高系统的可靠性，http要自己实现重传重试的补偿定时任务
1. MQ削峰/限流

### Token核心字段？

1. userId
1. 有效期

### user/admin 访问的身份如何控制

使用 Spring 中 AOP 配合自定义注解进行鉴权(目前只实现了方法拦截, 对类拦截要使用反射[后续可能更改为Spring Security]) ;
具体实现在/aop/AuthInterceptor 和 /annotation/AuthCheck 中

### 流程图

![流程图1](<https://raw.githubusercontent.com/lowoneko/public-imgs-1/main/public-imgs/image-20230112101821991.png>)
![流程图2](./assets/API开放平台项目整理/流程图.png)
