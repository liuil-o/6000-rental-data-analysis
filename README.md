
# 摘要

本研究深入分析了北京市租房市场，利用6000余条详细的租房数据记录，结合市场调研和网络数据搜集，揭示了不同区域、不同户型的租房价格分布，并探讨了影响租金价格的关键因素。

## 1.数据集概述

数据集包含6000余条租房信息，分布在8个结构统一的CSV文件中，涉及北京市多个区域。

## 2.数据处理流程

1. 利用pandas库将CSV文件合并，执行数据清洗，包括删除重复记录、处理缺失值、剔除异常数据。
2. 对数据进行转换和增强，例如，从楼层信息中提取出所在楼层和总楼层，从地铁信息中提取地铁线路数和距离地铁的距离。
3. 将清洗后的数据保存至SQLite数据库，以便于进一步分析。

## 3.数据分析与可视化结果

![数据分析图一](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/f16afd37-1b0b-4b3b-9ecc-0233428d7a8e)

![数据分析图二](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/db01796f-4362-4f7f-9cf8-38cc276aaa4e)


### 3.1市场概况

- 平均每平米租金为169元
- 平均每套房源出租面积为15.68平米
- 朝阳区和东城区因地理位置优越，租金普遍在2000元以上，部分高端住宅区超过9000元

### 3.2区域分布

- 朝阳区和通州区房源数量最多，分别占总房源的25%和22%，显示出这两个区域租赁市场的活跃度

### 3.3小区分析

- 半壁街南路1号院的租金最高，达到596元/平米，是市场平均值的3.5倍

### 3.4户型与楼层

- 2-4室户型占市场主导
- 电梯房平均每平米租金比非电梯房高10元
- 低楼层和高楼层的租金普遍高于中间楼层

### 3.5交通便利性

- 距离地铁站500米以内的房源，其价格比同区域远离地铁的房源高出10%至20%

# 一.数据集说明

这是一份北京的租房数据，总计6000多条记录，分为8个同样结构的CSV数据文件。

# 二.数据处理

```python
# 合并数据文件 
dir = r"C:\Users\Administrator\Desktop\RentFromDanke"
data_list = []
for i in range(1, 9):
    path = f"{dir}\\bj_danke_{i}.csv"
    data = pd.read_csv(path)
    data_list.append(data)
data = pd.concat(data_list)

## 数据清洗
#  数据重复处理: 删除重复值
# print(data[data.duplicated()])
data.drop_duplicates(inplace=True)
data.reset_index(drop=True, inplace=True)

# 缺失值处理：直接删除缺失值所在行，并重置索引
# print(data.isnull().sum())
data.dropna(axis=0, inplace=True)
data.reset_index(drop=True, inplace=True)

# 异常值清洗
data['户型'].unique()
# print(data[data['户型'] == '户型'])
data = data[data['户型'] != '户型']

# 清洗，列替换
data.loc[:, '地铁'] = data['地铁'].apply(lambda x: x.replace('地铁：', ''))

# 增加列
data.loc[:, '所在楼层'] = data['楼层'].apply(lambda x: int(x.split('/')[0]))
data.loc[:, '总楼层'] = data['楼层'].apply(lambda x: int(x.replace('层', '').split('/')[-1]))
data.loc[:, '地铁数'] = data['地铁'].apply(lambda x: len(re.findall('线', x)))
data.loc[:, '距离地铁距离'] = data['地铁'].apply(lambda x: int(re.findall('(\d+)米', x)[-1]) if re.findall('(\d+)米', x) else -1)

# 数据类型转换
data['价格'] = data['价格'].astype(np.int64)
data['面积'] = data['面积'].astype(np.int64)
data['距离地铁距离'] = data['距离地铁距离'].astype(np.int64)

#数据保存 
# 查看保存的数据
print(data.info)

# 保存清洗后的数据 sqlite
engine = create_engine('sqlite:///D:/GitHub/bigdata_analyse/rent.db')
data.to_sql('rent', con=engine, index=False, if_exists='append')
```

## 清洗后的数据

![清洗后的数据](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/9d900f68-1b66-4f02-9c2c-0ddb2a85fd4a)

# 三.数据分析可视化

## 3.1 整体情况

数据显示，北京市租房价格受多种因素影响，包括地理位置、交通便利性、房屋面积、户型结构等。例如，朝阳区和东城区由于其优越的地理位置和便捷的交通，租房价格普遍较高。而通州区和房山区作为相对较远的区域，租房价格相对较低。

![屏幕截图 2024-04-17 195547](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/79eba83f-d7d8-49b2-a614-09d4b766c5c7)

该数据集总共有6024个房源信息，平均每平米的租金为169元，每套房源的平均出租面积为15.68平米。

## 3.2 地区分析

房源数量分布情况如下，可以看到朝阳和通州这两个地区的房源数量要远大于其它区，说明这两个地方的租赁市场比较活跃，人员流动和人口密度可能也比较大。

- 核心区域：朝阳区、东城区的租房价格普遍在2000元以上，部分高端住宅区价格甚至超过9000元。
- 非核心区域：通州区、房山区的租房价格多在2000元以下，价格更加亲民。
- 
![屏幕截图 2024-04-17 195547](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/a90e1e60-97aa-41c5-a567-7c9417b710af)


## 3.3 小区分析

房租最贵的小区TOP 10。半壁街南路1号院的房租最高，达到596元/平米，是平均值169元/平米的3倍。

![屏幕截图 2024-04-17 204214](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/5aa9c228-d8c6-410b-a80c-9a4c872d7777)



## 3.4 户型楼层分析

从户型的房源数量分布来看，主要集中在2-4室的户型。之前也分析了，每套房源的平均出租面积为15.68平米，可见大部分房源都是合租，毕竟房租那么贵，生活成本太高了。

租房者对户型和面积的需求多样，从单身公寓到多室家庭住宅。数据分析显示，户型范围从1室1卫到8室3卫，面积从9平方米到超过100平方米不等。

1室1卫的户型在租房市场中占有较大比例，这可能与北京的单身人口和年轻专业人士较多有关。此外，3室1卫的户型也有一定的市场需求，这可能与家庭型租户的需求相匹配。例如，“京通苑阳光华苑”提供4室2卫的户型，面积达到116平方米，适合大家庭居住。

![屏幕截图 2024-04-17 204322](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/ab6dbba0-027e-4b36-99c5-480088c8ce25)

国家规定楼层7层以上需要装电梯，依据这个规定，我们根据楼层数来判断房源是否有电梯。

从下图可以看到，电梯房的房源数量比较多，毕竟楼层高，建的房子多，此外，电梯房平均每平米的租金也要比非电梯房贵10块钱。

![image](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/dd5b08a1-5f32-4188-867b-5e3f693c7f57)

![image](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/d6df03c1-0f31-4513-9092-efa67bfcdc3a)

在区分出电梯房之后，我们再引入楼层的纬度进行分析。

从租金上看，不管是电梯房还是非电梯房，低楼层的租金都会比较贵一些。因为北京地处北方，天气较干燥，不会有回南天，而且低楼层出行较为方便。电梯房的高楼层，租金也会比较贵，这大概是因为高楼层的风景较好。

南方天气潮湿，在春天的时候，有时会出现回南天这一气象，导致低楼层会出现地板、墙壁渗水，所以在南方一般都不爱租低层。

从房源数量上看，非电梯房的高层房源最多，低层房源最少。说明非电梯房的高层房源不容易租出去，这点在租金上也有所体现。

![image](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/5db08be0-e100-479a-ab34-8cb5f7c4a046)

## 3.5 交通分析

从地理位置上来看，交通越便利，租金也越贵，这个符合一般认知。

![image](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/17d5b278-01b6-4840-b20f-7040feffa134)

靠近地铁站的房源价格普遍高于同区域其他房源。例如，距离地铁站500米以内的房源，其价格可比同区域远离地铁的房源高出10%至20%。

![image](https://github.com/liuil-o/6000-rental-data-analysis/assets/150576077/ec16a249-9db7-4f40-a383-7c638a5b6eb7)
