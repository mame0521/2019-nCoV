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

![](img/2019-nCoV_trend.png)
![](img/2019-nCoV_map1.png)
![](img/2019-nCoV疫情地图.png)

| 省份名称 Province Name | 省份或城市名称 Provice/City Name | 确诊人数 Confirmed Count | 疑似人数 Suspected Count | 治愈人数 Cured Count | 死亡人数 Dead Count | 地区ID编码 Location ID |
|------------------------|----------------------------------|--------------------------|--------------------------|----------------------|---------------------|------------------------|
| 湖北省                 | 湖北                             | 16678                    | 0                        | 537                  | 479                 | 420000                 |
|                        | 武汉                             | 8351                     | 0                        | 374                  | 362                 | 420100                 |
|                        | 黄冈                             | 1645                     | 0                        | 52                   | 25                  | 421100                 |
|                        | 孝感                             | 1462                     | 0                        | 6                    | 18                  | 420900                 |
|                        | 襄阳                             | 735                      | 0                        | 7                    | 2                   | 420600                 |
|                        | 荆州                             | 713                      | 0                        | 12                   | 9                   | 421000                 |
|                        | 随州                             | 706                      | 0                        | 9                    | 8                   | 421300                 |
|                        | 黄石                             | 509                      | 0                        | 23                   | 2                   | 420200                 |
|                        | 宜昌                             | 496                      | 0                        | 9                    | 4                   | 420500                 |
|                        | 荆门                             | 422                      | 0                        | 16                   | 16                  | 420800                 |
|                        | 咸宁                             | 384                      | 0                        | 2                    | 0                   | 421200                 |
|                        | 鄂州                             | 382                      | 0                        | 6                    | 18                  | 420700                 |
|                        | 十堰                             | 318                      | 0                        | 9                    | 0                   | 420300                 |
|                        | 仙桃                             | 225                      | 0                        | 0                    | 4                   | 429004                 |
|                        | 恩施州                           | 138                      | 0                        | 10                   | 0                   | 422800                 |
|                        | 天门                             | 128                      | 0                        | 0                    | 10                  | 429006                 |
|                        | 潜江                             | 54                       | 0                        | 0                    | 1                   | 429005                 |
|                        | 神农架林区                       | 10                       | 0                        | 2                    | 0                   | 429021                 |
| 广东省                 | 广东                             | 895                      | 0                        | 49                   | 0                   | 440000                 |
|                        | 深圳                             | 291                      | 0                        | 16                   | 0                   | 440300                 |
|                        | 广州                             | 249                      | 0                        | 12                   | 0                   | 440100                 |
|                        | 珠海                             | 72                       | 0                        | 4                    | 0                   | 440400                 |
|                        | 佛山                             | 50                       | 0                        | 1                    | 0                   | 440600                 |
|                        | 东莞                             | 44                       | 0                        | 1                    | 0                   | 441900                 |
|                        | 中山                             | 36                       | 0                        | 3                    | 0                   | 442000                 |
|                        | 惠州                             | 34                       | 0                        | 1                    | 0                   | 441300                 |
|                        | 汕头                             | 20                       | 0                        | 3                    | 0                   | 440500                 |
|                        | 湛江                             | 18                       | 0                        | 4                    | 0                   | 440800                 |
|                        | 肇庆                             | 13                       | 0                        | 1                    | 0                   | 441200                 |
|                        | 阳江                             | 12                       | 0                        | 0                    | 0                   | 441700                 |
|                        | 梅州                             | 10                       | 0                        | 0                    | 0                   | 441400                 |
|                        | 清远                             | 10                       | 0                        | 2                    | 0                   | 441800                 |
|                        | 江门                             | 7                        | 0                        | 0                    | 0                   | 440700                 |
|                        | 韶关                             | 6                        | 0                        | 1                    | 0                   | 440200                 |
|                        | 揭阳                             | 6                        | 0                        | 0                    | 0                   | 445200                 |
|                        | 茂名                             | 6                        | 0                        | 0                    | 0                   | 440900                 |
|                        | 汕尾                             | 5                        | 0                        | 0                    | 0                   | 441500                 |
|                        | 潮州                             | 5                        | 0                        | 0                    | 0                   | 445100                 |
|                        | 河源                             | 1                        | 0                        | 0                    | 0                   | 441600                 |
| 浙江省                 | 浙江                             | 895                      | 0                        | 70                   | 0                   | 330000                 |
|                        | 温州                             | 364                      | 0                        | 28                   | 0                   | 330300                 |
|                        | 杭州                             | 141                      | 0                        | 7                    | 0                   | 330100                 |
|                        | 台州                             | 122                      | 0                        | 14                   | 0                   | 331000                 |
|                        | 宁波                             | 120                      | 0                        | 4                    | 0                   | 330200                 |
|                        | 金华                             | 44                       | 0                        | 8                    | 0                   | 330700                 |
|                        | 绍兴                             | 30                       | 0                        | 3                    | 0                   | 330600                 |
|                        | 嘉兴                             | 27                       | 0                        | 0                    | 0                   | 330400                 |
|                        | 丽水                             | 16                       | 0                        | 1                    | 0                   | 331100                 |
|                        | 衢州                             | 15                       | 0                        | 2                    | 0                   | 330800                 |
|                        | 湖州                             | 9                        | 0                        | 1                    | 0                   | 330500                 |
|                        | 舟山                             | 7                        | 0                        | 2                    | 0                   | 330900                 |
| 河南省                 | 河南                             | 764                      | 0                        | 46                   | 2                   | 410000                 |
|                        | 信阳                             | 138                      | 0                        | 6                    | 0                   | 411500                 |
|                        | 南阳                             | 107                      | 2                        | 2                    | 2                   | 411300                 |
|                        | 郑州                             | 92                       | 0                        | 13                   | 0                   | 410100                 |
|                        | 驻马店                           | 82                       | 0                        | 2                    | 0                   | 411700                 |
|                        | 商丘                             | 61                       | 0                        | 3                    | 0                   | 411400                 |
|                        | 周口                             | 56                       | 0                        | 1                    | 0                   | 411600                 |
|                        | 新乡                             | 39                       | 0                        | 0                    | 0                   | 410700                 |
|                        | 安阳                             | 35                       | 0                        | 1                    | 0                   | 410500                 |
|                        | 平顶山                           | 34                       | 0                        | 0                    | 0                   | 410400                 |
|                        | 漯河                             | 24                       | 0                        | 0                    | 0                   | 411100                 |
|                        | 许昌                             | 24                       | 0                        | 0                    | 0                   | 411000                 |
|                        | 洛阳                             | 17                       | 0                        | 1                    | 0                   | 410300                 |
|                        | 开封                             | 17                       | 0                        | 1                    | 0                   | 410200                 |
|                        | 焦作                             | 15                       | 0                        | 0                    | 0                   | 410800                 |
|                        | 鹤壁                             | 10                       | 0                        | 0                    | 0                   | 410600                 |
|                        | 三门峡                           | 7                        | 0                        | 0                    | 0                   | 411200                 |
|                        | 濮阳                             | 4                        | 0                        | 0                    | 0                   | 410900                 |
|                        | 济源                             | 2                        | 0                        | 0                    | 0                   | 419001                 |
| 湖南省                 | 湖南                             | 661                      | 0                        | 53                   | 0                   | 430000                 |
|                        | 长沙                             | 164                      | 0                        | 8                    | 0                   | 430100                 |
|                        | 岳阳                             | 89                       | 0                        | 2                    | 0                   | 430600                 |
|                        | 邵阳                             | 68                       | 0                        | 2                    | 0                   | 430500                 |
|                        | 常德                             | 56                       | 0                        | 3                    | 0                   | 430700                 |
|                        | 娄底                             | 45                       | 0                        | 3                    | 0                   | 431300                 |
|                        | 益阳                             | 40                       | 0                        | 6                    | 0                   | 430900                 |
|                        | 衡阳                             | 39                       | 0                        | 5                    | 0                   | 430400                 |
|                        | 株洲                             | 38                       | 0                        | 2                    | 0                   | 430200                 |
|                        | 怀化                             | 35                       | 0                        | 6                    | 0                   | 431200                 |
|                        | 永州                             | 30                       | 0                        | 8                    | 0                   | 431100                 |
|                        | 郴州                             | 24                       | 0                        | 2                    | 0                   | 431000                 |
|                        | 湘潭                             | 23                       | 0                        | 4                    | 0                   | 430300                 |
|                        | 湘西自治州                       | 7                        | 0                        | 1                    | 0                   | 433100                 |
|                        | 张家界                           | 3                        | 0                        | 2                    | 0                   | 430800                 |
| 江西省                 | 江西                             | 548                      | 0                        | 27                   | 0                   | 360000                 |
|                        | 南昌                             | 134                      | 0                        | 7                    | 0                   | 360100                 |
|                        | 九江                             | 86                       | 0                        | 3                    | 0                   | 360400                 |
|                        | 新余                             | 69                       | 0                        | 4                    | 0                   | 360500                 |
|                        | 上饶                             | 69                       | 0                        | 5                    | 0                   | 361100                 |
|                        | 宜春                             | 48                       | 0                        | 3                    | 0                   | 360900                 |
|                        | 赣州                             | 48                       | 0                        | 1                    | 0                   | 360700                 |
|                        | 抚州                             | 44                       | 0                        | 1                    | 0                   | 361000                 |
|                        | 萍乡                             | 22                       | 0                        | 1                    | 0                   | 360300                 |
|                        | 吉安                             | 15                       | 0                        | 1                    | 0                   | 360800                 |
|                        | 鹰潭                             | 9                        | 0                        | 0                    | 0                   | 360600                 |
|                        | 景德镇                           | 4                        | 0                        | 1                    | 0                   | 360200                 |
| 安徽省                 | 安徽                             | 530                      | 0                        | 23                   | 0                   | 340000                 |
|                        | 合肥                             | 93                       | 0                        | 5                    | 0                   | 340100                 |
|                        | 阜阳                             | 84                       | 0                        | 3                    | 0                   | 341200                 |
|                        | 亳州                             | 58                       | 0                        | 2                    | 0                   | 341600                 |
|                        | 安庆                             | 56                       | 0                        | 3                    | 0                   | 340800                 |
|                        | 蚌埠                             | 48                       | 0                        | 0                    | 0                   | 340300                 |
|                        | 六安                             | 34                       | 0                        | 1                    | 0                   | 341500                 |
|                        | 马鞍山                           | 30                       | 0                        | 1                    | 0                   | 340500                 |
|                        | 宿州                             | 24                       | 0                        | 0                    | 0                   | 341300                 |
|                        | 芜湖                             | 23                       | 0                        | 2                    | 0                   | 340200                 |
|                        | 铜陵                             | 19                       | 0                        | 1                    | 0                   | 340700                 |
|                        | 淮北                             | 14                       | 0                        | 0                    | 0                   | 340600                 |
|                        | 淮南                             | 12                       | 0                        | 2                    | 0                   | 340400                 |
|                        | 滁州                             | 11                       | 0                        | 1                    | 0                   | 341100                 |
|                        | 池州                             | 11                       | 0                        | 1                    | 0                   | 341700                 |
|                        | 黄山                             | 9                        | 0                        | 0                    | 0                   | 341000                 |
|                        | 宣城                             | 4                        | 0                        | 1                    | 0                   | 341800                 |
| 重庆市                 | 重庆                             | 376                      | 0                        | 15                   | 2                   | 500000                 |
|                        | 万州区                           | 67                       | 0                        | 2                    | 1                   | 500101                 |
|                        | 九龙坡区                         | 19                       | 0                        | 0                    | 1                   | 500107                 |
|                        | 开州区                           | 18                       | 0                        | 0                    | 0                   | 500154                 |
|                        | 云阳县                           | 18                       | 0                        | 2                    | 0                   | 500235                 |
|                        | 江北区                           | 16                       | 0                        | 0                    | 0                   | 500105                 |
|                        | 渝中区                           | 16                       | 0                        | 0                    | 0                   | 500103                 |
|                        | 忠县                             | 15                       | 0                        | 1                    | 0                   | 500233                 |
|                        | 垫江县                           | 15                       | 0                        | 1                    | 0                   | 500231                 |
|                        | 两江新区                         | 14                       | 0                        | 0                    | 0                   | \-1                    |
|                        | 渝北区                           | 12                       | 0                        | 0                    | 0                   | 500112                 |
|                        | 长寿区                           | 12                       | 0                        | 0                    | 0                   | 500115                 |
|                        | 石柱县                           | 12                       | 0                        | 0                    | 0                   | 500240                 |
|                        | 綦江区                           | 12                       | 0                        | 0                    | 0                   | 500110                 |
|                        | 巫溪县                           | 11                       | 0                        | 1                    | 0                   | 500238                 |
|                        | 合川区                           | 11                       | 0                        | 0                    | 0                   | 500117                 |
|                        | 奉节县                           | 11                       | 0                        | 1                    | 0                   | 500236                 |
|                        | 璧山区                           | 8                        | 0                        | 0                    | 0                   | 500120                 |
|                        | 巫山县                           | 8                        | 0                        | 4                    | 0                   | 500237                 |
|                        | 南岸区                           | 8                        | 0                        | 0                    | 0                   | 500108                 |
|                        | 荣昌区                           | 8                        | 0                        | 0                    | 0                   | 500153                 |
|                        | 大渡口区                         | 7                        | 0                        | 0                    | 0                   | 500104                 |
|                        | 大足区                           | 7                        | 0                        | 0                    | 0                   | 500111                 |
|                        | 铜梁区                           | 7                        | 0                        | 0                    | 0                   | 500151                 |
|                        | 潼南区                           | 7                        | 0                        | 0                    | 0                   | 500152                 |
|                        | 丰都县                           | 6                        | 0                        | 1                    | 0                   | 500230                 |
|                        | 江津区                           | 4                        | 0                        | 0                    | 0                   | 500116                 |
|                        | 高新区                           | 4                        | 0                        | 0                    | 0                   | 0                      |
|                        | 巴南区                           | 3                        | 0                        | 0                    | 0                   | 500113                 |
|                        | 永川区                           | 3                        | 0                        | 1                    | 0                   | 500118                 |
|                        | 梁平区                           | 3                        | 0                        | 0                    | 0                   | 500155                 |
|                        | 涪陵区                           | 3                        | 0                        | 0                    | 0                   | 500102                 |
|                        | 城口县                           | 2                        | 0                        | 0                    | 0                   | 500229                 |
|                        | 黔江区                           | 2                        | 0                        | 0                    | 0                   | 500114                 |
|                        | 沙坪坝区                         | 2                        | 0                        | 0                    | 0                   | 500106                 |
|                        | 彭水县                           | 2                        | 0                        | 0                    | 0                   | 500243                 |
|                        | 秀山县                           | 1                        | 0                        | 1                    | 0                   | 500241                 |
|                        | 武隆区                           | 1                        | 0                        | 0                    | 0                   | 500156                 |
|                        | 酉阳县                           | 1                        | 0                        | 0                    | 0                   | 0                      |
| 江苏省                 | 江苏                             | 341                      | 0                        | 23                   | 0                   | 320000                 |
|                        | 苏州                             | 55                       | 0                        | 4                    | 0                   | 320500                 |
|                        | 南京                             | 47                       | 0                        | 10                   | 0                   | 320100                 |
|                        | 徐州                             | 45                       | 0                        | 1                    | 0                   | 320300                 |
|                        | 无锡                             | 26                       | 0                        | 1                    | 0                   | 320200                 |
|                        | 泰州                             | 26                       | 0                        | 1                    | 0                   | 321200                 |
|                        | 淮安                             | 26                       | 0                        | 1                    | 0                   | 320800                 |
|                        | 南通                             | 25                       | 0                        | 0                    | 0                   | 320600                 |
|                        | 常州                             | 24                       | 0                        | 2                    | 0                   | 320400                 |
|                        | 连云港                           | 20                       | 0                        | 1                    | 0                   | 320700                 |
|                        | 扬州                             | 15                       | 0                        | 0                    | 0                   | 321000                 |
|                        | 盐城                             | 15                       | 0                        | 2                    | 0                   | 320900                 |
|                        | 宿迁                             | 11                       | 0                        | 0                    | 0                   | 321300                 |
|                        | 镇江                             | 6                        | 0                        | 0                    | 0                   | 321100                 |
| 山东省                 | 山东                             | 307                      | 0                        | 15                   | 0                   | 370000                 |
|                        | 青岛                             | 35                       | 0                        | 3                    | 0                   | 370200                 |
|                        | 临沂                             | 30                       | 0                        | 1                    | 0                   | 371300                 |
|                        | 济南                             | 29                       | 0                        | 0                    | 0                   | 370100                 |
|                        | 烟台                             | 27                       | 0                        | 0                    | 0                   | 370600                 |
|                        | 威海                             | 27                       | 0                        | 2                    | 0                   | 371000                 |
|                        | 济宁                             | 25                       | 0                        | 3                    | 0                   | 370800                 |
|                        | 德州                             | 24                       | 0                        | 1                    | 0                   | 371400                 |
|                        | 枣庄                             | 17                       | 0                        | 0                    | 0                   | 370400                 |
|                        | 潍坊                             | 16                       | 0                        | 0                    | 0                   | 370700                 |
|                        | 日照                             | 14                       | 0                        | 0                    | 0                   | 371100                 |
|                        | 淄博                             | 14                       | 0                        | 0                    | 0                   | 370300                 |
|                        | 泰安                             | 14                       | 0                        | 1                    | 0                   | 370900                 |
|                        | 聊城                             | 12                       | 0                        | 0                    | 0                   | 371500                 |
|                        | 菏泽                             | 12                       | 0                        | 2                    | 0                   | 371700                 |
|                        | 滨州                             | 11                       | 0                        | 2                    | 0                   | 371600                 |
| 四川省                 | 四川                             | 301                      | 0                        | 24                   | 1                   | 510000                 |
|                        | 成都                             | 92                       | 0                        | 17                   | 1                   | 510100                 |
|                        | 南充                             | 28                       | 0                        | 1                    | 0                   | 511300                 |
|                        | 广安                             | 24                       | 0                        | 1                    | 0                   | 511600                 |
|                        | 绵阳                             | 17                       | 0                        | 2                    | 0                   | 510700                 |
|                        | 达州                             | 17                       | 0                        | 1                    | 0                   | 511700                 |
|                        | 内江                             | 15                       | 0                        | 0                    | 0                   | 511000                 |
|                        | 巴中                             | 15                       | 0                        | 0                    | 0                   | 511900                 |
|                        | 德阳                             | 12                       | 0                        | 0                    | 0                   | 510600                 |
|                        | 攀枝花                           | 11                       | 0                        | 0                    | 0                   | 510400                 |
|                        | 泸州                             | 10                       | 0                        | 0                    | 0                   | 510500                 |
|                        | 甘孜州                           | 10                       | 0                        | 0                    | 0                   | 513300                 |
|                        | 自贡                             | 9                        | 0                        | 0                    | 0                   | 510300                 |
|                        | 宜宾                             | 7                        | 0                        | 0                    | 0                   | 511500                 |
|                        | 凉山州                           | 7                        | 0                        | 0                    | 0                   | 513400                 |
|                        | 广元                             | 6                        | 0                        | 0                    | 0                   | 510800                 |
|                        | 雅安                             | 6                        | 0                        | 0                    | 0                   | 511800                 |
|                        | 遂宁                             | 5                        | 0                        | 1                    | 0                   | 510900                 |
|                        | 眉山                             | 5                        | 0                        | 1                    | 0                   | 511400                 |
|                        | 乐山                             | 2                        | 0                        | 0                    | 0                   | 511100                 |
|                        | 资阳                             | 2                        | 0                        | 0                    | 0                   | 512000                 |
|                        | 阿坝州                           | 1                        | 0                        | 0                    | 0                   | 513200                 |
| 北京市                 | 北京                             | 253                      | 0                        | 24                   | 1                   | 110000                 |
|                        | 海淀区                           | 45                       | 0                        | 0                    | 0                   | 110108                 |
|                        | 朝阳区                           | 43                       | 0                        | 0                    | 0                   | 110105                 |
|                        | 大兴区                           | 29                       | 0                        | 2                    | 0                   | 110115                 |
|                        | 西城区                           | 29                       | 0                        | 0                    | 0                   | 110102                 |
|                        | 外地来京人员                     | 21                       | 0                        | 2                    | 0                   | \-1                    |
|                        | 丰台区                           | 20                       | 0                        | 0                    | 0                   | 110106                 |
|                        | 昌平区                           | 15                       | 0                        | 0                    | 0                   | 110114                 |
|                        | 通州区                           | 13                       | 0                        | 0                    | 0                   | 110112                 |
|                        | 房山区                           | 11                       | 0                        | 0                    | 0                   | 110111                 |
|                        | 石景山区                         | 7                        | 0                        | 0                    | 0                   | 110107                 |
|                        | 东城区                           | 6                        | 0                        | 0                    | 0                   | 110101                 |
|                        | 顺义区                           | 6                        | 0                        | 0                    | 0                   | 110113                 |
|                        | 怀柔区                           | 4                        | 0                        | 0                    | 0                   | 110116                 |
|                        | 门头沟区                         | 3                        | 0                        | 0                    | 0                   | 110109                 |
|                        | 延庆区                           | 1                        | 0                        | 0                    | 0                   | 110119                 |
|                        | 待明确地区                       | 0                        | 0                        | 20                   | 1                   | 0                      |
| 上海市                 | 上海                             | 243                      | 0                        | 15                   | 1                   | 310000                 |
|                        | 外地来沪人员                     | 86                       | 0                        | 6                    | 1                   | \-1                    |
|                        | 浦东新区                         | 46                       | 0                        | 0                    | 0                   | 310115                 |
|                        | 静安区                           | 13                       | 0                        | 0                    | 0                   | 310106                 |
|                        | 徐汇区                           | 12                       | 0                        | 0                    | 0                   | 310104                 |
|                        | 宝山区                           | 12                       | 0                        | 0                    | 0                   | 310113                 |
|                        | 闵行区                           | 12                       | 0                        | 0                    | 0                   | 310112                 |
|                        | 松江区                           | 10                       | 0                        | 0                    | 0                   | 310117                 |
|                        | 长宁区                           | 9                        | 0                        | 0                    | 0                   | 310105                 |
|                        | 虹口区                           | 7                        | 0                        | 0                    | 0                   | 310109                 |
|                        | 奉贤区                           | 7                        | 0                        | 0                    | 0                   | 310120                 |
|                        | 普陀区                           | 6                        | 0                        | 0                    | 0                   | 310107                 |
|                        | 杨浦区                           | 6                        | 0                        | 0                    | 0                   | 310110                 |
|                        | 黄浦区                           | 5                        | 0                        | 0                    | 0                   | 310101                 |
|                        | 嘉定区                           | 4                        | 0                        | 0                    | 0                   | 310114                 |
|                        | 青浦区                           | 4                        | 0                        | 0                    | 0                   | 310118                 |
|                        | 金山区                           | 2                        | 0                        | 0                    | 0                   | 310116                 |
|                        | 崇明区                           | 2                        | 0                        | 0                    | 0                   | 310151                 |
|                        | 未知地区                         | 0                        | 0                        | 9                    | 0                   | 0                      |
| 福建省                 | 福建                             | 205                      | 0                        | 11                   | 0                   | 350000                 |
|                        | 福州                             | 55                       | 0                        | 4                    | 0                   | 350100                 |
|                        | 莆田                             | 38                       | 0                        | 1                    | 0                   | 350300                 |
|                        | 泉州                             | 37                       | 0                        | 4                    | 0                   | 350500                 |
|                        | 厦门                             | 21                       | 0                        | 1                    | 0                   | 350200                 |
|                        | 漳州                             | 15                       | 0                        | 0                    | 0                   | 350600                 |
|                        | 宁德                             | 14                       | 0                        | 1                    | 0                   | 350900                 |
|                        | 南平                             | 12                       | 0                        | 0                    | 0                   | 350700                 |
|                        | 三明                             | 11                       | 0                        | 0                    | 0                   | 350400                 |
|                        | 龙岩                             | 2                        | 0                        | 0                    | 0                   | 350800                 |
| 黑龙江省               | 黑龙江                           | 190                      | 0                        | 7                    | 2                   | 230000                 |
|                        | 哈尔滨                           | 73                       | 0                        | 6                    | 0                   | 230100                 |
|                        | 绥化                             | 24                       | 0                        | 0                    | 2                   | 231200                 |
|                        | 齐齐哈尔                         | 20                       | 0                        | 0                    | 0                   | 230200                 |
|                        | 双鸭山                           | 16                       | 0                        | 1                    | 0                   | 230500                 |
|                        | 鸡西                             | 12                       | 0                        | 0                    | 0                   | 230300                 |
|                        | 大庆                             | 11                       | 0                        | 0                    | 0                   | 230600                 |
|                        | 佳木斯                           | 11                       | 0                        | 0                    | 0                   | 230800                 |
|                        | 牡丹江                           | 9                        | 0                        | 0                    | 0                   | 231000                 |
|                        | 七台河                           | 9                        | 0                        | 0                    | 0                   | 230900                 |
|                        | 大兴安岭                         | 2                        | 0                        | 0                    | 0                   | 232700                 |
|                        | 鹤岗                             | 2                        | 0                        | 0                    | 0                   | 230400                 |
|                        | 黑河                             | 1                        | 0                        | 0                    | 0                   | 231100                 |
| 陕西省                 | 陕西                             | 165                      | 0                        | 6                    | 0                   | 610000                 |
|                        | 西安                             | 70                       | 0                        | 5                    | 0                   | 610100                 |
|                        | 安康                             | 19                       | 0                        | 1                    | 0                   | 610900                 |
|                        | 汉中                             | 19                       | 0                        | 0                    | 0                   | 610700                 |
|                        | 咸阳                             | 15                       | 0                        | 0                    | 0                   | 610400                 |
|                        | 宝鸡                             | 10                       | 0                        | 0                    | 0                   | 610300                 |
|                        | 渭南                             | 9                        | 0                        | 0                    | 0                   | 610500                 |
|                        | 延安                             | 7                        | 0                        | 0                    | 0                   | 610600                 |
|                        | 铜川                             | 7                        | 0                        | 0                    | 0                   | 610200                 |
|                        | 商洛                             | 6                        | 0                        | 0                    | 0                   | 611000                 |
|                        | 榆林                             | 1                        | 0                        | 0                    | 0                   | 610800                 |
|                        | 韩城                             | 1                        | 0                        | 0                    | 0                   | 610581                 |
|                        | 杨凌                             | 1                        | 0                        | 0                    | 0                   | 0                      |
| 广西壮族自治区         | 广西                             | 150                      | 0                        | 13                   | 0                   | 450000                 |
|                        | 北海                             | 29                       | 0                        | 2                    | 0                   | 450500                 |
|                        | 南宁                             | 28                       | 0                        | 0                    | 0                   | 450100                 |
|                        | 桂林                             | 26                       | 0                        | 4                    | 0                   | 450300                 |
|                        | 柳州                             | 17                       | 0                        | 3                    | 0                   | 450200                 |
|                        | 防城港                           | 13                       | 0                        | 1                    | 0                   | 450600                 |
|                        | 河池                             | 9                        | 0                        | 0                    | 0                   | 451200                 |
|                        | 玉林                             | 8                        | 0                        | 1                    | 0                   | 450900                 |
|                        | 梧州                             | 5                        | 0                        | 2                    | 0                   | 450400                 |
|                        | 贵港                             | 5                        | 0                        | 0                    | 0                   | 450800                 |
|                        | 贺州                             | 4                        | 0                        | 0                    | 0                   | 451100                 |
|                        | 钦州                             | 4                        | 0                        | 0                    | 0                   | 450700                 |
|                        | 百色                             | 2                        | 0                        | 0                    | 0                   | 451000                 |
| 河北省                 | 河北                             | 135                      | 0                        | 6                    | 1                   | 130000                 |
|                        | 石家庄                           | 21                       | 0                        | 1                    | 0                   | 130100                 |
|                        | 沧州                             | 20                       | 0                        | 0                    | 1                   | 130900                 |
|                        | 保定                             | 17                       | 0                        | 4                    | 0                   | 130600                 |
|                        | 邢台                             | 16                       | 0                        | 0                    | 0                   | 130500                 |
|                        | 邯郸                             | 14                       | 0                        | 0                    | 0                   | 130400                 |
|                        | 廊坊                             | 13                       | 0                        | 1                    | 0                   | 131000                 |
|                        | 唐山                             | 12                       | 0                        | 0                    | 0                   | 130200                 |
|                        | 张家口                           | 10                       | 0                        | 0                    | 0                   | 130700                 |
|                        | 衡水                             | 5                        | 0                        | 0                    | 0                   | 131100                 |
|                        | 承德                             | 4                        | 0                        | 0                    | 0                   | 130800                 |
|                        | 秦皇岛                           | 3                        | 0                        | 0                    | 0                   | 130300                 |
| 云南省                 | 云南                             | 124                      | 0                        | 5                    | 0                   | 530000                 |
|                        | 昆明                             | 35                       | 0                        | 0                    | 0                   | 530100                 |
|                        | 西双版纳                         | 15                       | 0                        | 1                    | 0                   | 532800                 |
|                        | 玉溪                             | 12                       | 0                        | 1                    | 0                   | 530400                 |
|                        | 昭通                             | 10                       | 0                        | 0                    | 0                   | 530600                 |
|                        | 曲靖                             | 10                       | 0                        | 1                    | 0                   | 530300                 |
|                        | 保山                             | 8                        | 0                        | 0                    | 0                   | 530500                 |
|                        | 丽江                             | 7                        | 0                        | 0                    | 0                   | 530700                 |
|                        | 大理                             | 7                        | 0                        | 1                    | 0                   | 532901                 |
|                        | 红河                             | 5                        | 0                        | 1                    | 0                   | 532500                 |
|                        | 德宏                             | 5                        | 0                        | 0                    | 0                   | 533100                 |
|                        | 普洱                             | 4                        | 0                        | 0                    | 0                   | 530800                 |
|                        | 楚雄                             | 4                        | 0                        | 0                    | 0                   | 532301                 |
|                        | 临沧                             | 1                        | 0                        | 0                    | 0                   | 530900                 |
|                        | 文山                             | 1                        | 0                        | 0                    | 0                   | 532601                 |
| 海南省                 | 海南                             | 91                       | 0                        | 5                    | 1                   | 460000                 |
|                        | 三亚                             | 22                       | 0                        | 5                    | 0                   | 460200                 |
|                        | 海口                             | 16                       | 0                        | 0                    | 0                   | 460100                 |
|                        | 万宁                             | 11                       | 0                        | 0                    | 0                   | 469006                 |
|                        | 儋州                             | 10                       | 0                        | 0                    | 0                   | 460400                 |
|                        | 琼海                             | 6                        | 0                        | 0                    | 0                   | 469002                 |
|                        | 临高                             | 5                        | 0                        | 0                    | 0                   | 469024                 |
|                        | 陵水                             | 4                        | 0                        | 0                    | 0                   | 469028                 |
|                        | 澄迈                             | 4                        | 0                        | 0                    | 1                   | 469023                 |
|                        | 定安                             | 3                        | 0                        | 0                    | 0                   | 469021                 |
|                        | 昌江                             | 3                        | 0                        | 0                    | 0                   | 469026                 |
|                        | 东方                             | 2                        | 0                        | 0                    | 0                   | 469007                 |
|                        | 保亭                             | 2                        | 0                        | 0                    | 0                   | 0                      |
|                        | 琼中                             | 1                        | 0                        | 0                    | 0                   | 469030                 |
|                        | 乐东                             | 1                        | 0                        | 0                    | 0                   | 469027                 |
|                        | 文昌                             | 1                        | 0                        | 0                    | 0                   | 469005                 |
| 辽宁省                 | 辽宁                             | 88                       | 0                        | 4                    | 0                   | 210000                 |
|                        | 沈阳                             | 22                       | 0                        | 1                    | 0                   | 210100                 |
|                        | 大连                             | 14                       | 0                        | 3                    | 0                   | 210200                 |
|                        | 盘锦                             | 8                        | 0                        | 0                    | 0                   | 211100                 |
|                        | 锦州                             | 7                        | 0                        | 0                    | 0                   | 210700                 |
|                        | 丹东                             | 7                        | 0                        | 0                    | 0                   | 210600                 |
|                        | 葫芦岛                           | 7                        | 0                        | 0                    | 0                   | 211400                 |
|                        | 阜新                             | 7                        | 0                        | 0                    | 0                   | 210900                 |
|                        | 朝阳                             | 6                        | 0                        | 0                    | 0                   | 211300                 |
|                        | 本溪                             | 3                        | 0                        | 0                    | 0                   | 210500                 |
|                        | 铁岭                             | 3                        | 0                        | 0                    | 0                   | 211200                 |
|                        | 辽阳                             | 2                        | 0                        | 0                    | 0                   | 211000                 |
|                        | 营口                             | 1                        | 0                        | 0                    | 0                   | 210800                 |
|                        | 鞍山                             | 1                        | 0                        | 0                    | 0                   | 210300                 |
| 山西省                 | 山西                             | 81                       | 0                        | 4                    | 0                   | 140000                 |
|                        | 晋中                             | 21                       | 0                        | 2                    | 0                   | 140700                 |
|                        | 运城                             | 17                       | 0                        | 1                    | 0                   | 140800                 |
|                        | 大同                             | 9                        | 0                        | 0                    | 0                   | 140200                 |
|                        | 太原                             | 9                        | 0                        | 1                    | 0                   | 140100                 |
|                        | 晋城                             | 7                        | 0                        | 0                    | 0                   | 140500                 |
|                        | 吕梁                             | 5                        | 0                        | 0                    | 0                   | 141100                 |
|                        | 朔州                             | 4                        | 0                        | 0                    | 0                   | 140600                 |
|                        | 长治                             | 3                        | 0                        | 0                    | 0                   | 140400                 |
|                        | 忻州                             | 3                        | 0                        | 0                    | 0                   | 140900                 |
|                        | 阳泉                             | 2                        | 0                        | 0                    | 0                   | 140300                 |
|                        | 临汾                             | 1                        | 0                        | 0                    | 0                   | 141000                 |
| 天津市                 | 天津                             | 69                       | 0                        | 2                    | 1                   | 120000                 |
|                        | 宝坻区                           | 15                       | 0                        | 0                    | 1                   | 120115                 |
|                        | 河北区                           | 10                       | 0                        | 0                    | 0                   | 120105                 |
|                        | 河东区                           | 10                       | 0                        | 0                    | 0                   | 120102                 |
|                        | 和平区                           | 6                        | 0                        | 0                    | 0                   | 120101                 |
|                        | 外地来津人员                     | 6                        | 0                        | 0                    | 0                   | 0                      |
|                        | 河西区                           | 4                        | 0                        | 0                    | 0                   | 120103                 |
|                        | 宁河区                           | 4                        | 0                        | 0                    | 0                   | 120117                 |
|                        | 东丽区                           | 3                        | 0                        | 0                    | 0                   | 120110                 |
|                        | 南开区                           | 3                        | 0                        | 0                    | 0                   | 120104                 |
|                        | 西青区                           | 3                        | 0                        | 0                    | 0                   | 120111                 |
|                        | 滨海新区                         | 2                        | 0                        | 0                    | 0                   | 120116                 |
|                        | 红桥区                           | 2                        | 0                        | 1                    | 0                   | 120106                 |
|                        | 津南区                           | 1                        | 0                        | 0                    | 0                   | 120112                 |
| 贵州省                 | 贵州                             | 64                       | 0                        | 9                    | 1                   | 520000                 |
|                        | 贵阳                             | 12                       | 0                        | 4                    | 0                   | 520100                 |
|                        | 毕节                             | 11                       | 0                        | 0                    | 0                   | 520500                 |
|                        | 六盘水                           | 9                        | 0                        | 0                    | 1                   | 520200                 |
|                        | 遵义                             | 9                        | 0                        | 0                    | 0                   | 520300                 |
|                        | 铜仁                             | 6                        | 0                        | 2                    | 0                   | 520600                 |
|                        | 黔南州                           | 6                        | 0                        | 3                    | 0                   | 522700                 |
|                        | 黔东南州                         | 5                        | 0                        | 0                    | 0                   | 522600                 |
|                        | 黔西南州                         | 3                        | 0                        | 0                    | 0                   | 522300                 |
|                        | 安顺                             | 3                        | 0                        | 0                    | 0                   | 520400                 |
| 甘肃省                 | 甘肃                             | 57                       | 0                        | 4                    | 0                   | 620000                 |
|                        | 兰州                             | 30                       | 0                        | 3                    | 0                   | 620100                 |
|                        | 天水                             | 10                       | 0                        | 0                    | 0                   | 620500                 |
|                        | 陇南                             | 4                        | 0                        | 0                    | 0                   | 621200                 |
|                        | 临夏                             | 3                        | 0                        | 0                    | 0                   | 622901                 |
|                        | 定西                             | 3                        | 0                        | 1                    | 0                   | 621100                 |
|                        | 张掖                             | 2                        | 0                        | 0                    | 0                   | 620700                 |
|                        | 白银                             | 1                        | 0                        | 0                    | 0                   | 620400                 |
|                        | 金昌                             | 1                        | 0                        | 0                    | 0                   | 620300                 |
|                        | 平凉                             | 1                        | 0                        | 0                    | 0                   | 620800                 |
|                        | 甘南                             | 1                        | 0                        | 0                    | 0                   | 623000                 |
|                        | 庆阳                             | 1                        | 0                        | 0                    | 0                   | 621000                 |
| 吉林省                 | 吉林                             | 54                       | 0                        | 2                    | 0                   | 220000                 |
|                        | 长春                             | 28                       | 0                        | 1                    | 0                   | 220100                 |
|                        | 四平                             | 8                        | 0                        | 0                    | 0                   | 220300                 |
|                        | 吉林市                           | 5                        | 0                        | 1                    | 0                   | 220200                 |
|                        | 公主岭                           | 4                        | 0                        | 0                    | 0                   | 220381                 |
|                        | 延边                             | 3                        | 0                        | 0                    | 0                   | 222400                 |
|                        | 辽源                             | 3                        | 0                        | 0                    | 0                   | 220400                 |
|                        | 松原                             | 2                        | 0                        | 0                    | 0                   | 220700                 |
|                        | 通化                             | 1                        | 0                        | 0                    | 0                   | 220500                 |
| 内蒙古自治区           | 内蒙古                           | 42                       | 0                        | 3                    | 0                   | 150000                 |
|                        | 鄂尔多斯                         | 11                       | 0                        | 2                    | 0                   | 150600                 |
|                        | 包头                             | 7                        | 0                        | 0                    | 0                   | 150200                 |
|                        | 呼和浩特                         | 5                        | 0                        | 0                    | 0                   | 150100                 |
|                        | 巴彦淖尔                         | 4                        | 0                        | 0                    | 0                   | 150800                 |
|                        | 乌兰察布                         | 3                        | 0                        | 0                    | 0                   | 150900                 |
|                        | 呼伦贝尔                         | 3                        | 0                        | 1                    | 0                   | 150700                 |
|                        | 赤峰                             | 3                        | 0                        | 0                    | 0                   | 150400                 |
|                        | 锡林郭勒盟                       | 2                        | 0                        | 0                    | 0                   | 152500                 |
|                        | 乌海市                           | 2                        | 0                        | 0                    | 0                   | 150300                 |
|                        | 兴安盟                           | 1                        | 0                        | 0                    | 0                   | 152200                 |
|                        | 通辽                             | 1                        | 0                        | 0                    | 0                   | 150500                 |
| 宁夏回族自治区         | 宁夏                             | 34                       | 0                        | 1                    | 0                   | 640000                 |
|                        | 银川                             | 20                       | 0                        | 1                    | 0                   | 640100                 |
|                        | 吴忠                             | 7                        | 0                        | 0                    | 0                   | 640300                 |
|                        | 中卫                             | 3                        | 0                        | 0                    | 0                   | 640500                 |
|                        | 固原                             | 2                        | 0                        | 0                    | 0                   | 640400                 |
|                        | 宁东管委会                       | 1                        | 0                        | 0                    | 0                   | 0                      |
|                        | 石嘴山                           | 1                        | 0                        | 0                    | 0                   | 640200                 |
| 新疆维吾尔自治区       | 新疆                             | 32                       | 0                        | 0                    | 0                   | 650000                 |
|                        | 乌鲁木齐                         | 12                       | 0                        | 0                    | 0                   | 650100                 |
|                        | 伊犁州                           | 9                        | 0                        | 0                    | 0                   | 654000                 |
|                        | 第九师                           | 4                        | 0                        | 0                    | 0                   | 0                      |
|                        | 昌吉                             | 2                        | 0                        | 0                    | 0                   | 652301                 |
|                        | 吐鲁番                           | 1                        | 0                        | 0                    | 0                   | 650400                 |
|                        | 第七师                           | 1                        | 0                        | 0                    | 0                   | \-1                    |
|                        | 第八师                           | 1                        | 0                        | 0                    | 0                   | 0                      |
|                        | 阿克苏                           | 1                        | 0                        | 0                    | 0                   | 652901                 |
|                        | 第六师                           | 1                        | 0                        | 0                    | 0                   | 0                      |
| 香港                   | 香港                             | 21                       | 0                        | 0                    | 1                   | 810000                 |
| 青海省                 | 青海                             | 17                       | 0                        | 3                    | 0                   | 630000                 |
|                        | 西宁                             | 14                       | 0                        | 3                    | 0                   | 630100                 |
|                        | 海北州                           | 3                        | 0                        | 0                    | 0                   | 632200                 |
| 台湾                   | 台湾                             | 11                       | 0                        | 0                    | 0                   | 710000                 |
| 澳门                   | 澳门                             | 10                       | 0                        | 0                    | 0                   | 820000                 |
| 西藏自治区             | 西藏                             | 1                        | 0                        | 0                    | 0                   | 540000                 |
|                        | 拉萨                             | 1                        | 0                        | 0                    | 0                   | 540100                 |
|                        |                                  |                          |                          |                      |                     |                        |
