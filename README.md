# 一、项目简介
- 内容：循环抓取豆瓣影评中所有观众对《陈情令》的评论，存储在文本文档中，并运用可视化库--词云对其进行分析。
- 目标网站：https://movie.douban.com/subject/27195020/comments?start=
- 使用软件：pycharm
- 使用 python3.7 版本
- 涉及的python类库：requests、lxml、wordcloud、numpy、PIL、jieba


# 二、具体思路

```
#！/usr/bin/env python
#-*- coding:utf-8 -*-
```

## 1.安装、导入相应的类库（本机已安装类库）
```
import requests
from lxml import etree #xpath
from wordcloud import WordCloud
import PIL.Image as image  #引入读取图片的工具
import numpy as np
import jieba   # 分词
```

## 2.确定网页，获取请求头，解决反爬机制，并且循环获取所有页面
```
#获取html源代码
def getPage(url):
    headers = {
        "User-Agent":"Mozilla/5.0 (Windows NT 10.0; WOW64)"
                     " AppleWebKit/537.36 (KHTML, like Gecko)"
                     " Chrome/63.0.3239.132 Safari/537.36"
    }
    response = requests.get(url,headers = headers).text
    return response


#循环获得所有页面的url
def all_page(show_number):
    base_url = "https://movie.douban.com/subject/"+show_number+"/comments?start="
    #列表存放所有的网页，共10页
    urllist = []
    for page in range(0,200,20):
        allurl = base_url+str(page)
        urllist.append(allurl)
    return urllist
```

## 3.运用xpath获取短评
```
#解析网页
def parse(show_number):
    #列表存放所有的短评
    all_comment = []
    number = 1
    for url in all_page(show_number):
        #初始化
        html = etree.HTML(getPage(url))
        #短评
        comment = html.xpath('//div[@class="comment"]//p/span/text()')
        all_comment.append(comment)
        print('第'+str(number)+'页解析并保存成功')
        number += 1
    return all_comment
```

## 4.存入txt文档
```
#保存为txt
def save_to_txt(show_name,show_number):
    result = parse(show_number)
    for i in range(len(result)):
        with open(show_name+'评论集.txt','a+',encoding='utf-8') as f:
            f.write(str(result[i])+'\n')  #按行存储每一页的数据
            f.close()           
```

## 5.将文档的短评进行分词
```
#将爬取的文档进行分词
def trans_CN(text):
    word_list = jieba.cut(text)
    #分词后在单独个体之间加上空格
    result = " ".join(word_list)
    return result
```

## 6.制作词云
```
#制作词云
def getWordCloud(show_name):
    path_txt = show_name+"评论集.txt"                #文档
    path_jpg = "1.jpg"                          #词云形状图片
    path_font = "C:\\Windows\\Fonts\\msyh.ttc"  #字体

    text = open(path_txt,encoding='utf-8').read()

    #剔除无关字
    text = text.replace("真的"," ")
    text = text.replace("什么", " ")
    text = text.replace("但是", " ")
    text = text.replace("而且", " ")
    text = text.replace("那么", " ")
    text = text.replace("就是", " ")
    text = text.replace("可以", " ")
    text = text.replace("不是", " ")

    text = trans_CN(text)
    mask = np.array(image.open(path_jpg))  #词云图案
    wordcloud = WordCloud(
        background_color='white',   #词云背景颜色
        mask=mask,
        scale=15,
        max_font_size=80,
        font_path=path_font
    ).generate(text)

    wordcloud.to_file(show_name+'评论词云.jpg')
```    
    
# 三、主函数
- 长安十二时辰：26849758
- 陈情令：27195020

```
show_name="陈情令"
show_number=27195020
if __name__ == '__main__':
    save_to_txt(show_name,show_number)
    print('所有页面保存成功')
    getWordCloud(show_name)
    print('词云制作成功')
```
