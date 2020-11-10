

# xlearn


## 数据格式

### libffm

```
label field1:feature1:value1 field2:feature2:value2

0 0:0:1 1:1:1
1 0:2:1 1:3:1
```
#### 说明
* label   标签
* field   特征字段
* feature 特征分类值
* value   分类特征one-hot后的值，数字特征的值

#### 转换过程

##### 数据
* fields  广告投放者，广告发行者
* feature 广告投放者-Nike, 广告投放者-ESPN，广告发行者-CNN，广告发行者-BBC

##### 步骤
1. 使用一个字典，将所有的fields映射成编号
2. 使用一个字典，将所有features映射成编号


## 使用

### 接口
* create_xxx() 创建模型
* setTrain()  传入训练数据
* fit(param, model_path) 训练，param 超参数，model_path 模型输出文件
* setValidate() 传入验证数据
* cv(param)     交叉验证
* setTest()   传入测试数据
* predict(model_path) 预测 model_path 模型文件
* setSign()   转换预测值为0、1
* setSigmoid()  转换预测值为 0--1 之间
* disableNorm()  关闭特征归一化操作

### 超参数
* task 任务类型：binary 二分类，reg 回归
* metric 评价方法：分类 auc, f1, prec, recall 回归 mae, rmse
* lr   学习率，模型迭代步长
* lambda  正则化率
* k    FM,FFM模型隐向量的长度
* epoch  遍历训练数据集的次数
* fold  交叉验证划分数据的份数
* opt  优化算法：sgd, adagrad, ftrl

       