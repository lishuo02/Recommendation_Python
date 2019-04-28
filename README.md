# --python
基于无人售货机商品售卖情况推荐商品。

# 提供推荐

## 协作型过滤算法：对一大群人进行搜索，找出品味相近的一小群人。


1.搜集偏好。每位影评人对给定影片的喜好程度

2.寻找相近的用户。计算相似度评价值，欧几里得距离，皮尔逊相关度。

3.为评论者打分。找出最具有相似度的列表。

4.推荐物品。把用户之间的相似度最为加权值，为影片打分，推测得到用户没有看过影片的评分。

5.匹配商品。影评人可以和影片对换，为影片推荐人。

## 我们能做什么

为用户推荐商品

准备数据集，用户在一个月之内购买各种商品的数量

问题1：是否基于用户推荐，如果客户对象是运营商，基于用户推荐没有必要。

问题2：能否基于售货机推荐，为售货机推荐商品，同一个运营商区域内售货机相似度会很高，具体看结果。

目标改为：基于商品销售情况为售货机推荐商品。重点关注问题，同一运营商售货机可能会有很高相似度，是否对结果准确性造成影响。

准备数据集：售货机一个月内，售出各种商品数量。

## 开始进行推荐算法建模

1.准备数据

数据来源于开发数据库data_platform.svm_goods_stats_all

数据库配置


```python
import pymysql as MySQLdb
import pandas as pd
host=""
user=""
passwd=""
port=
db=""
```

打开数据库连接,关闭数据库链接函数,读取数据库函数


```python
def connect_sql():
    conn = MySQLdb.connect(host=host,port=port,user=user,passwd=passwd,db=db, charset='utf8')
    cur = conn.cursor()
    return conn,cur

def close_connect(conn,cur):
    conn.commit()
    cur.close()
    conn.close()

def select_sql(cur,sql):
    try:
        cur.execute(sql)
        alldata = cur.fetchall()
        frame = pd.DataFrame(list(alldata))
    except:
        frame = pd.DataFrame()
    return frame
```

读取数据,统计出2019年6月份，各个售货机每种商品的销量。


```python
conn,cur = connect_sql()
sql = "SELECT svm_id,svm_name,goods_gid,goods_name,SUM(sale_amount) as sale_amount FROM `svm_goods_stats_all` where count_month=201804 GROUP BY svm_id,svm_name,goods_gid,goods_name;"
result = select_sql(cur,sql)
print(result[0:10])
```

           0                1                                 2                3  \
    0  23561  微软大厦RE&F部门T1-8F  05CDE74A42B811E8B6E80017FA00B1BA     回头客枣泥蛋糕（两只装）   
    1  23561  微软大厦RE&F部门T1-8F  066D2F8EAF2411E798CE0017FA00E5A4              便利贴   
    2  23561  微软大厦RE&F部门T1-8F  070CF3DAC8E011E784ED0017FA00E5A4    统一果汁（250ML/盒）   
    3  23561  微软大厦RE&F部门T1-8F  10F6D1C0AF2411E798CE0017FA00E5A4              报事贴   
    4  23561  微软大厦RE&F部门T1-8F  1C5DB73FAF2111E798CE0017FA00E5A4            统一冰红茶   
    5  23561  微软大厦RE&F部门T1-8F  24146008AD8B11E79F3C0017FA00E9CA      零度可乐330ml/罐   
    6  23561  微软大厦RE&F部门T1-8F  27360F02AE4411E79F3C0017FA00E9CA        统一鲜橙多(盒装)   
    7  23561  微软大厦RE&F部门T1-8F  28DA970CAF2411E798CE0017FA00E5A4            彩色工字钉   
    8  23561  微软大厦RE&F部门T1-8F  35B52D06AE4311E79F3C0017FA00E9CA  三元纯牛奶(百利包)200ml   
    9  23561  微软大厦RE&F部门T1-8F  3A36BCD8AF2411E798CE0017FA00E5A4            套塑回形针   
    
        4  
    0  27  
    1   1  
    2  11  
    3   5  
    4  11  
    5  65  
    6  23  
    7   1  
    8  91  
    9  12  
    

添加列名


```python
result.columns = ["svm_id","svm_name","goods_gid","goods_name","sale_amount"]
print(result[0:10])
```

       svm_id         svm_name                         goods_gid       goods_name  \
    0   23561  微软大厦RE&F部门T1-8F  05CDE74A42B811E8B6E80017FA00B1BA     回头客枣泥蛋糕（两只装）   
    1   23561  微软大厦RE&F部门T1-8F  066D2F8EAF2411E798CE0017FA00E5A4              便利贴   
    2   23561  微软大厦RE&F部门T1-8F  070CF3DAC8E011E784ED0017FA00E5A4    统一果汁（250ML/盒）   
    3   23561  微软大厦RE&F部门T1-8F  10F6D1C0AF2411E798CE0017FA00E5A4              报事贴   
    4   23561  微软大厦RE&F部门T1-8F  1C5DB73FAF2111E798CE0017FA00E5A4            统一冰红茶   
    5   23561  微软大厦RE&F部门T1-8F  24146008AD8B11E79F3C0017FA00E9CA      零度可乐330ml/罐   
    6   23561  微软大厦RE&F部门T1-8F  27360F02AE4411E79F3C0017FA00E9CA        统一鲜橙多(盒装)   
    7   23561  微软大厦RE&F部门T1-8F  28DA970CAF2411E798CE0017FA00E5A4            彩色工字钉   
    8   23561  微软大厦RE&F部门T1-8F  35B52D06AE4311E79F3C0017FA00E9CA  三元纯牛奶(百利包)200ml   
    9   23561  微软大厦RE&F部门T1-8F  3A36BCD8AF2411E798CE0017FA00E5A4            套塑回形针   
    
      sale_amount  
    0          27  
    1           1  
    2          11  
    3           5  
    4          11  
    5          65  
    6          23  
    7           1  
    8          91  
    9          12  
    

转换成字典,格式为：{svm_id:{goods_name:sale_amount}}


```python
dict={}
for i in range(len(result)):
    dict_row={}
    row=result.loc[i]
    svm_id=row["svm_id"]
    goods_name=row["goods_name"]
    sale_amount=row["sale_amount"]
    dict2=dict.setdefault(svm_id,{})
    dict2.setdefault(goods_name,sale_amount)
#print(dict)
```

计算两个售货机皮尔逊相关系数


```python
from math import sqrt
import decimal
def sim_svm(dict,svm1,svm2):
    #得到两个售货机都售卖过商品列表
    si={}
    for item in dict[svm1]:
        if item in dict[svm2]:
            si[item]=1
            
    #获得列表元素的个数
    n=len(si)
    
    #如果两者共同之处<5，返回0
    if n<5 or n>20:
        return 0
    
    #对所有偏好求和
    sum1=sum([dict[svm1][it] for it in si])
    sum2=sum([dict[svm2][it] for it in si])
    
    #求平方和
    sum1Sq=sum([pow(dict[svm1][it],2) for it in si])
    sum2Sq=sum([pow(dict[svm2][it],2) for it in si])
    
    #求乘积之和
    pSum=sum([dict[svm1][it]*dict[svm2][it] for it in si])
    
    #计算皮尔逊评价值
    num=pSum-(sum1*sum2/n)
    den=sqrt((sum1Sq-pow(sum1,2)/n)*(sum2Sq-pow(sum2,2)/n))
    if den==0:
        return 0
    num=decimal.Decimal(num)
    den=decimal.Decimal(den)
    r=num/den
    
    return r
```

例如，计算北京腾讯众创空间4层G3-2：23804 和 北京腾讯众创空间B座3层G3-2：23805 皮尔逊相关系数，结果为0.6747359218147152627033075838


```python
sim_svm(dict,23804,23807)
```




    0



接下来计算一台售货机，与他最接近的匹配结果


```python
#返回结果个数为参数
def topMathes(dict,svm,n=5,similarity=sim_svm):
    scores=[(similarity(dict,svm,other),other) for other in dict if other!=svm]

#对列表进行排序，相似度最高的排在最前面
    scores.sort()
    scores.reverse()

    return scores[0:n]
```

以23804 北京腾讯众创空间4层G3-2 为例，相似度前10名为：  
24006	深圳宝安百旺创意工厂7栋G5-2  
24052	东莞理工学院行政楼G6-1  
24100	深圳南山区科兴科学园C1栋14楼腾讯茶水间G3-1  
24483	广州中大南方学院西学楼2号G5-1（男）  
24495	广州中大南方学院西学楼11号G6-1（女）  
24508	广州中大南方学院东学楼33号G6-1（女）  
24513	广州中大南方学院东学楼23栋G6-1（女）  
24601	广州TIT健身房门口G2  
24603	广州TIT B4栋拐角G2  
25655	北京创客广场A座3层P5  
可以发现，受同一经营商影响不大，具备参考价值。


```python
topMathes(dict,24147,n=10)
```




    [(Decimal('0.9823212393018347222349147787'), 24136),
     (Decimal('0.9719508383747784295317217650'), 24638),
     (Decimal('0.9656766675443123709255253711'), 24557),
     (Decimal('0.9618078821558180077873342986'), 24137),
     (Decimal('0.9521073218544146589857017012'), 24097),
     (Decimal('0.9294906056457545187788558502'), 24060),
     (Decimal('0.9255077576001855257316520260'), 24514),
     (Decimal('0.8984729592094900609570536544'), 24194),
     (Decimal('0.8924104525393517299226061508'), 25304),
     (Decimal('0.8839387698964820572321233092'), 24200)]



推荐商品

方法：得到售货机的相似度后，乘以商品的销量，相似度高的售货机将对整体推荐提供更大的影响。求和后除以全部相似度之和，减少因为商品在很多售货机售卖造成的影响。


```python
def getRecommendations(dict,svm,n=5,similarity=sim_svm):
    totals={}
    simSums={}
    for other in dict:
        #不要和本机作比较
        if other==svm:
            continue
        sim=similarity(dict,svm,other)
        
        #忽略相似度为0或者小于0的情况
        if sim<=0:
            continue
        
        for item in dict[other]:
            #只对没有出售的商品统计
            if item not in dict[svm]:
                #相似度*销量
                totals.setdefault(item,0)
                totals[item]+=dict[other][item]*sim
                #相似度之和
                simSums.setdefault(item,0)
                simSums[item]+=sim
                
    #建立一个归一化的列表
    rankings=[(total/simSums[item],item) for item,total in totals.items()]
    
    #返回经过排序的列表
    rankings.sort()
    rankings.reverse()
    
    return rankings[0:n]
```

以23804 北京腾讯众创空间4层G3-2 为例，推荐商品如下：


```python
getRecommendations(dict,24147,n=10)
```




    [(Decimal('265.0525274870319393328064328'), '达利园青梅绿茶'),
     (Decimal('157.5100028338143193087411658'), '农夫山泉（550ml瓶）'),
     (Decimal('107.8895927165601255672163058'), '味全乳酸菌饮品 草莓味 (450ml瓶)'),
     (Decimal('107.1198200283158929032631089'), '百事可乐(600ml/瓶)'),
     (Decimal('106.0000000000000000000000000'), '康师傅香辣牛肉面 (108g杯)'),
     (Decimal('92.83801177224463229020539481'), '冰露纯净水（550ml瓶)'),
     (Decimal('90.69816481581427204727284670'), '盼盼肉松饼(30g/袋)'),
     (Decimal('80.59939128255216851631953250'), '景田纯净水(560ml/罐)'),
     (Decimal('80.28597990368058623276120420'), '麒麟午后奶茶原味(500ml/瓶)'),
     (Decimal('75.99999999999999999999999999'), '黄鹤楼（软蓝）')]



## 调参过程

调参对象：计算售货机皮尔逊相关系数，售货机拥有共同商品数量n值,大于n值才计算相关系数。由于每月每台售货机商品种类较多，有几十种： 

当n过小时，如n=0,只要有一种商品共同售卖，则计算两台售货机相关系数过大，由于共同商品数量占比较小，不能反映两台售货机正常的相关系数。  

当n过大时，如n>10,数据显示相关度高的大部分为同一运营商下的售货机，售货机由同一家运营商运营，商品种类是很接近的，造成相关度高，不具有推荐价值。  

以下取n=5,10,8 svm_id为23804的售货机展示

2018.04 n>5  
(Decimal('0.9786200730071457015837720227'), 24077),  
 (Decimal('0.9785952494059722601910034694'), 24251),  
 (Decimal('0.9718629832926680068337899408'), 24241),  
 (Decimal('0.9690117014397064027364879548'), 24528),    
 (Decimal('0.9100647553336100121272323189'), 24253),  
 (Decimal('0.9038551160925817906111451915'), 24016),  
 (Decimal('0.9015406198643173343230210883'), 24053),  
 (Decimal('0.8988979765004685420165975777'), 24070),  
 (Decimal('0.8940838255256492836261510545'), 24041),  
 (Decimal('0.8840579125201337339223390679'), 25279) 
 

24016	深圳龙岗坪山主力宿舍B栋G5-1  
24041	深圳南方科技大学教师公寓6栋G3-1  
24053	东莞理工学院综合馆G6-1  
24070	深圳罗湖区深圳体育中心攀岩馆G3-1  
24077	深圳硅谷动力智能终端园区A4-B座G3-1  
24241	深圳市南山区西丽阳光工业园燕麦科技一楼G3-1  
24251	深圳顺荣公司19楼G3-1  
24253	深圳南山区百旺创意工厂7栋热饭区G3-1  
24528	深圳南山区卓越·后海中心13楼G5-1  
25279	广州海珠区逸景路海珠法院G/FG2  

2018.04 n>10  
(Decimal('0.8613278321515042256076360978'), 24107),  
 (Decimal('0.8047128623318818263408620095'), 23833),  
 (Decimal('0.7993363061220379677912798578'), 24611),  
 (Decimal('0.7853217985017786274568049621'), 23804),  
 (Decimal('0.7716792351064601225337047706'), 24566),  
 (Decimal('0.6408978040555316821860744214'), 23807),  
 (Decimal('0.5719628185572587922591246519'), 25298),  
 (Decimal('0.5669038392651221983059927370'), 24609),  
 (Decimal('0.4178989436433680834095254679'), 24582),  
 (Decimal('0.4132137518364929407830469620'), 24587)  
 
 23804	北京腾讯众创空间4层G3-2  
23807	北京腾讯众创空间A座5层G3-1  
23807	北京腾讯众创空间A座5层G3-2  
23833	北京腾讯众创空间A座2层G5-2  
24107	北京腾讯众创空间A座1层G5-1  
24566	广州肿瘤医院2号楼1楼G6-1  
24582	广州岭南电商产业园3楼G5-1  
24587	广州白云机场P2停车场三号G5-1  
24609	广州白云机场奥斯特酒店G6-1  
24611	广州白云机场P3停车场一楼G5-1  
25298	北京创客广场A座3层G5  

2018.04 n>8  

(Decimal('0.8613278321515042256076360978'), 24107),  
 (Decimal('0.8547413911344391514801827327'), 24714),  
 (Decimal('0.8047128623318818263408620095'), 23833),  
 (Decimal('0.7993363061220379677912798578'), 24611),  
 (Decimal('0.7853217985017786274568049621'), 23804),  
 (Decimal('0.7806301788636706538993692880'), 24102),  
 (Decimal('0.7716792351064601225337047706'), 24566),  
 (Decimal('0.7664002999952736182570821024'), 24606),  
 (Decimal('0.7571419380607181772692088246'), 24078),  
 (Decimal('0.6732402070760885938576218374'), 24090)  
 
 23804	北京腾讯众创空间4层G3-2  
23833	北京腾讯众创空间A座2层G5-2  
24078	深圳南方科技大学学生宿舍4栋G5-1  
24090	深圳南方科技大学教学楼28栋G3-1  
24102	深圳南山区科兴科学园C2栋12楼腾讯茶水间G3-1  
24107	北京腾讯众创空间A座1层G5-1  
24566	广州肿瘤医院2号楼1楼G6-1  
24606	广州蓝谷创意园g6-1  
24611	广州白云机场P3停车场一楼G5-1  
24714	广州全民时尚天河G6-1  


```python

```
