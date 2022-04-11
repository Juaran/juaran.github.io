---
title: 当当网电子书获取
date: 2021-04-16 10:53:29
category: project
tags: [反爬虫,toy]
---

## 一、引言

> ​        根据相关法律法规，为了保护作者及版权方的正当权益，当当网电子书目前在PC端不支持下载功能，只能在线阅读。若您订购的电子书支持iPhone、iPad、Android、E-ink设备阅读，请您使用当当读书客户端登陆您的当当账户，之后点击页面左上角图书下载按钮，之后页面中会显示您已购买未下载的图书，请您点击下载即可。

​        [当当云阅读](http://e.dangdang.com/)是当当网旗下的电子书阅读平台，具有丰富的正版图书资源。为了保护读物作者和版权方的正当权益，应最大限度地防止相关电子书在互联网上非法传播。当当云阅读在PC端不支持下载，只能在线阅读；在智能手机、平板上需安装当当云阅读客户端进行在线阅读或缓存。

​        但对大多数用户来说，仅仅能够阅读是不够的，在阅读过程中做批注、摘录等是常有的阅读习惯，但当当云阅读禁止用户复制、粘贴电子书内容，更不能导出到其他阅读器上。如果用户想要快速提取其内容，唯一的解决办法是：截取屏幕存为图片，然后使用OCR工具识别文字，手动校正误识点。——提取量少时这很有效，但如果想要获取章节乃至整本书籍，这就累死傻小子了。

​        本文对当当云阅读PC端对电子书的保护方式展开研究，提出了一种比传统OCR更高效的电子书内容提取方案：无需截屏、OCR识别、手动校正，并保证提取内容的准确度为100%。基本思路是：

1. 利用网络爬虫模拟登录云阅读网站以获取Cookies信息；

2. ”破解“云阅读页面文字的反爬取策略；

3. 解析数据并批量保存文字内容。

   本文提供的内容仅用于学习研究，请勿用于非法用途，否则后果自负。

## 二、分析过程

### 2.1 页面api解析

打开一本电子书，其链接如下：

> http://e.dangdang.com/pc/reader/index.html?id=1901168632

打开浏览器的网络调试工具或其他抓包程序，点击翻页获取到页面载入的[API](http://e.dangdang.com/media/api.go?action=getPcChapterInfo&epubID=1901168632&permanentId=20210416104157326126993695645533813&consumeType=1&platform=3&deviceType=Android&deviceVersion=5.0.0&channelId=70000&platformSource=DDDS-P&fromPaltform=ds_android&deviceSerialNo=html5&clientVersionNo=5.8.4&token=&chapterID=2&pageIndex=1&locationIndex=4&wordSize=2&style=2&autoBuy=0&chapterIndex=)：

> GET http://e.dangdang.com/media/api.go
>
> Query String Parameters:
>
> | **action**      | getPcChapterInfo                    |          |
> | --------------- | ----------------------------------- | -------- |
> | epubID          | 1901168632                          | 电子书id |
> | permanentId     | 20210416104157326126993695645533813 | 商品id   |
> | consumeType     | 1                                   |          |
> | platform        | 3                                   |          |
> | deviceType      | Android                             |          |
> | deviceVersion   | 5.0.0                               |          |
> | channelId       | 70000                               |          |
> | platformSource  | DDDS-P                              |          |
> | fromPaltform    | ds_android                          |          |
> | deviceSerialNo  | html5                               |          |
> | clientVersionNo | 5.8.4                               |          |
> | token           |                                     |          |
> | chapterID       | 2                                   | 章节     |
> | pageIndex       | 1                                   | 章节页码 |
> | locationIndex   | 4                                   | 全局页码 |
> | wordSize        | 2                                   |          |
> | style           | 2                                   |          |
> | autoBuy         | 0                                   |          |
> | chapterIndex    |                                     |          |

其中关键参数有：chapterID、pageIndex、locationIndex，在批量爬取时按照章节、页码顺序遍历。

API返回内容：

``` json
{
    "data": {
        "chapter": {},
        "chapterInfo": {
            "status": "ok",
            "code": 0,
            "pageType": 0,
            "snippet": "......"
        },
        "systemDate": "1618485378339",
        "currentDate": "2021-04-15 19:16:18",
        "wordSize": 2
    },
    "systemDate": 1618485378339,
    "status": {
        "code": 0
    }
}
```

`snippet`字段内容是HTML标签，这些标签经过浏览器渲染构成了页面文字，内容截取如下：

``` html
<div class="wraper" style="width: 660px; height: 880px; position: absolute; left:0px; top:0px; overflow: hidden; ">
    <div class="text" style="width: 660px; height: 880px; position: absolute; left:0px; top:0px;">
        <div>
            <style type="text/css">
                .fs-86b73afd-2f0 {
                    font-size: 19px;
                    font-family: 'Microsoft Yahei';
                    color: #000000;
                    position: absolute;
                }
                .fs-86b73afd-302 {
                    font-size: 19px;
                    font-family: Arial;
                    color: #000000;
                    position: absolute;
                }
            </style>
            <span class="fs-86b73afd-2f0" style="left:577px; bottom:591px; ">，</span>
            <span class="fs-86b73afd-2f0" style="left:140px; bottom:790px; ">到</span>
            <span class="fs-86b73afd-302" style="left:242px; bottom:392px; ">t</span>
            <span class="fs-86b73afd-2f0" style="left:539px; bottom:621px; ">具</span>
            <span class="fs-86b73afd-2f0" style="left:121px; bottom:760px; ">逻</span>
            <span class="fs-86b73afd-2f0" style="left:121px; bottom:469px; ">位</span>
            <span class="fs-86b73afd-2f0" style="left:141px; bottom:561px; ">界</span>
            <span class="fs-86b73afd-2f0" style="left:197px; bottom:760px; ">网</span>
            <span class="fs-86b73afd-2f0" style="left:500px; bottom:530px; ">络</span>
            <span class="fs-86b73afd-302" style="left:155px; bottom:392px; ">o</span>
            ...
        </div>
    </div>
</div>
```

### 2.2 文字排序

通过浏览器元素定位发现，每一个`<span class="fs-86b73afd-302" style="left:155px; bottom:392px; ">到</span>`标签对应页面上的一个文字，并通过样式`style="left:155px; bottom:392px; "`进行定位，这意味着`<span>`标签并非按照在HTML文档中的顺序排列，而是在浏览器中渲染后得到正确的显示顺序。效果如下图：

<img src="https://cdn.jsdelivr.net/gh/juaran/juaran.github.io@image/typora/image-20210416113326354.png" alt="image-20210416113326354" style="zoom:50%;" />

利用爬虫虽然能够直接解析API响应取到所有文字，但这是乱序的，必须利用定位信息`left`和`bottom`重新排列文字，否则所得结果毫无意义。

文字排序算法伪代码如下：

``` c
Array col_list    // 列定位数组，存储bottom.px信息
for span in <span>:        // 遍历span标签
    if span.bottom.px not in col_list:
        col_list.add(bottom.px)        // 添加列信息
    end if
end for
 
Array order_col_text    // 全部行文字
for col.px in col_list:    // 遍历列
    Array row_text        // 行文本数组，存储同一行的文字
    Array row_list    // 行定位数组，存储left.px信息
    for span in <span>:        // 遍历span标签
        if span.bottom.px == col.px:    // 获取同一行的span标签
            row_text.add(span.text)        // 添加文字到当前行
            row_list.add(span.bottom.px)    // 添加文字定位
        end if
    end for
    // 此处得到同一行的所有文字，但在同一行内仍是乱序
    // 对row_list进行排序，得到正确顺序后再对row_text进行排序
    Array order        // 对行定位排序后的索引列表
    for i in sorted(row_list):    // 遍历排序后的定位
        order.add(row_list.indexOf(i))    // 添加排序后定位所在索引
    end for
    // 按照正确顺序进行文字组合
    Array order_row_text    // 排序后的文字数组
    for i in row_list.length:   
        order_row_text[i] = row_text[order[i]]    // 依序组合文字
    end for
 
       order_col_text.add(order_row_text)
```

示例：

> row_text： ['3', '第', '1', '章']
>row_list ： [85, 45, 71, 99]
> order ： [1, 2, 0, 3]
>order_row_text：第13章

解释：

['3', '第', '1', '章'] 是根据<span>标签顺序获取的文字内容，[85, 45, 71, 99]是对应文字的left定位属性，'3'距离左侧85px，'第'距离左侧45px，'1'距离左侧71像素，'章'距离左侧99px，得到正确顺序应为：[1, 2, 0, 3]，即第0个文字取原文字数组的第1个，第1个文字取原数组的第2个，第2个文字取原第0个，第3个文字取原第3个，组合得到：”第十三章“。

## 三、程序

``` python
import requests
import json
from lxml import etree
 
# 自行替换Cookie信息
headers = {
    'Cookie': '',
    'Connection': 'keep-alive',
    'User-Agent':'Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36'
}
 
def get_page(chapterID=18, pageIndex=0, locationIndex=315):
    url = ""    # API，章节、页码id
 
    data = requests.get(url, headers=headers).json()
    content = data['data']['chapterInfo']
    eles = json.loads(content)
    html = eles['snippet']        # 解析得到内嵌html
 
    tree = etree.HTML(html)
    spans = tree.xpath('/html/body/div/div/div/span/@style')
 
    # 按照算法伪代码的实现：
    col_nums = []    # 列
    for span in spans:
        col_num = int( span.split(';', 2)[1].strip().split('bottom:')[1].split('px')[0] )
        if col_num not in col_nums:
            col_nums.append(col_num)
    # len(col_nums)
 
    page_text = '\n'   
    for bottom in sorted(col_nums, reverse=True):
        row_texts = tree.xpath('/html/body/div/div/div/span[contains(@style,"bottom:' + str(bottom) + 'px")]/text()')
        # print(row_texts)
        row_nums = []
        for location in tree.xpath('/html/body/div/div/div/span[contains(@style,"bottom:' + str(bottom) + 'px")]/@style'):
            row_num =  int(location.split(';', 1)[0].split('left:')[1].split('px')[0])
            row_nums.append(row_num)
        # print(row_nums)
        paragraph = False    # 是否为段落
 
        # new_row_nums = []
        if min(row_nums) != 45:
            paragraph = True
        else:
            paragraph = False
 
        row_results = ['']*len(row_texts)
        order = []
        for i in sorted(row_nums):
            order.append(row_nums.index(i))
 
        # print(order)
        for i in range(len(order)):
            row_results[i] = row_texts[order[i]]
 
        if paragraph is True:
            row_results.insert(0, '\n    ')
 
        # print(''.join(row_results))
        # return ''.join(row_results)
 
        page_text = page_text + ''.join(row_results)
 
    # 保存结果
    print(page_text)
    with open(r'D:\Downloads\当当\results.txt', 'a') as f:
        f.write(page_text)
 
# page_text
for i in range(0, 35):
    # get_page(18, 0, 315)
    get_page(16, i, 252+i)
```

<img src="https://cdn.jsdelivr.net/gh/juaran/juaran.github.io@image/typora/image-20210416143823445.png" alt="image-20210416143823445" style="zoom:50%;" />

## 四、结语

​        当当云阅读的此种反爬虫措施对大多数用户能起到约束作用，但对普通用户而言，失去了做笔记、复制文字内容等阅读的基本习惯，私以为是得不偿失的。尽管在很大程度上限制了电子书籍的非法传播，却失去了良好的用户体验，特别是对付费用户来说，他们购买书籍后理应享有对其内容的所有权。在互联网商业化、国内版权意识不断得到重视的现如今，这种割舍用户体验以换取权益损失最小化的做法，是互联网公司不得不做出的选择。希望在科技更迭的浪潮中，技术越来越服务于人类，法律法规愈加完善，生活越来越美好！