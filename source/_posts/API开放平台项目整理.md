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

> [对于此项目，别人的面经(点击跳转查看)，大致看看不一定对](API开放平台项目面经.md)

### 项目是你自己做的吗？你为什么做这样的一个项目？你做这个项目的背景(初衷)是什么？为什么你要使用网关？

1. 本来想做一个自己用的API接口整合项目，比如天气服务(目前使用高德的免费API，如果哪天要收费了就用Python爬虫自己实现一个接口)、用Docker部署的网易云音乐API、随机头像、一言......  只做前端展示 ( 自己使用**最重要的**其实就是**前端展示**并且**能在线调试**，**自己调用可以不走网关**直接用**真正的接口地址**省的被限流{自己**内部**使用，**不分享**给他人时**直接用接口的真实地址**}，当然走网关也行{例如：**个人主页**请求天气服务、一言、网易云音乐...... 要给**别人**展示使用的，肯定得走网关，防止接口真实地址被抓包获得})
1. 后来准备分享出去，为了保护自己的接口不被滥用，就用网关拦截请求(进行签名验证，**请求头**传递签名sign和access_key和其他字段{例如时间戳timestamp...... }，签名sign要能验证secret_key也要能保证**请求头**没被**篡改**) 隐藏真正的接口的地址(**网关保护接口，防止接口地址暴露**，前端在线调试的请求地址也得改成网关，防止被抓包查出接口真实地址)，并做了用户系统限制使用次数，统计API接口使用次数，后来慢慢扩展成这样的，算是个练手项目
1. 现在字节的Coze提供**免费的ChatGPT4**可带promots的机器人，就是**要科学上网** ; 我用自己的服务器把Coze的机器人**包装**成类似ChatGPT4的**API**供人**免翻墙**调用就能**收费**了
1. ChatGPT**要科学上网**，我提供用自己服务器包装的**免翻墙**的ChatGPT的**API**调用也可以**收费**了，或者让用户自己提供ChatGPT的api-key我服务器只提供转发**免翻墙**

> 注：因为我个人已经将项目上线，并能够提供一些真实的接口服务。有条件的同学尽量将项目上线。此外有两场的面试官想要**查看数据库**，我开了屏幕共享给他们看，所以要**对数据库的表结构和设计**有一定的了解。

### 数据库的表结构怎么设计的？

1. ```lowo_api_platform.user``` 表中
    - ```gender``` ```userRole``` ```status``` ```isDelete``` 字段都可以统一为 ```tinyint``` 类型(还没有改)
        - 目前前端通过 ```loginUser.userRole``` 的函数获取数据库中 ```userRole``` 字段的值来判断渲染 user/admin 权限对应的界面(没权限的不渲染，隐藏)
        - 所以修改 ```userRole``` 字段类型为 ```tinyint``` 的话，前端通过 ```loginUser.userRole``` 的函数获取到的值就是 ```tinyint``` 类型的，前端判断权限的语句得改
    - ```tinyint``` ```bigint``` 可以指定长度，例如 ```tinyint(1)```，**但长度只作用于显示几位数，存储空间不变，可存储的数值范围也不变** ，所以默认即可
    - 使用逻辑删除的原因是：即使误删还有机会恢复反悔，可以设定定时任务自动删除被标记删除的行
1. ```lowo_api_platform.daily_check_in``` 表中
    - ```description``` 字段可以废弃(浪费空间)，描述能用固定的模板消息与查出的 ```addPoints``` 字段值组合自动生成描述
1. ```lowo_api_platform.interface_info``` 表中
    - 使用逻辑删除
    - ```requestHeader``` ```responseHeader``` 前端暂时没展示
    - ```totalInvokes``` 用 Redis 存接口的总调用次数后此字段可以废弃(，或通过定时任务刷写到 MySQL 数据库中，Redis 宕机后可以查 MySQL 的数据)
1. ```lowo_api_platform.user_interface_invoke``` 表中
    - 给每个用户维护其调用各个接口的历史记录(最新调用时间、总次数......)，使用逻辑删除，
    - ```totalInvokes``` 也可以用 Redis 存当前用户调用某接口的次数，之后此字段可以废弃(，或通过定时任务刷写到 MySQL 数据库中，Redis 宕机后可以查 MySQL 的数据)
1. ```lowo_api_platform.product_info``` ```lowo_api_platform.product_order``` ```lowo_api_platform.payment_info``` ```lowo_api_platform.recharge_activity``` 支付相关的功能还没开发完(借鉴别人的设计)
    - ```lowo_api_platform.recharge_activity``` 大致显示用户的订单(先查需要的基本信息，若需要详细信息再查详细信息)(不查详细信息，可以再增加一个订单是否完成的状态，省的查两次才能显示是否成功)，要具体信息时可以通过其中的id查找对应 ```lowo_api_platform.product_order``` 的订单信息和 ```lowo_api_platform.payment_info``` 的支付信息

### 后端数据库里储存什么值来让前端判断渲染权限对应的界面(无权限的页面、按钮...... 隐藏)？

> 在数据库设计中，通常并不直接存放与前端按钮可见性或交互性相关的信息。这些通常是前端应用的实现细节。数据库层面关心的是数据的结构和完整性，而对如何展示或使用这些数据并不涉及。前端的可见性和可交互性通常是**基于用户的角色和权限**来决定的。(例如，我用的Vue/React框架)

1. 目前前端通过 ```loginUser.userRole``` 的函数获取数据库中 ```userRole``` 字段的值来判断渲染 user/admin 权限对应的界面(没权限的不渲染，隐藏)
1. 所以修改 ```userRole``` 字段类型为 ```tinyint``` 的话，前端通过 ```loginUser.userRole``` 的函数获取到的值就是 ```tinyint``` 类型的，前端判断权限的语句得改

**不过**，如果你需要后端控制前端按钮的显示与否，你可以在数据库中设置相关的权限控制表。例如：

1. 用户角色表：定义了可用的角色。
1. 权限表：定义了不同的权限，每个权限对应**前端界面**上的一个或多个**元素**，**如按钮**。
1. 用户角色权限关联表：标识每个角色拥有的权限。

这样前端应用可以根据用户的角色和关联的权限动态地显示或隐藏按钮，或设置其为不可点击状态。例如：

1. 当用户登录时，前端应用会查询数据库或通过API获取与用户角色相关的权限。
1. 前端根据这些权限，决定是否渲染特定的按钮或将其设置为不可用状态。

> 某些客户端**框架**可能会在渲染**之前**将权限数据用作渲染决策的**依据**。(例如，我用的Vue/React)
> **请注意**，即使前端隐藏了按钮，逻辑仍然应该在后端进行验证，以确保没有权限的用户不能通过直接调用API等方式执行不被允许的操作。始终在服务端进行权限检查是保护应用安全的重要措施。

### 类的变量加static和不加的区别是什么？是不是所有变量都要加static？

1. 存储位置和生命周期：
    - Static变量：也称为静态变量，它是类的一个类变量，不属于类的任何一个实例。静态变量在类被加载时创建，在程序结束时销毁。无论你创建了类的多少实例，都只有一份静态变量的副本。
    - 非Static变量：也称为实例变量，它属于类的一个实例。每当你创建类的新实例时，都会为非静态变量创建新的副本，且每个实例的非静态变量都是独立的。
2. 访问方式：
    - Static变量：可以不创建类的实例就直接通过类名访问。
    - 非Static变量：必须创建类的实例才能访问。
3. 内存管理：
    - Static变量：既然它们只有一份副本，静态变量被所有实例共享，所以它们是一种节约内存的方式，当你的变量值应该在类的所有实例之间共享时。
    - 非Static变量：每个实例有自己的变量副本，因此它们不共享状态，可能会占用更多的内存。
4. 默认值：
    - Static变量：如果未显式初始化，它们会获得类型的默认值（例如，int的默认值是0，对象引用的默认值是null）。
    - 非Static变量：同样未初始化时，也会获得类型的默认值。

> 是否所有变量都应该加上static取决于你的具体需求：
> 如果某个变量应该跨所有实例共享，比如常量、```serialVersionUID```、配置信息、状态标记或者计数器，那么它应该声明为static。
> 如果每个实例都应该拥有自己的变量副本，如对象的属性或者状态，那么它就不应该是static。
> 滥用static会导致设计上的问题，比如对于那些应该是实例独有的属性，如果进行了static声明，就会引起不必要的数据共享，可能会引发安全问题或是逻辑上的错误。因此，不是所有变量都适合加上static。

### MySQL怎么分页？

```sql
SELECT * FROM table_name LIMIT offset, count;
```

> 其中，```table_name```为要查询的表名；```offset```指定从第几条记录开始返回结果（索引值从0开始）；```count```指定每次返回多少条记录。

### 项目的架构你是怎么设计的？

1. 我采用**前后端分离**的架构，**前端 Ant Design Pro 使用Nginx部署**，通过**Nginx 反向代理**将请求转发到 web 项目(**注意通过反向代理这句话不一定对，我在 Ant Design Pro 打包生成的前端没用上**)，因为项目刚刚上线，所以这里暂时采用了单机部署的模式，未来可能采取水平扩容的方式，增加多台节点，通过**Nginx 的负载均衡**，将请求平均的分发到我的每个节点上，以支撑更高的并发。
1. 我的 web 项目使用**Spring Boot**开发，并连接到了数据库和 Redis，数据库使用的是 MySQL，主要用来存储用户的信息和接口的信息；通过 Redis 实现了**分布式 session**，因为考虑到未来要使用分布式架构，为了避免使用 tomcat 保存 session 有用户登录失效的问题。

> 注：这里我说出了**反向代理，水平扩容，负载均衡**等技术名词，很多面试官会根据这些名词进行延伸提问（引导面试官往自己熟悉的东西上提问）比如：**说说什么是正向代理/反向代理？什么是水平扩容？什么是负载均衡？你了解哪些负载均衡的算法？**提前准备好这些知识之后，就可以跟面试官一顿输出了。

### 为什么使用 RPC 调用？有了解过其他的方式吗？

1. 因为如果在网关引入数据库的操作的话，不仅会增加项目体积，还违背了设计原则的**单一职责原则**降低了系统的可维护性，所以我考虑通过服务间调用的方式，我了解过有两种方式，第一种是**Open feign**，原理是构造了一个 HTTP 请求，并会添加很多的请求头，body 是使用 json 字符串传输，所以调用效率会比较低，更加适合外部服务间的调用。
1. 然后我了解到**RPC**是可以基于 TCP 协议，避免了无用的请求头，以及可以通过**将数据序列化为二进制流的形式传输，效率更加高效，更加安全**，所以更适用于我这个场景。最终我选择了 Dubbo RPC 框架来实现这个功能。

### Spring Session + Redis 怎么实现登录？ JWT 怎么实现登录？

1. Spring Session 校验成功后就在 Session 中记录查询出来的登录用户的user信息(可放敏感信息)，下次通过cookie中的sessionid查找到session，并从session中得到登录用户的user信息(若无则没有登录)，用此来判断是否登录
1. JWT 校验成功后返回一个 token 给用户，用户下次发请求时带上此token，服务端校验此token若成功就相当于登录，并可以从token中获取基本信息(不可放敏感信息)
    - 用 JWT 实现登录，要把 user_id 和 user_role 和 过期时间 置于 payload 中

### Spring 默认单线程，常用方法与注解？

1. [见链接](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html)
1. Spring **默认单线程**，**但Controller方法默认多线程并发**，或者@Async注解标记异步实现的方法也是多线程并发
1. @PostConstruct是Java自带的注解，在方法上加该注解会在项目启动的时候执行该方法，也可以理解为在spring容器初始化的时候执行该方法，可作为一些数据的常规化加载，比如数据字典之类的。
1. @SpringBootApplication 包含 @ComponentScan 默认扫描配置类所在目录下的所有包(以配置类所在目录为根目录) 还要扫描别的位置的话，需要自己在注解里写明

### Spring Cloud Gateway网关有什么用？

1. [见链接](https://javaguide.cn/distributed-system/api-gateway.html)
1. 等调用完api接口并成功返回后才让数据库中的统计数据加1，容易形成并发写，用Redis代替MySQL

### 涉及浮点运算或钱时使用BigDecimal计算小数

1. [具体注意事项见链接](https://javaguide.cn/java/basis/bigdecimal.html)

### 怎么实现在Spring Boot中配置多个邮箱(spring-boot-starter-mail默认只支持单个邮箱)？

1. [见链接](https://blog.csdn.net/u012110298/article/details/106786456/)
1. spring-boot-starter-mail会根据spring.mail.xxx相关配置对JavaMailSender进行自动配置。但是**只支持单个邮箱**。为了**实现多邮件源**，可以参照上述逻辑。在配置文件内配置好，多个邮件源。然后读取配置文件，手动对JavaMailSender进行配置，并将其初始化完毕的JavaMailSender存储容器内。然后发送时随机取出JavaMailSender进行发送。
1. 定义配置属性：使用@ConfigurationProperties注解定义配置属性类，这样您可以在应用程序的配置文件（例如 ```application.yml``` 或 ```application.properties```）中保持多个邮箱配置。
1. 创建配置类：创建一个配置类，通过在该类上使用@Configuration注解，并通过@Bean为每个邮箱创建一个邮件发送器（JavaMailSender）实例。
1. 注入和使用：在您的服务或组件中，需要发送邮件时，注入对应的JavaMailSender实例，并使用它来发送邮件。
1. 如果你想要在发送邮件时轮流使用多个邮箱账户，可以通过在JavaMailSender上封装一个服务来管理邮箱账户的选择。下面的示例会展示一个简单的轮流切换邮箱账户来发送邮件的服务。
    - 所有的JavaMailSender实例都在构造函数中通过依赖注入注入到一个列表中。在sendEmail方法中，每次调用都会通过getNextMailSender方法来获取下一个JavaMailSender实例。
    - **不一定要**用 ```AtomicInteger``` Spring Boot默认单线程，但Controller方法默认多线程并发，或者@Async注解标记异步实现的方法也是多线程并发
    - **这里使用了 ```AtomicInteger``` 来保证索引的线程安全性**，```java.util.concurrent.atomic.AtomicInteger```。 AtomicInteger 是依靠 **[volatile](https://blog.csdn.net/u013967628/article/details/85291748)** 和 **CAS** 来保证原子性的

```java
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

@Service
public class RoundRobinEmailService {

    private final List<JavaMailSender> mailSenders;
    private final AtomicInteger index = new AtomicInteger(0);

    // 通过构造器注入所有的 JavaMailSender 实例
    public RoundRobinEmailService(List<JavaMailSender> mailSenders) {
        this.mailSenders = mailSenders;
    }

    private JavaMailSender getNextMailSender() {
        // 获取下一个 JavaMailSender 实例（在列表中轮流进行）
        return mailSenders.get(index.getAndUpdate(i -> (i + 1) % mailSenders.size()));
    }

    public void sendEmail(String to, String subject, String text) {
        JavaMailSender mailSender = getNextMailSender();
        // 创建并发送邮件...
        // MimeMessage message = mailSender.createMimeMessage();
        // MimeMessageHelper helper = new MimeMessageHelper(message, true);
        // helper.setTo(to);
        // helper.setSubject(subject);
        // helper.setText(text, true);
        // mailSender.send(message);

        // 打印日志或执行其他需要的逻辑
        System.out.println("邮件已通过邮箱 " + ((JavaMailSenderImpl)mailSender).getUsername() + " 发送");
    }
}
```

### 邮件验证码绑定邮箱或登录怎么使用 Redis缓存 实现？

> 生成的验证码放在Redis缓存中并设定30分钟的过期时间，用 **CAPTCHA_CACHE_KEY(Redis 验证码存储类的key抬头，例如"api:captcha:") + 邮箱地址** 做 Redis 的key保证只有最新的验证码有效(，每次生成后存储验证码到value时因为key相同所以会覆盖之前的value)

1. 验证码输入错误**次数限制**，比如设置短信验证码输入错误3次后，这个短信验证码就不能使用了。防止猜测短信验证码恶意注册。若**没有保护**则可能：
    - 用户在登录界面填写手机号，不获取短信验证码，直接通过爆破模块(burp suite)，生成一堆的验证码，直接脚本批量尝试登录。
    - 重置密码的逻辑，同样也可以通过这种爆破验证码的逻辑，跳过短信验证码，直接修改密码。
1. [验证码防爆破链接](https://blog.csdn.net/lilei1138494584/article/details/126637447)

### 邮件激活链接绑定邮箱或登录 或 文件临时一次性分享下载链接 怎么使用 JWT 实现？

> **适合使用jwt的场景**：有效期短 / 只希望被使用一次
> 比如，用户注册后发一封邮件让其激活账户，通常邮件中需要有一个链接，这个链接需要具备以下的特性：能够标识用户，该链接具有时效性（通常只允许几小时之内激活），不能被篡改以激活其他可能的账户，一次性的，等待自然过期即可，无需服务端控制其提前终止。这种场景就适合使用jwt。或文件临时一次性分享下载链接

1. [JWT介绍和基本信息](https://javaguide.cn/system-design/security/jwt-intro.html)
1. 实现JWT的时候，SECRET_KEY 是用于对JWT签名的密钥，它必须保持私密性。选择将 SECRET_KEY 存储在哪里通常取决于你的应用程序的需求和安全的最佳实践。以下是几种常见的存储方式：
    - **配置文件**：将 SECRET_KEY 存储在服务器的配置文件中是**比较常见**的做法。为了安全起见，配置文件不应该被包含在版本控制系统中，而是应该在部署的时候通过安全的方式（如使用配置管理工具）添加到应用程序环境中。这种方法对于不需要频繁更改密钥的应用程序来说，既安全又方便。
    - 环境变量：另一种方式是将 SECRET_KEY 存储为环境变量。这意味着密钥直接存储在应用程序运行环境中，这使得更改和管理密钥更加方便。它不会直接暴露在代码或配置文件中，提高了安全性。
    - 密钥管理服务：对于需要更高安全性的系统，可以使用密钥管理服务，如AWS KMS（Key Management Service）等。这些服务通常提供了密钥的存储、轮换和审计的能力。
    - **数据库/缓存系统**：在某些情况下，你也可以将 SECRET_KEY 存储在数据库或缓存系统（如Redis）中，尤其是当你有**一组密钥需要管理**，或者**需要定期更换密钥时**。不过，这种方法可能需要更复杂的管理和额外的安全措施，以确保密钥存储的安全。
    - 不管选择哪种存储方法，都需要确保对SECRET_KEY 的访问受到适当保护，以防止未经授权的访问。另外，密钥**不应该硬编码**在代码中，以**避免在代码库中无意中暴露**它。请根据应用程序的安全需求和可用的基础设施来选择最合适的密钥存储方案。
1. 我将用于对JWT签名的密钥 SECRET_KEY 放在**配置文件**里以后分布式可能还可以放到**配置中心**去
1. 要把 **user_id** 和 **过期时间** 置于 payload 中(用户名在JWT中可以放到 payload 中携带，但敏感的密码不可以放到 payload 中)
1. 点击链接后如果是**绑定邮箱**，要让用户**输入密码**(**防止输入错误的邮箱发给别人，别人一点击就绑定上错误的邮箱了**)，点击链接**注册**或**改密码**时可以不怕发错邮箱(，因为绑定邮箱时已经验证过邮箱了)点进去输要改的密码就好
1. **要实现邮箱激活链接跳转**
    1. 用户注册时，服务器创建一个包含用户信息（通常是用户ID）的JWT，并设置过期时间。
    1. 创建一个**包含JWT的激活链接**，并发送到用户提供的邮箱地址。
    1. 用户点击邮件中的链接，请求发送到服务器的**特定端点**。
    1. 服务器**端点解析JWT**，**验证其有效性**，如果有效，激活用户的邮箱，并可能**重定向**用户到登录页面或其他确认页面。

```yml
jwt:
  secret: YourJWTSecretKey
app:
  url: http://yourapp.com
```

用户注册和发送激活邮件的服务组件：

```java
@Service
public class UserService {

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${app.url}")
    private String appUrl;

    // ... 其他服务方法 ...

    public void registerUser(User user) {
        // 保存用户信息到数据库，设置邮箱未激活
        // userRepository.save(user);

        // 生成JWT
        String token = Jwts.builder()
                .setSubject(String.valueOf(user.getId()))
                .setExpiration(new Date(System.currentTimeMillis() + 86400000)) // 设置24小时过期
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();

        // 创建激活链接
        String activationLink = appUrl + "/activate?token=" + token;

        // 发送激活邮件，您需要配置邮件服务
        // emailService.sendActivationEmail(user.getEmail(), activationLink);
    }
}
```

控制器方法处理激活请求：(包含链接跳转，重定向)

```java
@RestController
public class ActivationController {

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Autowired
    private UserRepository userRepository;

    @GetMapping("/activate")
    public RedirectView activateAccount(@RequestParam String token) {
        String result = "Activation link expired.";
        try {
            Claims claims = Jwts.parser()
                    .setSigningKey(jwtSecret)
                    .parseClaimsJws(token)
                    .getBody();

            Long userId = Long.parseLong(claims.getSubject());
            User user = userRepository.findById(userId);
            if (user != null) {
                user.setActive(true);
                userRepository.save(user);
                result = "Account activated successfully.";
            } else {
                result = "Invalid activation link.";
            }
        } catch (ExpiredJwtException e) {
            result = "Activation link expired.";
        } catch (Exception e) {
            result = "Invalid activation link.";
        }
        // 假设每种激活状态你都有一个对应的前端页面来表示不同的结果
        String redirectUrl = "/activation-status"; // 根据实际情况修改
        
        // 链接跳转
        RedirectView redirectView = new RedirectView();
        redirectView.setUrl(redirectUrl); // 可以是相对路径或绝对路径
        return redirectView;
    }
}
```

### 发送验证邮件的接口怎么限流(例如，1小时内同邮箱只能发送5次验证邮件，同IP只能发送10次验证邮件，发送间隔至少1分钟)？前端限流，后端怎么使用 Redis缓存 实现？

1. 前端页面限流，防止用户连续点击(可能无恶意){前端做**倒计时限制**，时间未到**不能点击**发送短信按钮}
1. 后端限流，防止抓取到发送验证码的接口后构造请求恶意攻击
    1. [限制的方法](https://blog.csdn.net/TM007_/article/details/132964854)
    1. 用 **SEND_LIMIT_IP_KEY(Redis 存储类的key抬头，例如"api:send:limit:ip") + IP地址** 做 Redis 的key，设定1小时的过期时间，value存储已发次数
    1. 用 **SEND_LIMIT_EMAIL_KEY(Redis 存储类的key抬头，例如"api:send:limit:email") + 邮箱地址** 做 Redis 的key，设定1小时的过期时间，value存储已发次数
    1. 发送间隔至少1分钟(做时间间隔限制，时间未到**不发**送邮件/短信)，也可以类似的用Redis实现 {还是使用设定过期时间的方法} 或者 {value存储上一次发送时间，判断当前时间与上一次发送时间间隔是否大于1分钟}
1. 待使用的方法：
    1. 增加图片验证码：发送短信验证码时，要求输入图片验证码，每个图片验证码仅能使用1次，使用1次后，不管输入的图片验证码是否正确自动失效。如果输入错误更新图片验证码。图片验证码失效可以防止图片验证码识别软件尝试多次识别，可以考虑复杂的图片验证码或点触验证、滑动验证。
    1. 上行短信验证码：对于可疑用户要求其主动发短信。
    1. [链接](https://blog.csdn.net/zengdeqing2012/article/details/79625477)

### 为什么使用 {Spring Session + Redis}(通过 Redis 实现了*分布式Session*，*避免了*使用*分布式架构*时*用 tomcat 保存 session 时*会出现的用户登录失效的问题) 而不用 JWT？单点登录（SSO）？

1. JWT 的有效性无法主动注销用户的已登录状态，只能等待 token 过期，更改密码或点击退出后之前的 token 依然有效 (设置黑名单/Redis 保存 JWT 的状态可以撤销但与 JWT 不在服务端存储状态的理论相违背，变成类似session的东西了)，而 Spring Session + Redis 可以通过修改 Redis 中的 session 数据来主动撤销用户的登录状态 (单点退出后其他服务也一并退出，SSO)方便后续做单点登录
    - 单点登录（SSO）适合在多个应用系统中，只需登录一次就可以访问其他相互信任的应用系统时使用。这种方法提高了用户体验，减少了重复登录的繁琐，特别适用于企业内部网或外部网、学生门户网站、公有云服务等环境。通过单点登录，用户可以在一次登录后访问多个相关应用和服务，无需重复输入凭证，节省时间并提高便利性。然而，需要注意凭证泄露可能带来的安全风险，因此建议结合多因子身份验证（MFA）以增强安全性
1. 适合使用jwt的场景：有效期短 / 只希望被使用一次
    - **建议将 JWT 存放在 localStorage 中，放在 Cookie 中会有 CSRF 风险。**
    - 比如，用户注册后发一封邮件让其激活账户，通常邮件中需要有一个链接，这个链接需要具备以下的特性：能够标识用户，该链接具有时效性（通常只允许几小时之内激活），不能被篡改以激活其他可能的账户，一次性的，等待自然过期即可，无需服务端控制其提前终止。这种场景就适合使用jwt。或文件临时一次性分享下载链接
1. 而由于jwt具有一次性的特性。单点登录和会话管理非常不适合用jwt，如果在服务端部署额外的逻辑存储jwt的状态，那还不如使用session。基于session有很多成熟的框架可以开箱即用，但是用jwt还要自己实现逻辑。
1. 使用session与redis结合，因为单纯的使用session缓存数据的话，当一次会话结束后，session就会消失，如果结合redis使用则不会出现数据失效的情况。
1. JWT (类似身份证)无需在服务端存储 session 信息，不用数据库的查询(仅需存储解密密钥，使用密钥校验)，对跨域友好，可以在不同应用间共享用户信息
1. 用 OAuth 2.0 集成第三方登录 QQ、支付宝、钉钉、GitHub......
1. 使用 SpringSesion 的思路是：除了网关服务，每个服务都引入 SpringSession，使用 redis 实现 session 同步。获取信息时，根据 sessionId 从服务器 session 取，如果没有，连接 redis 服务器获取。
1. 使用 jwt 的话，我们在卡号服务里生成一个 jwtToken，放在 cookie 里，由网关服务解密后放在请求头里，这样的话，所有服务都可以从请求头里获取卡号信息。
1. JWT 只能传输非敏感的人员数据，base64解码后就可以得到json，每次请求都需要在header中携带token信息，增大了带宽的压力
1. JWT (Json Web Token) 分为三部分，header、payload(载荷)、signature(签名)。前后端分离开发模式下，token加密后前端请求服务端获取授权，完成登陆校验，其中我们可以拿到payload中的内容，来回显到前端展示界面
1. session是存在服务端的，他是后端的产物，在前后端不分离场景下，单体应用、分布式架构下都能很好的担任他的角色，特别是有SpringSession这一完美解决方案，将session的令牌存在浏览器cookie中，数据存储在redis；但是在前后端分离流行的当下，特别是微服务架构下，JWT以简洁、易用更能胜任校验状态的角色，JWT更是最流行的跨域认证解决方案，通过加密，载荷也相对安全的存储了用户的信息，前端也更容易拿到这一信息，并把信息存储在localStorge或cookie中

### 前后端分离时使用 {Spring Session + Redis}(*实现分布式Session*)，如果客户端是 手机app/小程序 等非浏览器，没有cookie实现保存不了cookie怎么办？

1. Spring Session **默认**使用 **cookie** 保存sessionid (对**浏览器**方便)，如果客户端是 **手机app/小程序** 等**非浏览器**，**没有**cookie实现**保存不了**cookie就不能通过cookie来访问session对象，需要在登录成功后，将sessionid添加到响应数据中，**保存在客户端**，下次用户访问，就在请求头当中带上sessionid ; 用 Spring Session 中的 ```HeaderHttpSessionIdResolver``` 实现
1. 用 Spring Session 中的 ```HeaderHttpSessionIdResolver``` 实现时，当会话被创建时，HTTP响应会包含一个指定名称的响应头和会话ID的值。例如，响应头可能如下所示：```X-Auth-Token: f81d4fae-7dec-11d0-a765-00a0c91e6bf6``` 客户端在每个请求中应该通过指定相同的头来包含会话ID。当会话被下线无效时，服务器将发送一个带有相同头名称但值为空的HTTP响应
1. 确保会话ID被改后用户不会登录成别人：
    - 使用安全的会话管理机制：确保在用户登录认证成功后，将用户的登录凭证放入会话中进行管理
        - 例如，会话代替方案：考虑采用会话代替方案，如使用单次令牌或其他安全机制来代替传统的会话ID
    - 注意会话ID的安全性：尽量避免会话ID泄露给他人，以防止他人伪造请求。通过HTTPS协议传输会话ID可以加密通信内容
    - 定期更新会话ID：定期更新会话ID可以增加安全性，减少被猜测或利用的可能性
    - 限制会话ID的有效期：设置会话ID的有效期，当超过一定时间后自动失效，减少被滥用的风险
    - 实施防护措施：考虑使用防护机制如防止会话固定攻击和跨站请求伪造等安全措施来保护会话安全

### 签名和加密有啥区别？MD5签名认证怎么实现的？怎么防止请求头被篡改？怎么防止重放攻击？

1. 签名是不可逆的，不存储原数据，只能做签名验证 ; 加密是可逆的，能够解密
1. 为防止重放攻击，加上了timestamp字段
1. 在数据库里的 **登录密码** 不用 **明文** 保存，保存用**MD5**算法对 **明文密码+盐值** 数据生成的 **签名**，网页表单Post提交用户输入的密码，将 **提交的密码加上盐值** 后用MD5生成签名，把生成的签名与保存在数据库里的签名对比，相同则登录成功
**登录密码** 通过网络传输了，而 **secret_key** 不通过网络传输，所以在数据库中是否保存明文、用于校验合法的方法...... 对于 **登录密码** 和 **secret_key** 使用的设计不一样
1. MD5签名没有信息所以传输时参数要包含原数据(Base64编码包裹传输)，例如access_key(方便服务器通过access_key查到secret_key)、timestamp......(请求头字段，但密钥secret_key别放在请求头明文传输)，secret_key包含在生成并传递过来的MD5签名中了 ; 服务器通过access_key查到secret_key用secret_key再加上**请求头的JSON**作为参数进行MD5签名与传递来的MD5签名对比，相同则签名验证通过(secret_key正确)
    - 为防止请求头被篡改：使用**请求头的JSON**加上**secret_key**一起签名，保证请求头没被篡改
1. MD5秒传(提取文件签名，对比签名，相同 {认为文件一致} 则秒传)
1. **RSA**也可用于签名，**MD5**不够安全

### 登录时怎么保护通过POST在网络上传递的账号密码明文数据安全？其与使用access_key和secret_key签名验证的网关接口服务有啥区别？Spring Boot中怎样将HTTP请求自动重定向到HTTPS请求？

1. Spring Boot使用HTTPS保证POST请求中传递的密码安全 ; 签名验证的网关接口有防重放保护，请求时secret_key不在网络上传递，无论使用的是 HTTP 还是 HTTPS 都能保证安全
1. Spring Boot配置SSL证书开启HTTPS请求，并将HTTP请求转换成HTTPS请求。用acme.sh脚本自动申请泛域名SSL证书，并自动续期(见**科学上网收藏夹**中申请SSL证书的步骤)【生成证书前必须将认证服务设置为letsencrypt（Let's Encrypt）】
    - [Spring Boot配置步骤见链接](https://blog.csdn.net/qq_42347616/article/details/120653643)
    - 生成SSL证书 — 你需要为你的应用生成SSL证书。你可以使用Java的keytool或者Let's Encrypt等服务来生成一个自签名的SSL证书。
    - 配置SpringBoot以使用SSL证书 — 一旦你拥有了SSL证书，需要将它配置到SpringBoot应用中。这通常需要在你的 ```application.properties``` 或 ```application.yml``` 文件中设置相关的SSL属性。
    - 在Spring Boot应用中配置Connector进行HTTP到HTTPS的重定向，创建一个新的配置类来添加额外的Connector：(代码见下)

```yml
server:
  port: 8080 ## 后台服务对外端口
  ssl:
    key-store: server.keystore ##秘钥库文件名称，即上面生成的自签名证书
    key-store-password: 123456 ## 生成秘钥库文件的密码
    key-store-type: JKS ## 秘钥库类型（JKS为jdk的keytool工具默认生成的秘钥库类型）
    key-alias: tomcat ## 秘钥别名
    enabled: true
```

```java
    import org.apache.catalina.connector.Connector;
    import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
    import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    @Configuration
    public class ServerConfig {
    
        @Bean
        public ServletWebServerFactory servletContainer() {
            TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
                @Override
                protected void postProcessContext(Context context) {
                    SecurityConstraint securityConstraint = new SecurityConstraint();
                    securityConstraint.setUserConstraint("CONFIDENTIAL");
                    SecurityCollection collection = new SecurityCollection();
                    collection.addPattern("/*");
                    securityConstraint.addCollection(collection);
                    context.addConstraint(securityConstraint);
                }
            };
            tomcat.addAdditionalTomcatConnectors(redirectConnector());
            return tomcat;
        }
        
        private Connector redirectConnector() {
            Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
            connector.setScheme("http");
            connector.setPort(8080); // 这是HTTP端口
            connector.setSecure(false);
            connector.setRedirectPort(8443); // 这是在上面配置的HTTPS端口
            return connector;
        }
    }
```

> 这段代码将会创建一个新的HTTP Connector，当它接收到HTTP请求时，它会将请求重定向到已配置的HTTPS端口。请确保redirectConnector方法中设置的端口与你的需求相符，setPort方法设置的是你希望HTTP请求监听的端口（默认是8080），而setRedirectPort是你的HTTPS端口（以上代码中为8443）。
> 现在当你尝试通过HTTP访问你的应用时，例如访问 ```http://localhost:8080``` ，Tomcat应当会自动将请求重定向到 ```https://localhost:8443``` 。

### RPC与MQ的区别以及MQ的使用场景？

1. [RPC介绍链接](https://javaguide.cn/distributed-system/rpc/rpc-intro.html)
1. [消息队列介绍链接](https://javaguide.cn/high-performance/message-queue/message-queue.html)
1. [见链接](https://zhuanlan.zhihu.com/p/97841943)
1. RPC是远程过程调用，MQ是消息队列，RPC通常是同步的 请求/响应 调用，MQ是异步的流处理
1. N个不同系统相互之间都有RPC调用，依赖程度深，引入MQ降低耦合度
1. MQ实现RPC会造成更大通讯开销，不要强行替代
1. MQ异步有自动重传重试，与http同步调用相比能提高系统的可靠性，http要自己实现重传重试的补偿定时任务
1. MQ削峰/限流

### 搜索怎么实现？

1. 在上线第一个版本，因为流量很少，搜索模块直接采用多字段分别模糊匹配的方案
1. 后续流量增长后，设计了mysql主从分离，并在从库进行全文检索(从 MySQL5.6 开始，InnoDB 开始支持全文检索{Full-Text Search}，看看索引介绍)
1. 超大流量才用 ElasticSearch 全文检索，会增加运营成本

### Token核心字段？

1. userId
1. 有效期

### user/admin 访问的身份如何控制

使用 Spring 中 AOP 配合自定义注解进行鉴权(目前只实现了方法拦截, 对类拦截要使用反射[后续可能更改为Spring Security]) ;
具体实现在/aop/AuthInterceptor 和 /annotation/AuthCheck 中

### 流程图

![流程图1](<https://raw.githubusercontent.com/lowoneko/public-imgs-1/main/public-imgs/image-20230112101821991.png>)
![流程图2](./assets/API开放平台项目整理/流程图.png)
