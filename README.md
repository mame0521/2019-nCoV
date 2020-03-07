# 2019-nCoV  
2019新型冠状病毒疫情信息的爬取与可视化- Crawling and Visualization about Daily Information of 2019 novel Coronavirus Epidemic in China Mainland
数据采集网站是丁香园, The data source is Ding Xiang Yuan, https://ncov.dxy.cn/ncovh5/view/pneumonia

```python
# -*- coding: utf-8 -*-
"""
Created on Wed Feb  2 16:39:51 2020
"""
import requests,os
import re
import xlwt
import time
import json

class get_yq_info:

    def get_data_html(self):
            headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.1 (KHTML, like Gecko) Chrome/14.0.835.163 Safari/535.1'}
            response = requests.get('https://ncov.dxy.cn/ncovh5/view/pneumonia?from=timeline&isappinstalled=0', headers=headers, timeout=3)
            # 请求页面

            response = str(response.content, 'utf-8')
            # 中文重新编码
            return response
            #返回了HTML数据

    def get_data_dictype(self):
            areas_type_dic_raw = re.findall('try { window.getAreaStat = (.*?)}catch\(e\)',self.get_data_html())
            areas_type_dic = json.loads(areas_type_dic_raw[0])
            return areas_type_dic
            #返回经过json转换过的字典化的数据

    def save_data_to_excle(self):
            self.make_dir()
            #调用方法检查数据目录是否存在，不存在则创建数据文件夹
            count = 2
            #数据写入行数记录
            newworkbook = xlwt.Workbook()
            worksheet = newworkbook.add_sheet('all_data')
            # 打开工作簿，创建工作表

            worksheet.write(1, 2, '省份名称')
            worksheet.write(1, 3, '省份简称或城市名称')
            worksheet.write(1, 4, '确诊人数')
            worksheet.write(1, 5, '疑似人数')
            worksheet.write(1, 6, '治愈人数')
            worksheet.write(1, 7, '死亡人数')
            worksheet.write(1, 8, '地区ID编码')
            #写入数据列标签

            for province_data in self.get_data_dictype():
                    provincename = province_data['provinceName']
                    provinceshortName = province_data['provinceShortName']
                    p_confirmedcount = province_data['confirmedCount']
                    p_suspectedcount = province_data['suspectedCount']
                    p_curedcount = province_data['curedCount']
                    p_deadcount = province_data['deadCount']
                    p_locationid = province_data['locationId']
                    #用循环获取省级以及该省以下城市的数据

                    worksheet.write(count, 2, provincename)
                    worksheet.write(count, 3, provinceshortName)
                    worksheet.write(count, 4, p_confirmedcount)
                    worksheet.write(count, 5, p_suspectedcount)
                    worksheet.write(count, 6, p_curedcount)
                    worksheet.write(count, 7, p_deadcount)
                    worksheet.write(count, 8, p_locationid)
                    #在工作表里写入省级数据

                    count += 1
                    #此处为写入行数累加，province部分循环

                    for citiy_data in province_data['cities']:
                            cityname = citiy_data['cityName']
                            c_confirmedcount = citiy_data['confirmedCount']
                            c_suspectedcount = citiy_data['suspectedCount']
                            c_curedcount = citiy_data['curedCount']
                            c_deadcount = citiy_data['deadCount']
                            c_locationid = citiy_data['locationId']
                            #该部分获取某个省下某城市的数据
                            print(cityname)
                            print(c_confirmedcount)
                            worksheet.write(count, 3, cityname)
                            worksheet.write(count, 4, c_confirmedcount)
                            worksheet.write(count, 5, c_suspectedcount)
                            worksheet.write(count, 6, c_curedcount)
                            worksheet.write(count, 7, c_deadcount)
                            worksheet.write(count, 8, c_locationid)
                            #该部分在工作表里写入某城市的数据

                            count += 1
                            #此处为写入行数累加，cities部分循环
            current_time = time.strftime("%Y-%m-%d-%H-%M-%S", time.localtime())
            newworkbook.save('F:\2019_nCoV_crawling_info\2019_nCoV_crawling_info-%s.xls' % (current_time))
            print('======数据爬取成功======')

    def make_dir(self):
            file_path = 'F:/2019_nCoV_crawling_info/'
            if not os.path.exists(file_path):
                    os.makedirs(file_path)
                    print('======数据文件夹不存在=======')
                    print('======数据文件夹创建成功======')
                    print('======创建目录为%s======'%(file_path))
            else:
                    print('======数据保存在目录：%s======' % (file_path))
            #检查并创建数据目录


get_yq_info().save_data_to_excle()


fig = matplotlib.figure.Figure()
fig.set_size_inches(width/100, height/100) # 设置绘图板尺寸
axes = fig.add_axes(rect)
    
# 兰博托投影模式，局部
m = Basemap(projection='lcc', llcrnrlon=77, llcrnrlat=14, urcrnrlon=140, urcrnrlat=51, lat_1=33, lat_2=45, lon_0=100, ax=axes)
    
# 兰博托投影模式，全图
#m = Basemap(projection='lcc', llcrnrlon=80, llcrnrlat=0, urcrnrlon=140, urcrnrlat=51, lat_1=33, lat_2=45, lon_0=100, ax=axes)
    
# 圆柱投影模式，局部
#m = Basemap(llcrnrlon=lon_min, urcrnrlon=lon_max, llcrnrlat=lat_min, urcrnrlat=lat_max, resolution='l', ax=axes)
    
# 正射投影模式
#m = Basemap(projection='ortho', lat_0=36, lon_0=102, resolution='l', ax=axes)
	
# 全球等经纬投影模式，
#m = Basemap(llcrnrlon=lon_min, urcrnrlon=lon_max, llcrnrlat=lat_min, urcrnrlat=lat_max, resolution='l', ax=axes)
#m.etopo()
    
m.readshapefile('res/china-shapefiles-master/china', 'province', drawbounds=True)
m.readshapefile('res/china-shapefiles-master/china_nine_dotted_line', 'section', drawbounds=True)
m.drawcoastlines(color='black') # 洲际线
m.drawcountries(color='black')  # 国界线
m.drawparallels(np.arange(lat_min,lat_max,10), labels=[1,0,0,0]) #画经度线
m.drawmeridians(np.arange(lon_min,lon_max,10), labels=[0,0,0,1]) #画纬度线
    
pset = set()
for info, shape in zip(m.province_info, m.province):
    pname = info['OWNER'].strip('\x00')
    fcname = info['FCNAME'].strip('\x00')
    if pname != fcname: # 不绘制海岛
      continue
        
        for key in data.keys():
            if key in pname:
                if data[key] == 0:
                    color = '#f0f0f0'
                elif data[key] < 10:
                    color = '#ffaa85'
                elif data[key] <100:
                    color = '#ff7b69'
                elif  data[key] < 1000:
                    color = '#bf2121'
                else:
                    color = '#7f1818'
                break
        
        poly = Polygon(shape, facecolor=color, edgecolor=color)
        axes.add_patch(poly)
        
        pos = provincePos[pname]
        text = pname.replace("自治区", "").replace("特别行政区", "").replace("壮族", "").replace("维吾尔", "").replace("回族", "").replace("省", "").replace("市", "")
        if text not in pset:
            x,  y = m(pos[0], pos[1])
            axes.text(x,  y, text, fontproperties=font_11, color='#00FFFF')
            pset.add(text)
    
axes.legend(handles, labels, bbox_to_anchor=(0.5, -0.11), loc='lower center', ncol=4, prop=font_14)
axes.set_title("2019-nCoV Epidemic Map 疫情地图", fontproperties=font_14)
FigureCanvasAgg(fig)
fig.savefig('2019-nCoV疫情地图.png')

```
# A sample of output was shown as followings:
 Mar 08, 2020

![](img/2019-nCoV_trend.png)
![](img/2019-nCoV_map1.png)
![](img/2019-nCoV-map2.png)



| 省份名称 Province Name | 省份或城市名称 Provice/City Name | 确诊人数 Confirmed Count | 疑似人数 Suspected Count | 治愈人数 Cured Count | 死亡人数 Dead Count | 地区ID编码 Location ID |
|------------------------|----------------------------------|--------------------------|--------------------------|----------------------|---------------------|------------------------|
| 湖北省                 | 湖北                             | 64287                    | 0                        | 16748                | 2495                | 420000                 |
|                        | 武汉                             | 46607                    | 0                        | 8950                 | 1987                | 420100                 |
|                        | 孝感                             | 3465                     | 0                        | 1177                 | 108                 | 420900                 |
|                        | 黄冈                             | 2904                     | 0                        | 1659                 | 103                 | 421100                 |
|                        | 荆州                             | 1383                     | 0                        | 470                  | 40                  | 420700                 |
|                        | 随州                             | 1574                     | 0                        | 734                  | 42                  | 421000                 |
|                        | 襄阳                             | 1303                     | 0                        | 551                  | 33                  | 421300                 |
|                        | 鄂州                             | 1174                     | 0                        | 521                  | 29                  | 420600                 |
|                        | 荆门                             | 924                      | 0                        | 304                  | 29                  | 420500                 |
|                        | 黄石                             | 920                      | 0                        | 389                  | 37                  | 420800                 |
|                        | 宜昌                             | 1005                     | 0                        | 491                  | 30                  | 420200                 |
|                        | 十堰                             | 836                      | 0                        | 445                  | 11                  | 421200                 |
|                        | 咸宁                             | 669                      | 0                        | 293                  | 2                   | 420300                 |
|                        | 仙桃                             | 573                      | 0                        | 268                  | 19                  | 429004                 |
|                        | 天门                             | 494                      | 0                        | 258                  | 13                  | 429006                 |
|                        | 恩施州                           | 194                      | 0                        | 82                   | 8                   | 429005                 |
|                        | 潜江                             | 251                      | 0                        | 146                  | 4                   | 422800                 |
|                        | 神农架林区                       | 11                       | 0                        | 10                   | 0                   | 429021                 |
| 广东省                 | 广东                             | 1345                     | 0                        | 786                  | 6                   | 440000                 |
|                        | 深圳                             | 417                      | 0                        | 249                  | 3                   | 440300                 |
|                        | 广州                             | 345                      | 0                        | 204                  | 0                   | 440100                 |
|                        | 佛山                             | 96                       | 0                        | 36                   | 1                   | 441900                 |
|                        | 东莞                             | 84                       | 0                        | 36                   | 0                   | 440600                 |
|                        | 珠海                             | 98                       | 0                        | 54                   | 1                   | 440400                 |
|                        | 中山                             | 62                       | 0                        | 46                   | 0                   | 441300                 |
|                        | 惠州                             | 66                       | 0                        | 51                   | 0                   | 442000                 |
|                        | 江门                             | 23                       | 0                        | 9                    | 0                   | 440700                 |
|                        | 汕头                             | 22                       | 0                        | 14                   | 0                   | 440800                 |
|                        | 茂名                             | 19                       | 0                        | 10                   | 1                   | 441200                 |
|                        | 湛江                             | 14                       | 0                        | 7                    | 0                   | 440900                 |
|                        | 肇庆                             | 16                       | 0                        | 11                   | 0                   | 441400                 |
|                        | 阳江                             | 25                       | 0                        | 21                   | 0                   | 440500                 |
|                        | 梅州                             | 14                       | 0                        | 10                   | 0                   | 441700                 |
|                        | 清远                             | 10                       | 0                        | 6                    | 0                   | 440200                 |
|                        | 韶关                             | 8                        | 0                        | 5                    | 0                   | 445200                 |
|                        | 揭阳                             | 5                        | 0                        | 2                    | 0                   | 441500                 |
|                        | 潮州                             | 4                        | 0                        | 1                    | 0                   | 441600                 |
|                        | 汕尾                             | 12                       | 0                        | 10                   | 0                   | 441800                 |
|                        | 河源                             | 5                        | 0                        | 4                    | 0                   | 445100                 |
|                        | 待明确地区                       | 1205                     | 0                        | 782                  | 1                   | 330000                 |
| 河南省                 | 河南                             | 504                      | 0                        | 306                  | 1                   | 330300                 |
|                        | 信阳                             | 146                      | 0                        | 98                   | 0                   | 331000                 |
|                        | 南阳                             | 157                      | 0                        | 114                  | 0                   | 330200                 |
|                        | 驻马店                           | 169                      | 0                        | 131                  | 0                   | 330100                 |
|                        | 郑州                             | 36                       | 0                        | 0                    | 0                   | 0                      |
|                        | 商丘                             | 45                       | 0                        | 28                   | 0                   | 330400                 |
|                        | 周口                             | 55                       | 0                        | 39                   | 0                   | 330700                 |
|                        | 新乡                             | 42                       | 0                        | 30                   | 0                   | 330600                 |
|                        | 安阳                             | 10                       | 0                        | 5                    | 0                   | 330900                 |
|                        | 平顶山                           | 14                       | 0                        | 10                   | 0                   | 330800                 |
|                        | 许昌                             | 17                       | 0                        | 14                   | 0                   | 331100                 |
|                        | 漯河                             | 10                       | 0                        | 7                    | 0                   | 330500                 |
|                        | 洛阳                             | 755                      | 0                        | 343                  | 5                   | 370000                 |
|                        | 焦作                             | 257                      | 0                        | 34                   | 0                   | 370800                 |
|                        | 开封                             | 47                       | 0                        | 20                   | 0                   | 370600                 |
|                        | 鹤壁                             | 47                       | 0                        | 24                   | 0                   | 370100                 |
|                        | 濮阳                             | 44                       | 0                        | 22                   | 0                   | 370700                 |
|                        | 三门峡                           | 60                       | 0                        | 38                   | 1                   | 370200                 |
|                        | 济源                             | 37                       | 0                        | 15                   | 2                   | 371400                 |
|                        | 待明确地区                       | 38                       | 0                        | 25                   | 0                   | 371000                 |
| 浙江省                 | 浙江                             | 38                       | 0                        | 25                   | 0                   | 371500                 |
|                        | 温州                             | 30                       | 0                        | 16                   | 1                   | 370300                 |
|                        | 宁波                             | 35                       | 0                        | 22                   | 1                   | 370900                 |
|                        | 杭州                             | 49                       | 0                        | 44                   | 0                   | 371300                 |
|                        | 台州                             | 24                       | 0                        | 19                   | 0                   | 370400                 |
|                        | 嘉兴                             | 16                       | 0                        | 11                   | 0                   | 371100                 |
|                        | 金华                             | 15                       | 0                        | 11                   | 0                   | 371600                 |
|                        | 绍兴                             | 18                       | 0                        | 17                   | 0                   | 371700                 |
|                        | 衢州                             | 989                      | 0                        | 663                  | 6                   | 340000                 |
|                        | 丽水                             | 174                      | 0                        | 103                  | 1                   | 340100                 |
|                        | 湖州                             | 155                      | 0                        | 104                  | 0                   | 341200                 |
|                        | 舟山                             | 160                      | 0                        | 107                  | 5                   | 340300                 |
| 安徽省                 | 安徽                             | 108                      | 0                        | 81                   | 0                   | 341600                 |
|                        | 蚌埠                             | 69                       | 0                        | 44                   | 0                   | 341500                 |
|                        | 合肥                             | 38                       | 0                        | 19                   | 0                   | 340500                 |
|                        | 阜阳                             | 83                       | 0                        | 69                   | 0                   | 340800                 |
|                        | 亳州                             | 41                       | 0                        | 29                   | 0                   | 341300                 |
|                        | 六安                             | 27                       | 0                        | 15                   | 0                   | 340400                 |
|                        | 安庆                             | 29                       | 0                        | 18                   | 0                   | 340700                 |
|                        | 宿州                             | 27                       | 0                        | 17                   | 0                   | 340600                 |
|                        | 芜湖                             | 33                       | 0                        | 27                   | 0                   | 340200                 |
|                        | 马鞍山                           | 17                       | 0                        | 11                   | 0                   | 341700                 |
|                        | 淮北                             | 13                       | 0                        | 9                    | 0                   | 341100                 |
|                        | 铜陵                             | 9                        | 0                        | 6                    | 0                   | 341000                 |
|                        | 淮南                             | 6                        | 0                        | 4                    | 0                   | 341800                 |
|                        | 池州                             | 1271                     | 0                        | 943                  | 19                  | 410000                 |
|                        | 滁州                             | 274                      | 0                        | 191                  | 2                   | 411500                 |
|                        | 黄山                             | 157                      | 0                        | 111                  | 0                   | 410100                 |
|                        | 宣城                             | 155                      | 0                        | 124                  | 3                   | 411300                 |
| 江西省                 | 江西                             | 76                       | 0                        | 51                   | 0                   | 411600                 |
|                        | 南昌                             | 139                      | 0                        | 121                  | 0                   | 411700                 |
|                        | 上饶                             | 91                       | 0                        | 74                   | 3                   | 411400                 |
|                        | 九江                             | 58                       | 0                        | 43                   | 1                   | 410400                 |
|                        | 新余                             | 39                       | 0                        | 26                   | 0                   | 411000                 |
|                        | 宜春                             | 53                       | 0                        | 41                   | 0                   | 410500                 |
|                        | 赣州                             | 35                       | 0                        | 23                   | 0                   | 411100                 |
|                        | 抚州                             | 31                       | 0                        | 18                   | 1                   | 410300                 |
|                        | 萍乡                             | 26                       | 0                        | 16                   | 0                   | 410200                 |
|                        | 鹰潭                             | 19                       | 0                        | 11                   | 0                   | 410600                 |
|                        | 吉安                             | 32                       | 0                        | 24                   | 1                   | 410800                 |
|                        | 景德镇                           | 17                       | 0                        | 10                   | 0                   | 410900                 |
|                        | 赣江新区                         | 57                       | 0                        | 48                   | 3                   | 410700                 |
| 湖南省                 | 湖南                             | 5                        | 0                        | 3                    | 0                   | 419001                 |
|                        | 长沙                             | 7                        | 0                        | 6                    | 1                   | 411200                 |
|                        | 岳阳                             | 0                        | 0                        | 2                    | 4                   | 0                      |
|                        | 邵阳                             | 934                      | 0                        | 645                  | 1                   | 360000                 |
|                        | 株洲                             | 118                      | 0                        | 57                   | 0                   | 360400                 |
|                        | 常德                             | 229                      | 0                        | 181                  | 0                   | 360100                 |
|                        | 娄底                             | 123                      | 0                        | 90                   | 0                   | 361100                 |
|                        | 益阳                             | 106                      | 0                        | 76                   | 0                   | 360900                 |
|                        | 永州                             | 130                      | 0                        | 103                  | 0                   | 360500                 |
|                        | 郴州                             | 76                       | 0                        | 49                   | 1                   | 360700                 |
|                        | 衡阳                             | 72                       | 0                        | 51                   | 0                   | 361000                 |
|                        | 湘潭                             | 33                       | 0                        | 16                   | 0                   | 360300                 |
|                        | 怀化                             | 18                       | 0                        | 6                    | 0                   | 360600                 |
|                        | 湘西自治州                       | 22                       | 0                        | 13                   | 0                   | 360800                 |
|                        | 张家界                           | 6                        | 0                        | 3                    | 0                   | 360200                 |
| 江苏省                 | 江苏                             | 1                        | 0                        | 0                    | 0                   | 0                      |
|                        | 苏州                             | 1016                     | 0                        | 731                  | 4                   | 430000                 |
|                        | 南京                             | 242                      | 0                        | 143                  | 2                   | 430100                 |
|                        | 徐州                             | 156                      | 0                        | 91                   | 1                   | 430600                 |
|                        | 淮安                             | 80                       | 0                        | 52                   | 0                   | 430200                 |
|                        | 无锡                             | 76                       | 0                        | 55                   | 0                   | 431300                 |
|                        | 连云港                           | 82                       | 0                        | 62                   | 0                   | 430700                 |
|                        | 南通                             | 102                      | 0                        | 88                   | 1                   | 430500                 |
|                        | 常州                             | 48                       | 0                        | 37                   | 0                   | 430400                 |
|                        | 泰州                             | 36                       | 0                        | 25                   | 0                   | 430300                 |
|                        | 盐城                             | 39                       | 0                        | 31                   | 0                   | 431000                 |
|                        | 扬州                             | 59                       | 0                        | 55                   | 0                   | 430900                 |
|                        | 镇江                             | 43                       | 0                        | 41                   | 0                   | 431100                 |
|                        | 宿迁                             | 40                       | 0                        | 38                   | 0                   | 431200                 |
| 山东省                 | 山东                             | 8                        | 0                        | 8                    | 0                   | 433100                 |
|                        | 烟台                             | 5                        | 0                        | 5                    | 0                   | 430800                 |
|                        | 青岛                             | 527                      | 0                        | 276                  | 3                   | 510000                 |
|                        | 济南                             | 143                      | 0                        | 78                   | 3                   | 510100                 |
|                        | 潍坊                             | 69                       | 0                        | 18                   | 0                   | 513300                 |
|                        | 聊城                             | 41                       | 0                        | 15                   | 0                   | 511700                 |
|                        | 临沂                             | 39                       | 0                        | 22                   | 0                   | 511300                 |
|                        | 济宁                             | 24                       | 0                        | 10                   | 0                   | 510500                 |
|                        | 威海                             | 24                       | 0                        | 10                   | 0                   | 511900                 |
|                        | 德州                             | 18                       | 0                        | 7                    | 0                   | 510600                 |
|                        | 泰安                             | 30                       | 0                        | 20                   | 0                   | 511600                 |
|                        | 淄博                             | 22                       | 0                        | 12                   | 0                   | 510700                 |
|                        | 枣庄                             | 22                       | 0                        | 13                   | 0                   | 511000                 |
|                        | 日照                             | 17                       | 0                        | 10                   | 0                   | 510900                 |
|                        | 菏泽                             | 13                       | 0                        | 8                    | 0                   | 513400                 |
|                        | 滨州                             | 12                       | 0                        | 8                    | 0                   | 511500                 |
| 重庆市                 | 重庆                             | 7                        | 0                        | 4                    | 0                   | 511800                 |
|                        | 万州区                           | 9                        | 0                        | 7                    | 0                   | 510300                 |
|                        | 江北区                           | 8                        | 0                        | 6                    | 0                   | 511400                 |
|                        | 綦江区                           | 3                        | 0                        | 2                    | 0                   | 512000                 |
|                        | 渝中区                           | 16                       | 0                        | 16                   | 0                   | 510400                 |
|                        | 合川区                           | 6                        | 0                        | 6                    | 0                   | 510800                 |
|                        | 垫江县                           | 3                        | 0                        | 3                    | 0                   | 511100                 |
|                        | 奉节县                           | 1                        | 0                        | 1                    | 0                   | 513200                 |
|                        | 九龙坡区                         | 480                      | 0                        | 227                  | 12                  | 230000                 |
|                        | 潼南区                           | 198                      | 0                        | 77                   | 3                   | 230100                 |
|                        | 两江新区                         | 52                       | 0                        | 20                   | 3                   | 230500                 |
|                        | 大足区                           | 46                       | 0                        | 20                   | 0                   | 230300                 |
|                        | 南岸区                           | 43                       | 0                        | 24                   | 1                   | 230200                 |
|                        | 忠县                             | 26                       | 0                        | 9                    | 1                   | 230600                 |
|                        | 渝北区                           | 47                       | 0                        | 29                   | 4                   | 231200                 |
|                        | 石柱县                           | 17                       | 0                        | 10                   | 0                   | 230900                 |
|                        | 云阳县                           | 14                       | 0                        | 8                    | 0                   | 231000                 |
|                        | 长寿区                           | 14                       | 0                        | 9                    | 0                   | 231100                 |
|                        | 丰都县                           | 5                        | 0                        | 4                    | 0                   | 230400                 |
|                        | 开州区                           | 1                        | 0                        | 0                    | 0                   | 230700                 |
|                        | 铜梁区                           | 15                       | 0                        | 15                   | 0                   | 230800                 |
|                        | 荣昌区                           | 2                        | 0                        | 2                    | 0                   | 232700                 |
|                        | 沙坪坝区                         | 575                      | 0                        | 335                  | 6                   | 500000                 |
|                        | 巫溪县                           | 118                      | 0                        | 66                   | 4                   | 500101                 |
|                        | 璧山区                           | 28                       | 0                        | 11                   | 0                   | 500105                 |
|                        | 巫山县                           | 23                       | 0                        | 12                   | 0                   | 500110                 |
|                        | 巴南区                           | 22                       | 0                        | 11                   | 0                   | 500115                 |
|                        | 大渡口区                         | 22                       | 0                        | 11                   | 0                   | 500236                 |
|                        | 永川区                           | 18                       | 0                        | 7                    | 0                   | 500152                 |
|                        | 高新区                           | 23                       | 0                        | 13                   | 0                   | 500117                 |
|                        | 江津区                           | 20                       | 0                        | 10                   | 0                   | 500231                 |
|                        | 涪陵区                           | 15                       | 0                        | 5                    | 0                   | 500108                 |
|                        | 梁平区                           | 20                       | 0                        | 11                   | 1                   | 500107                 |
|                        | 彭水县                           | 9                        | 0                        | 1                    | 0                   | 500106                 |
|                        | 武隆区                           | 17                       | 0                        | 10                   | 0                   | 500112                 |
|                        | 酉阳                             | 17                       | 0                        | 10                   | 0                   | \-1                    |
|                        | 万盛经开区                       | 10                       | 0                        | 4                    | 0                   | 500151                 |
|                        | 城口县                           | 10                       | 0                        | 4                    | 0                   | 500230                 |
|                        | 黔江区                           | 14                       | 0                        | 9                    | 0                   | 500111                 |
|                        | 秀山县                           | 14                       | 0                        | 9                    | 0                   | 500238                 |
|                        | 待明确地区                       | 25                       | 0                        | 21                   | 0                   | 500235                 |
| 黑龙江省               | 黑龙江                           | 21                       | 0                        | 16                   | 1                   | 500154                 |
|                        | 哈尔滨                           | 20                       | 0                        | 16                   | 0                   | 500103                 |
|                        | 鸡西                             | 15                       | 0                        | 11                   | 0                   | 500240                 |
|                        | 绥化                             | 10                       | 0                        | 6                    | 0                   | 500237                 |
|                        | 双鸭山                           | 9                        | 0                        | 5                    | 0                   | 500153                 |
|                        | 齐齐哈尔                         | 21                       | 0                        | 18                   | 0                   | 500233                 |
|                        | 七台河                           | 6                        | 0                        | 3                    | 0                   | 500113                 |
|                        | 大庆                             | 5                        | 0                        | 2                    | 0                   | 500102                 |
|                        | 牡丹江                           | 9                        | 0                        | 7                    | 0                   | 500120                 |
|                        | 佳木斯                           | 4                        | 0                        | 2                    | 0                   | 0                      |
|                        | 黑河                             | 2                        | 0                        | 0                    | 0                   | 500243                 |
|                        | 鹤岗                             | 7                        | 0                        | 6                    | 0                   | 500104                 |
|                        | 大兴安岭                         | 5                        | 0                        | 4                    | 0                   | 500118                 |
| 四川省                 | 四川                             | 1                        | 0                        | 0                    | 0                   | 500242                 |
|                        | 成都                             | 1                        | 0                        | 0                    | 0                   | 0                      |
|                        | 甘孜州                           | 4                        | 0                        | 4                    | 0                   | 500116                 |
|                        | 达州                             | 4                        | 0                        | 4                    | 0                   | 500155                 |
|                        | 南充                             | 2                        | 0                        | 2                    | 0                   | 500114                 |
|                        | 广安                             | 2                        | 0                        | 2                    | 0                   | 500229                 |
|                        | 巴中                             | 1                        | 0                        | 1                    | 0                   | 500156                 |
|                        | 内江                             | 1                        | 0                        | 1                    | 0                   | 500241                 |
|                        | 绵阳                             | 399                      | 0                        | 198                  | 4                   | 110000                 |
|                        | 泸州                             | 62                       | 0                        | 0                    | 0                   | 110108                 |
|                        | 德阳                             | 60                       | 0                        | 0                    | 0                   | 110105                 |
|                        | 宜宾                             | 53                       | 0                        | 0                    | 0                   | 110102                 |
|                        | 凉山州                           | 41                       | 0                        | 3                    | 0                   | 110106                 |
|                        | 自贡                             | 39                       | 0                        | 2                    | 0                   | 110115                 |
|                        | 遂宁                             | 29                       | 0                        | 0                    | 0                   | 110114                 |
|                        | 眉山                             | 26                       | 0                        | 2                    | 0                   | \-1                    |
|                        | 雅安                             | 19                       | 0                        | 1                    | 0                   | 110112                 |
|                        | 广元                             | 16                       | 0                        | 3                    | 0                   | 110111                 |
|                        | 攀枝花                           | 14                       | 0                        | 1                    | 0                   | 110107                 |
|                        | 乐山                             | 12                       | 0                        | 1                    | 0                   | 110101                 |
|                        | 资阳                             | 10                       | 0                        | 0                    | 0                   | 110113                 |
|                        | 阿坝州                           | 7                        | 0                        | 0                    | 0                   | 110116                 |
| 北京市                 | 北京                             | 7                        | 0                        | 0                    | 0                   | 110118                 |
|                        | 朝阳区                           | 3                        | 0                        | 2                    | 0                   | 110109                 |
|                        | 海淀区                           | 1                        | 0                        | 0                    | 0                   | 110119                 |
|                        | 西城区                           | 0                        | 0                        | 183                  | 4                   | 0                      |
|                        | 大兴区                           | 631                      | 0                        | 452                  | 0                   | 320000                 |
|                        | 丰台区                           | 93                       | 0                        | 57                   | 0                   | 320100                 |
|                        | 外地来京人员                     | 87                       | 0                        | 57                   | 0                   | 320500                 |
|                        | 昌平区                           | 48                       | 0                        | 24                   | 0                   | 320700                 |
|                        | 通州区                           | 66                       | 0                        | 46                   | 0                   | 320800                 |
|                        | 房山区                           | 55                       | 0                        | 41                   | 0                   | 320200                 |
|                        | 石景山区                         | 79                       | 0                        | 67                   | 0                   | 320300                 |
|                        | 东城区                           | 37                       | 0                        | 26                   | 0                   | 321200                 |
|                        | 顺义区                           | 51                       | 0                        | 41                   | 0                   | 320400                 |
|                        | 怀柔区                           | 40                       | 0                        | 33                   | 0                   | 320600                 |
|                        | 密云区                           | 23                       | 0                        | 17                   | 0                   | 321000                 |
|                        | 门头沟区                         | 13                       | 0                        | 9                    | 0                   | 321300                 |
|                        | 延庆区                           | 27                       | 0                        | 24                   | 0                   | 320900                 |
|                        | 待明确地区                       | 12                       | 0                        | 10                   | 0                   | 321100                 |
| 上海市                 | 上海                             | 251                      | 0                        | 112                  | 2                   | 450000                 |
|                        | 外地来沪人员                     | 44                       | 0                        | 14                   | 1                   | 450500                 |
|                        | 浦东新区                         | 27                       | 0                        | 4                    | 1                   | 451200                 |
|                        | 宝山区                           | 55                       | 0                        | 34                   | 0                   | 450100                 |
|                        | 徐汇区                           | 32                       | 0                        | 17                   | 0                   | 450300                 |
|                        | 闵行区                           | 19                       | 0                        | 6                    | 0                   | 450600                 |
|                        | 静安区                           | 24                       | 0                        | 12                   | 0                   | 450200                 |
|                        | 松江区                           | 11                       | 0                        | 0                    | 0                   | 451300                 |
|                        | 普陀区                           | 11                       | 0                        | 6                    | 0                   | 450900                 |
|                        | 长宁区                           | 8                        | 0                        | 3                    | 0                   | 450800                 |
|                        | 奉贤区                           | 4                        | 0                        | 2                    | 0                   | 451100                 |
|                        | 杨浦区                           | 8                        | 0                        | 7                    | 0                   | 450700                 |
|                        | 嘉定区                           | 3                        | 0                        | 2                    | 0                   | 451000                 |
|                        | 虹口区                           | 5                        | 0                        | 5                    | 0                   | 450400                 |
|                        | 黄浦区                           | 293                      | 0                        | 183                  | 1                   | 350000                 |
|                        | 青浦区                           | 55                       | 0                        | 30                   | 0                   | 350300                 |
|                        | 崇明区                           | 46                       | 0                        | 21                   | 0                   | 350500                 |
|                        | 金山区                           | 71                       | 0                        | 56                   | 1                   | 350100                 |
| 福建省                 | 福建                             | 35                       | 0                        | 23                   | 0                   | 350200                 |
|                        | 福州                             | 20                       | 0                        | 10                   | 0                   | 350700                 |
|                        | 莆田                             | 14                       | 0                        | 5                    | 0                   | 350400                 |
|                        | 泉州                             | 26                       | 0                        | 20                   | 0                   | 350900                 |
|                        | 厦门                             | 20                       | 0                        | 14                   | 0                   | 350600                 |
|                        | 宁德                             | 6                        | 0                        | 4                    | 0                   | 350800                 |
|                        | 南平                             | 335                      | 0                        | 261                  | 3                   | 310000                 |
|                        | 漳州                             | 111                      | 0                        | 86                   | 1                   | \-1                    |
|                        | 三明                             | 60                       | 0                        | 49                   | 0                   | 310115                 |
|                        | 龙岩                             | 21                       | 0                        | 12                   | 0                   | 310113                 |
| 河北省                 | 河北                             | 16                       | 0                        | 12                   | 0                   | 310106                 |
|                        | 唐山                             | 18                       | 0                        | 15                   | 0                   | 310104                 |
|                        | 沧州                             | 14                       | 0                        | 11                   | 0                   | 310117                 |
|                        | 邯郸                             | 9                        | 0                        | 6                    | 0                   | 310114                 |
|                        | 石家庄                           | 19                       | 0                        | 17                   | 0                   | 310112                 |
|                        | 张家口                           | 11                       | 0                        | 9                    | 0                   | 310107                 |
|                        | 保定                             | 9                        | 0                        | 7                    | 0                   | 310110                 |
|                        | 廊坊                             | 9                        | 0                        | 7                    | 0                   | 310120                 |
|                        | 邢台                             | 7                        | 0                        | 5                    | 0                   | 310109                 |
|                        | 秦皇岛                           | 6                        | 0                        | 4                    | 0                   | 310101                 |
|                        | 承德                             | 4                        | 0                        | 2                    | 0                   | 310151                 |
|                        | 衡水                             | 13                       | 0                        | 12                   | 0                   | 310105                 |
| 广西壮族自治区         | 广西                             | 3                        | 0                        | 2                    | 0                   | 310116                 |
|                        | 南宁                             | 5                        | 0                        | 5                    | 0                   | 310118                 |
|                        | 北海                             | 0                        | 0                        | 0                    | 2                   | 0                      |
|                        | 桂林                             | 311                      | 0                        | 234                  | 6                   | 130000                 |
|                        | 河池                             | 58                       | 0                        | 27                   | 1                   | 130200                 |
|                        | 防城港                           | 48                       | 0                        | 31                   | 3                   | 130900                 |
|                        | 柳州                             | 29                       | 0                        | 21                   | 0                   | 130100                 |
|                        | 来宾                             | 34                       | 0                        | 28                   | 0                   | 130700                 |
|                        | 玉林                             | 32                       | 0                        | 27                   | 0                   | 130400                 |
|                        | 钦州                             | 23                       | 0                        | 19                   | 1                   | 130500                 |
|                        | 贵港                             | 32                       | 0                        | 30                   | 0                   | 130600                 |
|                        | 贺州                             | 30                       | 0                        | 28                   | 0                   | 131000                 |
|                        | 百色                             | 10                       | 0                        | 8                    | 1                   | 130300                 |
|                        | 梧州                             | 8                        | 0                        | 8                    | 0                   | 131100                 |
| 陕西省                 | 陕西                             | 7                        | 0                        | 7                    | 0                   | 130800                 |
|                        | 西安                             | 245                      | 0                        | 173                  | 1                   | 610000                 |
|                        | 汉中                             | 120                      | 0                        | 85                   | 1                   | 610100                 |
|                        | 安康                             | 26                       | 0                        | 13                   | 0                   | 610700                 |
|                        | 渭南                             | 26                       | 0                        | 17                   | 0                   | 610900                 |
|                        | 咸阳                             | 15                       | 0                        | 8                    | 0                   | 610500                 |
|                        | 宝鸡                             | 17                       | 0                        | 15                   | 0                   | 610400                 |
|                        | 商洛                             | 8                        | 0                        | 6                    | 0                   | 610200                 |
|                        | 铜川                             | 7                        | 0                        | 5                    | 0                   | 611000                 |
|                        | 延安                             | 13                       | 0                        | 12                   | 0                   | 610300                 |
|                        | 榆林                             | 1                        | 0                        | 0                    | 0                   | 0                      |
|                        | 韩城                             | 8                        | 0                        | 8                    | 0                   | 610600                 |
|                        | 杨凌                             | 3                        | 0                        | 3                    | 0                   | 610800                 |
| 云南省                 | 云南                             | 1                        | 0                        | 1                    | 0                   | 610581                 |
|                        | 昆明                             | 79                       | 0                        | 19                   | 2                   | 810000                 |
|                        | 西双版纳                         | 168                      | 0                        | 116                  | 5                   | 460000                 |
|                        | 昭通                             | 39                       | 0                        | 20                   | 0                   | 460100                 |
|                        | 曲靖                             | 54                       | 0                        | 38                   | 1                   | 460200                 |
|                        | 玉溪                             | 13                       | 0                        | 9                    | 0                   | 469006                 |
|                        | 大理州                           | 7                        | 0                        | 3                    | 0                   | 469026                 |
|                        | 保山                             | 15                       | 0                        | 12                   | 1                   | 460400                 |
|                        | 丽江                             | 9                        | 0                        | 6                    | 1                   | 469023                 |
|                        | 红河州                           | 3                        | 0                        | 1                    | 0                   | 469029                 |
|                        | 普洱                             | 6                        | 0                        | 4                    | 1                   | 469002                 |
|                        | 楚雄州                           | 6                        | 0                        | 5                    | 0                   | 469024                 |
|                        | 德宏州                           | 3                        | 0                        | 2                    | 0                   | 469007                 |
|                        | 文山州                           | 4                        | 0                        | 4                    | 0                   | 469028                 |
|                        | 临沧                             | 3                        | 0                        | 2                    | 1                   | 469021                 |
|                        | 待明确地区                       | 3                        | 0                        | 3                    | 0                   | 469005                 |
| 海南省                 | 海南                             | 2                        | 0                        | 2                    | 0                   | 469027                 |
|                        | 三亚                             | 1                        | 0                        | 1                    | 0                   | 469030                 |
|                        | 海口                             | 0                        | 0                        | 4                    | 0                   | 0                      |
|                        | 儋州                             | 135                      | 0                        | 87                   | 3                   | 120000                 |
|                        | 万宁                             | 60                       | 0                        | 26                   | 2                   | 120115                 |
|                        | 昌江                             | 15                       | 0                        | 10                   | 1                   | 120102                 |
|                        | 澄迈                             | 6                        | 0                        | 2                    | 0                   | 120104                 |
|                        | 琼海                             | 6                        | 0                        | 3                    | 0                   | 120113                 |
|                        | 临高                             | 4                        | 0                        | 1                    | 0                   | 120117                 |
|                        | 陵水                             | 12                       | 0                        | 10                   | 0                   | 120105                 |
|                        | 保亭                             | 6                        | 0                        | 5                    | 0                   | 120101                 |
|                        | 东方                             | 6                        | 0                        | 5                    | 0                   | 0                      |
|                        | 文昌                             | 3                        | 0                        | 2                    | 0                   | 120116                 |
|                        | 乐东                             | 4                        | 0                        | 4                    | 0                   | 120110                 |
|                        | 定安                             | 4                        | 0                        | 4                    | 0                   | 120103                 |
|                        | 琼中                             | 4                        | 0                        | 4                    | 0                   | 120111                 |
| 贵州省                 | 贵州                             | 2                        | 0                        | 2                    | 0                   | 120114                 |
|                        | 贵阳                             | 2                        | 0                        | 2                    | 0                   | 120106                 |
|                        | 遵义                             | 1                        | 0                        | 1                    | 0                   | 120112                 |
|                        | 毕节                             | 0                        | 0                        | 6                    | 0                   | 0                      |
|                        | 黔南州                           | 174                      | 0                        | 128                  | 2                   | 530000                 |
|                        | 黔东南州                         | 53                       | 0                        | 40                   | 0                   | 530100                 |
|                        | 六盘水                           | 25                       | 0                        | 14                   | 0                   | 530600                 |
|                        | 铜仁                             | 9                        | 0                        | 4                    | 0                   | 0                      |
|                        | 安顺                             | 13                       | 0                        | 9                    | 0                   | 530300                 |
|                        | 黔西南州                         | 9                        | 0                        | 5                    | 0                   | 530500                 |
| 辽宁省                 | 辽宁                             | 5                        | 0                        | 2                    | 0                   | 0                      |
|                        | 沈阳                             | 15                       | 0                        | 12                   | 1                   | 532800                 |
|                        | 大连                             | 14                       | 0                        | 12                   | 1                   | 530400                 |
|                        | 锦州                             | 13                       | 0                        | 12                   | 0                   | 0                      |
|                        | 盘锦                             | 7                        | 0                        | 7                    | 0                   | 530700                 |
|                        | 葫芦岛                           | 4                        | 0                        | 4                    | 0                   | 530800                 |
|                        | 阜新                             | 4                        | 0                        | 4                    | 0                   | 0                      |
|                        | 铁岭                             | 2                        | 0                        | 2                    | 0                   | 0                      |
|                        | 朝阳                             | 1                        | 0                        | 1                    | 0                   | 530900                 |
|                        | 丹东                             | 76                       | 0                        | 30                   | 2                   | 650000                 |
|                        | 鞍山                             | 18                       | 0                        | 6                    | 0                   | 654000                 |
|                        | 本溪                             | 23                       | 0                        | 13                   | 0                   | 650100                 |
|                        | 辽阳                             | 10                       | 0                        | 0                    | 0                   | 0                      |
|                        | 营口                             | 4                        | 0                        | 1                    | 1                   | 0                      |
| 天津市                 | 天津                             | 3                        | 0                        | 1                    | 0                   | 650400                 |
|                        | 宝坻区                           | 4                        | 0                        | 1                    | 1                   | 0                      |
|                        | 河东区                           | 3                        | 0                        | 1                    | 0                   | 0                      |
|                        | 河北区                           | 2                        | 0                        | 0                    | 0                   | 0                      |
|                        | 北辰区                           | 4                        | 0                        | 3                    | 0                   | 0                      |
|                        | 南开区                           | 3                        | 0                        | 2                    | 0                   | 511902                 |
|                        | 和平区                           | 1                        | 0                        | 1                    | 0                   | 0                      |
|                        | 外地来津人员                     | 1                        | 0                        | 1                    | 0                   | 652900                 |
|                        | 宁河区                           | 146                      | 0                        | 102                  | 2                   | 520000                 |
|                        | 东丽区                           | 32                       | 0                        | 18                   | 0                   | 520300                 |
|                        | 河西区                           | 36                       | 0                        | 23                   | 1                   | 520100                 |
|                        | 滨海新区                         | 23                       | 0                        | 14                   | 0                   | 520500                 |
|                        | 西青区                           | 17                       | 0                        | 13                   | 0                   | 522700                 |
|                        | 红桥区                           | 10                       | 0                        | 8                    | 1                   | 520200                 |
|                        | 津南区                           | 10                       | 0                        | 9                    | 0                   | 522600                 |
|                        | 武清区                           | 4                        | 0                        | 3                    | 0                   | 520400                 |
|                        | 待明确地区                       | 10                       | 0                        | 10                   | 0                   | 520600                 |
| 山西省                 | 山西                             | 4                        | 0                        | 4                    | 0                   | 522300                 |
|                        | 晋中                             | 132                      | 0                        | 91                   | 0                   | 140000                 |
|                        | 太原                             | 36                       | 0                        | 23                   | 0                   | 140700                 |
|                        | 运城                             | 20                       | 0                        | 14                   | 0                   | 140100                 |
|                        | 大同                             | 10                       | 0                        | 5                    | 0                   | 140500                 |
|                        | 长治                             | 8                        | 0                        | 3                    | 0                   | 140400                 |
|                        | 忻州                             | 12                       | 0                        | 8                    | 0                   | 140200                 |
|                        | 晋城                             | 8                        | 0                        | 5                    | 0                   | 140600                 |
|                        | 朔州                             | 7                        | 0                        | 4                    | 0                   | 140900                 |
|                        | 阳泉                             | 19                       | 0                        | 17                   | 0                   | 140800                 |
|                        | 吕梁                             | 6                        | 0                        | 6                    | 0                   | 141100                 |
|                        | 临汾                             | 4                        | 0                        | 4                    | 0                   | 140300                 |
| 吉林省                 | 吉林                             | 2                        | 0                        | 2                    | 0                   | 141000                 |
|                        | 长春                             | 75                       | 0                        | 34                   | 0                   | 150000                 |
|                        | 四平市                           | 9                        | 0                        | 2                    | 0                   | 152500                 |
|                        | 辽源                             | 11                       | 0                        | 5                    | 0                   | 150200                 |
|                        | 公主岭                           | 9                        | 0                        | 3                    | 0                   | 150400                 |
|                        | 延边                             | 7                        | 0                        | 1                    | 0                   | 150500                 |
|                        | 通化                             | 7                        | 0                        | 2                    | 0                   | 150100                 |
|                        | 吉林市                           | 7                        | 0                        | 4                    | 0                   | 150700                 |
|                        | 松原                             | 11                       | 0                        | 9                    | 0                   | 150600                 |
|                        | 梅河口                           | 8                        | 0                        | 6                    | 0                   | 150800                 |
|                        | 白城                             | 3                        | 0                        | 1                    | 0                   | 150900                 |
| 新疆维吾尔自治区       | 新疆                             | 2                        | 0                        | 1                    | 0                   | 150300                 |
|                        | 乌鲁木齐                         | 1                        | 0                        | 0                    | 0                   | 152200                 |
|                        | 伊犁州                           | 121                      | 0                        | 80                   | 1                   | 210000                 |
|                        | 兵团第四师                       | 19                       | 0                        | 11                   | 0                   | 210200                 |
|                        | 兵团第九师                       | 28                       | 0                        | 22                   | 0                   | 210100                 |
|                        | 昌吉州                           | 12                       | 0                        | 5                    | 1                   | 211400                 |
|                        | 巴州                             | 7                        | 0                        | 3                    | 0                   | 211200                 |
|                        | 兵团第八师石河子市               | 4                        | 0                        | 0                    | 0                   | 210300                 |
|                        | 兵团第六师五家渠市               | 12                       | 0                        | 9                    | 0                   | 210700                 |
|                        | 兵团第十二师                     | 11                       | 0                        | 8                    | 0                   | 211100                 |
|                        | 吐鲁番市                         | 6                        | 0                        | 3                    | 0                   | 211300                 |
|                        | 兵团第七师                       | 8                        | 0                        | 6                    | 0                   | 210900                 |
|                        | 阿克苏地区                       | 7                        | 0                        | 6                    | 0                   | 210600                 |
| 内蒙古自治区           | 内蒙古                           | 3                        | 0                        | 3                    | 0                   | 210500                 |
|                        | 包头                             | 3                        | 0                        | 3                    | 0                   | 211000                 |
|                        | 鄂尔多斯                         | 1                        | 0                        | 1                    | 0                   | 210800                 |
|                        | 呼和浩特                         | 93                       | 0                        | 60                   | 1                   | 220000                 |
|                        | 通辽                             | 45                       | 0                        | 32                   | 0                   | 220100                 |
|                        | 巴彦淖尔                         | 7                        | 0                        | 1                    | 0                   | 220400                 |
|                        | 呼伦贝尔                         | 15                       | 0                        | 9                    | 1                   | 220300                 |
|                        | 赤峰                             | 6                        | 0                        | 2                    | 0                   | 220500                 |
|                        | 乌兰察布                         | 6                        | 0                        | 3                    | 0                   | 220381                 |
|                        | 锡林郭勒盟                       | 1                        | 0                        | 0                    | 0                   | 220800                 |
|                        | 乌海市                           | 5                        | 0                        | 5                    | 0                   | 220200                 |
|                        | 兴安盟                           | 5                        | 0                        | 5                    | 0                   | 222400                 |
| 香港                   | 香港                             | 2                        | 0                        | 2                    | 0                   | 220700                 |
| 甘肃省                 | 甘肃                             | 1                        | 0                        | 1                    | 0                   | 220581                 |
|                        | 兰州                             | 30                       | 0                        | 5                    | 1                   | 710000                 |
|                        | 平凉                             | 71                       | 0                        | 58                   | 0                   | 640000                 |
|                        | 甘南                             | 28                       | 0                        | 20                   | 0                   | 640300                 |
|                        | 天水                             | 33                       | 0                        | 29                   | 0                   | 640100                 |
|                        | 定西                             | 5                        | 0                        | 4                    | 0                   | 640400                 |
|                        | 白银                             | 3                        | 0                        | 3                    | 0                   | 640500                 |
|                        | 陇南                             | 1                        | 0                        | 1                    | 0                   | 0                      |
|                        | 庆阳                             | 1                        | 0                        | 1                    | 0                   | 640200                 |
|                        | 临夏                             | 91                       | 0                        | 80                   | 2                   | 620000                 |
|                        | 张掖                             | 36                       | 0                        | 31                   | 2                   | 620100                 |
|                        | 金昌                             | 9                        | 0                        | 6                    | 0                   | 620800                 |
|                        | 待明确地区                       | 9                        | 0                        | 8                    | 0                   | 621100                 |
| 宁夏回族自治区         | 宁夏                             | 4                        | 0                        | 3                    | 0                   | 620400                 |
|                        | 银川                             | 3                        | 0                        | 2                    | 0                   | 621000                 |
|                        | 吴忠                             | 12                       | 0                        | 12                   | 0                   | 620500                 |
|                        | 固原                             | 8                        | 0                        | 8                    | 0                   | 623000                 |
|                        | 中卫                             | 4                        | 0                        | 4                    | 0                   | 621200                 |
|                        | 宁东                             | 3                        | 0                        | 3                    | 0                   | 622900                 |
|                        | 石嘴山                           | 2                        | 0                        | 2                    | 0                   | 620700                 |
| 台湾                   | 台湾                             | 1                        | 0                        | 1                    | 0                   | 620300                 |
| 澳门                   | 澳门                             | 10                       | 0                        | 6                    | 0                   | 820000                 |
| 青海省                 | 青海                             | 18                       | 0                        | 18                   | 0                   | 630000                 |
|                        | 西宁                             | 15                       | 0                        | 15                   | 0                   | 630100                 |
|                        | 海北州                           | 3                        | 0                        | 3                    | 0                   | 632200                 |
| 西藏自治区             | 西藏                             | 1                        | 0                        | 1                    | 0                   | 540000                 |
|                        | 拉萨                             | 1                        | 0                        | 1                    | 0                   | 540100                 |
