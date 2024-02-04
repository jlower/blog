---
title: 用企业微信的API给微信发消息
date: 2024-02-05 03:00:00
categories: 实用脚本与解决方案
---

## 用企业微信的API

```python
import requests, sys


class SendWeiXinWork:
    def __init__(self):
        self.CORP_ID = ""  # 企业号的标识
        self.SECRET = ""  # 管理组凭证密钥
        self.AGENT_ID =   # 应用ID
        self.token = self.get_token()

    def get_token(self):
        url = "https://qyapi.weixin.qq.com/cgi-bin/gettoken"
        data = {"corpid": self.CORP_ID, "corpsecret": self.SECRET}
        req = requests.get(url=url, params=data)
        res = req.json()
        if res["errmsg"] == "ok":
            return res["access_token"]
        else:
            return res

    def send_message(self, to_user, content):
        url = (
            "https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=%s"
            % self.token
        )
        data = {
            "touser": to_user,  # 发送个人就填用户账号
            # "toparty": to_user,  # 发送组内成员就填部门ID
            "msgtype": "text",
            "agentid": self.AGENT_ID,
            "text": {"content": content},
            "safe": "0",
        }

        req = requests.post(url=url, json=data)
        res = req.json()
        if res["errmsg"] == "ok":
            print("send message succeed")
            return "send message succeed"
        else:
            print("send message error", res)
            return res


if __name__ == "__main__":

    # 导入时间模块
    import datetime

    # 获取当前时间
    now = datetime.datetime.now()
    # 创建一个指定时间的日期对象
    date = datetime.datetime(2024, 5, 15, 23, 59, 59)
    # 计算时间差
    delta = date - now
    msg0 = "网站my.freenom.com的域名还有 {} 天就过期, 请提前14天去更新".format(
        delta.days
    )

    SendWeiXinWork = SendWeiXinWork()
    SendWeiXinWork.send_message("USER NAME HERE", msg0)
```
