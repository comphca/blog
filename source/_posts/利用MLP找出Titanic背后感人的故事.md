---
title: 利用MLP找出Titanic背后感人的故事
date: 2018-06-05 13:04:45
tags: Keras
---

数据科学家在处理数据的时候，常常只会看到冷冰冰的数据，却忘记了这些数据背后的故事，每一项数据都是由生物的活动产生的。这里我使用mlp对一部比较经典的电影进行书数据背后的分析。

具体代码详见：[github](https://github.com/comphca/Basic-AI/blob/master/titanic/Multilay_titanic.py)

具体步骤

在网络上找到1309项数据，经过数据处理之后会产生features（一共有9个字段），与label标签字段（是否生存？0：否，1：是），然后输入多层感知器模型进行训练。
```
filepath = "./data/titanic3.xls"
all_df = pd.read_excel(filepath)
```

在原始数据集里面有很多我们不需要的features，所以要将其剔除掉。
```
#选取我们感兴趣的features
cols = ['survived','name','pclass','sex','age','sibsp','parch','fare','embarked']
all_df = all_df[cols]
```

在原始数据里面我们还发现很多字段的值是null，还有name属性在训练的阶段时不需要的，在预测阶段才会用到。还有性别字段和登船入口字段需要解决
```
df = all_df.drop(['name'],axis=1)
print(all_df.isnull().sum())

#把发现的空值数据进行处理，这里选择取字段平均值
age_mean = df['age'].mean()
df['age'] = df['age'].fillna(age_mean)

fare_mean = df['fare'].mean()
df['fare'] = df['fare'].fillna(fare_mean)

#性别字段转换为数字
df['sex'] = df['sex'].map({'female':0,'male':1}).astype(int)

#embarked字段有三个值分别的C、Q、S，要进行一位有效编码解决
x_OneHot_df = pd.get_dummies(data=df,columns=['embarked'])
```

到这里，数据还是以excel形式存储的，要想进行模型训练，必须要将数据转换为Array
```
#ndarray.shape=(1309,10)  共1309项数据，每项10个features   10个features值对应处理之后excel的字段，第0项时label，其他是features
ndarray = x_OneHot_df.values
```

使用slice分割label和features
```
#冒号提取所有项数，第0项是label
Label = ndarray[:,0]
Features = ndarray[:,1:]
```

在Features里面可以看到数值差值比较大，不利于收敛，在sklearn模块里面提供了preprocessing
```
minmax_scale = preprocessing.MinMaxScaler(feature_range=(0,1))
#.fit_transform函数传入要标准化的数据集  进行标准化
scaledFeature = minmax_scale.fit_transform(Features)
```

到这里，数据的预处理就已经完成了，接下来就需要把整个数据进行分割，一部分作为训练数据，一部分进行用于测试
```
msk = numpy.random.rand(len(all_df)) < 0.8
train_df = all_df[msk]
test_df = all_df[~msk]
```

训练模型
```
#两个隐藏层
model.add(Dense(units=240,
                input_dim=9,
                kernel_initializer='uniform',
                activation='relu'))

model.add(Dense(units=30,
                kernel_initializer='uniform',
                activation='relu'))

#输出层
model.add(Dense(units=1,
                kernel_initializer='uniform',
                activation='sigmoid'))

model.compile(loss='binary_crossentropy',
              optimizer='adam',metrics=['accuracy'])

train_history = model.fit(x = train_df,
                          y = label,validation_split=0.2,
                          epochs=30,batch_size=30,verbose=2)
```

模型训练完之后我们可以查看哪些游客预测的生存率高但是却没有活下来了，Pandas提供了很方便的方法
```
pd.[(pd['surivived'] == 0) & (pd['probabilityt'] > 0.9)]
```

