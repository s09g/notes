# Sensor 

对于无人驾驶系统而言，多传感器已经是默认配置
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/1.jpeg)

### Kalman Filter 卡尔曼滤波
Kalman Filter 经常运用于无人驾驶系统中感知模块，用于目标状态估计。用人话说，就是物体追踪。<br>
简单来说，有个运动的小车，用来测量小车运动的传感器其实有测量噪声 (Measurement Noise)，所以得到的结果是个高斯分布
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/2.jpeg)
如果我们用带误差的测量值来预测下一时刻的位置，由于加入了速度估计噪声，所以不确定性更大了
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/3.jpeg)
于是用传感器再做一次测量，新的测量依然带有误差(还是个高斯分布)
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/4.jpeg)
将得到的两个高斯分布加权取平均，得到新的高斯分布(绿)
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/5.jpeg)
这步操作中使用到的加权数值叫做**卡尔曼增益**，决定了我们对当前测量的信任程度。新得到的绿色高斯分布，拥有比前两次测量值更小的方差。说明**卡尔曼滤波从两个不确定较高的分布，得到了一个相对确定的分布**。并且新的高斯分布可以作为下次预测的初始值(卡尔曼滤波假设本次测量只和上次测量有关)，所以卡尔曼滤波可以迭代。

[小车插图图源《Understanding the Basis of the Kalman Filter Via a Simple and Intuitive Derivation》](https://www.cl.cam.ac.uk/~rmf25/papers/Understanding%20the%20Basis%20of%20the%20Kalman%20Filter.pdf)

### 多传感融合 Lidar and Radar Fusion

一个简单的感知反馈模型其实只有两步：**状态预测**与**测量更新**
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/6.jpeg)
在多传感器条件下，各传感器之间想要同步反馈速度其实并无必要。每个传感器异步地参与感知反馈
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/7.jpeg)
在任何时间，只要收到传感器的数据，就会触发一次测量更新
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/8.jpeg)
这个过程中，会一直使用KF预测和KF更新
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/9.jpeg)

### 状态预测 State Prediction
线性模型假设，物体在运动时，每段时间间隔中速度恒定。实际上，每次测量时间之间的间隔是不定的，物体的加速也是不定的
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/10.jpeg)
时间和加速度的不确定性决定了**过程噪声`process Noise`**
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/11.jpeg)

下图引入了状态转移矩阵
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/12.jpeg)

#### 过程协方差矩阵 Process Covariance Matrix

由于状态向量只包含位置和速度信息，实际上加速度在模型中是作为随机噪声的
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/13.jpeg)
由于加速度不确定，所以直接当成随机成分。然后对上面的式子求导，就得到下面的随机加速向量v
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/14.jpeg)
v是服从于N(0, Q)分布
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/15.jpeg)

再把v分解成两个矩阵。一个4x2的矩阵G，其中不包含随机变量。一个2x1的矩阵a，包含随机加速项。
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/16.jpeg)
根据定义，协方差矩阵Q又是v乘上v的转置的数学期望。由于G不包含随机项，所以移到了数学期望的外面
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/17.jpeg)
剩下的就是ax的方差，ay的方差，ax和ay的协方差。由于ax和ay不相关，所以协方差是0. 前前后后放到一起，就是下面这玩意儿
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/18.jpeg)

### 激光检测 Laser Measurement
使用激光传感器，获取点云数据，探测物体。利用卡尔曼滤波进行转台预测
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/19.jpeg)


预测效果如下
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/20.jpeg)
选取近似直线部分放大，发现预测跟实际物体运行高度吻合
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/21.jpeg)
选取转向部分放大，预测偏离实际运行轨迹。因为在每个小的时间间隔中，我们一直假设方向不变
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/22.jpeg)

### 雷达检测 Radar Measurement
激光可以获得车辆的位置信息，要完成传感融合还需要从雷达获取速度信息
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/23.jpeg)

雷达信息包含了三个变量：`Range`范围(与路人的距离); `Bearing`方位(从x轴开始逆时针转向路人方向的角度); `Radial Velocity`是车速在行人方向上的速度分量，也叫`range rate`
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/24.jpeg)

测量函数`h(x')`会将测量空间投射到预测空间
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/25.jpeg)

### Extended Kalman Filter 扩展卡尔曼滤波

**EKF(Extended Kalman Filter）**是卡尔曼滤波的**非线性版本**。

![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/26.jpeg)
原本的卡尔曼滤波假设被`Prediction(Estimate)`服从高斯分布，且`Measurement(Noisy)`也服从高斯分布。但是现实状态中，基本都是非线性变换（简单说就是因为现实中被测物体多半处于受力状态，比如摩擦力）。

![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/27.jpeg)

所以EKF利用**一阶泰勒展开**，用局部的线性系统接近整个非线性系统。
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/28.jpeg)


新的结果依旧服从高斯分布

![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/29.jpeg)

#### 雅可比矩阵 Jacobian Matrix 
做泰勒展开的时候，需要对`x`求偏导，得到雅可比矩阵。
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/30.jpeg)
而`x`是由4部分组成的
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/31.jpeg)
最后会得到一个3X4的矩阵
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/32.jpeg)


#### EKF 算法总结
![](https://github.com/s09g/notes/raw/master/self-driving/4.%E4%BC%A0%E6%84%9F%E5%99%A8/assets/33.jpeg)