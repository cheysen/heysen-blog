# 浅谈XSS

## 1. XSS简介

跨站脚本(Cross Site Script)为了避免与CSS混淆,简称XSS。XSS是指攻击者利用网站没有对用户提交数据进行转义处理或者过滤不足的缺点，在输入框添加一些代码，嵌入到web页面中，使别的用户访问会执行相应的嵌入代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。XSS又分为反射型、存储型和DOM-Based型。

XSS的危害包括：

- 盗取用户Cookie

- DDOS客户端浏览器
- 盗窃企业重要的具有商业价值的资料
- 爆发web2.0蠕虫
- 强制发送电子邮件
- 网站挂马，钓鱼攻击
- 劫持用户web行为甚至渗透内网

## 2. 原理

（1）反射型

> 服务端返回脚本，客户端执行。如URL注入非法脚本，然后诱导用户点击该链接，但是一般浏览器都会有基本防御措施，自带拦截过滤；服务端返回的富文本中包含非法脚本，被直接展示。

（2）存储型

> 后台被动存储了非法脚本，并且前端直接展示。如发帖或留言中发出包含恶意代码的内容,其他用户访问该内容后,满足特定条件即触发,这种需要后台不过滤信息,并且前端展示时也不过滤信息。

（3）DOM-Based型

> 基于DOM或本地的XSS攻击。如wifi流量劫持、DNS劫持并直接返回钓鱼页面。本质是需要更改DOM，再排除自己攻击自己。某些反射型的攻击也能造成这个后果-通过url控制DOM。在传统的XSS中，恶意JavaScript脚本在页面加载时执行，而在基于DOM的XSS中，恶意JavaScript脚本在页面加载后的某个时刻执行。

## 3.自动化XSS

Browser Exploitation Framework(BeEF),是目前强大的浏览器开源渗透测试框架,通过XSS漏洞配合JS脚本和Metasploit进行渗透。它是基于Ruby编写，支持图形化界面，操作简单。
{{< image src="https://cdn.heysen.xyz/201905022124_805.jpg" >}}


Kali linux已经自带BeEF,另外我使用一个开源的靶机OWASP Broken Web Apps来练习。

在靶机的低安全级别下执行一个存储型的XSS：
{{< image src="https://cdn.heysen.xyz/201905022117_458.jpg" >}}


然后使用win10的Google浏览器访问靶机的相应被注入脚本的页面后，BeEF中的记录：
{{< image src="https://cdn.heysen.xyz/201905022119_667.png" >}}


之后就可以使用beef控制用户的浏览器进行多种操作（比如钓鱼），但是这里并没有win10的记录，原因是chrome对该XSS进行了过滤。
{{< image src="https://cdn.heysen.xyz/201905022121_317.png" >}}


## 4. 如何预防

（1）编码，转义用户输入，编码可在前端和后端中进行。常见的如下图的HTML转义
{{< image src="https://cdn.heysen.xyz/201905022217_212.png" >}}


但是编码并不适用所有情况，比如当用户需要提供一个URL或者编写用户配置文件时，对输入编码是不友好的，不应对用户的自定义配置做限制，所以需要使用验证。

（2）验证，过滤用户输入。有两种实现方式

 - 分类策略，使用白名单或黑名单分类

 - 验证结果，拒绝或删除不合法的输入

   

（3）CSP内容安全策略（CSP）

CSP用于约束浏览器只能使用从可信来源下载的资源。资源是页面引用的脚本，样式表，图像或其他类型的文件。这样即使攻击者成功将恶意内容注入到网站，CSP也可以防止它被执行。

CSP可用于执行以下规则：

- 没有不受信任的来源。外部资源只能从一组明确定义的可信来源加载。

- 没有内联资源。不会评估内联JavaScript和CSS。

- 无法使用JavaScript eval函数。

```http
Content‑Security‑Policy:
    script‑src 'self' scripts.example.com;
    media‑src 'none';
    img‑src *;
    default‑src 'self' http://*.example.com
```

以上CSP表明脚本只能从提供页面的主机和scripts.example.com下载。

无法从任何地方下载音频和视频文件。可以从任何主机下载图像文件。

所有其他资源只能从提供该页面的主机和example.com的任何子域下载。

 (4) 其他方法

- Cookie设置http-only。http-only标志可以防止cookie被“读取”，但不能防止被“写”,已证明有些浏览器的http-only标记可以被JavaScript写入覆盖，而这种覆盖可能被攻击者利用发动session fixation攻击。
- WAF,大多数使用规则匹配,也能被绕过。
- [X-XSS-Protection](https://link.zhihu.com/?target=https%3A//www.owasp.org/index.php/Security_Headers)也有助于在一些浏览器中防止某些XSS，但在某些情况下可以被绕过。
