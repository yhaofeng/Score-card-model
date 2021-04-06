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
其中，$B$代表风险用户，$G$代表正常用户，$i$代表第变量的$i$个bin或group，$B_T=\sum_{i}B_i$,$G_T=\sum_{i}G_i$。

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
一般我们根据IV取值在选择变量。
