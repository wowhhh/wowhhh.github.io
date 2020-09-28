---
layout: post
title:  "Spider for stackoverflow"
tags:   Python Scraper
date:   2020-09-17 10:00:00 +0800
categories: [Spider]
---

- 参考链接

  https://www.zbpblog.com/blog-109.html

  https://github.com/CharlesPikachu/DecryptLogin

## 需求

- 实现登录StackOverflow功能，并且可以获取cookie
- 根据传入的keywords，获取所有检索结果
- 根据检索结果，获取每个post中的相关信息

## 登录StackOverflow

主要使用Session完成整个过程，其中Session由```requests.Session()```进行获取。

#### 1：初始化请求头

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



#### 2：初始化数据

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

#### 3：提交登录

```python
# 提交登录
res = self.session.post(self.login_url, data=data, params=params)
```

#### 4：判断登录状态，获取Cookie

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

先用命令行创建一个Scrapy的项目：```scrapy startproject SoScrapySpider```

#### 需要获取的内容

如下图所示，Votes、Answers、Title、Tags、Time、User这六项为我们需要获取是内容

![image-20200917103228253](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-15/image-20200917103228253.png)

除此之外SO的检索结果还包括Answer，对于这种，获取的内容如下：

votes、title、time、user

![image-20200917103254198](https://raw.githubusercontent.com/ARP2019/ImageUpload/master/img/2020-09-15/image-20200917103254198.png)

#### 定义字段

字段指的是用于保存获取到的内容，将其定义在Items中:

```python
# 这里定义我们需要获取的字段
class StackoverflowPythonItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    _id = scrapy.Field() # Post ID
    questions = scrapy.Field() # Q||A 的内容
    votes = scrapy.Field() # votes
    answers = scrapy.Field()  # answer数目
    # views = scrapy.Field() 搜索页面不显示view数目
    soLinks = scrapy.Field() # url
    askTime = scrapy.Field() # time
    tags = scrapy.Field() # tags
    userLinks = scrapy.Field()# user
    pass
```

#### 爬取逻辑

爬取过程分为：获取URL集合，循环处理单个URL获取response页面，对response页面分析获取div集合，循环处理单个div。

并且请求获取页面的过程模拟一下浏览器的请求头，加入cookie。据本人判断，StackOverflow的cookie信息中，```acct```最为重要，可以根据刚才写的登录代码进行获取。

```python
class SO_Post_Spider(scrapy.Spider):
    name = 'SO_KeywordsResults'
    # 请求链接的格式
    '''
    https://stackoverflow.com/search?page={}}&tab=Relevance&q=android%20runtime%20permission
    2020-09-17 : page range is : 1-188
    '''
    cookieString = "prov=0cdec603-244b-ad55-8a4e-42e658c35d67; _ga=GA1.2.1006163756.1598961205; __qca=P0-2133854003-1598961206504; __gads=ID=70fa45c44fcc54a3:T=1598961206:S=ALNI_MZScVyvqfXve35l7kAU36gWrG_47A; notice-ssb=1%3B1599107227134; _gid=GA1.2.812044257.1600265005; usr=p=%5b10%7c15%5d; _gat=1; acct=t=pItCB47IYi%2fuYPCKRnHhJcjTjdCUN97q&s=CPRVITNrnkvqjiqOgw2PlayW3UHHjhE1"
    headers = {
        # ':authority': 'stackoverflow.com',
        # ':method': 'GET',
        # ':scheme': 'https',
        'accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
        "accept-encoding": "gzip, deflate, br",
        "accept-language": "zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6",
        'cache-control': 'max-age=0',
        # "Content-Type": "text/html; charset=utf-8",
        "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36",
        "sec-fetch-dest": 'document',
        "sec-fetch-mode": "navigate",
        "sec-fetch-site": "none",
        "sec-fetch-user": "?1",
        "upgrade-insecure-requests": "1"
    }

    def start_requests(self):
        urls = []
        _url = 'https://stackoverflow.com/search?page={}&tab=Relevance&q=android%20runtime%20permission'
        for page in range(1, 189):  # 189 next start 34 ;123 in 16:52
            urls.append(_url.format(page))
        for url in urls:
            self.headers["cookie"] = self.cookieString
            yield scrapy.Request(url=url, headers=self.headers,
                                 callback=self.parse)  # 回调方法写到parse中
    '''
    此方法解析标签，提取值
    '''
    def parse(self, response):
        question_list = response.xpath('//*[@class="question-summary search-result"]')
        # 每一个字段都包含在了id为：questions-summary-postID的div内
        if len(question_list) == 0:
            print("需要通过人机验证，然后重试。")
        for question in question_list:  # 获取它下面的所有 div
            item = StackoverflowPythonItem()
            item['_id'] = question.attrib['id']  # id="question-summary-30719047" 注意这里id可能会重复，因为Q和A，只有Q的话不会重复
            temp = question.xpath('div[2]/div[1]')
            temp2 = question.xpath('div[2]/div[1]/h3/a')
            item['questions'] = question.xpath('div[2]/div[1]/h3/a/text()').get()
            item['votes'] = question.xpath('div[1]/div[1]/div[1]/div[1]/span/strong/text()').get()
            item['answers'] = question.xpath('div[1]/div[1]/div[2]/strong/text()').get()
            item['soLinks'] = question.xpath('div[2]/div[1]/h3/a/@href').get()
            item['askTime'] = question.xpath('div[2]/div[4]/span/@title').get()
            tagLists = question.xpath('div[2]/div[3]/a')
            tempTags = []
            for tag in tagLists:
                tempTags.append(tag.xpath('@href').get())
            item['tags'] = tempTags
            item['userLinks'] = question.xpath('div[2]/div[4]/a/@href').get()
            yield item
       		print("Item Done")
```

#### 爬取结果

上述实现后的爬取结果如下展示：

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

#### 持久化数据到MySql

对于上面的爬取代码，我将数据保存到了两个表中：```question-info```和```answer-info```，持久化数据的代码将其写到```piplines.py```中

```python
class SoscrapyspiderPipeline:
    def __init__(self):
        # 连接数据库 For New PC
        self.connect = pymysql.connect(host='localhost', user='root', password='password',db='db', port=3306)
        self.cursor = self.connect.cursor()
    def process_item(self, item, spider):
        # 往数据库里面写入数据
        # 判断post是Question还是answer
        flag = item["_id"]
        if 'question' in flag:
            self.cursor.execute(
            'insert into question_info()VALUES ("{}","{}","{}","{}","{}","{}","{}","{}")'.format
            (item['_id'], item['answers'], item['votes'], item['questions'], item['soLinks'], item['tags'], item['userLinks'], item['askTime']))
        else:
            self.cursor.execute(
                'insert into answer_info()VALUES ("{}","{}","{}","{}","{}","{}","{}")'.format
                (item['_id'], item['votes'], item['questions'], item['soLinks'], item['tags'],
                 item['userLinks'], item['askTime']))
        self.connect.commit()
        return item

    def close_spider(self):
        self.cursor.close();
        self.connect.close();
```

写完之后还是需要在```settings.py```中设置一下：

```python
ITEM_PIPELINES = {
   'SoScrapySpider.pipelines.SoscrapyspiderPipeline': 300,
}
```



## 爬取StackOverflow单个Post的内容

这里有与上面重复的步骤就不多说了，主要写一下Item的定义，标签的匹配。

#### 需要获取的信息

Post以及Answer的Vote和、Views、是否有accept的答案、collects、asked time、active time 

#### 字段的定义

```python
class SoPostItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    postId = scrapy.Field()
    askedTime = scrapy.Field()
    activeTime = scrapy.Field()
    viewed = scrapy.Field()  # answer数目
    # views = scrapy.Field() 搜索页面不显示view数目
    collects = scrapy.Field()
    allVotes = scrapy.Field()
    acceptedAnswer = scrapy.Field()
    pass
```

#### 爬取逻辑

其他部分代码与上述过程基本相同：

```python
def parse(self, response):
    item = SoPostItem()
    # ask time/activity time/views
    try:
        postId = response.xpath('//*[@class="question"]').attrib['data-questionid']
    except Exception:
        postId = ""
    item["postId"] = postId

    try:
        askedTime = response.xpath('//*[@class="grid--cell ws-nowrap mr16 mb8"]').attrib['title']
    except Exception:
        askedTime = ""
    item["askedTime"] = askedTime

    try:
        activeTime = response.xpath('//*[@class="s-link s-link__inherit"]').attrib['title']
    except Exception:
        activeTime = ""
    item["activeTime"] = activeTime

    try:
        viewed = response.xpath('//*[@class="grid--cell ws-nowrap mb8"]').attrib['title']
    except Exception:
        viewed = ""
    item["viewed"] = viewed.replace(",", "").replace("Viewed ", "").replace("times", "")
    try:
        collects = response.xpath('//*[@class="js-bookmark-count mt4"]').attrib['data-value']
    except Exception:
        collects = '0'
    item["collects"] = collects

    # 包括post和answer的votes
    voteSum = 0
    try:
        allVotes = response.xpath(
            '//*[@class="js-vote-count grid--cell fc-black-500 fs-title grid fd-column ai-center"]')
        for single in allVotes:
            vote = single.attrib['data-value']
            voteSum += int(vote)
    except Exception:
        voteSum = 0
    item["allVotes"] = voteSum

    try:
        acceptedAnswer = response.xpath('//*[@class="answer accepted-answer"]').attrib['id']
    except Exception:
        acceptedAnswer = '0'
    item["acceptedAnswer"] = acceptedAnswer
    yield item

    print("Item Done")
```



