## Kaggle 准备

1.安装`Anaconda`

安装没什么好说的。

就是一点小问题。我用的`shell`是`zsh`，安装完之后不能在`terminal`使用`conda`命令。
因为默认`conda`会把自己的加载路径写进`~/.bashrc`或者`~/.bash_profile`。这里需要手动复制粘贴到`~/.zshrc` *(我寻思`fish`也会有这个问题)*

另外`conda`会自动启动`base`环境，这个有点不好了。因为我会有多个project同时在开发，依赖不同的环境。所以可以用下面这条关闭。
```bash
conda config --set auto_activate_base false
```

**修改默认配置**

使用下面这条命令，生成一个配置文件
```bash
jupyter notebook --generate-config
```
mac下，配置文件的路径为`~/.jupyter/jupyter_notebook_config.py`

公司电脑上有权限管理，所以我需要在服务器上安装`jupyter`再通过`http`登录。那么修改`jupyter_notebook_config.py`文件.

首先允许所有IP访问`jupyter server`, 默认只允许`localhost`访问
```python
c.NotebookApp.ip = '*'
```

对于`5.3`之后的`jupyter notebook`，这时候打开会要求输入密码
![](https://github.com/s09g/notes/raw/master/kaggle/1.Kaggle%E5%87%86%E5%A4%87/assets/1.png)

使用下面这条命令，配置密码
```bash
jupyter notebook password
```
设定好密码之后就可以登录了。

此外，`jupyter server`还允许配置`SSL/HTTPS`，[相关文档参考此处](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html)。

2.注册Kaggle & 下载数据集

#### 思路

>1.这是一个什么类型的问题？<br>
>以`house price`为例，是靠回归做预测<br>
>2.哪些算法可以做回归<br>
>线性回归等<br>
>3.线性回归需要什么样的数据<br>
>4.数据中是否有字符串，或者缺失值？如何变为数值型？<br>
>5.数据特征工程思路：EDA、特征选择、特征组合、特征分割……<br>
>6.算法的选择

#### 数据清洗

>Data cleaning is the process of detecting and correcting (or removing) corrupt or inaccurate records from a record set, table, or database and refers to identifying incomplete, incorrect, inaccurate or irrelevant parts of the data and then replacing, modifying, or deleting the dirty or coarse data.

**方法**

1. 解决缺失值：平均值、最大值、最小值或者概率估计
2. 去重：合并相同的记录
3. 解决错误值：
    + 用统计方法识别可能的错误值或异常值
    + 用简单的规则库检查数据值
    + 使用不同属性间的约束、外部的数据清理数据
4. 解决数据的不一致性：类别型、次序型数据

**场景**

1. 删除多列
2. 更改数据类型
3. 将分类变量变为数字变量
4. 检查缺失值
5. 删除字符串
6. 删除空格
7. 字符串连接两列
8. 转换时间戳
