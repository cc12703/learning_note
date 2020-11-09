[TOC]

# sklearn


## 功能

* 分类
* 回归
* 聚类
* 数据降维
* 模型选择
* 数据预处理


## 选择
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109183403.png)



## 模型

* 转换器 Transformer
* 估计器 Estimator
* 管道 Pipeline

### 转换器
* 有输入，有输出
* 输出可以放入 转换器或估计器 中当输入
* fit接口用于计算
* transform接口用于输出

### 估计器
* 所有的机器学习算法模型，都被作为估计器
* fit接口用于训练
* predict接口用于预测

### 管道
* 将转换器和估计器组合起来
* 管道可以包含多个转换器
* 管道最后一个只能放估计器

#### 串行化模型
* 使用Pipeline类实现
* 继承最后一个类的所有方法
* 参数steps设置处理流程，是(name, processor)列表
* 函数set_params设置每个步骤的参数，格式：stepname__paramname=value

#### 并行化模型
* 使用FeatureUnion类实现
* 每个step分开计算，将计算结果合并


## 特征提取（feature_extraction）

### CountVectorizer （获取词频矩阵）

#### 输入
* 字符串列表
* 一个字符串代表一篇文章
* 字符串已经分好词

#### 输出
* fit_transform()  返回一个矩阵：行是文本，列是词，值为词频
* vocabulary_dict  返回一个map：键是词，值是词频


#### 参数
* encodeing 默认值utf-8
* stop_words 停用词，输入一个列表
* binary  将所有非零次数设置为1


### FeatureHasher （特征散列化）

#### 功能
* 将字符串序列转换为scipy的稀疏矩阵

#### 参数
* n_features 输出矩阵的特征数
* input_type 输入数据格式
* alternate_sign  是否给hash值交替添加一个符号


### TfidfVectorizer （文本TF-IDF）

#### 功能
* 将文本转化成TF-IDF的特征矩阵

#### 输入
* 字符串列表
* 一个字符串代表一篇文章
* 字符串已经分好词

#### 输出
* fit_transform()  返回一个稀疏矩阵，行是文本，列是词，值是TF-IDF分
* vocabulary 词汇表

#### 参数

##### token_pattern 
* 单词匹配模式。默认值匹配长度大于2的单词
* 中文时需要修改为 r"(?u)\b\w+\b" 可以匹配长度为1的单词

##### max_df/min_df
* 用于过滤掉无意义单词
* 值：0.0 -- 1.0
* 过滤出现在超过max_df, 低于min_df比例文本中的单词

##### stop_words
* 直接指定要过滤的单词
* 传入一个单词列表

##### ngram_range
* 设置用多少个单词进行组合
* 传入一个元组 （1，2）允许用1，2个单词进行组合

##### max_feature
* 设置使用单词的上限，模型会优先使用词频高的单词




## 特征选择（feature_selection）

### 假设检验

#### SelectKBest
* 根据统计指标，选择最佳的几个特征
* 统计指标：f_regression, chi2, f_classif

#### SelectPercentile
* 根据统计指标，选择最佳的百分之几的特征
* 统计指标：f_regression, chi2, f_classif

### 模型选择

#### SelectFromModel
* 根据重要性指标，选择高于阀值的特征
* 需要模型有feature_importances_属性(树模型)或者coef_属性(线性模型)


## 模型选择（model_selection）

### 函数
* cross_val_score 使用交叉验证来计算模型评分
* cross_validate  允许使用多个评估方法
* cross_val_predict 使用交叉验证获取模型输出值


### 参数

#### 公共
* estimator 估计器
* X 样本数据
* Y 目标数据

#### cv
* 交叉验证参数
* 默认使用5折验证
* 整数，为KFold的折数
* 函数，为分割器函数

#### scoring
* 模型的度量指标

##### 指标
* accuracy 分类准确度
* f1      分类F1值
* precision 分类精度度
* recall    分类召回值




## 数据降维

### LatentDirichletAllocation（LDA主题模型）

#### 输入
* 词频矩阵：行为文本，列为词
* 格式可以为 dataframe, 稀疏矩阵，数组

#### 输出
* fit_transform() 返回一个ndarray的数组，一项代表一个文章，内容为各个主题的概率

#### 参数

##### n_topics 
* 主题数K，需要调参

##### doc_topic_prior  
* 文档主题先验分布的参数
* 可以使用默认值 1/K

##### topic_word_prior 
* 主题词先验分布的参数
* 可以使用默认值 1/K

##### learning_method  
* 求解算法：batch, online
* batch 使用变分推断EM算法，用于样本量不大的时候
* online 使用了分布训练，用于样本太多太大的时候，需要调整的参数比较多

##### max_iter
* EM算法的最大迭代次数


#### 输出




## 数据预处理

### OneHotEncoder (one-hot编码)
* 用于将类别特征的每个元素转化成一个可以计算的值
* 传入的数据必须是数值格式的

### LabelEncoder
* 将类型变量转换为数值
* 值 0 --- class-num - 1

### LabelBinerizer
* 将标签转成成一对多的格式



## 线性模型（linear_model）


### 线性回归
* LinearRegression  普通的线性回归，无正则化处理
* Ridge，RidgeCV 加入L2正则化
* Lasso，LassoCV 加入L1正则化
* ElasticNet、ElasticNetCV 加入L1、L2混合正则化



### LogisticRegression (逻辑回归)

#### 功能
* 概率预测，分类


#### 参数

##### penalty 
* 用于选择正则化类型：l1, l2(默认)
* l2主要用于解决过拟合
* l2可选所有4种优化算法
* l1只能选 liblinear 算法

##### solver
* 用于设置对损失函数的优化方法：liblinear, lbfgs, newton-cg, sag
* liblinear 使用坐标轴下降法来求解，适用于小数据集
* lbfgs  使用二阶导数矩阵来求解
* newton-cg 使用二阶导数矩阵来求解
* sag    使用随机小批量梯度下降来求解


##### multi_class
* 用于设置多元分类的方式：ovr(默认), multinomial
* ovr 实现简单，效果相对略差，可以使用全部4种优化算法
* multinomial 实现复杂，效果相对精确，速度比较慢，只能使用 lbfgs/newton-cg/sag 算法


##### class_weight
* 解决分类类型不平衡问题
* 用于设置类型权重：balanced, 自定义值
* balanced  类库自动计算权重，样本量越多则权重越低
* 自定义   { 类型值 : 百分比， 类型值 : 百分比 }

##### sample_weight
* 解决分类类型不平衡问题
* 在fit函数中传入

##### max_iter
* 设置算法最大的迭代次数
* 适用于 lbfgs/newton-cg/sag 算法

##### random_state
* 设置随机数种子
* 适用于 sag, liblinear 算法

##### tol
* 设置算法迭代终止的误差范围
* 默认值：1e-4

##### n_jobs
* 设置CPU并行数： -1 和CPU核数一致，1（默认）  



#### 特殊情况

##### 误分类代价高
* 将非法用户分类成合法用户的代价很高
* 可以适当提高非法用户的权重


##### 样本高度失衡
* 非法用户样本很少很少
* 可以将class_weight设置为balanced，来提高非法用户样本的权重


#### 算法使用总结

##### l1 + liblinear
* 适用于小数据集
* 使用l2还是过拟合

##### l2 + liblinear
* 只支持OvR多元回归

##### l2 + lbfgs/newton-cg/sag
* 用于较大数据集
* 支持OvR，MvM多元回归

##### l2 + sag
* 用于超大数据集
* 无法使用l1正则化


## 树模型


### 决策树

#### 函数
* DecisionTreeClassifier 决策分类树
* DecisionTreeRegressor 决策回归树


#### 参数

##### criterion
* 特征选择方法
* 分类树：gini (基尼系数), entropy（信息增益）
* 回归树：mse（均方差）, mae（均值差的绝对值之和）


##### splitter
* 特征划分点的选择标准
* 值：best, random
* best：在所有划分点中找出最优点
* random：随机在部分划分点中找局部最优点


##### max_features
* 划分时考虑的最大特征数
* 值：none, log2, sqrt, auto，整数、浮点数
* none：考虑所有的特征数
* log2：最多考虑log2N个特征
* 整数：特征数的最大值
* 浮点数：特征数的百分比


##### max_depth
* 决策树的最大深度
* 如果样本多，特征多，可以设置该值（范围：10 -- 100）

##### max_leaf_nodes
* 最大叶子节点数，可以防止过拟合
* 值：none, 整数


## 近邻算法（neighbors）

### 函数
* KNeighborsClassifier KNN分类
* KNeighborsRegressor KNN回归
* RadiusNeighborsClassifier 限定半径的近邻分类
* RadiusNeighborsRegressor 限定半径的近邻回归


### 通用参数 

#### n_neighbors
* k值，用于KNN算法
* 通过交叉验证来选择

#### randius
* 半径值，用于限定半径算法
* 通过交叉验证来选择


#### weights
* 标识每个样本的权重
* 值 uniform（默认值）, distance，自定义
* uniform 表示所有样本的权重都一样
* distance 表示权重和距离成反比，距离更近，权重更高


#### algorithm
* 指定使用的实现算法
* 值 brute, kd_tree, ball_tree, auto
* brute 暴力实现
* kd_tree KD树实现。用于样本大，特征多
* ball_tree 球树实现。用于样本大，特征多
* auto 自动在上面三种中选择。用于样本少，特征少


#### metric
* 指定使用的距离度量算法
* 值 euclidean 欧式距离，manhattan 曼哈顿距离，minkowski 闵可夫斯基距离（默认）


#### leaf_size
* 停止建子树的叶子节点数量的阀值
* 值越小，生成的树就越大，层数越深
* 一般依赖于样本的数量，样本越多值越大



## 朴素贝叶斯（native_bayes）

### GaussianNB 

#### 概述
* 先验概率为高斯分布
* 适用于样本特征大部分是连续值的情况


#### 参数

##### priors 
* 先验概率，默认为样本中的比例值



### MultinomialNB 

#### 概述
* 先验概率为多项式分布
* 适用于样本特征大部分是多元离散值的情况

#### 参数

##### alpha 
* 多项式分布中的参数
* 默认为1，使用拉普拉斯平滑

##### fit_prior
* 使用考虑先验概率
* false 使用相同的概率 1/k
* true  算法自己计算、使用clas_prior参数的值



### BernoulliNB

#### 概述
* 先验概率为伯努利分布
* 适用于样本特征大部分是二元离散值、稀疏的多元离散值的情况


## 集成算法(ensemble)

### GradientBoostingClassifier （梯度提升分类树）

#### 特点
* 使用boosting集成方式
* 弱学习器使用 CART分类树，是决策树的增强版本，可以处理连续值

#### 调参
* 需要n_estimators 和 learning_rate 一起调

#### boosting参数

##### n_estimators
* 设置弱学习器的最大个数
* 太小会导致欠拟合，太大会导致过拟合

##### learning_rate
* 设置每个弱学习器的权重缩减系数（步长）

##### subsample
* 设置子采样率，值 0 -- 1
* 默认值为1，不使用子采样
* 推荐值 0.5 -- 0.8

##### loss
* 设置损失函数：deviance(默认), exponential
* deviance 对数似然损失函数，对二元、多元分类有较好的优化
* exponential 指数损失函数


#### 弱学习器参数


##### max_features
* 设置划分时考虑的最大特征数：none, sqrt, auto，log2
* none 默认值，考虑所有特征数


##### max_depth
* 设置决策树的最大深度
* 若样本多，特征也多，可以设置该值
* 推荐值：10 -- 100

##### max_leaf_nodes
* 设置最大叶子节点数： none, 整数
* none 默认值，不限制个数


### bagging算法

#### 函数
* BaggingClassifier bagging分类器


#### 参数
* base_estimator 基本模型,默认为决策树
* n_estimator  基本模型的数量

* max_samples 抽取用于训练每个基本模型的样本数，具体的个数、百分比，默认值 1.0
* max_features 抽象用于训练基本模型的特征数，具体的个数、百分比，默认值 1.0

* bootstrap 是否抽取样本进行替换，默认值 true

