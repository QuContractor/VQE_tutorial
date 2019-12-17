# VQE_tutorial
A simple VQE tutorial for beginners.

在量子计算的众多应用中，量子化学被认为是可以在近期取得量子优势的领域之一。变分量子本征求(Variational Quantum Eigensolver, VQE) 采用量子-经典混合计算架构，其中量子计算部分可在较短时间内进行，减少退相干的影响，从而有利于在近期的量子硬件上实施。

在这个系列中，我们提供一个面向初学者的tutorial, 主要是几个在VQE和量子化学领域入门的例子。抛砖引玉，希望更多的同仁加入到量子化学的学习中。

* [VQE部分介绍](https://github.com/QuContractor/VQE_tutorial/blob/master/VQE_tutorial_1.ipynb)
* [OpenFermion部分介绍](https://github.com/QuContractor/VQE_tutorial/blob/master/OpenFermion_tutorial.ipynb)

## VQE 简单示例

变分量子本征求解 (VQE) 通过制备带有参数的试探波函数(Ansatz)，通过测量体系哈密顿量在该波函数下面的平均值，最后使用经典优化算法来最小化这个平均值，从而确定波函数相关参数。根据基本的原理：


$$
\frac{<\varphi(\theta)|H|\varphi(\theta)>}{<\varphi(\theta)|\varphi(\theta)>}\geq E_0
$$


通过对参数变分从而最小化哈密顿量的平均值，同时得到的试探波函数就近似对应着系统的基态波函数。

比如我们需要求解的单比特哈密顿量为：
$$
H = 0.5\sigma^0_x + 0.7\sigma_y^0.
$$
使用HiQ/ProjectQ框架来模拟量子线路，并定义系统哈密顿量：

```python
import projectq
from projectq.ops import All, Measure, QubitOperator, TimeEvolution
from projectq.ops import CNOT, H, Rz, Rx, Ry, X, Z, Y
import matplotlib.pyplot as plt
from scipy.optimize import minimize_scalar, minimize
import numpy as np
# 定义哈密顿量
hamil = 0.5 * QubitOperator("X0") + 0.7 * QubitOperator("Y0")
```

对于这样一个简单的哈密顿量，我们可以对角化其哈密顿量矩阵得到这个二能级系统的能级：

```python
hamil_matrix = 0.5* X.matrix + 0.7 * Y.matrix
np.linalg.eigh(hamil_matrix)
(array([-0.86023253,  0.86023253])
```

接下来我们选取带有变分参数的试探波函数，并在量子线路中测量哈密顿量再该试探波函数下的平均值。这个平均值可以写为变分参数\theta的函数：

```python
def small_vqe(theta, hamiltonian):
    eng = projectq.MainEngine()
    qubits = eng.allocate_qureg(1)
	# 定义试探波函数
    H | qubits[0]
    Rz(theta[0]) | qubits[0]
    
    eng.flush()
    energy = eng.backend.get_expectation_value(hamiltonian, qubits)
    All(Measure) | qubits
    return energy
```

然后，我们通过经典计算的优化算法来优化测量后得到的平均值：

```python
minimum = minimize(lambda theta: small_vqe(theta,hamil), [0.])
minimum.fun
fun: -0.8602325267042619
```

对比发现，VQE的所得到的基态能量和精确对角化取得的结果相符。

然而在实际中，试探波函数的形式并不一定能完全描述真正的基态波函数。如果选取另一个试探波函数：

```python
def small_vqe(theta, hamiltonian):
    eng = projectq.MainEngine()
    qubits = eng.allocate_qureg(1)
	# 定义新的试探波函数
    Z | qubits[0]
    Rx(theta[0]) | qubits[0]
    
    eng.flush()
    energy = eng.backend.get_expectation_value(hamiltonian, qubits)
    All(Measure) | qubits
    return energy
```

```python
minimum = minimize(lambda theta: small_vqe(theta,hamil), [0.])
minimum.fun
-0.6999999999999997
```

则最小化后的“基态能”与真实基态能相差甚远。因此，根据具体问题选取合适的试探波函数也是VQE求解的关键因素。

## References:

* [“Quantum chemistry calculations on a trapped-ion quantum simulator,” Physical Review X 8, 031022 (2018).](http://dx.doi.org/10.1103/PhysRevX.8.031022)

* ["Quantum computational chemistry", arXiv:1808.10402](https://arxiv.org/abs/1808.10402)

* ["Collective optimization for variational quantum eigensolvers", arXiv: 1910.14030](https://arxiv.org/abs/1910.14030)
