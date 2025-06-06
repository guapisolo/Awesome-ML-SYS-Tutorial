# 马尔科夫决策过程

书接上文，想不到两次写作这一笔记，中间隔了整整一周。究竟是什么事情让我的学习进度一直被打断呢？

按照朋友所言，作为一个项目的 leader，我太在乎话语权了。话语权是最大的权利，同时也让我完全没法将手头的事情分享给他人。基于此，我决定每天至少有 1/3 的时间，关掉微信和 slack，全心自己学习，来琢磨一些事情。

## 马尔科夫过程

马尔科夫过程意味着"此时此刻，过去不会影响未来"，这一抽象能够建模相当多的实际问题。与多臂老虎机问题不同，马尔可夫决策过程包含状态信息以及状态之间的转移机制。如果要用强化学习去解决一个实际问题，第一步要做的事情就是把这个实际问题抽象为一个马尔可夫决策过程。

> 具有马尔可夫性并不代表这个随机过程就和历史完全没有关系。因为虽然 $t$ 时刻的状态只与 $t-1$ 时刻的状态有关，但是 $t-1$ 时刻的状态其实包含了 $t-2$ 时刻的状态的信息，通过这种链式的关系，历史信息被传递到了现在。马尔可夫性可以大大简化运算，因为只要当前状态可知，所有的历史信息都不再需要了，利用当前状态信息就可以决定未来。

马尔可夫过程 (Markov process) 指具有马尔可夫性质的随机过程，也被称为马尔可夫链 (Markov chain)。我们通常用集合 $(S, P)$ 描述一个马尔可夫过程，其中 $S$ 是数量有限的状态集合，$P$ 是状态转移矩阵 (state transition matrix)。假设一共有 $n$ 个状态，此时 $S = \{s_1, s_2, \cdots, s_n\}$。状态转移矩阵 $P$ 标记状态之间的转移概率，即：

$$
P = \begin{bmatrix}
P(s_1|s_1) & \cdots & P(s_n|s_1) \\
\vdots & \ddots & \vdots \\
P(s_1|s_n) & \cdots & P(s_n|s_n)
\end{bmatrix}
$$

矩阵 $P$ 中第 $i$ 行第 $j$ 列元素 $P(s_j|s_i) = P(S_{t+1} = s_j|S_t = s_i)$ 表示从状态 $s_i$ 转移到状态 $s_j$ 的概率。我们称 $P(s|s)$ 为状态转移函数。从某一个状态 $s_i$ 出发，到达其他状态的概率和应该为 1，即状态转移矩阵每一行的和为 1。

从一个马尔可夫过程，我们可以以某个状态出发，根据转移矩阵状态转移矩阵产生一个状态序列（episode)，设这个序列也称为样本路径 (sampling)。例如，可能发生序列 $s_1 \to s_2 \to s_3 \to s_6$ 或者序列 $s_1 \to s_1 \to s_2 \to s_3 \to s_4 \to s_5 \to s_3 \to s_6$ 等。生成这些序列的概率和状态转移矩阵有关。

### 奖励函数

在马尔可夫过程的基础上加入奖励函数 $r$ 和折扣因子 $\gamma$，就可以得到马尔可夫奖励过程 (Markov reward process)。一个马尔可夫奖励过程由 $(S, P, r, \gamma)$ 构成，其中组成元素的含义如下所示。

- $S$ 是有限状态集合。
- $P$ 是状态转移矩阵。
- $r$ 是奖励函数，某个状态 $s$ 的奖励 $r(s)$ 指转移到该状态时可以获得的奖励的期望。
- $\gamma$ 是折扣因子 (discount factor)，取值范围为 [0, 1)。

引入折扣因子的理由为远期利益具有一定不确定性，有时我们更希望能够尽快获得一些奖励，所以我们需要对远期利益打一些折扣。接近 1 的 $\gamma$ 更关注长期的累计奖励，接近 0 的 $\gamma$ 更希望短期奖励。

在一个马尔可夫奖励过程中，从第t时刻状态$S_t$开始，直到终止状态时，所有奖励的衰减之和称为回报$G_t$（Return），公式如下：

$$
G_t = R_t + \gamma R_{t+1} + \gamma^2 R_{t+2} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k}
$$

### 价值函数

在马尔可夫奖励过程中，一个状态的期望回报（即从这个状态出发的未来累积奖励的期望）被称为这个状态的价值（value）。所有状态的价值构成价值函数（value function），价值函数的输入为某一个状态，输出为这个状态的价值。我们将价值函数表示成 $V(s) = \mathbb{E}[G_t|S_t = s]$，根据回报的定义，可以得到：

$$
V(s) = \mathbb{E}[G_t|S_t = s] \\
= \mathbb{E}[R_t + \gamma R_{t+1} + \gamma^2 R_{t+2} + \cdots | S_t = s] \\
= \mathbb{E}[R_t + \gamma (R_{t+1} + \gamma R_{t+2} + \cdots) | S_t = s] \\
= \mathbb{E}[R_t + \gamma G_{t+1} | S_t = s] \\
= \mathbb{E}[R_t + \gamma V(S_{t+1}) | S_t = s]
$$

一方面，立即奖励的期望正是奖励函数的输出，即 $\mathbb{E}[R_t|S_t = s] = r(s)$；另一方面，学习中剥壳部分 $\mathbb{E}[\gamma V(S_{t+1})|S_t = s]$ 可以根据从状态 $s$ 出发的转移概率算出，即可以得

$$
V(s) = r(s) + \gamma \sum_{s' \in S} p(s'|s)V(s')
$$

上面就是马尔可夫奖励过程中非常有名的贝尔曼方程（Bellman equation），对每一个状态都成立。考虑一个马尔可夫奖励过程一共有 $n$ 个状态，即 $S = \{s_1, s_2, \cdots, s_n\}$，我们将所有状态的价值装表示成一个列向量 $V = [V(s_1), V(s_2), \cdots, V(s_n)]^T$，同理，将奖励函数变成一个列向量 $R = [r(s_1), r(s_2), \cdots, r(s_n)]^T$。于是我们可以将贝尔曼方程写成矩阵的形式：

$$
V = R + \gamma P V
$$

$$
\begin{aligned}
\begin{bmatrix}
V(s_1) \\
V(s_2) \\
\vdots \\
V(s_n)
\end{bmatrix} &= 
\begin{bmatrix}
r(s_1) \\
r(s_2) \\
\vdots \\
r(s_n)
\end{bmatrix} + \gamma
\begin{bmatrix}
P(s_1|s_1) & P(s_2|s_1) & \cdots & P(s_n|s_1) \\
P(s_1|s_2) & P(s_2|s_2) & \cdots & P(s_n|s_2) \\
\vdots & \vdots & \ddots & \vdots \\
P(s_1|s_n) & P(s_2|s_n) & \cdots & P(s_n|s_n)
\end{bmatrix}
\begin{bmatrix}
V(s_1) \\
V(s_2) \\
\vdots \\
V(s_n)
\end{bmatrix}
\end{aligned}
$$

我们可以直接将矩阵运算写成转移要求解，得到以下解析解：

$$
V = R + \gamma P V
$$

$$
(I - \gamma P)V = R
$$

$$
V = (I - \gamma P)^{-1} R
$$

以上解析解的计算复杂度是 $O(n^3)$，主要的开销来自矩阵取逆。因此这种方法只适用于状态较小的马尔可夫奖励过程。求解较大规模的马尔可夫奖励过程中的价值函数时，可以使用动态规划（dynamic programming）算法，蒙特卡洛方法（Monte-Carlo method）和时序差分（temporal difference）。

## 马尔科夫决策过程

马尔科夫过程是自发随机过程，而智能体的动作可能会影响环境。我们在 MRP 基础上加入动作，就得到了 MDP。一个马尔可夫决策过程由 $(S, A, P, r, \gamma)$ 构成，其中组成元素的含义如下所示。

- $S$ 是状态的集合；
- $A$ 是动作的集合；
- $\gamma$ 是折扣因子；
- $r(s, a)$ 是奖励函数，此时奖励可以同时取决于状态 $s$ 和动作 $a$，在奖励函数只取决于状态 $s$ 时，则退化为 $r(s)$；
- $P(s'|s, a)$ 是状态转移概率，表示在状态 $s$ 执行动作 $a$ 后到达状态 $s'$ 的概率。


注意，在上面 MDP 的定义中，我们不再使用类似 MRP 定义中的状态转移矩阵方式，而是直接表示成了状态转移函数。这样做一是因为此时状态转移与动作也有关，变成了一个三维数组，而不再是一个矩阵（二维数组）；二是因为状态转移函数更具有一般意义，例如，如果状态集合不是有限的，就无法用数组表示，但仍然可以用状态转移函数表示。

### 策略

智能体选择动作的策略（Policy）通常用字母 $\pi$ 表示。策略 $\pi(a|s) = P(A_t = a|S_t = s)$ 是一个函数，表示在状态 $s$ 下选择动作 $a$ 的概率。策略可以分为两类：

- **确定性策略（deterministic policy）**：当一个策略是确定性策略（deterministic policy）时，它在每个状态下只选择一个确定的动作。
- **随机策略（stochastic policy）**：在随机策略（stochastic policy）中，$\pi(a|s)$ 表示从状态 $s$ 选择动作 $a$ 的概率。随机策略允许在给定的状态下选择多个可能的动作，每个动作有一定的概率分布。

在马尔可夫决策过程（MDP）中，由于马尔可夫性质的存在，策略只需要基于当前状态 $s$ 来决定动作 $a$，而不依赖于过去的动作或状态序列。因此，MDP 中的策略可以看作是状态到动作的映射函数 $\pi: S \to A$。在 MDP 中，当前状态的转移概率由策略 $\pi$ 和状态转移概率 $P(s'|s, a)$ 共同决定。

### 价值函数

在强化学习中，策略 $\pi$ 的目标是最大化长期累积奖励（discounted cumulative reward）。因此，策略的优化问题可以表示为寻找最优策略 $\pi^*$，以最大化期望回报 $V^\pi(s)$ 或 $Q^\pi(s, a)$。

不同于 MRP，在 MDP 中，由于动作的存在，我们额外定义一个动作价值函数（action-value function）。我们用 $Q^{\pi}(s, a)$ 表示在 MDP 遵循策略 $\pi$ 的，对于当前状态 $s$ 执行动作 $a$ 得到期望回报：

$$
Q^{\pi}(s, a) = \mathbb{E}_{\pi}[G_t | S_t = s, A_t = a]
$$

状态价值函数和动作价值函数之间的关系：在使用策略 $\pi$ 中，状态 $s$ 的价值等于在该状态下基于策略 $\pi$ 采取所有动作的概率与相应动作的价值采用求和的结果：

$$
V^{\pi}(s) = \sum_{a \in A} \pi(a|s) Q^{\pi}(s, a)
$$

使用策略 $\pi$ 的，状态 $s$ 下采取动作 $a$ 的价值等于立即奖励加上经过衰减后的所有可能的下一状态的状

态转移概率与相应的价值的乘积：

$$
Q^{\pi}(s, a) = r(s, a) + \gamma \sum_{s' \in S} P(s'|s, a) V^{\pi}(s')
$$

### 贝尔曼期望方程

基于此，我们继续推导贝尔曼期望方程（Bellman Expectation Equation），看着很复杂，其实就是基于基础定义相互展开：

$$
V^{\pi}(s) = E_{\pi}[R_t + \gamma V^{\pi}(S_{t+1})|S_t = s]
$$

$$
V^{\pi}(s) = \sum_{a \in A} \pi(a|s) \left( r(s,a) + \gamma \sum_{s' \in S} p(s'|s,a)V^{\pi}(s') \right)
$$

$$
Q^{\pi}(s,a) = E_{\pi}[R_t + \gamma Q^{\pi}(S_{t+1}, A_{t+1})|S_t = s, A_t = a]
$$

$$
Q^{\pi}(s,a) = r(s,a) + \gamma \sum_{s' \in S} p(s'|s,a) \sum_{a' \in A} \pi(a'|s') Q^{\pi}(s', a')
$$