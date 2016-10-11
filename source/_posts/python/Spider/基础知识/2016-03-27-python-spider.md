---
layout: post
title: Python Spider入门
category: Python
tags: [python,spider,pythonspider]
keywords:
description:
---

## 添加header和data

```python
request=urllib.request.Request(url)
response=urllib.request.urlopen(request)
#response是一个http.client.HTTPResponse对象
print(response.geturl()) #获取网址
print(response.info())  #获取信息
print(response.getcode()) # http状态码

html=response.read()
```

`urllib.request.Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)`
1.data参数：the HTTP request will be a POST instead of a GET when the data parameter is provided.data should be a buffer in the standard application/x-www-form-urlencoded format. The urllib.parse.urlencode() function takes a mapping or sequence of 2-tuples and returns a string in this format.<br>

```python
data={}
data['type']='AUTO'
data['i']=content
data['doctype']='json'
data['xmlVersion']=1.8
data['keyfrom']='fanyi.web'
data['ue']='UTF-8'
data['action']='FY_BY_CLICKBUTTON'
data['typoResult']='true'

data=urllib.parse.urlencode(data).encode('utf-8')
```

2.headers：一个字典，可以直接从浏览器中复制过来

```python
header={}
#添加header的第一种方法
header['User-Agent']='Mozilla/5.0 (Windows NT 10.0; WOW64; rv:43.0) Gecko/20100101 Firefox/43.0'
request=urllib.request.Request(url,data,header)
#也可使用以下方法
request.add_header('User-Agent','Mozilla/5.0 (Windows NT 10.0; WOW64; rv:43.0) Gecko/20100101 Firefox/43.0')
```


## 获取Response Headers

下面代码是用来判断 网页是否使用`gzip`压缩过。
```python
for i in response.getheaders():
    if i[0]=="Content-Encoding":
        if(i[-1]=="gzip"):
            html=gzip.decompress(html)
```

## 使用代理



1.参数是一个字典{'类型’：‘代理ip:端口号’}
`proxy_support=urllib.request.ProxyHandler({})`
2.定制、创建一个openner
`opener=urllib.request.build_opener(proxy_support)
3a.安装opener
`urllib.request.install_opener(opener)`
3b.调用opener
`opener.open(url)`

### 代码

```python
proxy_support=urllib.request.ProxyHandler({'http':random.choice(iplist)})

opener=urllib.request.build_opener(proxy_support)
opener.addheaders=[('User-Agent','Mozilla/5.0 (Windows NT 10.0; WOW64; rv:43.0) Gecko/20100101 Firefox/43.0')]
urllib.request.install_opener(opener)

req=urllib.request.Request(url)
response=urllib.request.urlopen(req)
```


## 爬知乎图片

```python
import urllib.request
import os
import random

#打开网页
def url_open(url):
    iplist=[
        '49.77.22.1:8118',
        '58.134.102.3:12696',
        '120.26.213.55:9999'...]

    proxy_support=urllib.request.ProxyHandler({'http':random.choice(iplist)})

    opener=urllib.request.build_opener(proxy_support)
    opener.addheaders=[('User-Agent','Mozilla/5.0 (Windows NT 10.0; WOW64; rv:43.0) Gecko/20100101 Firefox/43.0')]

    urllib.request.install_opener(opener)

    req=urllib.request.Request(url)
    response=urllib.request.urlopen(req)
    html=response.read()
    return html

#获取图片地址，返回图片地址的list
def get_imgs(url):
    html=url_open(url).decode('utf-8')

    img_address=[]
    a=html.find('data-original')
    while(a!=-1):
        b=html.find('.jpg',a,a+300)
        if(b!=-1):
            # print(html[a+15:b+4])
            img_address.append(html[a+15:b+4])
        else:
            b=a+9
        a=html.find('data-original=',b)


    for i in img_address:
        print(i)

    return img_address

#存储到本地
def save_imgs(img_address):

    for i in img_address:
        # print(i)
        filename=i.split('/')[-1]
        with open(filename,'wb') as f:
            img=url_open(i)
            f.write(img)


def zhihuPic(url,folder="zhihu"):
    if(os.path.exists(folder)):
        os.chdir(folder)
    else:
        os.mkdir(folder)
        os.chdir(folder)
    img_address=get_imgs(url)
    save_imgs(img_address)





if __name__=='__main__':
    zhihuPic("https://www.zhihu.com/question/22070147")
```
