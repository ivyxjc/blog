---
author: ivyxjc
date: 2016-03-29
title: BeautifulSoup4入门
category: Python
tags: [python,spider,bs4]
keywords:
description: bs4可以非常高效地获取网页中的指定内容. 
---

## 对象的种类
BeautifulSoup会将HTML文档抓换成一个树形结构, 每个节点都是Python对象，所有对象可以分为4类：`Tag`,`NavigableString`,`BeautifulSoup`,`Comment`。


### Tag
`Tag`与XML或HTML中的tag相同：

```python3
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
print(type(tag))
print(tag)

--<class 'bs4.element.Tag'>
--<b class="boldest">Extremely bold</b>
```

#### name

每一个Tag都有自己的name，可以通过`.name`。改变tag的name，将会影响通过当前BS对象生成的HTML文档

```python3
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>',"html.parser")
tag = soup.b
print(tag)
print(tag.name)
tag.name="blockquote"
print(tag)

---<b class="boldest">Extremely bold</b>
---b
---<blockquote class="boldest">Extremely bold</blockquote>
```

#### attributes
一个tag可能有多个属性

```python3
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>',"html.parser")
tag = soup.b
print(tag['class'])
print(tag.attrs)

---['boldest']
---{'class': ['boldest']}
```

tag的属性可以被添加，删除或修改

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>',"html.parser")
tag = soup.b
#更改属性
tag['class']="verybold"
tag['id']=1
print(tag)

#删除属性
del tag['class']
print(tag)
print(tag.get('class'))

---<b class="verybold" id="1">Extremely bold</b>
---<b id="1">Extremely bold</b>
---
```

##### 多值属性
多值属性即为可包含多个值的属性，最常见的即为`class`
`xml`中没有多值属性

多值属性的返回类型是list

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>',"html.parser")
print(css_soup.p['class'])

---['body', 'strikeout']
```

若某个属性看起来像多值属性，但是在任何版本HTML中都没有被定义为多值属性，那么BS会将这个属性作为字符串返回

```python
id_soup = BeautifulSoup('<p id="my id"></p>',"html.parser")
print(id_soup.p['id'])

---my id
```

### NavigableString

`NavigableString`不支持`.content`,`.string`,`find()`方法。

如果想在Beautiful Soup之外使用`NavigableString`对象,需要调用`unicode()`方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束,该对象的输出也会带有对象的引用地址.这样会浪费内存.

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>',"html.parser")
tag = soup.b
print(tag.string)
print(type(tag.string))
tag.string.replace_with("No long bold")
print(tag)

---Extremely bold
---<class 'bs4.element.NavigableString'>
---<b class="boldest">No long bold</b>
```

## BeautifulSoup

`BeautifulSoup`对象表示的是一个文档的全部内容.大部分时候,可以把它当作 Tag 对象,它支持 遍历文档树 和 搜索文档树 中描述的大部分的方法.

`BeautifuSoup`没有name和attribute属性。但有一个值为`[document]`的特殊属性`.name`

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>',"html.parser")
print(soup.name)

---[document]
```

### Comment

## 遍历文档树

```
html_doc = """
<html><head><title>The Dormouse's story</title></head>

<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
```

### `.content`和`.children`
tag的`.contents`属性可以将tag的子节点以列表的方式输出:
通过tag的`.children`生成器,可以对tag的子节点进行循环:

```python
print(soup.title.contents)
print(soup.title.contents[0])

---["The Dormouse's story"]
---The Dormouse's story
```

```python
for i in soup.head.children:
    print(i)

---<title>The Dormouse's story</title>
```

### `.descendants`

`.descendants`属性可以对所有tag的子孙节点进行递归循环

```python
print(soup.head.contents)
for child in soup.head.descendants:
    print(child)

print(len(list(soup.children)))
print(len(list(soup.descendants)))

---[<title>The Dormouse's story</title>]
---<title>The Dormouse's story</title>
---The Dormouse's story
---2
---25
```

### `.string`

如果tag只有一个`NavigableString`类型子节点,那么这个tag可以使用`.string`得到子节点

```python
print(soup.head.string)

---The Dormouse's story
```

### `.strings`和`stripped_strings`
如果tag中有多个字符串，可以使用`.strings`循环获取
使用`.stripped_strings`可以去除多余空白内容：

```python
for string in soup.stripped_strings:
    print(repr(string))
```

### `.parent`和`.parents`
通过`.parent`属性来获取某个元素的父节点
通过元素的`.parents`属性可以递归得到元素的所有父辈节点。

```python
for parent in soup.a.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)

---p
---html
---[document]
```

### 兄弟节点

#### `.next_sibling`和`.previous_sibling`

使用`。next_sibling`和`.previous_sibling`属性来查询兄弟节点

实际文档中的tag的 .next_sibling 和 .previous_sibling 属性通常是字符串或空白。因为中间可能会隔着一些字符以及标点。

#### `.next_siblings`和`.previous_siblings`

```python3
for sibling in soup.a.next_siblings:
    print(repr(sibling))

---',\n'
---<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
---' and\n'
---<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
';\nand they lived at the bottom of a well.'
```

### 重现解析过程

#### `.next_element`和`.previous_element`
`next_element`属性指向解析过程中下一个被解析的对象(字符串或tag),结果可能与`.next_sibling`相同,但通常是不一样的.

```python
last_a_tag=soup.find("a",id="link3")
print(last_a_tag)
print(last_a_tag.next_sibling)
print(last_a_tag.next_element)

---<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
---;
---and they lived at the bottom of a well.
---Tillie
```

#### `.next_elements`和`.previous_elements`

## 搜索文档树

### 过滤器

#### 字符串

#### 正则表达式

```python
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
```

#### 列表
如果传入列表参数,Beautiful Soup会将与列表中任一元素匹配的内容返回.

#### True

True可以匹配任何值,但是不会返回字符串节点

#### 方法
如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数,如果这个方法返回True表示当前元素匹配并且被找到,如果不是则反回False。

```python3
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
```


### find_all()

#### name参数

#### keyword参数

`soup.find_all(id='link2')`


#### 按CSS搜索

因为`class`是python保留字，所以用`class_`
例：
`soup.find_all("a",class_="sister")`

`class_`参数同样接受不同类型的过滤器 ,字符串,正则表达式,方法或 True :

tag的`class`属性是 多值属性 .按照CSS类名搜索tag时,可以分别搜索tag中的每个CSS类名:

#### text参数
通过`text`参数可以搜搜文档中的字符串内容.

#### limit参数

`limit`参数限制返回结果的数量

#### recursive参数
调用tag的`find_all()`方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 `recursive=False`.

#### 简写

```
soup.find_all("a")
soup("a")
```

```
soup.title.find_all(text=True)
soup.title(text=True)
```

### find()

只返回一个结果

### `find_parents()`和`find_parent()`

### `find_next_siblings()和`find_next_sibling()`

### `find_all_next()`和`find_next()`

### `find_all_previous()`和`find_previous()`

### CSS选择器
在Tag或BeautifulSoup对象的`.select()`方法中传入字符串参数,即可使用CSS选择器的语法找到tag:
