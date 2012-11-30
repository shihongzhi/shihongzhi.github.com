---
layout:     post
title:      两道python面试题
category: tech
description: 最近做了两道知乎的面试题，收获还是比较大的。
tags: python, interview
---

最近做了两道知乎的面试题，收获还是比较大的。

##写一个邮件发送的脚本

接受参数输入发件人邮箱、发件人密码，收件人地址，以及若干文件名，要求以文件为附件，发到收件人邮箱去。

要求: 至少支持gmail,163,sohu,sina,qq邮箱

[code](https://gist.github.com/2780008):

<script src="https://gist.github.com/2780008.js">
</script>


##antispam系统

假设我们可以获得线上的实时请求（按时间顺序）

每个请求包含如下信息：

    时间（unix时间戳）

    用户名

    动作（提问、回答、评论）

    内容

依次考虑如何解决以下问题：

>1. 当发现动作频率较高的用户时，输出报警（1分钟内超过10次评论、回答、提问）
>2. 当发现一个用户连续发重复的内容时，输出报警（连续发了10个相同的评论、回答、提问）
>3. 使用你觉得最优美的方法将上面的策略与程序分离开。使得上面的策略可以随时更新而不用修改程序。

要求:

服务监听一个端口，使用测试程序模拟用户行为不断向服务发送请求

请求格式为json如：
`{"time":1323333,"user":"xxx","action":"answer","content":"abcde"}`

服务输出报警格式如下：
`xxx,"频繁提问"`

[code](https://gist.github.com/2791877):
<script src="https://gist.github.com/2791877.js">
</script>
