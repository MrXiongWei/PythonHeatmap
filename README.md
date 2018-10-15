# PythonHeatmap
python调用百度地图API实现经纬度换算、热力地图全流程指南
基于地图的数据可视化应用愈来愈广泛，目前，有很多方法来实现地图可视化，包括excel的power map包、各种数据分析软件的地图库以及在线交互地图可视化操作工具，如Echarts、Tableau Public、polyMaps等等。另外还有一种手段就是通过软件调用百度、google或者其他地图的api，自己DIY可视化地图，但是这种办法需要操作者本身既要对相关软件的编程熟悉，又要熟悉不同地图api的具体用法。本文就是用这种手段，以一个简单的表格文件出发，在不知道相关地点经纬度的情况下，通过python调用百度地图API实现热力地图，这其中需要申请密钥、批量经纬度换算、转换成js数据、百度热力地图api相关参数的调整等等。

（1）初始数据：csv格式的数据表格

初始数据为2017年1月70个大中城市新建住宅价格指数同比值，直接从国家统计局网站公布的数据copy过来（下图），数据已整理好，为两列（城市city、房价指数price），并保存为csv格式。在实际中，我们常常通过爬取网站上万条地区数据并存为csv格式来分析，在这里为简化流程，初始数据来源直接copy已有数据。
参考house_price.csv

（2）城市转换成经纬度第一步：注册密钥
在百度地图api上相关位置的展现是以经纬度为基础的（这里暂不介绍百度地图坐标体系与其他地图的区别），如北京，其经度（longitude）为：116.395645，纬度（latitude）为：39.929986，在这里既需要通过百度的Geocoding API来获取不同城市的经纬度坐标，又要求将csv数据文件导入python，批量获取这70个城市的坐标信息。在做这些之前，需要注册百度地图api（首先你要用百度的账号）以获取免费的密钥，才能完全使用该api。登录网址：http://lbsyun.baidu.com/
首页点击申请密钥按钮，经过填写个人信息、邮箱注册等，成功之后在开放平台上点击“创建应用”，填写相关信息，在这里特别说明的是，在IP白名单框里，如果不清楚自己的IP地址，最好设置为：0.0.0.0/0，虽然百度提醒它会有泄露使用的风险，但是有时候你把你自己的IP地址输进去可能也不行。提交后，在你创建应用的访问应用（AK）那一栏就是你的密钥。
（3）城市转换成经纬度第二步：构造经纬度获取函数
注册密钥后就可以在百度Web服务API下的Geocoding API接口来获取你所需要地址的经纬度坐标并转化为json结构的数据，其网址为：
http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-geocoding
网页中有相关说明，根据示例URL，采用python3软件，写出如下函数：
import json
from urllib.request import urlopen, quote
import requests,csv
import pandas as pd #导入这些库后边都要用到

def getlnglat(address):
    url = 'http://api.map.baidu.com/geocoder/v2/'
    output = 'json'
    ak = '你申请的密钥***'
    add = quote(address) #由于本文城市变量为中文，为防止乱码，先用quote进行编码
    uri = url + '?' + 'address=' + add  + '&output=' + output + '&ak=' + ak
    req = urlopen(uri)
    res = req.read().decode() #将其他编码的字符串解码成unicode
    temp = json.loads(res) #对json数据进行解析
    return temp 
（4）城市转换成经纬度第三步：批量获取城市经纬度坐标
在构造完获取坐标函数后，我们就需要用python读取csv文件的数据，并将city列单独读出来，批量获取经度、纬度坐标，并生成json的数据文件，其代码如下：

file = open(r'E:\\爬虫数据分析\调用百度地图api\point.json','w') #建立json数据文件
with open(r'E:\\爬虫数据分析\调用百度地图api\各区域房价.csv', 'r') as csvfile: #打开csv
    reader = csv.reader(csvfile)
    for line in reader: #读取csv里的数据
        # 忽略第一行
        if reader.line_num == 1: #由于第一行为变量名称，故忽略掉
            continue
            # line是个list，取得所有需要的值
        b = line[0].strip() #将第一列city读取出来并清除不需要字符
        c= line[1].strip()#将第二列price读取出来并清除不需要字符
        lng = getlnglat(b)['result']['location']['lng'] #采用构造的函数来获取经度
        lat = getlnglat(b)['result']['location']['lat'] #获取纬度
        str_temp = '{"lat":' + str(lat) + ',"lng":' + str(lng) + ',"count":' + str(c) +'},'
        #print(str_temp) #也可以通过打印出来，把数据copy到百度热力地图api的相应位置上
        file.write(str_temp) #写入文档
file.close() #保存
在这里特别要注意str_temp = '{"lat":' + str(lat) + ',"lng":' + str(lng) + ',"count":' + str(c) +'},'，这一行的命令，这是参照百度地图JavaScript API热力图制作的相应格式而生成的，生成的json数据格式为：{"lat":39.92998577808024,"lng":116.39564503787867,"count":124.7},如下图所示，来自于网址：http://developer.baidu.com/map/jsdemo.htm#c1_15。
Paste_Image.png

（5）生成热力地图
接下来就比较简单，我们先建立一个html文件，将http://lbsyun.baidu.com/jsdemo.htm#c1_15
网址中源代码复制过来，首先将代码中的ak换成你自己的密钥；

然后将生成的point.json文件里的数据复制出来，在替换掉var points =[ ]里的内容，即可。这里要注意的是，由于百度地图JavaScript API热力图默认的是以天安门为中心的北京区域地图，而我们的数据是全国性的，所以这里还需要对热力图中“设置中心点坐标和地图级别”的部分进行修改（见下图），具体设置可以参考百度创建地图api中：
http://api.map.baidu.com/lbsapi/creatmap/
自己可以去调试出合适的中心点与地图级别。

最后，由于我们的大部分price数据（也就是points里的count）都超过了100（默认最大为100），还需要对热点图代码中的点最大值进行设定（这里设为140）。

保存后，用浏览器打开，即得到了2017年1月70个大中城市新建住宅价格指数同比的热力地图

图形可以看出，2017年1月房价上涨的热点地区主要是合肥、南京、杭州一带，福州、厦门一带以及广州一带。
