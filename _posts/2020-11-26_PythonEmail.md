---
layout: post
title:  "Gmail & Python"
tags:   Python
date:   2020-11-26 00:00:10 +0800
categories: [Utils]
---



```python
import smtplib
from email.mime.text import MIMEText
from email.utils import make_msgid
import time

def sendEmail(receiverName, receiverEmail):
    # 设置服务器所需信息
    # 163邮箱服务器地址
    mail_host = 'smtp.gmail.com'
    # 163用户名
    mail_user = 'xxx@gmail.com'
    # 密码(部分邮箱为授权码)
    mail_pass = 'pass'
    # 邮件发送方邮箱地址
    sender = 'xxx@gmail.com'
    # 邮件接受方邮箱地址，注意需要[]包裹，这意味着你可以写多个邮件地址群发
    receivers = [receiverEmail]

    # 设置email信息
    # 邮件内容设置
    emailContent = '''
    Dear Ms./Mr. ''' + receiverName + ''',
    We are postgraduate students working on a research project about Android runtime permission issues. We are looking for developers who have experience in Android development. If you are one of them, we need your help.
    
    We would like to invite you to participate in our study. The goal of this research is to explore the problems encountered by developers in the process of managing runtime permissions.
    
    We would appreciate if you can share your experiences in a survey. We believe that your sharing can benefit the Android development community. The survey involves 14 questions and would take you 2-3 minutes to complete. 
    
    Suvery link: https://forms.gle/URJUp1XcHPcR3e1NA 
    
    Looking forward to hearing from you. Thanks a lot for your time.
    
    Best regards,
    Android runtime permission research team
    
    '''
    message = MIMEText(emailContent, 'plain', 'utf-8')
    # 邮件主题
    message['Subject'] = 'A Request for Android Runtime Permission Survey'
    # 发送方信息
    message['From'] = sender
    # 接受方信息
    message['To'] = receivers[0]
    message['Message-ID'] = make_msgid()
    # 登录并发送邮件
    try:
        smtpObj = smtplib.SMTP('smtp.gmail.com', 587, timeout=120)
        # 连接到服务器
        # smtpObj = smtplib.SMTP()
        # smtpObj.connect(mail_host,587)
        smtpObj.ehlo()
        smtpObj.starttls()
        smtpObj.ehlo()
        # 登录到服务器
        smtpObj.login(mail_user, mail_pass)
        # 发送
        smtpObj.sendmail(
            sender, receivers, message.as_string())
        # 退出
        smtpObj.quit()
        print('success')
    except smtplib.SMTPException as e:
        print('error', e)  # 打印错误
```