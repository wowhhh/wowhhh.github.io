---
layout: post
title:  "Spider for stackoverflow"
tags:   Python Scraper
date:   2020-09-17 10:00:00 +0800
categories: [Spider]
---

## 需求

- 实现登录StackOverflow功能，并且可以获取cookie
- 根据传入的keywords，获取所有检索结果
- 根据检索结果，获取每个post中的相关信息

## 登录StackOverflow

主要使用Session完成整个过程，其中Session由```requests.Session()```进行获取。

1：初始化请求头

```python
   def __initForBrowser(self):
        # 设置请求头
        # User-Agent 表示浏览器请求头
        # Content-Type 表示网络文件的类型和网页的编码
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119 Safari/537.36',
            'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'
        }
        # 登录URL
        self.login_url = 'https://stackoverflow.com/users/login'
        self.home_url = 'https://stackoverflow.com/'
        # 将请求头更新到session中
        self.session.headers.update(self.headers)
```



2：初始化数据

- **fkey**、ssrc、returnurl

  每次访问https://stackoverflow.com/users/login?ssrc=head&returnurl=https%3a%2f%2fstackoverflow.com%2f的时候，都会返回fkey用于后续登录传输数据验证。

  获取方式为：通过链接https://stackoverflow.com/users/login?ssrc=head&returnurl=https%3a%2f%2fstackoverflow.com%2f访问，并获得response，在response中使用正则```"fkey":"([^"]+)"```解析得到fkey

  ```python
  def __getFkey(self):
      params = {
          'ssrc': 'head',
          'returnurl': 'https://stackoverflow.com/'
      }
      res = self.session.get(self.login_url, params=params)
      fkey = re.findall(r'"fkey":"([^"]+)"', res.text)[0]
      return fkey
  ```

- username & password

  StackOverflow的用户名以及密码

- 参数封装

  ```python
  data = {
      'openid_identifier': '',
      'password': password,
      'fkey': fkey,
      'email': username,
      'oauth_server': '',
      'oauth_version': '',
      'openid_username': '',
      'ssrc': 'head'
  }
  # ssrc=head&returnurl=https%3a%2f%2fstackoverflow.com%2f
  params = {
      'ssrc': 'head',
      'returnurl': 'https://stackoverflow.com/'
  }
  ```

3：提交登录

```python
# 提交登录
res = self.session.post(self.login_url, data=data, params=params)
```

4：判断登录状态，获取Cookie

```python
if res.history:
    res = self.session.get(self.home_url)
    profile_url = 'https://stackoverflow.com' + re.findall(r'<a href="(.+)" class="my-profile', res.text)[0]
    print('[INFO]: Account -> %s, login successfully...' % username)
    infos_return = {'username': username, 'fkey': fkey, 'profile_url': profile_url}
    return infos_return, self.session
# 登录失败
else:
    raise RuntimeError('Account -> %s, fail to login, username or password error...' % username)
```

## 爬取StackOverflow检索的概述

- 检索的内容

![image-20200917103228253](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-15/image-20200917103228253.png)

- 定义字段
- Spider

![image-20200917103254198](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-15/image-20200917103254198.png)

定义字段->

```python
_id = scrapy.Field()
questions= scrapy.Field()
votes = scrapy.Field()
answers = scrapy.Field() #answer数目
# views = scrapy.Field() 搜索页面不显示view数目
soLinks = scrapy.Field()
askTime = scrapy.Field()
tags = scrapy.Field()
userLinks = scrapy.Field()
```

```json
{'_id': 'question-summary-30719047',
 'answers': '25',
 'votes': '313',
 
 'questions': '\r\nQ: Android M - check runtime permission - how to determine if the user checked “Never ask again”?        ',
 'soLinks': '/questions/30719047/android-m-check-runtime-permission-how-to-determine-if-the-user-checked-nev?r=SearchResults',
 'tags': ['/questions/tagged/android',
          '/questions/tagged/android-permissions',
          '/questions/tagged/android-6.0-marshmallow'],
 'userLinks': '/users/534471/emanuel-moecklin',
 'askTime': '2015-06-08 21:03:34Z',
 }
```

保存数据的话需要根据id区分

question的id为：question-summary-POSTID

answer的id为：answer-id-POSTID



设置header、cookie、延时爬取

https://www.zbpblog.com/blog-109.html



StackOverflow设置的cookie主要是关注：

```
prov=0cdec603-244b-ad55-8a4e-42e658c35d67; _ga=GA1.2.1006163756.1598961205; __qca=P0-2133854003-1598961206504; __gads=ID=70fa45c44fcc54a3:T=1598961206:S=ALNI_MZScVyvqfXve35l7kAU36gWrG_47A; notice-ssb=1%3B1599107227134; _gid=GA1.2.812044257.1600265005; usr=p=%5b10%7c15%5d; _gat=1; **acct=t=jMDeVTl85Wm7%2fAj2FjtTGAYazgFszpG%2f&s=Vf2kO0wPL755g2FqVARg4l480Opxu%2b4A**
```



已经爬取搜索页面的结果的概述，

下一步需不需要爬取单独的post页面，获取：

- 总的Votes

- 总的Views

- 是否由accept的答案

- 单个votes比较高的answer

  Post：

  ​	Votes、collects、asked time,active time ,viewed,users

  answer：

  ​	votes、answerID、answer/edit time（last modified）、accepted？、users

##### 创建scrapy项目

```scrapy startproject 项目名称```

先创建数据库

再定义Item

再写爬取逻辑