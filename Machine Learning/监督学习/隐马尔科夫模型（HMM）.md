
> **前置知识：马尔可夫性 (Markov Property)**
> 
> 系统在时刻 $t$ 的状态只与其在时刻 $t-1$ 的状态有关，与更早的状态无关。
> $$P(q_t | q_{t-1}, q_{t-2}, \dots, q_1) = P(q_t | q_{t-1})$$

---

## 1. 隐马尔可夫模型 (Model) 定义

隐马尔可夫模型（HMM）是关于时序的概率模型，描述由一个**隐藏的马尔可夫链**随机生成不可观测的状态序列，再由各个状态生成观测序列的过程。

设 $Q$ 是所有可能的状态集合， $V$ 是所有可能的观测集合：
$Q = \{q_1, q_2, \dots, q_N\}, \quad V = \{v_1, v_2, \dots, v_M\}$
其中 $N$ 是可能的状态数，$M$ 是可能的观测数。

设 $I$ 是长度为 $T$ 的状态序列，$O$ 是对应的观测序列：
$I = (i_1, i_2, \dots, i_T), \quad O = (o_1, o_2, \dots, o_T)$

### 1.1 模型的三个基本要素
模型可表示为三元组：$\lambda = (A, B, \pi)$

- **$A$ (状态转移概率矩阵)**：$A = [a_{ij}]_{N \times N}$
  $a_{ij} = P(i_{t+1} = q_j | i_t = q_i)$ 
- **$B$ (观测概率矩阵 / 发射矩阵)**：$B = [b_j(k)]_{N \times M}$
  $b_j(k) = P(o_t = v_k | i_t = q_j)$ 
- **$\pi$ (初始状态概率向量)**：$\pi = (\pi_i)$
  $\pi_i = P(i_1 = q_i)$ 

### 1.2 两个核心条件独立性假设 (公式推导的基石)

1. **齐次马尔可夫性假设**：
   $$P(i_t | i_{t-1}, o_{t-1}, \dots, i_1, o_1) = P(i_t | i_{t-1})$$
2. **观测独立性假设**：
   $$P(o_t | i_T, o_T, \dots, i_{t+1}, o_{t+1}, i_t, i_{t-1}, \dots) = P(o_t | i_t)$$

---

## 2. 问题一：概率计算问题 (Evaluation) 与公式推导

**目标**：给定模型 $\lambda = (A, B, \pi)$ 和观测序列 $O$，计算 $P(O|\lambda)$。

### 2.1 暴力计算法 (直接计算)
理论上，我们可以穷举所有可能的状态序列 $I$。
$P(O|\lambda) = \sum_{I} P(O, I | \lambda) = \sum_{I} P(O | I, \lambda) P(I | \lambda)$

- $P(I | \lambda) = \pi_{i_1} a_{i_1 i_2} a_{i_2 i_3} \dots a_{i_{T-1} i_T}$
- $P(O | I, \lambda) = b_{i_1}(o_1) b_{i_2}(o_2) \dots b_{i_T}(o_T)$

代入得：$P(O|\lambda) = \sum_{i_1, \dots, i_T} \pi_{i_1} b_{i_1}(o_1) a_{i_1 i_2} b_{i_2}(o_2) \dots a_{i_{T-1} i_T} b_{i_T}(o_T)$
**缺陷**：时间复杂度为 $O(TN^T)$，呈指数级爆炸，实际不可行。

### 2.2 前向算法 (Forward Algorithm) 推导
引入**前向概率 $\alpha_t(i)$**：到时刻 $t$ 为止的观测序列为 $o_1, \dots, o_t$ 且时刻 $t$ 处于状态 $q_i$ 的联合概率。
$$\alpha_t(i) = P(o_1, o_2, \dots, o_t, i_t = q_i | \lambda)$$

**【递推公式推导】**：
我们要用时刻 $t$ 的 $\alpha_t(j)$ 来表示时刻 $t+1$ 的 $\alpha_{t+1}(i)$。
$$
\begin{aligned}
\alpha_{t+1}(i) &= P(o_1, \dots, o_{t+1}, i_{t+1} = q_i | \lambda) \\
&= \sum_{j=1}^N P(o_1, \dots, o_t, o_{t+1}, i_t = q_j, i_{t+1} = q_i | \lambda) \quad \text{(引入全概率公式)} \\
&= \sum_{j=1}^N P(o_{t+1} | o_1, \dots, o_t, i_t = q_j, i_{t+1} = q_i) P(o_1, \dots, o_t, i_t = q_j, i_{t+1} = q_i) \quad \text{(条件概率乘法公式)} \\
\end{aligned}
$$
应用**观测独立性假设**，第一项 $P(o_{t+1} | \dots, i_{t+1}=q_i)$ 只和 $i_{t+1}$ 有关，即为 $b_i(o_{t+1})$。
应用**马尔可夫性假设**，第二项可拆解为：$P(o_1, \dots, o_t, i_t=q_j) P(i_{t+1}=q_i | i_t=q_j) = \alpha_t(j) a_{ji}$。
故得出最终递推式：
$$\alpha_{t+1}(i) = \left[ \sum_{j=1}^N \alpha_t(j) a_{ji} \right] b_i(o_{t+1})$$

### 2.3 后向算法 (Backward Algorithm) 推导
引入**后向概率 $\beta_t(i)$**：在时刻 $t$ 处于状态 $q_i$ 的条件下，从 $t+1$ 到 $T$ 的观测序列为 $o_{t+1}, \dots, o_T$ 的条件概率。
$$\beta_t(i) = P(o_{t+1}, \dots, o_T | i_t = q_i, \lambda)$$

**【递推公式推导】**：
$$
\begin{aligned}
\beta_t(i) &= P(o_{t+1}, \dots, o_T | i_t = q_i, \lambda) \\
&= \sum_{j=1}^N P(o_{t+1}, \dots, o_T, i_{t+1} = q_j | i_t = q_i, \lambda) \\
&= \sum_{j=1}^N P(o_{t+2}, \dots, o_T | o_{t+1}, i_{t+1}=q_j, i_t=q_i) P(o_{t+1}, i_{t+1}=q_j | i_t=q_i) \\
\end{aligned}
$$
应用两个假设化简得：
$$\beta_t(i) = \sum_{j=1}^N a_{ij} b_j(o_{t+1}) \beta_{t+1}(j)$$

---

## 3. 问题二：学习问题 (Learning) 与 Baum-Welch 算法推导

**目标**：已知观测序列 $O$，估计模型参数 $\lambda = (A, B, \pi)$。此时隐状态序列 $I$ 未知，需要用 EM 算法。

### 3.1 确定完全数据的对数似然函数
完全数据为 $(O, I)$，其联合分布为：
$P(O, I | \lambda) = \pi_{i_1} b_{i_1}(o_1) a_{i_1 i_2} b_{i_2}(o_2) \dots a_{i_{T-1} i_T} b_{i_T}(o_T)$

### 3.2 E步：求 Q 函数
$Q$ 函数定义为完全数据对数似然在给定观测序列 $O$ 和当前参数 $\bar{\lambda}$ 下隐变量 $I$ 的条件期望：
$$Q(\lambda, \bar{\lambda}) = \sum_I P(I | O, \bar{\lambda}) \log P(O, I | \lambda)$$
因为 $P(I | O, \bar{\lambda}) = \frac{P(I, O | \bar{\lambda})}{P(O | \bar{\lambda})}$，且分母是常数，可等价最大化：
$$Q(\lambda, \bar{\lambda}) \propto \sum_I P(O, I | \bar{\lambda}) \log P(O, I | \lambda)$$
代入联合分布并取对数：
$$Q(\lambda, \bar{\lambda}) = \sum_I P(O, I | \bar{\lambda}) \left( \log \pi_{i_1} + \sum_{t=1}^{T-1} \log a_{i_t i_{t+1}} + \sum_{t=1}^T \log b_{i_t}(o_t) \right)$$

### 3.3 M步：极大化 Q 函数求解参数 (拉格朗日乘子法)
上式中 $\pi_i, a_{ij}, b_j(k)$ 分别在三项中，可以分别利用约束条件求极大值。

**推导 $\pi_i$**：
取出包含 $\pi$ 的项：$\sum_{i=1}^N P(O, i_1=q_i | \bar{\lambda}) \log \pi_i$
加上等式约束 $\sum_{i=1}^N \pi_i = 1$，构造拉格朗日函数：
$L = \sum_{i=1}^N P(O, i_1=q_i | \bar{\lambda}) \log \pi_i + \gamma \left(\sum_{i=1}^N \pi_i - 1\right)$
对 $\pi_i$ 求导令其为 0：
$\frac{\partial L}{\partial \pi_i} = \frac{P(O, i_1=q_i | \bar{\lambda})}{\pi_i} + \gamma = 0$
求和得 $\gamma = -P(O | \bar{\lambda})$，代回得：
$$\pi_i = \frac{P(O, i_1=q_i | \bar{\lambda})}{P(O | \bar{\lambda})} = \gamma_1(i)$$
*(注：同理可推导 $a_{ij}$ 和 $b_j(k)$，均是利用拉格朗日乘子结合期望频数得到更新公式)*。

---

## 4. 问题三：预测/解码问题 (Decoding) 与 Viterbi 算法

**目标**：已知 $\lambda$ 和 $O$，求最有可能的状态序列 $I^*$。
**策略**：动态规划。

定义在时刻 $t$ 状态为 $i$ 的所有单个路径中的最大概率为 $\delta_t(i)$：
$$\delta_t(i) = \max_{i_1, \dots, i_{t-1}} P(i_t=i, i_{t-1}, \dots, i_1, o_t, \dots, o_1 | \lambda)$$

递推公式：
$\delta_{t+1}(i) = \max_{1 \le j \le N} [\delta_{t}(j) a_{ji}] \cdot b_i(o_{t+1})$

同时为了找回路径，需要用 $\Psi_t(i)$ 记录时刻 $t$ 状态为 $i$ 的所有路径中概率最大的路径的第 $t-1$ 个节点的索引：
$\Psi_t(i) = \arg\max_{1 \le j \le N} [\delta_{t-1}(j) a_{ji}]$

---

## 5. 课后习题详细解答 (10.1 - 10.5)

**【全局参数设定】**
$Q = \{1, 2, 3\}, \quad V = \{r(\text{红}), w(\text{白})\}$
$$
A = \begin{bmatrix} 0.5 & 0.2 & 0.3 \\ 0.3 & 0.5 & 0.2 \\ 0.2 & 0.3 & 0.5 \end{bmatrix}, \quad
B = \begin{bmatrix} 0.5 & 0.5 \\ 0.4 & 0.6 \\ 0.7 & 0.3 \end{bmatrix}, \quad
\pi = \begin{bmatrix} 0.2 \\ 0.4 \\ 0.4 \end{bmatrix}
$$

### 习题 10.1：利用后向算法计算 $P(O|\lambda)$
**观测**：$O = (r, w, r, w)$
1. **$t=4$**：$\beta_4 = (1, 1, 1)^T$
2. **$t=3$ ($o_4 = w$)**：
   $\beta_3(1) = 0.5(0.5)(1) + 0.2(0.6)(1) + 0.3(0.3)(1) = 0.46$
   $\beta_3(2) = 0.3(0.5)(1) + 0.5(0.6)(1) + 0.2(0.3)(1) = 0.51$
   $\beta_3(3) = 0.2(0.5)(1) + 0.3(0.6)(1) + 0.5(0.3)(1) = 0.43$
3. **$t=2$ ($o_3 = r$)**：
   $\beta_2(1) = 0.5(0.5)(0.46) + 0.2(0.4)(0.51) + 0.3(0.7)(0.43) = 0.2461$
   $\beta_2(2) = 0.3(0.5)(0.46) + 0.5(0.4)(0.51) + 0.2(0.7)(0.43) = 0.2312$
   $\beta_2(3) = 0.2(0.5)(0.46) + 0.3(0.4)(0.51) + 0.5(0.7)(0.43) = 0.2577$
4. **$t=1$ ($o_2 = w$)**：
   $\beta_1(1) = 0.5(0.5)(0.2461) + 0.2(0.6)(0.2312) + 0.3(0.3)(0.2577) \approx 0.11246$
   $\beta_1(2) = 0.3(0.5)(0.2461) + 0.5(0.6)(0.2312) + 0.2(0.3)(0.2577) \approx 0.12174$
   $\beta_1(3) = 0.2(0.5)(0.2461) + 0.3(0.6)(0.2312) + 0.5(0.3)(0.2577) \approx 0.10488$
5. **终结 ($o_1 = r$)**：
   $P(O|\lambda) = \sum \pi_i b_i(r) \beta_1(i) = 0.2(0.5)(0.11246) + 0.4(0.4)(0.12174) + 0.4(0.7)(0.10488) = \mathbf{0.06009}$

### 习题 10.2：前后向概率混合计算
求 $T=8$ 序列中 $P(i_4 = q_3 | O, \lambda)$。
利用公式：$P(i_t = q_i | O, \lambda) = \frac{\alpha_t(i) \beta_t(i)}{\sum_j \alpha_t(j) \beta_t(j)}$
1. 推导前向概率 $\alpha_4 = (0.02108, 0.01679, 0.03226)^T$。
2. 推导后向概率 $\beta_4 = (0.05320, 0.05776, 0.04973)^T$。
3. 计算：分子 $\alpha_4(3)\beta_4(3) \approx 0.001604$；分母累加 $\approx 0.003695$。
结果为 $\mathbf{0.434}$。

### 习题 10.3：用维特比算法求最优路径
1. **$t=1$ ($o_1 = r$)**：$\delta_1 = (0.10, 0.16, \mathbf{0.28})$
2. **$t=2$ ($o_2 = w$)**：
   $j=1: \max(0.05, 0.048, \mathbf{0.056}) \cdot 0.5 \implies \Psi_2(1)=3$
   $j=2: \max(0.02, 0.08, \mathbf{0.084}) \cdot 0.6 \implies \Psi_2(2)=3$
   $j=3: \max(0.03, 0.032, \mathbf{0.14}) \cdot 0.3 \implies \Psi_2(3)=3$
3. **$t=3$ ($o_3 = r$)**：
   推导得最大局部来自节点2，即 $\Psi_3(1)=2, \Psi_3(2)=2, \Psi_3(3)=3$
4. **$t=4$ ($o_4 = w$)**：
   $\delta_4(2)$ 取到所有状态最大概率 $0.003024$，故 $i_4^* = \mathbf{2}$。
**回溯路径**：$I^* = (3, 2, 2, 2)$。

### 习题 10.4：全局概率公式的证明
**证明**：根据全概率公式对 $t$ 处于状态 $q_i$ 且 $t+1$ 处于 $q_j$ 进行展开：
$$P(O|\lambda) = \sum_{i=1}^N \sum_{j=1}^N P(O, i_t=q_i, i_{t+1}=q_j | \lambda)$$
拆分 $O$ 序列为 $\{o_1..o_t\}$，$\{o_{t+1}\}$，$\{o_{t+2}..o_T\}$ 三段：
联合概率可拆解为：
1. $P(o_1 \dots o_t, i_t=q_i | \lambda) = \alpha_t(i)$
2. $P(i_{t+1}=q_j | i_t=q_i) = a_{ij}$
3. $P(o_{t+1} | i_{t+1}=q_j) = b_j(o_{t+1})$
4. $P(o_{t+2} \dots o_T | i_{t+1}=q_j, \lambda) = \beta_{t+1}(j)$
四项相乘即得：$\sum_i \sum_j \alpha_t(i) a_{ij} b_j(o_{t+1}) \beta_{t+1}(j)$。

### 习题 10.5：比较 $\delta$ 与 $\alpha$ 计算的区别
- **前向变量 $\alpha_t(i)$**：使用的是 **求和 (Sum)** 算子。它聚合了到达当前节点的所有可能路径的概率，属于**期望累积**，用于解决全局概率计算（Evaluation）。
- **维特比变量 $\delta_t(i)$**：使用的是 **求最大值 (Max)** 算子。它直接阻断了劣势路径的概率累加，只保留到达当前节点的最优单次路径，属于**硬对齐匹配**，用于解决序列解码问题（Decoding）。