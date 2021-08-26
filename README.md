[English](./README_EN.md) | 简体中文

# AutoX是什么？
AutoX一个高效的自动化机器学习工具，它主要针对于表格类型的数据挖掘竞赛。
它的特点包括:
- 效果出色: AutoX在多个kaggle数据集上，效果显著优于其他解决方案(见[效果对比](#效果对比))。
- 简单易用: AutoX的接口和sklearn类似，方便上手使用。
- 通用: 适用于分类和回归问题。
- 自动化: 无需人工干预，全自动的数据清洗、特征工程、模型调参等步骤。
- 灵活性: 各组件解耦合，能单独使用，对于自动机器学习效果不满意的地方，可以结合专家知识，AutoX提供灵活的接口。
- 比赛上分点总结：整理并公开历史比赛的上分点。

# 目录
<!-- TOC -->

- [AutoX是什么？](#AutoX是什么？)
- [目录](#目录)
- [安装](#安装)
- [架构](#架构)
- [快速上手](#快速上手)
- [比赛上分点总结](#比赛上分点总结)
- [效果对比](#效果对比)

<!-- /TOC -->
# 安装
```
1. git clone https://github.com/4paradigm/autox.git
2. cd autox
3. python setup.py install
```

# 架构
```
├── autox
│   ├── ensemble
│   ├── feature_engineer
│   ├── feature_selection
│   ├── file_io
│   ├── join_tables
│   ├── metrics
│   ├── models
│   ├── process_data
│   └── util.py
│   ├── CONST.py
│   ├── autox.py
├── run_oneclick.py
└── demo
└── test
├── setup.py
├── README.md
```

# 快速上手
- 全自动: 适合于想要快速获得一个不错结果的用户。只需要配置最少的数据信息，就能完成机器学习全流程的构建。
```
from autox import AutoX
path = data_dir
autox = AutoX(target = 'loss', train_name = 'train.csv', test_name = 'test.csv', 
               id = ['id'], path = path)
sub = autox.get_submit()
sub.to_csv("submission.csv", index = False)
```
- 半自动: run_demo.ipynb
```
适合于想要获得更优预测结果的用户。AutoX提供了易用且丰富的接口，用户可以方便地根据实际数据场景进行配置，以获得更优的预测结果。
```

# 效果对比：
| index |data_type | data_name(link)     | AutoX         | AutoGluon   | H2o |
| ----- |----- | ------------- | ---------------- | ----------------|----------------|
| 1    |regression | [zhidemai](https://www.automl.ai/competitions/19)   | 1.1231 | 1.9466 | 1.1927|
| 2    |regression | [Tabular Playground Series - Aug 2021](https://www.kaggle.com/c/tabular-playground-series-aug-2021)   | 7.87731 | 10.3944 | 7.8895|
| 3    |binary classification | [Titanic](https://www.kaggle.com/c/titanic/)   | x | 0.78229 | 0.79186 |


# 数据类型
- cat: Categorical，类别型无序变量
- ord: Ordinal，类别型有序变量
- num: Numeric，连续型变量
- datetime: Datetime型时间变量
- timestamp: imestamp型时间变量

# pipeline的逻辑
- 1.初始化AutoX类
```
1.1 读数据
1.2 合并train和test
1.3 识别数据表中列的类型
1.4 数据预处理
```
- 2.特征工程
```
特征工程包含单表特征和多表特征。
每一个特征工程类都包含以下功能：
    一、自动筛选要执行当前操作的特征；
    二、查看筛选出来的特征
    三、修改要执行当前操作的特征
    四、执行特征数据的计算，返回和主表样本条数以及顺序一致的特征
```
- 3.特征合并
```
将构造出来的特征进行合并，行数不变，列数增加，返回大的宽表
```
- 4.训练集和测试集的划分
```
将宽表划分成训练集和测试集
```
- 5.特征过滤
```
通过train和test的特征列数据分布情况，对构造出来的特征进行过滤，避免过拟合
```
- 6.模型训练
```
利用过滤后的宽表特征对模型进行训练
模型类提供功能包括：
    一、查看模型默认参数；
    二、模型训练；
    三、模型调参；
    四、查看模型对应的特征重要性；
    五、模型预测
```
- 7.模型预测

# AutoX类
```
AutoX类自动为用户管理数据集和数据集信息。
初始化AutoX类之后会执行以下操作：
一、读数据；
二、合并train和test；
三、识别数据表中列的类型；
四、数据预处理。
```
## 属性
###  info_: info_属性用于保存数据集的信息。
- info_['id']: List，用于标识样本的唯一Key
- info_['target']: String，用于标识数据表的标签列
- info_['shape_of_train']: Int，train数据集的数据样本条数
- info_['shape_of_test']: Int，test数据集的数据样本条数
- info_['feature_type']: Dict of Dict，标识数据表中特征列的数据类型
- info_['train_name']: String，用于训练集主表表名
- info_['test_name']: String，用于测试集主表表名

### dfs_: dfs_属性用于保存所有的DataFrame，包含原始表数据和构造的表数据。
- dfs_['train_test']: train数据和test数据合并后的数据
- dfs_['FE_feature_name']:特征工程所构造出的数据，例如FE_count，FE_groupby
- dfs_['FE_all']:原始特征和所有特征工程合并后的数据集

## 方法
- concat_train_test: 将训练集和测试集拼接起来，一般在读取数据之后执行
- split_train_test: 将训练集和测试集分开，一般在完成特征工程之后执行
- get_submit: 获得预测结果(中间过程执行了完成的机器学习pipeline，包括数据预处理，特征工程，模型训练，模型调参，模型融合，模型预测等)

# AutoX的pipeline中的操作对应的具体细节：

## 读数据
```
读取给定路径下的所有文件。默认情况下，会将训练集主表和测试集主表进行拼接，
再进行后续的数据预处理以及特征工程等操作，并在模型预测开始前，将训练集和测试进行拆分。
```

## 数据预处理
```
- 对时间列解析年, 月, 日, 时、星期几等信息
- 在每次训练前，会对输入到模型的数据删除无效(nunique为1)的特征
- 去除异常样本，去除label为nan的样本
```

## 特征工程
- 1-1拼表特征
```
```

- 1-M拼表特征
```
- time diff特征
- 聚合统计类特征
```

- count特征
```
对要操作的特征列，将全体数据集中，和当前样本特征属性一致的样本计数作为特征
```

- target encoding特征

- 统计类特征
```
使用两层for训练提取统计类特征。
第一层for循环遍历所有筛选出来的分组特征(group_col)，
第二层for循环遍历所有筛选出来的聚合特征(agg_col)，
在第二层for循环中，
若遇到类别型特征，计算的统计特征为nunique，
若遇到数值型特征，计算的统计特征包括[median, std, sum, max, min, mean].
```

- shift特征
```
```

## 模型训练
```
AutoX目前支持以下模型：
1. Lightgbm
2. Xgboost
3. TabNet
```

## 模型融合
```
AutoX支持的模型融合方式包括一下两种，默认情况下，使用Bagging的方式进行融合。
1. Stacking；
2. Bagging。
```


# 比赛上分点总结：
|比赛|magics|
|------|------|
|kaggle criteo|对于nunique很大的特征列，进行分桶操作。例如，对于nunique大于10000的特征，做hash后截断保留4位，再进行label_encode。|
|zhidemai|article_id隐含了时间信息，增加article_id的排序特征。例如，groupby(['date'])['article_id'].rank()。|


## 错误排查
|错误信息|解决办法|
|------|------|

