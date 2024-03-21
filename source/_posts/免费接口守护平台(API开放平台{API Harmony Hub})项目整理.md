---
title: 免费接口守护平台(API开放平台{API Harmony Hub})项目整理
date: 2024-03-02 03:00:00
categories: Java项目
---

## 项目部署

### 数据库建表的SQL代码

> 数据库创建表的SQL代码(**待优化更改**)

```sql
-- 创建库
create database if not exists lowo_api_platform;
-- 切换库
use lowo_api_platform;

-- 用户表
create table if not exists lowo_api_platform.user
(
    id             bigint auto_increment comment 'id' primary key,
    userName       varchar(128)                           null comment '用户昵称',
    userAccount    varchar(128)                           not null comment '账号',
    userAvatar     varchar(1024)                          null comment '用户头像',
    email          varchar(128)                           not null comment '邮箱',
    gender         tinyint                                null comment '性别 1-男 2-女',
    userRole       tinyint      default 1                 not null comment '用户角色 1-user 2-admin',
    userPassword   varchar(256)                           null comment '密码',
    accessKey      varchar(256)                           null comment 'accessKey',
    secretKey      varchar(256)                           null comment 'secretKey',
    status         tinyint      default 1                 not null comment '账号状态 0-被删除 1-正常 2-禁用 3-审核中 4-审核失败',
    createTime     timestamp    default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime     timestamp    default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    constraint uni_email unique (email), -- 给email添加唯⼀约束
    constraint uni_userAccount unique (userAccount) -- 给userAccount添加唯⼀约束, UNI通常指的是 用户识别号(User Identification Number) 的缩写
)
    comment '用户表';

-- 接口信息表
create table if not exists lowo_api_platform.interface_info
(
    id                   bigint auto_increment comment 'id' primary key,
    name                 varchar(256)                           not null comment '接口名称',
    url                  varchar(256)                           not null comment '接口地址',
    userId               bigint                                 null comment '发布人',
    tagId                varchar(256)                           null comment '标签id 标签id间用逗号分隔开',
    method               tinyint      default 1                 not null comment '请求方法 0-方法未填写为空 1-GET 2-POST',
    requestParams        text                                   null comment '接口请求参数',
    responseParams       text                                   null comment '接口响应参数',
    requestExample       text                                   null comment '请求示例',
    requestHeader        text                                   null comment '请求头',
    responseHeader       text                                   null comment '响应头',
    returnFormat         tinyint      default 1                 null comment '返回格式 0-返回格式未填写为空 1-JSON',
    description          text                                   null comment '描述信息',
    status               tinyint      default 3                 not null comment '接口状态 0-被删除 1-正常上线 2-下线禁用 3-审核中 4-审核失败',
    statusCode           int          default 200               not null comment '接口状态码 判断接口是否正常',
    totalViews           bigint       default 0                 not null comment '接口被查看次数',
    totalSubscriptions   bigint       default 0                 not null comment '接口被订阅次数',
    avatarUrl            varchar(1024)                          null comment '接口头像',
    createTime           timestamp    default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime           timestamp    default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    -- FULLTEXT (name, description) WITH PARSER ngram -- 创建(name, description)的联合全文索引,联合全文索引查单独列时不会使用索引,所以搜素就两个字段都查(不再单独建立每列的索引了)
    FULLTEXT (name) WITH PARSER ngram,
    FULLTEXT (description) WITH PARSER ngram
)
    comment '接口信息表';

-- 用户订阅调用接口表(查不到的就是没订阅过)
create table if not exists lowo_api_platform.user_interface_subscribe
(
    id           bigint auto_increment comment 'id' primary key,
    userId       bigint                                  not null comment '调用人id',
    interfaceId  bigint                                  not null comment '接口id',
    status       tinyint       default 0                 not null comment '订阅状态 0-已取消订阅 1-已订阅',
    totalInvokes bigint        default 0                 not null comment '用户调用接口次数',
    createTime   timestamp     default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime   timestamp     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    INDEX user_interface_idx (userID, interfaceId) -- 创建(userId, interfaceId)的联合索引
)
    comment '用户接口调用表';

-- 标签表
create table if not exists lowo_api_platform.daily_check_in
(
    id          bigint auto_increment comment 'id' primary key,
    description varchar(64)    default '暂无描述'          not null comment '标签名称',
    status      tinyint        default 0                  not null comment '标签状态 0-被删除 1-正常上线 2-下线禁用 3-审核中 4-审核失败',
    createTime  timestamp      default CURRENT_TIMESTAMP  not null comment '创建时间',
    updateTime  timestamp      default CURRENT_TIMESTAMP  not null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '标签表';

-- 接口标签表
create table if not exists lowo_api_platform.user_interface_invoke
(
    id           bigint auto_increment comment 'id' primary key,
    tagId        bigint                                  not null comment '标签id',
    interfaceId  bigint                                  not null comment '接口id',
    createTime   timestamp     default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime   timestamp     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '接口标签表';

-- 支付模块还未实现，待实现，仅粗略设计了下表

-- 每日签到表
create table if not exists lowo_api_platform.daily_check_in
(
    id          bigint auto_increment comment 'id' primary key,
    userId      bigint                                  not null comment '签到人',
    addPoints   bigint        default 10                not null comment '签到增加积分个数',
    createTime  timestamp     default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime  timestamp     default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '每日签到表';

-- 产品信息
create table if not exists lowo_api_platform.product_info
(
    id             bigint auto_increment comment 'id' primary key,
    name           varchar(256)                           not null comment '产品名称',
    description    text                                   null comment '产品描述',
    userId         bigint                                 null comment '创建人',
    total          bigint                                 null comment '支付金额(单位:分)',
    addPoints      bigint       default 0                 not null comment '增加积分个数',
    productType    tinyint      default 2                 not null comment '产品类型 1-VIP会员 2-RECHARGE充值 3-RECHARGEACTIVITY充值活动',
    status         tinyint      default 0                 not null comment '商品状态 0-被删除 1-正常上线 2-下线禁用 3-审核中 4-审核失败',
    expirationTime timestamp                              null comment '过期时间',
    createTime     timestamp    default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime     timestamp    default CURRENT_TIMESTAMP not null comment '更新时间'
)
    comment '产品信息';

-- 产品订单
create table if not exists lowo_api_platform.product_order
(
    id             bigint auto_increment comment 'id' primary key,
    orderNo        varchar(256)                           not null comment '订单号',
    codeUrl        varchar(256)                           null comment '二维码地址',
    userId         bigint                                 not null comment '创建人',
    productId      bigint                                 not null comment '商品id',
    orderName      varchar(256)                           not null comment '商品名称',
    total          bigint                                 not null comment '支付金额(单位:分)',
    status         tinyint      default 3                 not null comment '交易状态 1-SUCCESS支付成功 2-REFUND转入退款 3-NOTPAY未支付 4-CLOSED已关闭 5-REVOKED已撤销(仅付款码支付会返回) 6-USERPAYING用户支付中(仅付款码支付会返回) 7-PAYERROR支付失败(仅付款码支付会返回)',
    payType        tinyint      default 1                 not null comment '支付方式 1-WX微信 2-ZFB支付宝',
    productInfo    text                                   null comment '商品信息',
    formData       text                                   null comment '支付宝的formData',
    addPoints      bigint       default 0                 not null comment '增加积分个数',
    expirationTime timestamp                              null comment '过期时间',
    createTime     timestamp    default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime     timestamp    default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '商品订单';

-- 付款信息
create table if not exists lowo_api_platform.payment_info
(
    id             bigint auto_increment comment 'id' primary key,
    orderNo        varchar(256)                           null comment '商户订单号',
    transactionId  varchar(256)                           null comment '支付订单号',
    tradeType      varchar(256)                           null comment '交易类型',
    tradeState     tinyint     default 3                  not null comment '交易状态 1-SUCCESS支付成功 2-REFUND转入退款 3-NOTPAY未支付 4-CLOSED已关闭 5-REVOKED已撤销(仅付款码支付会返回) 6-USERPAYING用户支付中(仅付款码支付会返回) 7-PAYERROR支付失败(仅付款码支付会返回)',
    tradeStateDesc varchar(256)                           null comment '交易状态描述',
    successTime    timestamp                              null comment '支付完成时间',
    openid         varchar(256)                           null comment '用户标识',
    payerTotal     bigint                                 null comment '用户支付金额',
    currency       tinyint      default 1                 null comment '货币类型 1-CNY',
    payerCurrency  tinyint      default 1                 null comment '用户支付币种',
    content        text                                   null comment '接口返回内容',
    total          bigint                                 null comment '总支付金额(单位:分)',
    createTime     timestamp    default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime     timestamp    default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间'
)
    comment '付款信息';

-- 充值活动表
create table if not exists lowo_api_platform.recharge_activity
(
    id         bigint auto_increment comment 'id' primary key,
    userId     bigint                               not null comment '用户id',
    productId  bigint                               not null comment '商品id',
    orderNo    varchar(256)                         null comment '商户订单号',
    createTime timestamp  default CURRENT_TIMESTAMP not null comment '创建时间',
    updateTime timestamp  default CURRENT_TIMESTAMP not null on update CURRENT_TIMESTAMP comment '更新时间',
    status     tinyint    default 1                 not null comment '状态 0-被删除 1-正常'
)
    comment '充值活动表';
```

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
        # 使用 http
        listen       8002;        # 自定义端口
        # # 如果有资源，建议使用 https + http2，配合按需加载可以获得更好的体验
        # listen       8002 ssl;        # 自定义端口
        # # 证书的公私钥(绝对位置)
        # ssl_certificate /root/CA/lowoneko.eu.org/cert.pem;
        # ssl_certificate_key /root/CA/lowoneko.eu.org/key.pem;

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

### 后端打包后使用 Docker-jdk11 部署

> 除了jar包外还有, mysql, redis, nacos

#### 打包注意点

1. 先install SDK代码
1. 再打包主项目(父子项目打包要注意)
    - 先注释掉父项目pom中<modules></modules>标签包含的子模块，install父pom
    - 再取消注释掉<modules></modules>标签包含的子模块，即可统一打包子模块

#### Dockerfile 编写

```dockerfile
FROM openjdk:11-oracle

# 描述作者和邮箱,可只写其中一个,也可二个都写
LABEL authors="lowo"

# 时区与字符设置UTF-8并配置环境
ENV TZ=Asia/Shanghai
ENV LANG=C.UTF-8

WORKDIR /app

COPY ./lowo-api-backend-0.0.3.jar /app/

EXPOSE 8002

ENV SPRING_PROFILES_ACTIVE prod

# 设置默认执行命令
CMD ["sh", "-c", \
    "cd /app && java -jar api-backend-0.0.3.jar"]
```

## 自我介绍(整理)

1.

## 项目真实性

### 你讲讲整个业务的流程，模块间的关系？

> - [ ] 等待完善文字描述(，并自己画个流程图)
> - [ ] 简历上的项目不写网关、RPC、消息队列、分布式，但面试时讨论优化点可以讲；3、为保证接口服务的可用性和稳定性，接口服务被独立部署在一台机器上。为了隐藏接口服务的真实地址和端口，避免接口服务被恶意攻击，前端请求先发送给网关，网关鉴权后再路由转发给接口服务处理；4、为了避免在网关中引入数据库操作，提高服务的可维护性，用Dubbo RPC实现服务间的方法远程调用；

#### 自己的流程

1.

### 项目是你自己做的吗？你为什么做这样的一个项目？你做这个项目的背景(初衷)是什么？

> - [ ] 换成监控MC服务器状态也行，但MC监控平台有很多，API监控推送平台没看到
> - [ ] 准备做让用户能够自己上传接口，要改sql
> - [ ] 简历上的项目不写网关、RPC、消息队列、分布式，但面试时讨论优化点可以讲；3、为保证接口服务的可用性和稳定性，接口服务被独立部署在一台机器上。为了隐藏接口服务的真实地址和端口，避免接口服务被恶意攻击，前端请求先发送给网关，网关鉴权后再路由转发给接口服务处理；4、为了避免在网关中引入数据库操作，提高服务的可维护性，用Dubbo RPC实现服务间的方法远程调用；
> - [ ] 邮件推送没做测试，并发量大的时候可能会出现漏发的问题，看看[QQ如何实现高可用的订阅推送系统](https://cloud.tencent.com/developer/article/2216345)
>   - 可以考虑引入消息队列(一般来说，消息队列的意义主要是削峰填谷、异步解耦。)对本项目而言，引入消息队列有以下好处：
>     - 将**任务调度和任务执行解耦**（调度服务并不需要关心任务执行结果）；
>     - 异步化，保证调度服务的高效执行，调度服务的执行是以 ms 为单位；
>     - 借助消息队列实现任务的可靠消费（ At least once ）；
>     - 将瞬时高并发的任务量打散执行，达到削峰的作用。
>   - 实现用户级别的可靠性，即要保证所有订阅用户都被至少推送一次（At least once）。At least once推送如何实现？
>     - 前提是当把用户 uin 从订阅列表中取出进行推送后，在推送结果返回之前，必须保证用户 uin 被妥善保存，以防止推送失败后没有机会再推送。由于 Redis 没有提供从一个 set 中批量 move 数据到另一个set中，这里采取的做法是通过 redis lua 脚本来保证这个操作的原子性，具体 lua 代码如下（近似）：

1. 自己做个人主页的时候用了一些别人提供的免费API，想着自己收集一下实用的免费API
1. 免费API分享一般就是在GitHub上维护了一个文档不好调试，分享免费API的网站不提供消息提醒功能
1. 待实现，让用户能自己上传API接口，并打标签
1. 平台汇集了众多网络上的免费接口服务，包括我自行开发的实用接口服务，能自动检测并监控接口状态，一旦用户订阅的**接口出现故障**，系统会通过**邮件通知用户**，让用户能及时**去检查**他调用免费接口的**项目是否出错**(例如，他个人主页调用的天气接口失效，他接到通知后就能及时更换成其他能用的)并解决。还对于**不好访问**的**接口实现了代理**，让用户能够用我提供的**SDK简化接口调用**的过程，便于开发者集成和使用这些接口服务，通过对接口代理还能让用户访问到像ChatGPT这样**国内不好访问**的接口。平台还实现了在线接口测试、统计接口查看次数等功能。
1. 调研类似此项目免费API分享平台：[夏柔 - 免费API](https://api.aa1.cn/)
    - **重点功能(还没有人做，一般需要自己写代码实现)**：发现有接口自动检测接口能否调通的功能但**没有**实现接口**订阅功能**，接口**出错不可调通**或接口**恢复正常**时**无法通知用户**，不能让用户及时了解到自己调用的免费接口有没有挂掉，以便用户及时切换提供服务的接口，或自己想用的免费接口服务有没有恢复上线
    - 发现**没有提供统一的SDK**，**但可以用户自己上传接口**(需人工审核){，**我的项目用户不能**自己上传接口}
1. 除了收集别人免费公开的API接口，我还提供了一些自己实现服务的API接口
1. 实现自动检测接口是否可调通(**不检测**返回是否正确，按要求请求，有返回则认为正常)，若发现接口不可调通(**失败多次**才认为接口出错)或参数发生变更时向所有订阅了此接口状态的人发送邮件提醒
1. 本来想做一个自己用的API接口整合项目收集免费的API，比如天气服务(目前使用高德的免费API，如果哪天要收费了就用Python爬虫自己实现一个接口)、用Docker部署的网易云音乐API、随机头像、一言......  只做前端展示 ( 自己使用**最重要的**其实就是**前端展示**并且**能在线调试**，**自己调用可以不走网关**直接用**真正的接口地址**省的被限流{自己**内部**使用，**不分享**给他人时**直接用接口的真实地址**}，当然走网关也行{例如：**个人主页**请求天气服务、一言、网易云音乐...... 要给**别人**展示使用的，肯定得走网关，防止接口真实地址被抓包获得})
1. **为什么要自己部署API？**有的对我来说实用的API别人没有实现，比如生成bilibili视频文章的短链接(不想打开手机去生成，要在电脑上直接出){之前还能生成跳转到外部网址的bilibili短链接，bug}，我个人主页上常用网易云音乐的免费API接口的作者说他服务器压力大，我就不白嫖自己搭建了
1. 后来准备分享出去，为了保护自己的接口不被滥用，~~就用网关~~用Spring的AOP拦截请求(进行签名验证，**请求头**传递签名sign和access_key和其他字段{例如时间戳timestamp...... }，签名sign要能验证secret_key也要能保证**请求头**没被**篡改**) 隐藏真正的接口的地址(**网关保护接口，防止接口地址暴露**，前端在线调试的请求地址也得改成网关，防止被抓包查出接口真实地址)，并做了用户系统限制使用次数，统计API接口使用次数，后来慢慢扩展成这样的，算是个练手项目
1. 现在字节的Coze提供**免费的ChatGPT4**可带promots的机器人，就是**要科学上网** ; 我用自己的服务器把Coze的机器人**包装**成类似ChatGPT4的**API**供人**免翻墙**调用就能**收费**了
1. ChatGPT**要科学上网**，我提供用自己服务器包装的**免翻墙**的ChatGPT的**API**调用也可以**收费**了，或者让用户自己提供ChatGPT的api-key我服务器只提供转发**免翻墙**
1. 对我来说项目最重要的地方在于前端展示，对上线的项目来说最重要的地方在于用户注册与调用api接口时在网关的鉴权

> 注：因为我个人已经将项目上线，并能够提供一些真实的接口服务。有条件的同学尽量将项目上线。此外有两场的面试官想要**查看数据库**，我开了屏幕共享给他们看，所以要**对数据库的表结构和设计**有一定的了解。

### 你的开发流程是什么？先实现还是先技术选型？

1. 这里可以参考鱼皮大佬直播开发时对**企业中做项目流程**的讲解

1. 调研类似此项目免费API分享平台：[夏柔 - 免费API](https://api.aa1.cn/)
1. 我先**参考了一些已有的产品**，根据这些产品，总结出来比较好的功能点，再**结合自己想要实现的一些功能特色**，去做了一个项目整体设计，**有了产品原型后再进行技术选型。使用什么样的技术去解决什么样的业务问题**。

> 总而言之，**面试官会从各个角度去深挖项目的细节，考量你是不是真的自己做的，是不是真的理解？**所以要做到对项目的所有细节都非常的熟悉。当完全理解项目之后，就能够**提前预测到面试官会怎么问**，并在面试过程中说出一些技术名词引导面试官，然后对这些问题，和延伸的知识点能够完全掌握后，相信一定可以征服面试官。

### 数据库的表结构是怎么设计的？

1. 各个表设计了**冗余字段**方便查询(冗余常需的字段，免的查询两次)
1. ​Mysql建表你一般会**遵守什么原则或者规范**，哪些**通用字段**，**主键**一般用啥
    - 一般有**四个通用字段**，```id```、```status```、```addTime```、```updateTime```，其中 ```id``` 一般用 ```int``` 或 ```bigint``` 自增， ```status``` 一般用 ```tinyint``` ， ```addTime``` 一般用 ```timestamp``` ， ```updateTime``` 一般用 ```timestamp``` ( ```datetime``` 类型人易直接读，但占用空间大， ```timestamp``` 类型占用空间小但人不易直接读) {流水表(例如，物流表......)不更新所以不用加 ```updateTime``` 字段}
    - 优先级高(**与业务相关**)的字段放上面
    - 一般认为 **0** 为**异常值**
    - 一般使用**逻辑删除**，改变 ```status``` 字段的值，而非真正删除；如果用户投诉可以**恢复**数据查看具体问题证据，每隔一段时间再统一清理(例如180天)
    - ```status``` 可以设定为 0 表示被删除， 1 表示正常， 2 表示禁用， 3 表示审核中， 4 表示审核失败，无需多加一个 ```isDelete``` 字段，
1. ```lowo_api_platform.user``` 表中
    - [x] ```gender``` ```userRole``` ```status``` 字段都可以统一为 ```tinyint``` 类型(还没有改)
        - 目前前端通过 ```loginUser.userRole``` 的函数获取数据库中 ```userRole``` 字段的值来判断渲染 user/admin 权限对应的界面(没权限的不渲染，隐藏)
        - 所以修改 ```userRole``` 字段类型为 ```tinyint``` 的话，前端通过 ```loginUser.userRole``` 的函数获取到的值就是 ```tinyint``` 类型的，前端判断权限的语句得改
    - 给 ```email``` 和 ```userAccount``` 添加**唯⼀约束**(会**自动的创建唯一索引**)，加快用户创建账号时查表的速度
        - 在MySQL中，唯一约束和唯一索引都可以实现列数据的唯一，列值可以有null(但**只能有一个**null)。**创建唯一约束时，会自动的创建唯一索引**。唯一约束是通过唯一索引来实现数据的唯一。
    - ```tinyint``` ```bigint``` 可以指定长度，例如 ```tinyint(1)```，**但长度只作用于显示几位数，存储空间不变，可存储的数值范围也不变** ，所以默认即可
    - 使用逻辑删除的原因是：即使误删还有机会恢复反悔，可以设定定时任务自动删除被标记删除的行
1. ```lowo_api_platform.interface_info``` 表中
    - 使用逻辑删除
    - ```requestHeader``` ```responseHeader``` 前端暂时没展示
    - ```totalInvokes``` 用 Redis 存接口的总调用次数后此字段可以废弃(，或通过定时任务刷写到 MySQL 数据库中，Redis 宕机后可以查 MySQL 的数据)
1. ```lowo_api_platform.user_interface_invoke``` 表中
    - 用户查自已有没有订阅过某个接口时查MySQL不查Redis，创建 ```userID``` 和 ```interfaceId``` 的**联合索引**方便查询
        - **为啥用联合索引而不单独索引？** 因为要实现用户查自已有没有订阅过某个接口这个功能WHERE语句需要同时限定userID和interfaceId，所以用联合索引
        - 要**防止索引失效**([联合索引遵循**最左匹配原则**](https://xiaolincoding.com/mysql/index/index_lose.html#%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95%E9%9D%9E%E6%9C%80%E5%B7%A6%E5%8C%B9%E9%85%8D))
        - 若经常以 ```userID``` 字段查询则创建 ```(userID,interfaceId)``` 的联合索引比创建 ```(interfaceId，userID)``` 的联合索引更有效
    - 给每个用户维护其调用各个接口的历史记录(最新调用时间、总次数......)，使用逻辑删除，
    - ```totalInvokes``` 也可以用 Redis 存当前用户调用某接口的次数，之后此字段可以废弃(，或通过定时任务刷写到 MySQL 数据库中，Redis 宕机后可以查 MySQL 的数据)
1. ```lowo_api_platform.daily_check_in``` 表中(暂时废弃)
    - [x] ```description``` 字段可以废弃(浪费空间)，描述能用固定的模板消息与查出的 ```addPoints``` 字段值组合自动生成描述
1. ```lowo_api_platform.product_info``` ```lowo_api_platform.product_order``` ```lowo_api_platform.payment_info``` ```lowo_api_platform.recharge_activity``` 支付相关的功能还没开发完(借鉴别人的设计)
    - ```lowo_api_platform.recharge_activity``` 大致显示用户的订单(先查需要的基本信息，若需要详细信息再查详细信息)(不查详细信息，可以再增加一个订单是否完成的状态，省的查两次才能显示是否成功)，要具体信息时可以通过其中的id查找对应 ```lowo_api_platform.product_order``` 的订单信息和 ```lowo_api_platform.payment_info``` 的支付信息

### 若是实习项目，怎么跟产品配合的？怎么测试的？怎么上线的？

1. 产品原型图: 产品经理把分布的位置画好，备注了要实现的**功能点**，UI还没有开始画
1. M站：手机端
1. 产品原型图 -> 产品需求文档 -> 产品设计文档 -> 产品开发文档 -> 产品测试文档 -> 产品上线文档 -> 产品上线

### 为啥不用HTTPS？

后续更新再统一上线

### ​在这个项目中线程池参数怎么配置的？你这个项目是根据 I/O密集型 还是 CPU密集型 计算？

Web应用大部分是 I/O密集型 的，与数据库、缓存交互多，网络传输，查数据库最消耗性能，少查数据库

## 简历上亮点的回答

- [ ] 要写为啥要重复造个轮子？例如RPC，不想用那么重的；Spring 要实现不同module相互调用时需要别的包**先发布才能调用**，**不能调**用不同module还**没发布**的东西
- [ ] 面试时引导面试官追问，顺便说说编码规范和技巧(校招vip里讲的){别全信校招vip}
- [ ] 简历上体现自己代码质量好，编写了单元测试，项目单元测试覆盖率达80%；
- [ ] 简历写：**出现了问题，如何定位问题，如何解决问题，达到什么样效果，实现了什么业务价值**；例如，项目优化中能否加入性调优，背景是在访问流量突增，出现热点，会如何应对？出现问题：cpu、内存、网络/io等异常，通过使用xx限流、热点检测、缓存......解决了问题
- [ ] CPU占用飙升100%排查，用top/jps/jstack/监控......指令定位到问题(平时没问题流量大后就出现问题){问题一般不会是死锁(除非代码太烂)，一般是并发问题(很可能流量小时没并发问题，流量大了后才出现问题)}，解决问题(例如，之前没考虑并发的地方考虑了并发，出现死锁的地方定位到了并解决，卡住的地方多线程优化、同步改成异步)

### SSM是啥？

Spring + SpringMVC + MyBatis
SpringMVC的注解 @RestController = @Controller + @ResponseBody、@RequestMapping("/user")、@PostMapping("/register")、@GetMapping("/getCaptcha")、@RequestBody、@RequestParam("username")、@Controller、@ResponseBody

### SDK，MD5签名

### 为啥单测用Mockito不用postman？

### 频繁 Full GC 排查

> [见黑马实战篇笔记](JVM调优(内存调优、GC调优、性能调优)与检测频繁Full%20GC.md)

#### 为什么OOM(内存溢出后JVM不会崩溃)？


### 设计了查询接口状态的短连接接口，并引入Redis缓存接口状态，避免了前端轮询判断接口状态是否变化时频繁到MySQL中去查全量数据

#### Redis中缓存时间设置多长？

变化较快的数据，比如点赞......缓存时间要短(一两分钟)，变化不大的数据，比如详情......缓存时间可以长点(一天)；流量上去了再用缓存，公司多项目用同一个Redis要注意key别重复了，覆盖掉别人的缓存了

### 为了给用户推荐好用的接口，实现了实时的接口热度榜，利用Redis的有序集合（ZSET）实现对接口查看次数的统计和排序。并使用Quartz定时任务将接口的查看次数同步到MySQL中

#### 为什么要用Redis？

为了构建**实时**的排行榜，且记录查看次数的操作有大量写入需求
使用ZADD命令向有序集合ZSET中添加成员key和对应的分数score(查看次数)。然后，我们使用 ```ZRANGE``` 命令获取排名，并打印出排行榜
在Redis中，Zset（有序集合）确实可以获取保存的所有成员（key）及其分数（score）。您可以使用ZRANGE命令配合WITHSCORES选项来实现这一点。例如：

```bash
ZRANGE yourzset 0 -1 WITHSCORES
```

这条命令会返回yourzset中的所有成员和它们的分数，成员按分数从低到高排序。其中0是起始索引，-1表示最后一个成员，所以这个范围包含了所有成员。WITHSCORES选项确保返回的结果中包含了分数。

在Redis中，您可以使用```ZINCRBY```命令来修改Zset中成员的分数。这个命令可以增加或减少成员的分数，并返回新的分数值。使用方法如下：

```bash
ZINCRBY yourzset increment member
```

其中yourzset是您的有序集合名称，increment是您想要增加或减少的分数（可以是负数），member是集合中的成员。
如果您想将成员member1的分数增加10，可以使用：```ZINCRBY yourzset 10 member1```
如果想减少10分，则increment使用-10。

> 如果你需要快速的读写操作，且数据结构相对简单，Redis可能是一个不错的选择。如果你需要处理大量的数据，需要复杂的SQL查询，或者需要完整的事务支持，那么MySQL可能更适合你。

#### 多久同步一次？

每两小时一次，不需要太精准

#### 定期将Redis的数据同步到MySQL中有啥方法？为啥不用Redis持久化策略 (AOF, RDB快照)？

1. 定时任务 (Cronjob/Scheduled Task):
    - 优点:可靠，已经广泛使用，容易理解和设置。符合“设置并忘记”的模式，不需要实时监控。定时任务可以很容易地调整频率来满足业务需求。
    - 缺点:如果任务失败，可能需要额外的机制来重新执行或报警。对于非常频繁的更新，定时任务可能会产生额外的开销。
1. Redis持久化策略 (AOF, RDB快照):
    - 优点:利用Redis自身的持久化功能，可以定期备份数据。备份文件可以用于多种用途，包括数据恢复和数据迁移。
    - 缺点:备份和迁移是离线的处理，不是实时的。需要额外的处理步骤，如解析备份文件，并将数据导入MySQL。
1. 事件驱动的更新:
    - 优点:可以实时更新，保持数据的实时性。减少不必要的操作，只有在数据变更时才进行更新，从而提高效率。
    - 缺点:实现复杂，需要额外的逻辑来监控数据变化。需要确保事件机制本身的高可用性和故障转移。
1. 消息队列 (如RabbitMQ, Kafka):
    - 优点:可以缓冲大量更新请求，平衡负载，避免瞬间压力。通过消息队列的特性提供了自然的重试机制。
    - 缺点:引入新的组件，增加了系统复杂性。需要维护消息队列的健康状态。
1. MySQL外部数据源特性 (如Federated Engine, Connect Engine):
    - 优点:允许MySQL直接访问外部数据库数据，无需复制数据。
    - 缺点:不同数据库之间的性能差异可能会成为瓶颈。依赖数据库的特定扩展，可能有兼容性和可移植性问题。

> 选择哪种方法，取决于数据更新的频率、实时性需求、系统的容错需求以及维护成本等多个因素。
> 对于绝大多数情况，使用定时任务来定期更新数据是一种简单并且有效的策略。如果更新非常频繁或者对实时性要求较高，则可以考虑事件驱动或消息队列等方案。一般来说，定时任务是最常见的策略，适用于多数需求不是特别高的场景。

#### @Scheduled注解和Quartz是Spring框架中用于实现定时任务的两种不同机制，区别？

1. 以后如果要做定时任务**集群**，保证定时任务稳定执行，使用Quartz
1. 简易性：@Scheduled注解是Spring内置的，使用起来非常简单。只需在方法上添加@Scheduled注解，并配置cron表达式即可实现定时任务。
1. 功能性：Quartz提供了更强大的功能，如任务持久化、事务管理、任务监听等。它支持分布式环境，可以在多个节点上执行任务。
1. 并发支持：@Scheduled默认不支持并发执行，如果需要并发执行，可以结合@Async注解使用。而Quartz则可以通过配置来支持并发执行。
1. 持久化：Quartz支持任务的持久化，即使系统重启，任务也能继续执行。而@Scheduled不支持持久化，重启后需要重新调度。
1. 表达式：两者都支持cron表达式，但Quartz的表达式更为强大，支持更复杂的调度需求。

> 总的来说，如果你的应用只需要简单的定时任务，@Scheduled注解足以满足需求。如果你需要更复杂的任务调度，或者在分布式环境中运行任务，Quartz会是更合适的选择。选择哪一个取决于你的具体需求和项目的复杂度。

#### 如果用MySQL实现，如何实现？

以下查询将返回“scores”表中的前10名用户及其分数和排名。FIND_IN_SET 函数用于确定每个用户的排名。

```sql
SELECT 
    user_id,
    username,
    score,
    FIND_IN_SET( score, (
        SELECT GROUP_CONCAT( score
        ORDER BY score DESC ) 
        FROM scores )
    ) AS rank
FROM scores
ORDER BY score DESC
LIMIT 10;
```

当存在并列情况时，上述的FIND_IN_SET和GROUP_CONCAT的组合可能不会准确地反映排名，因为FIND_IN_SET会返回分数在由GROUP_CONCAT创建的列表中第一次出现的位置，这可能导致并列的分数有不同的排名。
为了处理并列情况并获得正确的排名，您可以用窗口函数（如果您的MySQL版本是8.0或以上）来简化这个过程。
这里的RANK()窗口函数会为每行数据分配一个排名，分数相同的用户将获得相同的排名，接下来的用户排名会跳过并列的数量。

```sql
SELECT 
    user_id,
    username,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank
FROM scores;
```

如果您的MySQL版本低于8.0，可以考虑其他方法，如变量来计算排名：
这段代码使用变量来追踪前一个值和当前排名，并据此计算每行的排名。如果当前分数与前一个分数相同，则排名不变；否则，排名加一。注意，使用此方法时，须确保查询包括ORDER BY score DESC子句以保持正确的顺序。

```sql
SET @prev_value := NULL;
SET @rank_count := 0;
SELECT 
    user_id,
    username,
    score,
    CASE
        WHEN @prev_value = score THEN @rank_count
        WHEN @prev_value := score THEN @rank_count := @rank_count + 1
        ELSE @rank_count := @rank_count + 1
    END AS rank
FROM scores
ORDER BY score DESC;
```

### 定时执行接口检测方法，若有接口状态变化，则在Kafka中发布通知用户的任务。引入消息队列将接口状态检测和通知用户解耦，并将瞬时高并发的任务打散执行，达成了削峰的作用

Kafka消息队列，使操作**异步**执行

#### 怎么实现接口订阅功能(当用户订阅的接口状态改变时，平台通过邮件通知用户)？

**用Redis的set**存每个接口的订阅了的用户的列表（看看[文章](https://cloud.tencent.com/developer/article/2216345)中的实现，我自己实现直接取的话**没必要用**），MySQL里也存接口用户订阅关系表（**可以不存**，备份Redis即可，存就要保证高可靠一致性），当要用户订阅和取消订阅时要同时更改Redis、MySQL中的数据，[数据库和缓存如何**保证一致性**](https://xiaolincoding.com/redis/architecture/mysql_redis_consistency.html#%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E7%BC%93%E5%AD%98%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E4%B8%80%E8%87%B4%E6%80%A7)

- 我一开始准备按[文章](https://cloud.tencent.com/developer/article/2216345)中实现分块多用户推送，但我用户又不可能很多，而且我发邮件时对速率有限制，并且没实现At least once推送，所以用不用都行，反正部署了Redis就用一用呗；我现在只需要**取**，如果也要按文章上写的实现At least once推送则需要**存和取** 那就要用Redis

系统主要会有两部分数据：

- 业务方创建的任务数据。包含任务的提醒时间和提醒内容；
- 用户订阅生成的订阅数据。主要是订阅用户 uin 列表数据，这个列表元素级别可达到千万以上，并且必须要能够快速读取。

该项目存储选型主要从访问速度上考虑。任务数据可靠性要求高，不需要快速存取，使用MySQL即可。订阅列表数据需要频繁读写，且推送触发时对于**存取效率**要求较高，考虑使用内存型数据库。

- Redis 单线程模型，有效避免读写冲突；
- Redis set 底层基于 整数集合(intset) 和 hash表 实现，存储整型 uin(User Identification Number，用户识别号) 在空间和时间上均高效；
- 用uin再去用户表里查email时因为uin有唯一**索引**所以快
- 原生支持去重；
- 原生支持高效的**批量取一定数量**字段的接口（```spop```），适合于推送时使用。
- 这个**我不用**，我的订阅用户**数量少**直接 ```SMEMBERS``` 取出set中的**所有值**（但这样就没必要用Redis了，自己要了解只有MySQL时怎么实现）

一般来说，消息队列的意义主要是**削峰填谷、异步解耦**。对本项目而言，引入消息队列有以下好处：

- 借助消息队列实现任务的**可靠消费**,保证所有订阅用户都被至少推送一次（ At least once ）；
- 将**任务调度和任务执行解耦**（调度服务并不需要关心任务执行结果）；
- 异步化，保证调度服务的高效执行，调度服务的执行是以 ms 为单位；
- 将瞬时高并发的任务量打散执行，达到削峰的作用。

设置定时任务(**使用Spring的@Scheduled注解来运行定时任务**{是**单线程**的，所以定时任务内异步调用方法，如果某个定时任务耗时太长会影响其他定时任务的执行})，每小时检测一遍全部的接口，定时任务异步调用检测接口状态的异步方法，检测方法中引入[Spring Retry](https://javaguide.cn/high-availability/timeout-and-retry.html#java-%E4%B8%AD%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E9%87%8D%E8%AF%95)避免网络波动影响(有一次调通即可)检测结果~~(，可以在Redis中记录接口状态持续的次数，在**第六次**时状态保持时通知用户{，状态变化时接口状态持续的次数**归零**})~~，若接口状态变化则写入数据库(数据库里接口信息字段用statusCode存检测结果)并在kafka中发布通知用户的任务消息(消息包含：**邮箱信息、接口名字、接口描述、用户id、接口id**组成**不同的**邮件样式，还能防止被识别成**垃圾邮件**)。100个左右的接口，单个线程检测就行，查接口表，记录含接口id和url和检测方法的list到set中，用iterator迭代器迭代检测。

```java
// [注意简历上写了用Quartz来实现定时任务]使用Spring的@Scheduled注解来运行定时任务 (是单线程的，所以定时任务内异步调用方法，如果某个定时任务耗时太长会影响其他定时任务的执行)
@Scheduled(fixedRate = 1800000) // 每1800000毫秒 = 30分钟
// 使用Sprin的@Async注解来定义一个异步方法。在执行这个方法时，Spring会在单独的线程上运行它，这样不会干扰到其他的进程。
@Async
// 调用异步方法，调用自定义类AsyncService中自定义的process()异步方法
@GetMapping("/startAsync")
public ResponseEntity<String> startAsyncProcess() {
    CompletableFuture<String> future = asyncService.process();
    // 如果你需要处理返回的Future，获取结果并处理可能的异常，你可以提供回调处理，例如：
    future.whenComplete((result, throwable) -> {
        if (throwable != null) {
            // 处理异常情况
        } else {
            // 处理正常的返回结果
        }
    });
    // 不需要等待结果，直接响应请求
    return ResponseEntity.accepted().body("请求已接受，正在处理中...");
    
    // 如果你想在某个地方等待结果，可以这样做：
    // String result = future.get(); // 注意这会阻塞直到异步过程结束
    // return ResponseEntity.ok().body(result);
}
```

~~消费者异步调用发送邮件的函数，提供回调处理，让发送出错没发出去的邮件任务重新放回消息队列中，保证所有订阅用户都被至少推送一次（At least once），实现At least once推送~~

#### 看文章的整理

实现订阅的接口变动(若有变动就发消息邮件)的功能(类似平台发送广告邮件的业务逻辑，用户没订阅则不发)，看看[QQ如何实现高可用的订阅推送系统](https://cloud.tencent.com/developer/article/2216345)，我不做分布式，选用 Redis 的 set 类型来存储订阅列表{双set保证所有用户在接口状态发生变化时都能收到通知(接口状态变化需要连续几次检测相同才判定(设置下线6次检测，下线或上线3小时才发邮件)，防止频繁误判，而且发邮件也需要间隔时间发送，不能一次性大量发，防止QQ、网易......的邮箱服务器封我邮箱服务器的IP，**会被判定为垃圾邮件的问题还要解决**，**自建邮箱服务器发大量邮件容易被VPS厂家封的问题还要解决**)，发送邮件时从Redis里取，省的查用户接口订阅调用表了}，并记录到MySQL的表中{用户添加或删除订阅时记录到MySQL，要注意使Redis缓存失效([实现缓存与数据库一致](https://xiaolincoding.com/redis/architecture/mysql_redis_consistency.html#%E6%95%B0%E6%8D%AE%E5%BA%93%E5%92%8C%E7%BC%93%E5%AD%98%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E4%B8%80%E8%87%B4%E6%80%A7))}(用户查自已有没有订阅过某个接口时查MySQL不查Redis，创建 ```userID``` 和 ```interfaceId``` 的**联合索引**{**为啥用联合索引而不单独索引？**因为要实现用户查自已有没有订阅过某个接口这个功能WHERE语句需要同时限定userID和interfaceId，所以用联合索引}方便查询，要**防止索引失效**([联合索引遵循**最左匹配原则**](https://xiaolincoding.com/mysql/index/index_lose.html#%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95%E9%9D%9E%E6%9C%80%E5%B7%A6%E5%8C%B9%E9%85%8D))，若经常以 ```userID``` 字段查询则创建 ```(userID,interfaceId)``` 的联合索引比创建 ```(interfaceId，userID)``` 的联合索引更有效)，{问问**ChatGPT**怎么实现，及实现业务的设计难点在哪}，可以使用**消息队列**(kafka ......？)优化(消息队列中的**任务**是发送给**一个**用户的信息，没像文章中那样5000个用户打个包**一起**处理，是**一个一个用户**当作任务处理的{邮件发送有限制})

要**实现At least once推送**，看看[QQ如何实现高可用的订阅推送系统](https://cloud.tencent.com/developer/article/2216345)

- 可以考虑引入消息队列(一般来说，消息队列的意义主要是削峰填谷、异步解耦。)对本项目而言，引入消息队列有以下好处：
  - 将**任务调度和任务执行解耦**（调度服务并不需要关心任务执行结果）；
  - 异步化，保证调度服务的高效执行，调度服务的执行是以 ms 为单位；
  - 借助消息队列实现任务的可靠消费（ At least once ）；
  - 将瞬时高并发的任务量打散执行，达到削峰的作用。
- 实现用户级别的可靠性，即要保证所有订阅用户都被至少推送一次（At least once）。At least once推送如何实现？
  - 前提是当把用户 uin 从订阅列表中取出进行推送后，在推送结果返回之前，必须保证用户 uin 被妥善保存，以防止推送失败后没有机会再推送。由于 Redis 没有提供从一个 set 中批量 move 数据到另一个set中，这里采取的做法是通过 redis lua 脚本来保证这个操作的原子性，具体 lua 代码如下（近似）：

#### 为什么用Kafka，而不是用其它的MQ中间件，区别是什么？

> [MQ选型](https://juejin.cn/post/6964029068450725896)
> [MQ选型](https://blog.csdn.net/u011539836/article/details/110228525)
> [MQ选型](https://blog.csdn.net/ww2651071028/article/details/130357196)
> [MQ选型](https://developer.aliyun.com/article/1337502)
> [消息队列对比](https://zhuanlan.zhihu.com/p/60288391)
> [消息队列对比](https://zhuanlan.zhihu.com/p/163246737)
> [消息队列对比](https://cloud.tencent.com/developer/article/2314814)
> [消息队列对比](https://javaguide.cn/high-performance/message-queue/message-queue.html#%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9)
> Spring Boot 里集成了kafka，注解使用方便，随便用用
> Kafka的缺点是有可能消息重复消费而且**消费者消费失败不支持重试，只能自己扔回消息队列**，但我的通知重复发也没啥问题，又不是金融服务，邮件发送失败设置了Spring Retry如果失败，并且我使用的 topic 数量较少正好可以让Kafka保证其超高吞吐量
> 大数据领域的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。

Kafka最初是为了日志统计分析而设计的，它适用于处理大量的流式数据。选择Kafka而不是其他MQ中间件的主要原因在于其高吞吐量、分布式架构、适用于大数据场景以及开源社区支持。以下是Kafka与其他MQ中间件的主要区别：

1. 高吞吐量：Kafka单机每秒处理几十上百万条消息，并且即使存储了许多TB的消息，它也保持稳定的性能
1. 持久化数据存储：Kafka将消息持久化到磁盘，而不依赖内存缓存，从而提高了数据的持久性和容错性
1. 分布式系统：Kafka在一个或多个可以跨越多个数据中心的服务器上作为集群运行，并且支持主题划分为多个分区，每个分区是一个有序的消息队列，分区之间可以并行地读写数据，提高了系统的并发能力
1. 零拷贝：Kafka利用操作系统的零拷贝特性，减少了数据在内核空间和用户空间之间的复制，降低了 CPU 和内存的开销
1. 数据批量发送：Kafka支持生产者和消费者批量发送和接收数据，减少了网络请求的次数和开销
1. 数据压缩：Kafka支持多种压缩算法，如 gzip，snappy，lz4 等，可以有效地减少数据的大小和传输时间
1. 开源社区支持：Kafka是一个开源解决方案，拥有庞大的开发者社区支持，提供广泛的连接器、框架和工具，使其更具灵活性和可扩展性
1. 异步通信：Kafka支持异步处理机制，允许用户将消息放入队列中，但不立即处理它们
1. 冗余：Kafka支持一对多的消息发布，允许一个生产者的消息被多个订阅者消费
1. 扩展性与解耦：Kafka可以作为一个接口层，解耦业务流程，并提供扩展能力
1. 数据可靠性：Kafka通过复制机制和ack应答机制来保证数据的可靠性

> 相比之下，其他MQ如RabbitMQ和RocketMQ等，通常在实时性、可靠性要求较高的场景中使用，它们遵循AMQP协议，支持事务和消息确认机制，但在吞吐量上可能不如Kafka。每种MQ中间件都有其优势和适用场景，选择哪一种取决于具体的业务需求和技术要求。如果你需要处理大规模的数据流并且对性能有较高要求，Kafka可能是一个更好的选择。如果你需要**确保消息的可靠传递并且对实时性**有更高的要求，那么其他MQ中间件可能更适合。

选择Kafka、RocketMQ或RabbitMQ取决于你的具体业务需求和场景。以下是每种消息队列中间件的适用场景：

1. Kafka: 适用于高吞吐量、低延迟的实时数据处理和事件驱动架构场景。有良好的可伸缩性和持久性。常用于日志收集、流处理、消息传递和实时监控等。
1. RocketMQ: 适用于高性能、高可用性的消息传递场景，尤其是在金融互联网领域，如电商订单处理和业务削峰。支持丰富的消息过滤和分布式事务特性。常用于交易、订单处理、分布式事务等需要高可靠性和低延迟的场景。如果你的业务需要可靠性极高的消息传递，或者在大量交易涌入时后端可能无法及时处理的情况，RocketMQ可能更适合。
1. RabbitMQ: 适用于需要可靠消息传递和灵活消息模型的场景，具有丰富的插件和社区支持。常用于在对速度需求不极端高、但对消息可靠性要求较高的场景。如果你的数据量不是特别大，或者你需要一个功能完备且社区支持良好的消息队列，RabbitMQ可能是更好的选择。

在选择消息队列时，需要根据应用的具体需求来定。比如，对于大数据处理、实时分析、大量数据收集的场景可能会选择Kafka；在需要处理高价值交易、保证消息可靠性、以及支持事务性消息的应用场景中，则可能选择RocketMQ；而RabbitMQ可能会在需要良好的消息可靠性、多种消息队列模型和插件支持的时候得到采用。
总的来说，Kafka适合大规模数据处理，RocketMQ适合高可靠性场景，而RabbitMQ适合灵活性和可靠性都重要的场景。你应该根据你的业务需求和技术栈来选择最合适的消息队列中间件。如果你有更具体的场景，我可以提供更详细的建议。

#### Kafka为什么可能会重复发送？

### 将MySQL读写分离，在读库上为接口的名称和介绍字段建立全文索引实现搜索，比用模糊匹配搜索的方法更加高效

> [MySQL全文索引怎么解决like模糊匹配查询慢](https://www.php.cn/faq/497183.html)

#### MySQL 读写分离/主从分离 ？

> [MySQL 读写分离/主从分离 和分库分表详解](https://javaguide.cn/high-performance/read-and-write-separation-and-library-subtable.html)
> 通过接口传递不同数据库源之间的数据，配置多源数据库？怎么切换源？怎么切换事务？

1. 在上线第一个版本，因为**流量很少**(万级以下)，搜索模块直接采用多字段分别**模糊匹配** ```LIKE``` 的方案
1. 后续流量增长后(十万，百万)，设计了mysql主从分离(主库写，从库读)，并在**从库**进行**全文索引**(**对写库没有压力**)(从 MySQL5.6 开始，InnoDB 开始支持全文检索{Full-Text Search}，看看索引介绍)
    - 全文检索通常可以实现对数据库中文本内容的快速搜索，这可以包括对标签、详情、文章内容、评论、产品描述等文本字段的搜索。具体能对哪些字段使用全文检索，要看你所使用的数据库系统，以及如何设计数据库的schema。
    - MySQL中的全文检索通常**用于MyISAM和InnoDB**存储引擎，可以**对VARCHAR和TEXT类型**的字段创建全文索引。但对于InnoDB，MySQL从版本5.6开始支持全文索引。全文索引可以让你针对那些包含大量文本的字段进行搜索优化，用于匹配用户查询中的关键词，它可以有效地搜索含这些词的记录。(就用InnoDB就好)
    - 全文检索是可以**多字段**的**索引**
    - **难点**：怎么主从同步
    - 怎么使用全文检索？ ```SELECT * FROM articles WHERE MATCH(title, body) AGAINST('search keyword');``` 用 ```MATCH() AGAINST()``` 方法当条件，不用 ```LIKE```
1. **超大流量(千万级)**才用 **ElasticSearch** 全文索引，会**增加运营成本**，没必要

#### 追问ElasticSearch？

1. [Elasticsearch常见面试题总结](https://javaguide.cn/database/elasticsearch/elasticsearch-questions-01.html)

#### MySQL的全文索引怎么用？建立联合全文索引后，查他的单独列时，不会使用全文索引，所以搜素就两个字段都查(或单独建立每列的索引了)

> [MySQL全文索引介绍](https://zhuanlan.zhihu.com/p/589076491)
> **创建(name, description)的联合全文索引,联合全文索引查单独列时不会使用索引,所以搜素就两个字段都查(或单独建立每列的索引了)**

用全文索引的格式：  MATCH (columnName) AGAINST ('string')

```sql
SELECT * FROM `student` WHERE MATCH(`name`) AGAINST('聪')
```

当查询多列数据时：建议在此多列数据上创建一个联合的全文索引，否则使用不了索引的。(建立**联合索引**后，查他的单独列时，**不会使用**索引)

```sql
SELECT * FROM `student` WHERE MATCH(`name`,`address`) AGAINST('聪 广东')
```

使用全文索引需要注意的是：(基本单位是词) 分词，全文索引以词为基础的，MySQL默认的分词是所有非字母和数字的特殊符号都是分词符(外国人嘛)，**有中文分词插件**

#### 怎么创建全文索引？创建全文索引时为什么用了 ```WITH PARSER ngram``` ？

```sql
FULLTEXT (name, description) WITH PARSER ngram
```

> ```WITH PARSER ngram``` 是MySQL中的一个选项，它用于创建全文索引时指定使用ngram解析器。ngram解析器的主要功能是将文本序列标记为n个字符的连续序列。
> 例如，你可以将“abcd”标记为不同值n的文本序列：
> n=1: ‘a’, ‘b’, ‘c’, ‘d’
> n=2: ‘ab’, ‘bc’, ‘cd’
> n=3: ‘abc’, ‘bcd’
> n=4: ‘abcd’
> 这对于处理中文、日文、韩文等表意语言的全文搜索非常有用，因为这些语言不使用分词符。
> MySQL的内置全文解析器使用空格确定单词的开始和结束，这对于处理这些语言是一个限制。为了解决这个问题，MySQL提供了ngram全文解析器

### 引入Redis缓存接口状态，避免了前端轮询判断接口状态是否变化时到MySQL中查询全量数据

> [如何保证缓存和数据库数据的一致性](https://javaguide.cn/database/redis/redis-questions-02.html#%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E7%BC%93%E5%AD%98%E5%92%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E6%95%B0%E6%8D%AE%E7%9A%84%E4%B8%80%E8%87%B4%E6%80%A7)
> [短连接是啥](https://developer.aliyun.com/article/1159871)
> [短连接是啥](https://blog.51cto.com/u_16213722/8893794)
> MySQL单机支持并发一般300~500/s
> 先用缓存优化，不行再做MySQL主从分离

**新增**了一个**短连接接口**(只查接口状态)和查全量数据的接口分开(2个接口操作不同)，方便调用，不用判断执行哪个操作
每十秒前端调用一次后端的短连接接口查询接口状态有没有更新，若接口状态有更新就调用全量查询接口查询接口全量数据~~(不一定要刷新，接口信息中**会变的只有热度和状态**，其中状态最重要，变状态就好)~~，然后重新传给前端并刷新页面数据，引入 Redis 减少全量接口查询的次数，短连接接口查的是Redis缓存，若发现接口状态有变化，则先存到MySQL里，Redis设置**失效时间为1分钟**(1分钟内必显示出接口状态变化)，注意[如何保证缓存和数据库数据的一致性](https://javaguide.cn/database/redis/redis-questions-02.html#%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E7%BC%93%E5%AD%98%E5%92%8C%E6%95%B0%E6%8D%AE%E5%BA%93%E6%95%B0%E6%8D%AE%E7%9A%84%E4%B8%80%E8%87%B4%E6%80%A7)

### 由于接口最新查看榜实时变动较大，可以采取时间戳+唯一ID的组合作为新的分页参数，确保翻页时数据不会重复(，或使用Redis Feed 流)

> [滑动分页时列表数据重复或丢失问题解决方法介绍](https://baijiahao.baidu.com/s?id=1761347544743891908)
> [滑动分页时列表数据重复或丢失问题解决方法介绍](https://www.cnblogs.com/mouseleo/p/17624516.html)

#### 为什么用常规动态分页的方法会出现数据重复或丢失的问题？主要是在哪些场景中出现？

#### Redis Feed流也可以实现？Redis Feed是啥？主要用在哪？怎么实现？

#### MySQL深度分页怎么优化？

1. [深度分页介绍及优化建议](https://javaguide.cn/high-performance/deep-pagination-optimization.html)

### 怎么用Junit、Mockito写单元测试？怎么用JaCoCo生成单测报告？

### 解决了部分网站使用字体加密反爬虫的问题

#### 为啥要爬虫？

以前用别人的免费爬 番茄免费小说/塔读文学 小说的接口挂了，作者GitHub上归档不维护了，我自己比较常用就只好自己实现喽。
我用Kindle看小说较多，有的时候就下txt下来看，要解决编码冲突，Python 实现起来比较方便，就用了Python。

#### 商业避免反爬虫怎么实现？

1. 修改请求头：有些网站会识别爬虫程序的请求头，我们可以通过修改请求头来改变程序的请求方式，让其看起来像是普通用户在浏览网页。具体实现可通过 Python 的 requests 库中的 headers 参数来设置。
1. 延时爬取：有些网站会限制短时间内的访问次数，所以我们可以通过设置延时，让爬虫程序在一定时间间隔后再访问页面，从而降低访问频率。具体实现可通过 Python 中的 time 模块来实现。
1. 使用代理IP(IP池子)：有些网站为了防止爬虫程序的访问，会封禁爬虫程序所在的 IP 地址，我们可以通过使用代理 IP 来实现每次访问使用不同的 IP 地址，从而避免被封 IP。具体实现可通过 Python 中的 requests 库中的 proxies 参数来设置。
1. 登录账号：有些网站会设置只有登录用户才能进行浏览访问，我们可以通过模拟用户登录行为，以登录状态进行爬取。具体实现可通过 Python 中的 requests 库中的 session 对象来完成。
1. 绕过人机验证：有开源的绕过谷歌验证和亚马逊验证的模块，现在**用AI绕过验证**也行了

> 我爬的少且频率低无需太多操作。

### 为避开邮箱提供者对免费邮箱发信次数的限制，在Spring中配置了多个邮箱并实现轮流发信

#### 为啥现在还没改发送的邮箱？

流量小，免费邮箱QQ、Google一天50封，网易一天100封，Spring虽然默认只能配置1个邮箱，但自己写Bean可以配置多个邮箱轮询使用
并且，Amazon Simple Email Service (SES)不好用，还是会被识别成垃圾邮件，只防止了被封ip被拒收，并且可能产生费用

#### 还要限制发送速度？

免费邮箱发送速率受限，发太快的话号会被封且邮件也发不出去，用Guava的RateLimiter限制了消费者消费发送邮件通知任务的速率

#### 自建邮件服务器

自建邮件服务器**避免被主流邮箱屏蔽**的方法：**使用中继主机** 例如 Amazon Simple Email Service (SES)
使用[Guava的RateLimiter](https://javaguide.cn/high-availability/limit-request.html#%E5%8D%95%E6%9C%BA%E9%99%90%E6%B5%81%E6%80%8E%E4%B9%88%E5%81%9A)限制每秒发送邮件的速率，满足Amazon Simple Email Service (SES)限制，~~避免被封IP和被当成垃圾邮件的问题~~
消费者调用发送邮件的函数，使用Spring Retry，保证邮件发送成功

会被判定为**垃圾邮件**的问题要解决
待实现[根据不同状态码调整发送策略](https://blog.csdn.net/xjmfc/article/details/105837572)
自建邮箱服务器发大量邮件**容易被VPS厂家封**的问题还要解决，[解决方法](https://blog.csdn.net/weixin_39850365/article/details/111616226)
    - 发信模板：不管使用文字、图片、附件还是HTML，都**必须要使用变量**，同一内容被大量群发后，腾讯就会识别然后设置为垃圾邮件内容，所以变量是防止垃圾内容的方法之一。
    - 发信ip：通过vps切换ip来达到模拟人工操作的要求。
      - [自建邮局方法](https://nigzu.com/self-hosted-email/)并非不可行，有**避免被主流邮箱屏蔽**的方法：**使用中继主机**。我开通了[Amazon Simple Email Service (SES)](https://zhuanlan.zhihu.com/p/358780018?ref=nigzu.com)（主用）和Mailgun（备用）作为发件中继主机，SES每个月免费 62000封，Mailgun每个月免费1000封，两者免费版的都使用共享 IP 地址，理论上仍有可能因其他共享用户发送垃圾邮件或自己大量发件被收件方邮局屏蔽，送达率可能不如大厂的邮箱，解决办法是购买专用IP，但这半年来我没有碰到过邮件被拒收的情况，除了163邮箱莫名收不到Mailgun的邮件，163邮箱似乎会屏蔽大部分海外来件。
      - 此外通过中继主机发送的邮件，有些收件人客户端会显示“由 <xxxxxxxxx@xxx.xxx>代发”字样，可能会给对方带来困惑，因此最好在Amazon SES或Mailgun上配置与发件地址同一域名下的子域作为发件域。
    - Amazon Simple Email Service (SES)限制：每天50000封、每秒14封、每条消息 50 个收件人、每封邮件包括附件最大 40MB，带宽上行 40MB/秒
    - 发送时用Guava RateLimiter来限制每秒发10封邮件，这样就能避免被封IP的问题，发送出错的**重新放回消息队列**并记录重发次数超限就不发了，(千用户以内，用户多了就买企业邮箱，我不收费就用免费的方案，不行的话就不自建邮箱服务器了，还是配多个邮箱账号轮询来发邮件吧)
    - [自建邮箱服务也不可以大量发送](https://www.zhihu.com/question/283149109)
    - [SMTP/POP3协议介绍](https://zhuanlan.zhihu.com/p/84174651)。采用常用的**SMTP**作为邮件**发送协议**，采用常用的**POP3**作为邮件**读取协议**。请注意，SMTP和 POP3 (或IMAP)都是使用**TCP连接**来传送邮件的，使用TCP的目的是为了可靠地传送 邮件。

### 算法题参数一定要用校验，不然就挂

### 默认先http访问，之后再转https

## 项目追加问答

### 为简化查询接口对应标签的常用操作，在“接口信息表”中增加了一个varchar类型的冗余字段来存放接口对应标签的id(标签id间用逗号分隔开)

为简化查询接口对应标签的常用操作，在“接口信息表”中增加了一个varchar类型的冗余字段来存放接口对应标签的id（标签id间用逗号分隔开）；流量大了上Redis缓存

- 使用VARCHAR字段来存储多个标签ID，然后用逗号分隔，是一种常见的简化数据库设计的做法，尤其是在一些**小型或不需要高度正规化**的数据库中。然而，这样做**会影响到数据库的拓展性、性能和数据完整性**。
- 一个接口仅拥有少量标签时，可以在“接口信息表”中用一个varchar类型的字段存放多个标签的id（id间用逗号分隔开），使用MySQL的FIND_IN_SET函数能实现按**单个标签**查询接口，不用创建“接口标签表”，优化了按标签查询接口和查询接口标签的操作；
- MySQL手册中[FIND_IN_SET函数](https://www.cnblogs.com/jinxiang1224/p/8468261.html)的语法解释：FIND_IN_SET(str,strlist)
  - str 要查询的字符串
  - strlist 字段名 参数以”,”分隔 如 (1,2,6,8,10,22)
  - 查询字段(strlist)中包含(str)的结果，返回结果为 null或记录
- MySQL的**IND_IN_SET函数**确实能够查询出含有特定标签的记录，但这种方式**并不高效**，尤其是在标签数量较多或是记录数量庞大时。这是因为FIND_IN_SET是一种字符串函数，它**不会利用索引**，所以每次查询都需要进行全表扫描。
- 如果考虑到可维护性和性能，建议使用一种更可扩展的设计，比如创建一个标签表和一个接口与标签的关联表（通常称为“联结表”或"中间表"）
- fooId字段**可以用逗号分隔**储存成**varchar类型** 一个字段储存多个fooId，一查查个String出来再**按逗号分割**可以分出多个fooId，MySQL的**varchar类型**支持输入逗号并且MySQL有[FIND_IN_SET函数](https://www.cnblogs.com/jinxiang1224/p/8468261.html)支持逗号，fooId**数量少**的时候实现了存**可变**数量的fooId数据(**可以设置not null**)；不用设置 fooId_1、fooId_2、fooId_3...... **多个字段**(有的时候只存一个数据，但查得全查则查出的后面几个字段会是**空**，**还要判空**，且浪费空间)
