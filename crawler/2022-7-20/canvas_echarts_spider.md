## 采集canvas图片案例 (仅限学习)


目标是生成这样的图片

![](https://img-blog.csdnimg.cn/3fccbabb7e044351a659d0ae98ea302d.png)



F12 打开开发者工具，发现存在无限debugger
![](https://img-blog.csdnimg.cn/1ccd32674c0d43d9b5cabab29be6adc8.png)


解决方法就是先禁用 debugger， 后续需求调试的时候, 再 条件debug 或者直接替换js,

我使用的是chrome浏览器, 禁用 debugger 如图:

![](https://img-blog.csdnimg.cn/5ebcedad64494289a251e43960dad04c.png)


点击禁用以后F5再次刷新整个页面， 然后打开开发者工具， 打开后是这样的
![](https://img-blog.csdnimg.cn/79965df964d94a9fbe2bdd7fe40ed6d6.png)


发现可以正常使用开发者工具了.



然后我们查看html 源码， html如下:

![](https://img-blog.csdnimg.cn/5d8c6fae4e1e4d8e9396e93e4550086a.png)

我们全局搜索一下 "data-zr-dom-id"

![](https://img-blog.csdnimg.cn/0c375deb7d0043708c470a231feabe65.png)

发现貌似是canvas 结合 echarts 进行画图的。利用搜索引擎搜了一圈，给出的解决方案都是selenium 结合webdriver 做的。

我想试试看能不能拦截数据直接使用echarts画出这个图呢?

于是我仔细查找网络请求,发现了一个可疑的js文件, "tides.js"

![](https://img-blog.csdnimg.cn/d77c8d38a1e04352ae276b6e69162dbc.png)

我们使用翻译软件翻译一下tides 是什么意思?

![](https://img-blog.csdnimg.cn/c94745061b124739a2d21c28764ab91e.png)

所以我们有理由怀疑这个js是一个值得一看的js文件.

![](https://img-blog.csdnimg.cn/a880d1c2925c419c8f731cf9c07ab733.png)


好家伙 jsjiami.v6, 好吧，刚才是值得一看, 这么看来不仅是值得一看, 还是值得一干的js，解铃还须系铃人, 我们找个在线网站进行解密,

使用 [obfuscator解密](https://www.dejs.vip/encry_decry/obfuscator.html) 这个网站进行在线解密，解密以后长这样

![](https://img-blog.csdnimg.cn/7f27ee20bdb847f98cc960793bfc2cb9.png)


我们复制到 文本文件中看一下
![](https://img-blog.csdnimg.cn/18f0ffa557df45ad838f88b199e64eac.png)

好家伙， 可读性好了很多

然后我们写个echarts hello world
利用搜索引擎搜索一下 "在线echarts", 我使用的是这个网站 aHR0cHM6Ly93d3cucnVub29iLmNvbS90cnkvdHJ5LnBocD9maWxlbmFtZT10cnllY2hhcnRzX2ludHJv

不看不知道，一看吓一跳啊

<br>

![](https://img-blog.csdnimg.cn/52c2d67b50174b478eed507df954dee3.png)

这个 echarts hello world 的代码怎么跟目标网站中的代码这么相像啊.目标网站代码如下

![](https://img-blog.csdnimg.cn/936753772dbb4f7da53675a0860d8cd0.png)

这个data好像是 参数传进来的, 也就是说， 我们只要找到哪里调用了这个函数, 并且找到data这个参数, 就可以自己调用echarts 来进行画图了,

我们全局来搜索一下 "showTides"

![](https://img-blog.csdnimg.cn/4056761b79b14fb1a2b77461aa665f38.png)

原来这个data 就是写在 html 的啊, 我们可以用正则拿出来

等等, data 怎么是 "eJwNyckJADAMxMCGFMjGd/+NxS+BJoIUmZTTzgxSIUsUgbJR2Xafrz1jLlG8JjnFGc765TnWH1rXDyY=" 这样的， 而不是我们直接可以阅读的 "55,61,66,74,84,99,117,136,155,168,173,167,149,123,90,57,28,6,-7,-9,-3,10,24,38" 这样的数据呢？

不过这串数据看起来像是 base64编码过的, 我们直接用在线网站解码一下

![](https://img-blog.csdnimg.cn/03484b00bb9f45e686ab538040334b07.png)


乱码了， 貌似行不通!!!!

我们再回过头来看一下刚才tides.js 中有没有对数据做什么处理

![](https://img-blog.csdnimg.cn/5c5fb9f2fad34530bcacc19c549d9afb.png)

哦， 原来是 先 atob一下, 然后再pako.inflate 一下，就是我们要的可以直接查看的可阅读的数据了. 我们找个在线网站 进行   pako.inflate 一下,  使用这个网站在线 pako.inflate 一下

aHR0cHM6Ly93d3cucWl1dGlhbmFpbWVpbGkuY29tL2h0bWwvX3BhZ2UvMjAxOS8xMi9zb3VyY2UvcGFrby9leGFtcGxlLmh0bWw=  

解压结果如下

![](https://img-blog.csdnimg.cn/b5853c16142b4f6fbc3dbda81a7276c1.png)

然后我们将解压后的数据使用 echarts 画个图看看, 还是使用菜鸟教程的 hello world demo, 在原来的基础上改改.  将数据替换成我们解密出来的 "55,61,66,74,84,99,117,136,155,168,173,167,149,123,90,57,28,6,-7,-9,-3,10,24,38"


代码如下
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>第一个 ECharts 实例</title>
    <!-- 引入 echarts.js -->
    <script src="https://cdn.staticfile.org/echarts/4.3.0/echarts.min.js"></script>
</head>
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 600px;height:400px;"></div>
    <script type="text/javascript">
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));
 
        // 指定图表的配置项和数据
        var option = {
            title: {
                text: '第一个 ECharts 实例'
            },
            tooltip: {},
            legend: {
                data:['销量']
            },
            xAxis: {
                data: [
                "00:00",
                "01:00",
                "02:00",
                "03:00",
                "04:00",
                "05:00",
                "06:00",
                "07:00",
                "08:00",
                "09:00",
                "10:00",
                "11:00",
                "12:00",
                "13:00",
                "14:00",
                "15:00",
                "16:00",
                "17:00",
                "18:00",
                "19:00",
                "20:00",
                "21:00",
                "22:00",
                "23:00"
            ]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [55,61,66,74,84,99,117,136,155,168,173,167,149,123,90,57,28,6,-7,-9,-3,10,24,38]
            }]
        };
 
        // 使用刚指定的配置项和数据显示图表。
        myChart.setOption(option);
    </script>
</body>
</html>
```

在线运行以后， 效果图如下:

![](https://img-blog.csdnimg.cn/eec3514226d04c2babaa733418598d53.png)

是不是跟我们目标网站中的有点像了呢

![](https://img-blog.csdnimg.cn/9745d79092f9400b82fb27a2152112a8.png)

好吧, 我承认还是有点差异的，为了尽善尽美, 我们直接把目标网站上的 echats 配置copy过来, 同样在我们的 echarts hello world 网站进行效果测试, 代码如下:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>第一个 ECharts 实例</title>
    <!-- 引入 echarts.js -->
    <script src="https://cdn.staticfile.org/echarts/4.3.0/echarts.min.js"></script>
</head>
<body>
    <!-- 为ECharts准备一个具备大小（宽高）的Dom -->
    <div id="main" style="width: 600px;height:400px;"></div>
    <script type="text/javascript">
        // 基于准备好的dom，初始化echarts实例
        var myChart = echarts.init(document.getElementById('main'));
 
        // 指定图表的配置项和数据
        var option = {
            "title": {
            "subtext": "天文潮数据\uFF0C不包含气象原因造成的影响",
            "left": "center",
            "align": "right",
            "bottom": 10
        },
            "tooltip": {
            "trigger": "axis",
            "formatter": "时间 : {b}<br/>潮高 : {c}cm"
        },
            legend: {
                data:['销量']
            },
            xAxis: {
                data: [
                "00:00",
                "01:00",
                "02:00",
                "03:00",
                "04:00",
                "05:00",
                "06:00",
                "07:00",
                "08:00",
                "09:00",
                "10:00",
                "11:00",
                "12:00",
                "13:00",
                "14:00",
                "15:00",
                "16:00",
                "17:00",
                "18:00",
                "19:00",
                "20:00",
                "21:00",
                "22:00",
                "23:00"
            ]
            },
            "yAxis": {
            "name": "潮高(cm)",
            "type": "value"
        },
            series: [{
                name: '销量',
                "type": "line",
				"lineStyle": { "normal": { "color": "#2196f3" } },
                "areaStyle": { "normal": { "color": "#2196f3" } },
                data: [55,61,66,74,84,99,117,136,155,168,173,167,149,123,90,57,28,6,-7,-9,-3,10,24,38]
            }]
        };
 
        // 使用刚指定的配置项和数据显示图表。
        myChart.setOption(option);
    </script>
</body>
</html>
```

运行以后再来看看效果图, hello world demo

![](https://img-blog.csdnimg.cn/1e5acb93ed7646c7b603950410a3e6ee.png)

![](https://img-blog.csdnimg.cn/89d6d41542c547b1a434e8e5e62f9459.png)

怎么样， 是不是和目标网站 如出一辙了呢

respect !!

aHR0cHM6Ly93d3cuY2hhb3hpYmlhby5uZXQvdGlkZXMvMTM5Lmh0bWw=

仅限学习使用