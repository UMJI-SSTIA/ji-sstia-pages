# 凸优化算法

Presented by Yang Mo

在人工智能中，需要对于数据进行学习。而学习过程的步骤之一就是找到使得设定好的损失函数（loss function）达到最小值的参数。优化本质上就是找到一个函数最小值的过程。而为了简化问题，凸优化指的是研究定义于凸集中的凸函数最小化的问题。


## 无约束的优化

我们现在想要找到函数$$f_0(x)$$的最小值。我们使用一种迭代的算法来找。

我们先随便指定一个$$x^k$$,我们只要找到一个$$x^{k+1}$$使得$$f_0(x^k)>f_0(x^{k+1})$$就可以使用$$x^{k+1}$$来代替$$x^k$$进入下一轮迭代。

为了让我们更快的计算，我们希望每次迭代效率更高，也就是使得函数$$f_0$$减少更多。
$$
x^{k+1}  = x^k + \alpha^k \cdot d^k
$$
其中，$$x$$是需要被优化的变量。$$\alpha$$为每一步走的长度，因此为标量。d是一个单位矢量，指每一步走的方向。


### 无约束优化算法举例——梯度下降法

对于一个函数$$f_0(x)$$，我们先找一个$$x_0$$。我们使用下面的方法对每一个$$x_k$$进行迭代。
$$
x^{k+1} = x^k - c \cdot \nabla f(x^k)
$$
其中$$c$$是一个常数。

当$$x^k$$与$$x^{k+1}$$接近到一定程度时停止迭代。

一个梯度下降法的python实现如下：

```python
import numpy as np

def gradient_descent(f, grad, init_x, lr=0.01, tol=1e-6, max_iter=1000):
    """
    梯度下降法求解函数的最小值
    :param f: 目标函数
    :param grad: 目标函数的梯度
    :param init_x: 初始点
    :param lr: 学习率
    :param tol: 容差
    :param max_iter: 最大迭代次数
    :return: 最小值点和最小值
    """
    x = init_x
    for i in range(max_iter):
        g = grad(x)
        x_new = x - lr * g
        if np.linalg.norm(x_new - x) < tol:
            break
        x = x_new
    return x, f(x)

# 例子：求解函数 f(x) = (x-1)^2 + y^2 的最小值
def f(x):
    return (x[0]-1) ** 2 + x[1] ** 2

def grad(x):
    return np.array([2 * (x[0] - 1), 2 * x[1]])

init_x = np.array([1.0, 1.0])
x_min, f_min = gradient_descent(f, grad, init_x)
print("最小值点为：", x_min)
print("最小值为：", f_min)

```

也就是说，在上面的例子当中，每一步走的方向是$$\nabla f(x^k)$$，每一步的步长为向量$$c \cdot \nabla f(x^k)$$的长度。

从上面的例子可以看出，不同的优化算法本质上不同之处体现在步长选择和方向选择的不同。


###     Amijo规则

但是上述的方法也有一定的缺点。比如每一步的步长可能相对较短。

Amijo Rule仍然选择了$$\nabla f(x^k)$$作为方向，但是改变了步长的选择方法。


若  $$f_0(x^k + \alpha \cdot d^k)>f_0(x^k)+\gamma \cdot \alpha \cdot \nabla f(x^k) \cdot d^k$$

则 $$\alpha := \alpha \cdot \beta$$

否则停止

其中$$\gamma$$决定了下降的显著程度，一般在$$[0,0.5]$$之间，$$\beta$$一般在$$[0,1]$$ 之间。

![](https://www.researchgate.net/profile/Dominic-Kafka/publication/338621028/figure/fig5/AS:847815165108229@1579146295992/Schematic-diagrams-for-a-the-function-value-based-inexact-line-search-that-is-based-on.ppm)

这种方法也可以称为 Back-tracing Algorithm，因为一般会选一个较大的步长，随后逐渐减小。

python实现如下：

```python
import numpy as np

def gradient_descent(f, grad, init_x, lr=1, c=0.1, rho=0.5, max_iter=1000):
    """
    使用 Amijo rule 的梯度下降法求解函数的最小值
    :param f: 目标函数
    :param grad: 目标函数的梯度
    :param init_x: 初始点
    :param lr: 初始步长
    :param c: Amijo rule 中的参数
    :param rho: Amijo rule 中的参数
    :param max_iter: 最大迭代次数
    :return: 最小值点和最小值
    """
    x = init_x
    for i in range(max_iter):
        g = grad(x)
        alpha = lr
        while f(x - alpha * g) > f(x) - c * alpha * np.dot(g, g):
            alpha *= rho
        x_new = x - alpha * g
        if np.linalg.norm(x_new - x) < 1e-6:
            break
        x = x_new
    return x, f(x)

# 例子：求解函数 f(x) = x^2 + y^2 的最小值
def f(x):
    return (x[0] - 1) ** 2 + x[1] ** 2

def grad(x):
    return np.array([2 * (x[0] - 1), 2 * x[1]])

init_x = np.array([1.0, 1.0])
x_min, f_min = gradient_descent(f, grad, init_x)
print("最小值点为：", x_min)
print("最小值为：", f_min)
```

但不仅我们可以改变选择步长的方法，我们也可以改变方向的办法。


### 坐标轮换法

坐标轮换本质上的目的在于，因为每次计算多元函数梯度的成本较高，因此每次只对一个坐标进行优化。

例如，对于函数$$f_0(x_0,x_1,x_2,...,x_n)$$, 可以在固定一部分坐标的情况下，对一个坐标进行优化，然后轮换。

![](https://handwiki.org/wiki/images/e/e3/Coordinate_descent.svg)

python实现如下：

```python
import numpy as np

def coordinate_descent(f, grad, x0, max_iter=1000, tol=1e-6):
    """
    坐标轮换法求解函数的最小值
    :param f: 目标函数
    :param grad: 目标函数的梯度
    :param x0: 初始点
    :param max_iter: 最大迭代次数
    :param tol: 收敛阈值
    :return: 最小值点和最小值
    """
    x = x0.copy()
    n = x0.size
    for iter in range(max_iter):
        for i in range(n):
            ei = np.zeros(n)
            ei[i] = 1
            gi = grad(x).dot(ei)
            #这个地方采取了梯度点成方向向量的方法计算一个方向上的梯度，其实没有体现出可以只计算一个方向上偏导的有点。。。
            if abs(gi) > 1e-10:
                alpha = -f(x) / gi
                x[i] += alpha
        if np.linalg.norm(grad(x)) < tol:
            break
    return x, f(x)

# 例子：求解函数 f(x) = (x1-1)^2 + x2^2 + x3^2 的最小值
def f(x):
    return (x[0] - 1) ** 2 + x[1] ** 2 + x[2] ** 2

def grad(x):
    return np.array([2 * (x[0] - 1), 2 * x[1], 2 * x[2]])

x0 = np.array([1.0, 2.0, 3.0])
x_min, f_min = coordinate_descent(f, grad, x0)
print("最小值点为：", x_min)
print("最小值为：", f_min)
```

有时候也被叫做Zig-zag

这个方法也可以进行拓展。对于多元函数，不一定需要轮换坐标，而是把多个坐标拆分成若干子空间进行循环优化。


### 牛顿法

牛顿法是一种对于梯度下降的改进。梯度下降的缺点在于它只关注了一个函数的一阶导数。为了更好的知道这个函数的性质，牛顿法采用的不只是线性模拟，而包括了二次项。

$$f(x^k+v)$$ 可以用 $$f(x^k)+\nabla f^T(x^k) v + \frac{1}{2} v^T \nabla^2 f(x^k)v$$ 来模拟

为了让上面的式子取到最小值， $$v=(\nabla^2f(x^k) )^{-1}\nabla f(x^k)$$

因此牛顿法的流程是:
$$
d^k = -(\nabla^2f(x^k) )^{-1}\nabla f(x^k)\\
\alpha^k = \arg \min f(x^k+\alpha^k\cdot d^k)\\
x^{k+1} = x^k + \alpha^k \cdot d^k
$$
一个python实现的牛顿法求解如下

```python
import numpy as np

def newton_method(f, x0, tol=1e-8, max_iter=1000):
    # f是目标函数，x0是初始点，tol是容差，max_iter是最大迭代次数
    x = x0
    for i in range(max_iter):
        # 计算函数值、一阶导数和二阶导数
        fx = f(x)
        grad_f = np.array([f(x + h) - fx for h in np.eye(len(x))]) / 1e-6
        hess_f = np.array([(grad_f(x + h) - grad_f(x)) / 1e-6 for h in np.eye(len(x))])
        # 计算搜索方向和步长
        dx = np.linalg.solve(hess_f, -grad_f)
        alpha = 1.0
        while f(x + alpha * dx) > fx:
            alpha *= 0.5
        # 更新x
        x = x + alpha * dx
        # 判断是否达到容差
        if np.linalg.norm(dx) < tol:
            break
    return x

# 定义目标函数
def f(x):
    return (x[0]-1)**2 + x[1]**2

# 调用牛顿法
x0 = np.array([0.0, 0.0])
x_opt = newton_method(f, x0)

print("优化结果：", x_opt)
print("目标函数最小值：", f(x_opt))

```

牛顿法的问题主要在于计算Hessian矩阵计算量很大。基于这一点，后来也有不少拟牛顿法。这些方法通过找到一个方便计算的近似Hessian矩阵来实现目的。




## 有约束的优化


### 等式约束下的拉格朗日乘子法

一般的优化问题会被一定条件束缚。

因此一般的优化问题有以下的形式：
$$
min \space f_0(x)\\ s.t. \space f_i(x) \le 0, \space i = 1,2,...,m\\ h_i(x) = 0,i=1,2,...,n
$$
$$s.t.$$意即subject to

对于这种问题，最简单得方法是采用拉格朗日乘子法

我们先讨论只有等式约束的情况，即
$$
\min \space f_0(x)\\ s.t.  h_i(x) = 0,i=1,2,...,n
$$

$$min \space f_0(x)$$等价于$$min \space f_0(x) + \lambda_i \cdot h_i(x)$$，因此问题转化为
$$
min \space f_0(x) + \lambda_i \cdot h_i(x)\\ s.t.  h_i(x) = 0,i=1,2,...,n
$$

对于凸问题，我们直接可以说极值点在一阶导为零处，也就是说
$$
\frac{\partial}{\partial x}(f_0(x)+ \lambda_i \cdot h_i(x)) = 0
$$
我们发现$$h_i(x) = 0$$也可以被改成类似的形式
$$
\frac{\partial}{\partial \lambda_i}(f_0(x)+ \lambda_i \cdot h_i(x)) = 0
$$
因此整个问题可以转化为
$$
min \space g(x,\lambda)\\ g(x,\lambda) = f_0(x)+ \lambda_i \cdot h_i(x)
$$


一个拉格朗日乘子法求最小值的python实现

```python
import numpy as np
from scipy.optimize import minimize

# 定义目标函数
def f(x):
    return x[0]**2 + x[1]**2

# 定义等式约束
def g(x):
    return x[0] + x[1] - 1

# 定义拉格朗日函数
def lagrangian(x, lmbda):
    return f(x) - lmbda * g(x)

# 定义求解函数
def solve_lagrangian(x0):
    # 定义约束条件
    cons = {'type': 'eq', 'fun': g}
    # 初始拉格朗日乘子为0
    lmbda0 = 0.0
    # 定义求解函数
    def lagrangian_min(x):
        return lagrangian(x, lmbda0)
    # 调用优化函数求解
    res = minimize(lagrangian_min, x0, constraints=cons)
    # 提取优化结果和乘子
    x_opt = res.x
    lmbda_opt = res.fun / g(x_opt)
    return x_opt, lmbda_opt

# 求解优化问题
x0 = [0.5, 0.5]
x_opt, lmbda_opt = solve_lagrangian(x0)

# 输出结果
print("优化结果：", x_opt)
print("拉格朗日乘子：", lmbda_opt)
print("目标函数最小值：", f(x_opt))
print("约束条件值：", g(x_opt))

```




### 不等式约束与KKT条件

我们继续讨论含有不等式约束的情况，以及继续使用拉格朗日乘子
$$
min \space f_0(x)\\ s.t. \space f_i(x) \le 0, \space i = 1,2,...,m\\ h_i(x) = 0,i=1,2,...,n
$$
定义 $$L(x,\lambda,\mu) = f_0(x)+ \lambda_i \cdot f_i(x) + \mu_i \cdot h_i(x)$$ 其中 $$\lambda_i \ge 0$$ 我们称 $$L(x,\lambda,\mu)$$为拉格朗日函数

令$$g(\lambda,\mu) = inf_x \space L(x,\lambda,\mu) $$

那么一定有 $$g(\lambda,\mu) \le inf_x \space f_0(x)$$

因此 $$max_{\lambda,\mu} \space g(x,\lambda,\mu) \le inf_x f_0(x)$$

以下我们不加证明的给出下面的定理(Slater's Condition):

对于凸问题$$min \space f_0(x) \space s.t. \space f_i(x) \le 0 \space (i = 1,2,...,n), \space Ax=b$$， 满足 $$max_{\lambda,\mu} \space g(x,\lambda,\mu) = inf_x f_0(x)$$。

为什么会出现这种情况？下面不加数学的给出一个经济学解释。

所以我们可以知道 $$\lambda_i \cdot f_i(x) = 0$$

从上面的各种情况，我们可以总结出一下达到最小值的条件（Karush-Kuhn-Tucker Conditions）

对于能使得$$f_0(x)$$在符合约束条件下达到最小值的$$x^*$$，一定符合KKT条件
$$
f_i(x^*)\le0\\
h_i(x^*)=0\\
\lambda_i \ge 0\\
\lambda_i \cdot f_i(x) = 0\\
\nabla f_0(x) + \Sigma\lambda_i \cdot \nabla f_i(x) + \mu_i \cdot \nabla h_i(x) = 0
$$
(最后一个条件是求导等于零)


### KKT条件应用实例——灌水算法

Water-filling Problem:
$$
min \space \Sigma^{n}_{i=1} \log (\alpha_i + x_i) \\ s.t. \space x \ge 0, \space 1^T x = 0
$$
该问题的KKT条件：
$$
x^* \ge 0\\ 1^Tx^* = 1\\ \lambda_i^* \cdot x_i^*=0,\space i=1,2,...,n\\ -\frac{1}{\alpha_i + x_i^*}-\lambda_i^*+v^* = 0
$$
消掉$$\lambda^*$$,得到
$$
x^*\cdot(v^*-\frac{1}{\alpha_i+x_i^*}) = 0,i=1,2,...,n\\
v^* \ge \frac{1}{\alpha_i+x_i^*},i=1,2,...,n
$$
也就是
$$
v^* \ge \frac{1}{\alpha_i} \Rightarrow x^*_i = 0\\
v^* \lt \frac{1}{\alpha_i} \Rightarrow x^*_i = \frac{1}{v_i^*}-\alpha_i
$$
![](https://wikidocs.net/images/page/20961/water-fill.png)


### 惩罚函数与增广拉格朗日法

需要解决的问题是：
$$
\min \space f_0(x)\\ s.t.  h_i(x) = 0,i=1,2,...,n
$$
惩罚函数法：

不在纠结于一定要满足等式条件。把等式约束下的优化问题改成一个无约束优化问题
$$
\min \space f_0(x)+\frac{c}{2}\cdot  \Sigma^n_{i=1} h_i^2(x)
$$
当参数c趋于无穷时，得到的最优解逐渐趋近于原本的最优解。


所以贪婪的人类必定会把两个好东西放在一起。

相比于传统的拉格朗日函数，我们引入一些很新的东西：
$$
L_c(x,v) = f(x) + v^T h(x) + \frac{c}{2} |h(x)|^2
$$
这个问题和原来的拉格朗日函数是等价的

我们考虑约束函数 $$h(x) = Ax-b$$ 的情况

原来的拉格朗日法迭代的函数是这样的：
$$
x^{k+1}=x^k-\alpha^k \cdot (\nabla f(x^k)+A^Tv^k)\\ v^{k+1}=v^k+\alpha^k(Ax^k-b)
$$
现在我们的迭代过程变成了这样：
$$
x^{k+1}=\arg \min_x L_c(x,v_k)\\ v^{k+1}=v^k+\alpha^k(Ax^k-b)
$$
（这里的两个方法只是找到了下降的方向，至于具体的步长需要继续步长搜索，例如Amijo Rule）

```python
import numpy as np

def augmented_lagrangian(f, g, x0, lambd0, mu0, maxiter, tol, rho=1.0, gamma=2.0):
    x = x0
    lambd = lambd0
    mu = mu0
    it = 0
    converged = False
    
    while not converged and it < maxiter:
        # 求解Lagrange乘子
        def L(x):
            return f(x) + np.dot(lambd, g(x)) + (mu/2.0) * np.linalg.norm(g(x))**2
        
        # 梯度下降
        x = gradient_descent(lambda x: L(x), x, tol=tol, rho=rho, gamma=gamma)
        
        # 更新Lagrange乘子
        lambd = lambd - mu * g(x)
        
        # 更新惩罚参数
        mu = gamma * mu
        
        # 检查是否收敛
        if np.linalg.norm(g(x)) < tol:
            converged = True
        
        it += 1
    
    return x

def gradient_descent(f, x0, tol, rho, gamma, maxiter=1000):
    x = x0
    it = 0
    converged = False
    
    while not converged and it < maxiter:
        gradient = compute_gradient(f, x)
        x_new = x - rho * gradient
        if np.linalg.norm(x_new - x) < tol:
            converged = True
        x = x_new
        rho *= gamma
        it += 1
        
    return x

def compute_gradient(f, x, eps=1e-8):
    gradient = np.zeros_like(x)
    for i in range(len(x)):
        x_plus = x.copy()
        x_plus[i] += eps
        x_minus = x.copy()
        x_minus[i] -= eps
        gradient[i] = (f(x_plus) - f(x_minus)) / (2.0 * eps)
    return gradient

```


### 交替方向的拉格朗日乘子法与分布式计算

对于大型人工智能或者计算程序，分布式计算有利于提高效率。同样，优化问题也有办法被拆分进行。例如下面的问题：
$$
\min (f(x) + g(x))
$$
可以转化为
$$
\min f(x) + g(x)\\s.t.x-z=0
$$
我们定义增广拉格朗日函数 $$L_c(x,z,v) = f(x) + g(z) + v\cdot (x-z) + \frac{c}{2}\cdot ||x-z||^2_2$$

分成两步解决：
$$
1.[x^{k+1},z^{k+1}]=\arg \min_{x,z} (f(x)+g(z)+v(x-z)+\frac{c}{2}\cdot ||x-z||^2_2)\\
2.v^{k+1}=v^k+c \cdot(x^{k+1}-z^{k+1})
$$
而对于第一步拆分成以下的步骤。考虑对于大循环第k步的情况
$$
1A. x^{k|t+1}=\arg \min_x f(x)+\frac{c}{2}\cdot ||x^{k|t}-z^{k|t}+\frac{v^k}{2}||^2_2)\\
1B. z^{k|t+1}=\arg \min_z g(z)+\frac{c}{2}\cdot ||x^{k|t}-z^{k|t}+\frac{v^k}{2}||^2_2)
$$
我们可以把1A和1B拆分到两个小计算机上进行，把步骤2在大计算机上运行，实现优化的分布。




## 参考

1. 中科大-凸优化 凌青 https://www.bilibili.com/video/BV1Jt411p7jE/?spm_id_from=333.999.0.0&vd_source=a17cf2a11436a72227c8a1d1d96ca44a

2. *Cnvex Optimization* Stephen Boyd

3. 感谢chatGPT写的python


