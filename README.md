# Score-card-model

评分卡模型是常用的金融风控手段之一。我们根据客户的各种属性和行为数据，利用信用评分模型，对客户的信用进行评分，从而决定是否给予授信，授信的额度和利率，减少在金融交易中存在的交易风险。按照不同的业务阶段，可以划分为三种评分卡：

$\quad\cdot$ 贷前：申请评分卡（Application score card），称为A卡

$\quad\cdot$ 贷中：行为评分卡（Behavior score card），称为B卡

$\quad\cdot$ 催收评分卡（Collection score card），称为C卡

评分卡模型开发步骤：

$\quad$ 1.数据获取，包括获取存量客户及潜在客户的数据存量客户，已开展融资业务的客户，包括个人客户和机构客户；潜在客户，将要开展业务的客户。

$\quad$ 2.EDA，获取样本整体情况，进行直方图、箱形图可视化。

$\quad$ 3.数据预处理，包括数据清洗、缺失值处理、异常值处理。

$\quad$ 4.变量筛选，通过统计学的方法，筛选出对违约状态影响最显著的指标。主要有单变量特征选择和基于机器学习的方法。

$\quad$ 5.模型开发，包括变量分段、变量的WOE（证据权重）变换和逻辑回归估算三个部分。

$\quad$ 6.模型评估，评估模型的区分能力、预测能力、稳定性，并形成模型评估报告，得出模型是否可以使用的结论。

$\quad$ 7.生成评分卡（信用评分），根据逻辑回归的系数和WOE等确定信用评分的方法，将Logistic模型转换为标准评分的形式。

$\quad$ 8.建立评分系统（布置上线），根据生成的评分卡，建立自动信用评分系统。

从上面的步骤中，涉及一些重要的概念，比如WOE等，我们将在后面一一介绍。
### WOE编码
WOE(Weight of Evidence)，翻译为“证据权重”。它是自变量的一种编码，常用于特征变换，用来衡量自变量与因变量的相关性。计算公式如下：
\begin{equation}
woe_i = \ln(\frac{B_i/B_T}{G_i/G_T})
\end{equation}
其中，$B$代表风险用户，$G$代表正常用户，$i$代表变量的$i$个bin或group，$B_T=\sum_{i}B_i$,$G_T=\sum_{i}G_i$。

下面的数据表展示了这些$woe_i$的计算。
\begin{equation}
\begin{array}{|r|r|r|r|r|r|}
\hline \text { 记录value } & \begin{array}{r}
\text { Label=1 } \\
\text { 个数 }
\end{array} & \begin{array}{r}
\text { Label=0 } \\
\text { 个数 }
\end{array} & \begin{array}{r}
\text { Label=1 } \\
\text { 的比率 }
\end{array} & \begin{array}{r}
\text { Label=0 } \\
\text { 的比率 }
\end{array} & \text { Woe } \\
\hline \text { Value1 } & 40 & 10 & \begin{array}{r}
40 /(40+25)= \\
62 \%
\end{array} & \begin{array}{r}
10 /(10+25 \\
1=28 \%
\end{array} & \begin{array}{r}
\operatorname{Ln}(62 \% / 28 \%)= \\
\operatorname{In}(2.2)=0.79
\end{array} \\
\hline \text { Value2 } & 25 & 25 & \begin{array}{r}
25 /(40+25)= \\
38 \%
\end{array} & \begin{array}{r}
25 /(10+25 \\
1=72 \%
\end{array} & \begin{array}{c}
\operatorname{Ln}(38 \% / 72 \%)= \\
\operatorname{In}(0.52)=-0.64
\end{array} \\
\hline
\end{array}
\end{equation}
一般地，WOE差异越大，对风险的区分越明显。

关于WOE计算，如果是连续型数据，分成N个bins，对于分类型变量保持类别group不变。在计算的过程中，需要注意一下几点：

$\quad$1.每个bin or group记录不能过少，至少有5%的记录

$\quad$2.不要用过多的bin or group，会导致不稳定性

$\quad$3.对bin or group中全为0或者1的特列，用下面修正的woe防止分母为0的情况。
\begin{equation}
woe_i = \ln(\frac{(B_i+0.5)/B'_T}{(G_i+0.5)/G'_T})
\end{equation}

此时，$B'_T=\sum_{i}B_i+0.5$,$G'_T=\sum_{i}G_i+0.5$。

### IV
IV（Information Value),woe只考虑了风险区分的能力，没有考虑能区分的用户有多少。IV衡量一个变量的风险区分能力,即衡量各变量对y的预测能力，用于筛选变量。

\begin{equation}
IV_i=(\frac{B_i}{B_T}-\frac{G_i}{G_T})\ln(\frac{B_i/B_T}{G_i/G_T})
\end{equation}
\begin{equation}
IV=\sum_{i}IV_i
\end{equation}

IV是与WOE密切相关的一个指标，在应用实践中，评价标准可参考如下：
\begin{equation}
\begin{array}{|c|c|}
\hline \text { IV范围 } & \text { 变量评估 (预测效果) } \\
\hline \text { 小于0.02 } & \text { 几平没有 } \\
\hline 0.02\sim 0.1 & \text { 弱 } \\
\hline 0.1 \sim 0.3 & \text { 中等 } \\
\hline 0.3 \sim 0.5 & \text { 强 } \\
\hline \text { 大于0.5 } & \text { 难以置信，需要确认 } \\
\hline
\end{array}
\end{equation}
一般我们根据IV取值在选择变量。通常筛选IV值>0.1的特征字段。

### 逻辑回归
在通过IV选择了合适的特征，并计算了各自特征的woe值，我们就根据所得到的特征的woe值构造逻辑回归模型，即
\begin{equation}
\ln\frac{p}{1-p}=\theta_1x_1+\cdots+\theta_nx_n
\end{equation}
其中，$p$为客户逾期的概率。

最后，得到相应的系数$\theta_i$。

## 评分卡模型转换
设p为客户违约的概率，那么正常的概率为1-p，则
$$
O d d s=\frac{p}{1-p}
$$
评分卡的分值计算，可以通过分值表示为比率对数的线性表达式来定义，即
$$
\text { Score }=A-B * \ln (O d d s)
$$
常数A、B可以通过将两个假设的分值带入计算得到：
1) 基准分, 即给某个特定的比率 $\theta_{0}$ 时，预期的分值为 $P_{0}$，通常，业内的基准分为500/600/650。

2) PDO（ point of double odds ），即比率翻倍时的分数。比如，odds翻倍时，分值减少50。即比率为 $2 \theta_{0}$ 的点的分值应该为 $P_{0}-P D O$

由以上两个规则，我们可以得到：
$$
\begin{array}{l}
P_{0}=A-B* \ln \left(\theta_{0}\right) \\
P_{0}-P D O=A-B* \ln \left(2 \theta_{0}\right)
\end{array}
$$
求解得：
$$
\begin{array}{l}
B=\frac{P D O}{\ln 2}   \\
A=P_{0}+B^{*} \ln \left(\theta_{0}\right)
\end{array}
$$
通过上面的逻辑回归模型，求得的违约概率： $p=\frac{1}{1+e^{-} \theta^{T} x}$，将公式变化下，可得
$$
\ln \left(\frac{p}{1-p}\right)=\theta^{T} x \quad, \text { 即 } \ln (o d d s)=\theta^{T} x
$$
评分卡的逻辑是Odds的变动与评分变动的映射，即把Odds映射为评分，因此，可以得到
$$
\text { Score }=A-B\left\{\theta_{0}+\theta_{1} x_{1}+\ldots+\theta_{n} x_{n}\right\}
$$
以上就是评分卡模型的转换规则。
以下，我们以基准分600，此时对应的的odd为2%，并假设odds翻倍时，分值减少50，在这上面构建评分卡。
