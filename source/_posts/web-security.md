---
title: 《白帽子讲 Web 安全》总结
date: 2017-02-28 18:45:26
tags: 总结
---
简介：介绍XSS（跨站脚本攻击）、CSRF (跨站请求伪造)和原理、应用和防御
<!-- more -->

## XSS（跨站脚本攻击）
**使用场景**
1. cookie劫持
2. 模拟请求：删文章，发评论等
3. 钓鱼：画出登录窗口，让用户输入密码
4. 利用扩展程序、浏览器控件

**防御方法**
1. httpOnly 防御 XSS 对 Cookies 的劫持
2. CSP 防止插入来历不明的脚本
3. 输入检查
4. 输出检查

## CSRF (跨站请求伪造)
原理：模拟用户发送请求
案例：引诱用户访问某页面，在该页面发送请求，删除用户的博客

**防御方法**
* 验证码：即需要用户确认操作
* Referer：验证请求的源
* Token：CSRF 的本质是因为请求可预测，如`delete=blogId`，如果请求需要带上不可预测的 token，CSRF 就没办法了，如`delete=blogId&token=tokenId`，实现步骤后端把 token 站在 cookie 里，因为 CSRF 只是“请求”伪造，是没办法获得 cookie 的

## Cookie 的分类
1. session cookie：浏览器关闭后失效，没有指定 expire 时生成的 cookie，存于浏览器内存
2. Third-party cookie：到了 expire 后失效，指定 expire 时生成的 cookie，存于**本地**！