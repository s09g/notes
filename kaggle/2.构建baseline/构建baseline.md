## 构建baseline

#### 数据探索

>思路：数据探索，做一点点的修改-->数据清洗(空值的填充)-->数据预处理（数据的归一化，标准化等）-->模型构建-->训练预测-->保存提交

```python
import pandas as pd
train = pd.read_csv("./train.csv")
train.head()
```

使用**pandas_profiling**完成**EDA`(explore data analysis)`**
```python
import pandas_profiling as ppf
ppf.ProfileReport(train) 
```
![](https://github.com/s09g/notes/raw/master/kaggle/2.%E6%9E%84%E5%BB%BAbaseline/assets/1.png)
![](https://github.com/s09g/notes/raw/master/kaggle/2.%E6%9E%84%E5%BB%BAbaseline/assets/2.png)

使用**箱型图**查看**异常值**
```python
plt.figure(figsize=(15,8))
sns.boxplot(train.YearBuilt, train.SalePrice)
```
![](https://github.com/s09g/notes/raw/master/kaggle/2.%E6%9E%84%E5%BB%BAbaseline/assets/3.png)

使用**散点图**用于观察是否存在**线性关系**
```python
plt.figure(figsize=(12,6))
plt.scatter(x=train.GrLivArea, y=train.SalePrice)
plt.xlabel("GrLivArea", fontsize=13)
plt.ylabel("SalePrice", fontsize=13)
# plt.ylim(0,800000)
```
![](https://github.com/s09g/notes/raw/master/kaggle/2.%E6%9E%84%E5%BB%BAbaseline/assets/4.png)
删除图中右下角有两个异常点
```python
outlier = train[(train.GrLivArea > 4000) & (train.SalePrice < 300000)]
train.drop(outlier.index, inplace=True)
```

#### 数据清洗

```python
miss = full.isnull().sum()#统计出空值的个数
miss[miss > 0].sort_values()
```

1. **缺失值/空值**的填充与删除
    + 对于字符
        + 用`None`填充
    + 数值：
        + 均值填充
        + 众数填充 `full["MSZoning"].mode()[0]`
    + *如何选择处理方式，选均值or众数？正确的方法应该都试一下，然后根据validation结果进行选择*
2. 数据预处理:字符变成数值型，归一化，标准化...
    + 使用`StandardScaler`、`Normalizer`、`MinMaxScaler`、`LabelBinarizer`、`LabelEncoder`、`OneHotEncoder`和`get_dummies`等
    ```python
    from sklearn.preprocessing import LabelEncoder
    pd.get_dummies(full)##独热编码
    ```
    + *我一般用`sklearn`的`LabelEncoder`和`OneHotEncoder`, `pd.get_dummies`还是倒是头一次遇上*

3.异常值的处理
+ 箱型图
+ 散点图

4.模型的构建

```python
from sklearn.linear_model import LinearRegression#导入模型
```

5.模型训练好了后就对test进行预测 & 提交结果

```python
result.to_csv("baseline.csv",index=False
```

直接使用`sklearn`的`LinearRegression`作为`baseline`, 排名应该在3k之后。所以接下来要开始做特征工程