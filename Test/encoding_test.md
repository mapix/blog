Title: WebPage Encoding Test
Date: 2013-04-14 23:50
Tags: Test, Encoding, Python
Category: Test 
Slug: webpage_encoding_test
Author: qingchen
Lang: en

很多时候我们从搜索引擎跳转到一个新的站点的时候站点会作出一些站内引导，引导的方式其中有一类就是抓取搜索引擎的关键词，然后根据关键词引导，这个时候在跨站操作的过程中就出现了编码问题，以至于一些时候我们看到的引导词是乱码的，在这种情况下，网站要想更好的引导用户，就需要尽最大可能的处理好encoding的问题。而从web传递进来的字符本身是没有任何编码信息的，真实的编码信息由于OS和WebBrowser的不同而不同，这个时候其实就需要有个选择的过程。

首先，需要明确自己网站的编码是什么，现如今大部分是UTF-8，当然不乏一些大网站还是用的GBK，下面使用Python演示一下自动处理这个问题的一些代码片段, 应该根据自己站点用户使用浏览器，OS的情况稍微做一些调查，然后列出一个可用的列表，原则上，排第一个的应该是站点自己的编码，其次是多字节的编码，然后是单字节编码，其中类似Latin1这种编码，占据全部8位的空间，所以需要放在最后一位，因为对它来说，什么都是编码正确的。

    :::Python
    CHARSETS = (
      'utf-8',                  # Unicode多语言
      'gbk', 'gb18030',         # 简体中文 
      'big5',                   # 繁体中文， 台湾香港澳门
      'EUC_KR',                 # 韩语
      'EUC_JP', 'Shift_JIS',    # 日语
      'KOI8-R', 'Windows-1251', # 俄语
      'iso-8859-5',             # 斯拉夫语系(保加利亚语,Byelorussian语,马其顿语,俄语,塞尔维亚语,乌克兰语等)
      'iso-8859-2',             # 中欧语系
      'iso-8859-1',             # 西欧语系(荷兰语,英语,法语,德语,意大利语,挪威语,葡萄牙语,瑞士语.等十八种语言), Latin1 
    )

处理的逻辑很简单，就是遍历之前的这个决策列表，除非有解码异常，否则就认为识别编码。

    :::Python
    def detect_charset(s):
        '''事实上，一个rawstr 过来是无法确定它是什么东西的， It all depends'''
        if not isinstance(s, unicode):
            for charset in CHARSETS:
                try:  
                    return unicode(s, charset)
                except:
                    pass
            return ""
        return s

这里你可能想到了chardet等专门用于探测字符编码的库来完成，但是这些基于规则的探测库只有在大量输入的情况下才能形成统计结论，而对于这种一个两个搜索词的缺毫无办法。函数完成了，就需要看到对其进行一个测试了，这里使用Python的标准库模拟从百度过来一个搜索:
    
    :::Python
    import urllib2
    referer = "http://www.baidu.com/s?wd=Hello&rsv_spt=1&issp=1&rsv_bp=0&ie=utf-8&tn=baiduhome_pg&rsv_sug3=3"
    req = urllib2.Request(<origin_url>)
    req.add_header('Referer', referer)
    r = urllib2.urlopen(req)
    return r.read()

这样通过这个web地址就可以对页面识别搜索引擎关键词做出模拟，而在这里的wd参数也可替换成自己需要的编码，例如：

    :::Python
    In [1]: s = u'你好'
    In [2]: s.encode('gbk')
    Out[2]: '\xc4\xe3\xba\xc3'
    In [3]: s.encode('utf-8')
    Out[3]: '\xe4\xbd\xa0\xe5\xa5\xbd'

需要测试其他编码的时候只需要将上面得到的编码后的字符替换上面的Hello就可以了。

还有一个手动测试编码的好方法，以豆瓣小组匿名引导为例，步骤如下:

 1. Firefox 匿名状态下打开豆瓣搜索页面，注意在豆瓣搜索页面有个搜索的Form;
 2. 更改这个页面的编码为你需要测试的编码，这个时候Form要处理的编码会随着你的设定而走;
 3. 在乱码的网页那个搜索Form里边输入需要测试的搜索词`你好`, 点击提交; 
 4. 这个时候结果页是编码正常的，也能顺利搜索到内容，但搜索框里边的搜索词与你搜索的`你好`又相去甚远，不过不用沮丧，因为你已经得到了一个你设定编码的reffer了;
 5. 在这个页面点击搜索出来的小组，小组会弹出匿名引导框，这个时候再看匿名引导框上的文字`你好`又好了;

这个过程其实就是应用到了之前说过的那个多级解码的函数了， 第4步的时候其实我们是获取了一个可用的需要测试编码内容的reffer了，再进入下一个页面的时候，会自带reffer信息，就被这个函数成功处理了。
