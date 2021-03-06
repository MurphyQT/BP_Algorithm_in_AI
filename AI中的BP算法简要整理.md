# 论文算法中文整理



<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic1.jpg">

### 一、Batch Gradient Decent (BGD)

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic2.png">

η为学习率，后面乘以的项为Loss function J(θ;x;y)的梯度，负学习率乘以梯度得到参数的增量Δθ，更新参数θ = θ + Δθ。

值得注意的是和大家所熟悉的SGD算法不同，BGD为最原始的BP算法，一次输入整个数据集（dataset）计算梯度。

---

#### Pros:

- 能确保收敛

#### Cons:

- 计算所需内存和计算速度随着数据集的增大而增大
- model容易引起overfit的情况



### 二、Stochastic Gradient Decent (SGD)

公式同BGD，区别在于，SGD的梯度计算一次只用到了数据集的一部分（minibatch），用小部分数据集的梯度来近似整个数据集的梯度。

---

#### Pros:

- 计算所需内存和计算速度相比BGD有了很大的提升。

#### Cons:

- 当数据稀疏时，minibatch的梯度不能真实反映数据集梯度，梯度噪声较大，影响收敛。



### 三、Stochastic  Gradient Decent with Momemtum (SGDM)

刚说到SGD当数据稀疏时，容易遇到minibatch梯度不能有效近似真实梯度的情况。所以容易在模型训练过程中遇到梯度方向不稳定，”反复横跳“的现象：

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic3.png">

因此，为了防止这样的情况，SGDM对SGD进行了一些改进：

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic4.png">

设置了一个参数动量vt来记录之前t-1步迭代中梯度的加权和，然后把原来的公式：

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic2.png">

中的θ增量Δθ换成了- γ × v(t-1) - η × gradient。

也就是说每次的增量不单单包括学习率乘梯度，是动量和学习率梯度积的加权和，让增量Δθ拥有了“惯性”，不仅仅取决于当前的梯度大小方向，也考虑之前的梯度的大小方向。

当当前梯度方向与动量方向相同，那么能够加速向该方向迭代，反之，抑制当前梯度方向对模型迭代的影响。

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic5.png">

棕色线是当前两步迭代的学习率*梯度；红色线是历史累计动量。蓝色线是不加动量，两步迭代的结果；绿色线是加了动量之后的结果。可以看到动量对于梯度“横跳”的抑制效果。



### 四、AdaGrad & RMSProp

两种方法放一起是因为本质上做的优化都属于同一种方法：修改learning rate。

模型训练中，尤其到了接近收敛的关键时刻，大家都希望Loss function能尽快收敛，此时我们不希望这时有个过大的gradient直接导致模型参数偏离optimum，比较适合“小碎步”慢慢向optimum接近。

因此，如果我们能控制learning rate,在快要收敛的时候减小learning rate就好了。

#### 4.1 AdaGrad

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic6.png">

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic7.JPG">

 η是全局学习速率，实时学习速率等于η除以梯度的累计平方和的开方。这样可以保证学习速率是逐渐下降的。

---

#### Pros:

- 学习速率逐渐下降，可以确保模型在接近收敛时的稳定性

#### Cons:

- 在后期，学习速率会趋近于0，过早结束训练，陷入局部最优，尤其对于深度网络来说是致命的缺陷。

#### 4.3 RMSProp

接上文，AdaGrad会导致在深度网络中，学习率到后期几乎为0，模型无法正常迭代的情况。RMSProp对AdaGrad做出了小小的改进：

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic8.JPG">

γ是衰减系数，因此Gt并不会随t单调下降，之前的gt对当前学习率的影响会逐渐下降，同时Gt是增加还是减小取决于被γ衰减的多还是被（1-γ）gt * gt增加的多。

---

#### Pros: 

- 一定程度上缓解了AdaGrad学习速率到后期几乎为0的问题
- 一定程度上能限制训练后期出现过大梯度的问题

#### Cons:

- 模型训练后期，△θ过大或者过小的问题没有得到根本的解决，仍会发生



### 五、 集大成者：Adam

Adam目前是CNN中较为主流的Optimizor。它采用了之前用到的对SGD的大部分优化算法。

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic9.png">

### 六、 Conclusions of Adaptive Methods

---

#### Pros:

- 训练速度更快
- 更平滑过渡的学习速率
- 超参数易于选取
- 当数据稀疏时表现更好

#### Cons:

- 测试集表现普遍低于SGD
- 非减的学习率带来的收敛问题
- **会导致过大和过小的学习率，影响收敛**



### 七、 Recent State-of-Art Methods

#### 7.1 AMSGrad

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic10.png">

AMSGrad对于Adam的优化在于，限制了vt的最大值，因此学习速率有了下限：

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic11.png">

#### 7.2 AdaBound

基本算法与Adam一致，区别在于对学习速率同时设上限和下限，同时上下限为动态边界， ηl下边界，ηu是上边界。

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic12.png">

开始训练时，上限无穷大，下限为0，使模型自由迭代，快速降低Loss function,随着迭代步数t的增加，上下限开始发挥作用并逐渐缩小学习速率的取值范围。

最后，t趋于无穷时，上下限都趋于同一个值，此时，学习速率可以视为恒定值。

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic13.JPG">

SGDM可以视作lower bound=upper bound =α* 的特例；Adam可以视作lower bound = 0, upper bound = ∞ 的特例。

**AdaBound可以视作是Adam随t逐渐向SGDM的平滑过渡。**

<img src="https://github.com/MurphyQT/BP_Algorithm_in_AI/raw/master/pics/pic14.png">

