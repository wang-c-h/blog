---
title: "Stoat_论文阅读笔记"
date: 2023-06-27T17:23:06+08:00
description: Guided, Stochastic Model-Based GUI Testing of Android Apps
tags: [Paper,GUI_Testing,Android]
categories: Paper
math: true
# draft: true
---

[原文地址](https://doi.org/10.1145/3106237.3106298)  
## 论文信息
软工A类会议  
ESEC/FSE'17: Joint Meeting of the European Software Engineering Conference and the ACM SIGSOFT Symposium on the Foundations of Software Engineering
## 作者信息
Ting Su, Guozhu Meng, Yuting Chen, Ke Wu  
 Weiming Yang, Yao Yao, Geguang Pu, Yang Liu, Zhendong Su   
 School of Computer Science and Software Engineering, East China Normal University, China   
 School of Computer Engineering, Nanyang Technological University, Singapore   
 Department of Computer Science and Engineering, Shanghai Jiao Tong University, China   
 Department of Computer Science, University of California, Davis, USA

## 摘要
移动应用无处不在，在复杂的环境中运行，并且是在上市时间的压力下开发的。确保其正确性和可靠性因此成为一个重要的挑战。本文介绍了一种新的引导方法**Stoat**，用于在Android应用程序上进行基于随机模型的测试。stoat分为两个阶段:  
- (1)给定一个应用程序作为输入，它使用由**加权UI探索策略**增强的**动态分析**和**静态分析**来逆向 应用程序GUI交互的随机模型;  
- (2)采用Gibbs抽样对随机模型进行**迭代突变/改进**，并指导从突变模型生成的测试，以实现高代码和模型覆盖率，并展示多样化的序列。  

在测试过程中，随机注入系统级事件，进一步提高测试效率。Stoat在93个开源应用程序中进行了评估。  

结果表明:  
- (1)与现有建模工具相比，由Stoat生成的模型覆盖的代码多17% ~ 31%;  
- (2)与Monkey和Sapienz这两种最先进的测试工具相比，Stoat检测到的独特崩溃多出3倍。
- 此外，Stoat测试了1661个最受欢迎的Google Play应用，并检测到2110个以前未知的独特崩溃。  

到目前为止，已经有43个开发者回应说他们正在调查我们的报告。报告的崩溃中有20个已经确认，8个已经修复。

## 第一章 介绍

近年来，移动应用程序变得无处不在，数量急剧增加。
据最近的统计数据[29]显示，每个月有超过5万的新Android应用提交到Google Play。
然而，如何保证它们的质量是一个挑战。  
首先，它们是**以事件为中心**的程序，具有**丰富的图形用户界面(gui)**，并与**复杂的环境(例如，用户、设备和其他应用程序)**交互。  
其次，它们通常是在上市时间的压力下开发的，因此在发布之前可能没有进行充分的测试。
在执行测试时，开发人员倾向于执行那些他们认为**重要的功能或使用场景**，但可能会错过他们设计的测试无法暴露的错误。

为了应对这一挑战，已经提出了许多技术[2,4,6,39 - 41]。
符号执行[4,60]在**源代码级别跟踪事件的起源和句柄**，并通过**详尽地探索程序路径**来生成测试。
随机测试[28,39]通过**生成随机事件流**来模糊应用程序。
进化算法[40,41]通过**随机突变和交叉事件序列**来生成测试，以实现其优化目标。

**基于模型的测试(MBT)**[23,54]是另一种流行的自动化GUI测试方法，它通过模型**抽象应用程序的行为**，然后从中**派生测试**来验证应用程序。
然而，从模型中详尽地生成测试来验证应用程序的行为是非常困难的。  
例如，Bites[20]是一个简单的食谱应用程序(如图2a所示)，它有1027行代码，它的模型有21个状态和70个转换(由我们的方法生成)。
该模型可生成6个单事件序列，36个双事件序列，567K个三事件序列，执行时间较长。  
由于这种**路径爆炸**问题，导出所有潜在的测试并执行它们实际上是不可行的。
因此，传统的MBT技术选择生成**随机测试**，并使用**模型级覆盖标准**[2,43]\(例如，覆盖所有转换)作为测试目标。  
然而，如果没有**强有力的指导**，这样的测试对于检测bug来说往往是多余的和无效的。  
此外，以往对基于模型的GUI测试的研究[1,2,7,15,18,31,32,42,42,48,57,67]在测试过程中只考虑了**ui级事件**(如点击、编辑)，而忽略了**系统级事件**(如屏幕旋转、来电)。
如果不结合这两种类型的事件，MBT的有效性可能会由于不充分的测试而进一步受到限制。  

此外，大多数应用程序在实践中没有模型。
尽管在手工或自动构建模型来表示GUI交互方面做了很多努力[1,2,19,42,57,67]，但由于*UI探索的不完整*，MBT的有效性仍然有限。
例如，最近一项广泛的研究[16]表明，**现有GUI探索工具生成的模型覆盖率相当低(仅为Monkey覆盖率的一半)**。

上述挑战强调了开发有效的基于模型的测试技术以释放其潜力的重要性。
为此，我们提出了一种新的基于随机模型的测试方法Stoat (stochastic model App Tester)，以改进Android应用程序的GUI测试。  
它旨在从GUI模型中**彻底测试应用程序的功能**，并通过**强制各种用户/系统交互**来验证应用程序的行为[66]。
给定一个应用程序作为输入，Stoat分两个阶段运行。  
首先，它从应用程序中生成一个随机模型来描述其GUI交互。
在我们的设置中，应用程序的随机模型是一个有限状态机(FSM)，其边 与测试生成的概率相关。

特别是，Stoat采用 通过**加权UI探索策略**和**静态分析**来增强的**动态分析技术**，以**探索应用程序的行为**并构建随机模型。

其次，Stoat对随机模型进行**迭代突变**，并从模型突变中生成测试。
通过*扰动*概率，Stoat能够生成**带有各种事件组合**的测试，以充分测试GUI交互，并有意将测试转向**较少的路径**以检测深层漏洞。  
特别是，Stoat采用了一种受 马尔可夫链蒙特卡罗(Markov Chain Monte Carlo, MCMC)采样启发的引导搜索算法来搜索“好”模型(在4.2节中讨论)——期望派生的测试多样化，并实现高代码和模型覆盖率。

此外，Stoat采用了一种简单而有效的策略来增强MBT:在MCMC采样期间将各种**系统级事件***随机*注入UI测试中[44]。
它避免了将系统级事件合并到行为模型中的复杂性，并进一步施加外部环境的影响来检测复杂的bug。

总的来说，本文做出了以下贡献:

- **模型构建**。Stoat采用动态分析技术，通过加权UI探索策略和静态分析来增强，有效地探索应用程序行为并构建模型。该增强有助于实现更完整的模型，与现有GUI探索工具生成的模型相比，它可以覆盖17 ~ 31%的代码。

- **错误检测**。我们采用Gibbs抽样，MCMC抽样的一个实例，来指导基于随机模型的检验。在这93款开源应用中，Stoat达到了令人满意的覆盖率，检测到的独特崩溃比最先进的测试工具Monkey和Sapienz多3倍，这清楚地证明了我们方法的好处。特别是，通过在MBT期间注入系统级事件，Stoat检测到91个以上的崩溃。

- **实施和评估**。我们已经将Stoat作为一种自动化工具，并在Google Play的1661款最受欢迎的应用中对其进行了进一步评估。Stoat从691个应用程序中检测到2110个独特的崩溃。到目前为止，20个崩溃被确认为真正的故障，8个已经修复。结果表明，Stoat在测试实际应用程序时是有效的。



![Figure 1: Stoat’s workflow](/img/stoat/figure1.png)

## 2方法概述

Stoat采用独特的两阶段流程来测试应用程序。
图1显示了它的工作流程。

**阶段1:模型构建** 

Stoat首先构建一个**随机有限状态机(FSM)**来描述应用程序的行为。
它使用动态分析技术，通过加权UI探索策略(图1中的第2步)进行增强，以有效地探索应用行为。
它通过分析应用程序页面的**UI层次结构**来**推断**输入事件，并动态地**优先考虑它们的执行**以**最大化代码覆盖率**。
此外，为了识别一些可能丢失的事件，执行静态分析(第1步)来扫描应用程序代码中**已注册的事件侦听器**。
Stoat记录在探索过程中所有UI事件的**执行频率**，然后使用它们来生成模型中转换的**初始概率值**。细节将在第3节中解释。

**阶段2:模型突变、测试生成和执行**  

为了彻底测试一个应用程序，Stoat利用阶段1的模型，迭代地指导测试生成，以获得高覆盖率，并展示不同的事件序列。  
具体来说，Stoat作为一个循环工作:  
-    随机改变当前随机模型的转移概率(步骤4)，
-    从模型w.r.t.概率(步骤5)中生成测试，
-    随机将系统级事件(在步骤3中通过静态分析分析)注入这些ui级测试中以增强MBT(步骤6)，
-    在应用程序上重播(步骤7)并收集测试结果，例如代码和模型覆盖率以及事件序列多样性(步骤8)。
-    根据测试结果，Stoat利用**Gibbs抽样**来决定是否接受或拒绝新提出的模型(步骤9)，接受**目标值更好**的模型进行下一次**突变**和**抽样迭代**; 否则，将以一定概率被拒绝，以避免局部最优(如果被拒绝，将重用原模型)。
-    一旦检测到任何错误(即，崩溃或无响应)，将执行进一步的分析，以使用相应的测试诊断错误(步骤10)。

细节将在第4节中解释。




**一个说明性示例** 

Bites[20]是一个简单的食谱应用程序(如图2a所示)，支持食谱创建和分享。 
- 用户可以通过单击Recipes页面(页面A)中的插入菜单项来创建菜谱。 
- 当用户点击食谱名称时，应用程序会导航到配料页面(页面b)，用户可以在这里查看或添加配料，通过短信分享配料，或者将配料添加到购物清单中。 
- 用户还可以切换到Method页面(页面c)，在这里可以查看烹饪方法。
- 通过单击插入菜单项，用户可以填写步骤号和方法(第d页)。  

图2b显示了为Bites构建的应用模型的一部分，其中每个节点表示一个应用状态，每个边表示一个状态转换(与一个概率值相关联)。

例如，当e6出现时(即，单击Recipes页面上的配方项)，可以导航到Ingredients，概率为p6。 Stoat从该模型生成ui级测试，并在Gibbs采样期间随机地将系统级事件注入其中。 例如，Bites可以通过短信和浏览器激活，阅读别人分享的食谱。 在测试期间，Stoat通过向Bites发送特定的Broadcast intent来模拟这些系统级事件。

![Figure 2: Exmple app Bites and its app model](/img/stoat/figure2.png)


## 3基于随机模型的测试
### 3.1随机模型
Stoat使用随机有限状态机(FSM)模型来表示应用程序的行为。在形式上，将随机FSM定义为5元组$ M = (Q， Σ，δ， S_0, F) $ ， 
其中: 
- $Q$和$Σ$分别为应用程序状态集和输入事件集，
- $S_0∈Q$为应用程序初始状态集，
- $F⊆Q$为最终状态集，
- $δ: Q × Σ→P(Q ×[0,1])$ 为概率转移函数。$P(·)$是幂集算子，
- 每个转换都是$(s,e， (s'， P))$的形式，这意味着事件e触发从$s$到$s'$的状态转换的概率是$P$。

让一个应用程序状态$s$有$k$个事件转换比如: $ e_1，…e_i……，e_k, 1≤i≤k $ ， $P_i$为$e_i$的概率值。对于$s$, $\displaystyle \sum_{i=1}^{k}p_{i} = 1$成立。  

&emsp;&emsp;在我们的设置中，**一个应用状态被抽象为一个应用页面**(表示为一个**小部件层次树**，其中非叶节点表示布局小部件(如LinearLayout)，叶节点表示可执行小部件(如Button));  
&emsp;&emsp;当页面的结构(和属性)发生变化时，会创建一个新的状态(例如，在图2a中，Recipes页面和Ingredients页面对应于两个应用状态)。如果应用程序退出/崩溃，结束状态被视为最终状态(例如，页面f)。  
&emsp;&emsp;**边** 对应于表示**UI动作的输入事件e**(例如，点击，编辑)。  
&emsp;&emsp;一个应用程序**通过处理一个输入事件从一个状态移动到另一个状态**。例如，当用户按下食谱页面上的菜单键时，将弹出一个菜单，并创建一个新的应用程序状态(对应于食谱菜单)。给每个*转换*$e$赋一个概率值$p$，表示$e$在测试生成中的选择权值。初始概率值由模型构建过程中每个事件的执行频率决定。$p$初始分配为$e$观察到的执行次数与所有事件 $s (e∈s)$的总执行次数之比。

**从模型生成测试。**  Stoat采用概率策略从随机模型生成事件序列。它从进入状态$s_0$开始，按照概率值从对应的app状态中选择一个事件，直到达到最大序列长度或结束状态。事件概率值越高，该事件被选择的可能性越大。

### 3.2模型构造
Stoat采用动态UI探索策略，并辅以静态分析，为待测应用构建随机模型。  
**动态UI探索。**  
&emsp;&emsp;应用程序可以在具有不同ui的各种页面之间导航，以提供其功能。为了有效地构建更完整的应用行为模型，我们从前10个类别(如教育、商业、工具)中调查了50款最受欢迎的Google Play应用，并手动探索了尽可能多的功能。最后，我们总结了三个对提高勘探性能至关重要的观察结果，它们构成了我们加权UI勘探策略的基础:
- *事件执行频率*。所有UI事件都有机会执行。一个事件执行的频率越低，就越有可能在随后的探索中被选中。
- *事件类型*。不同类型的事件并不是平等选择的。例如，与普通的UI事件(如点击)相比，导航事件(如后退、滚动和菜单)被赋予不同的优先级，以确保它们在正确的时间被触发，否则它们可能会极大地破坏探索效率。
- *未访问的子组件的数量*。如果一个事件在下一页上请求更多新的UI小部件，它将被优先处理，因为应该在具有新功能的页面上花费更多的精力。

为了实现这些规则，Stoat为每个事件分配了一个**执行权重**，并在运行时**动态调整**。事件的权重定义为:

$$
{ execution\underline{\ }weight }=\frac{\alpha * T_e+\beta * C_e}{\gamma * F_e} \tag{1}
$$  
其中
- $T_e$由其事件类型决定(1代表普通UI事件，0.5代表后退和滚动，2代表菜单)，
- $C_e$表示未访问的子部件的数量，
- $F_e$是其历史执行频率，
- $α， β$和$γ$是权重参数[^2]
[^2]: 我们在调查50款Google Play应用时调整了权重参数，但在最终评估过程中，所有应用的权重参数都保持不变。


算法1概述了随机模型的构建过程。  
它以一个应用程序作为输入，并输出其对应的模型m。  
- 算法根据当前应用程序页面$s$上的UI小部件推断出可调用的事件$E$，然后将它们添加到动态分析期间存储所有事件的事件列表中(第6-7行)。  
&emsp;&emsp;例如，在图2a中的应用页面Insert Method (d)中，由于有两个edittext(“Step number”和“Method”)和两个button(“Ok”和“Cancel”)(它们的可点击属性为true)， Stoat推断出四个事件，即编辑(“Step number”)、编辑(“Method”)、点击(“Ok”)和点击(“Cancel”)。
- 在每次执行之前，列表中所有事件的权重将按照公式(1)(第8-9行)进行更新。
- 选择在页面s上具有最大权重(通过函数getMaxWeightEvent计算)的事件e来执行(第10-11行)。
- 函数expandFSM接受返回的状态和执行的事件来构造模型(第15行)。
- 最后，根据观察到的执行次数，为M的所有 转换 分配初始概率值(第17、18-24行)。
- 在UI探索过程中，如果在一些事件被执行后，应用程序进入未知状态(例如，应用程序崩溃，退出，变得无响应，或导航到一个不相关的应用程序)，将执行restoreApp以重新启动应用程序或导航到上一页(第12-14行)。这个未知状态被作为*最终状态*。这个事件将被添加到$T$中，并从以后的执行中排除(第10行)，以防止影响建模效率。


![Algorithm 1: App Stochastic Model Construction](/img/stoat/algorithm1.png)


**静态事件识别。**  动态分析技术通常从UI层次结构推断事件(由Android UIAutomator[27]转储)，它只捕获静态GUI布局信息。然而，它可能会错过一些动态事件，例如，Activity的菜单操作只能通过按下菜单键来调用，或者一些在应用程序代码中编程的事件，例如，在TextView上注册的longClick操作。  
&emsp;&emsp;为了进一步提高建模能力，算法1使用静态分析来识别这些被动态分析遗漏的**潜在事件**(第4行)。它通过扫描应用程序代码中的**事件侦听器**来检测事件，然后通过它们的**唯一资源id**将这些事件与运行时观察到的小部件关联起来。Stoat**检测在UI小部件(例如，setOnLongClickListener)上注册的事件，并通过重写类方法(例如，onCreateOptionsMenu)实现**。

**压缩模型。** 
一个应用的状态和转换的数量可以很大，甚至是无限的[7,15,42,50,67]。为了提高测试效率，Stoat通过只**将结构上不同的页面识别为不同的状态**来压缩模型，并合并相似的页面。
- (1)将状态的**层次树**编码为字符串，并将其转换为**哈希值**，以便有效地检测重复状态;
- (2)**次要的UI信息**，例如文本更改(例如TextViews/EditTexts的内容)和UI属性更改(例如RadioButtons/ checkboxes的checked属性)，**在不创建新状态的情况下被省略**;
- (3) **listview[^3]只区分为空和非空**。例如，在图2a中，应用程序页面(d)和(e)对应于相同的状态，因为只有EditTexts中的内容不同。

[^3]: listview 是常用的控件，以列表的形式展示具体数据内容。是android系统提供的一种列表显示控件，可以显示常见的列表形式。

## 4指导随机突变模型
Stoat利用Gibbs采样来指导随机模型的变异，从而生成一组具有**代表性**的检验。在我们的设置中，我们打算从可以实现我们期望的目标的测试套件中 找到“好的”模型。我们把这个问题看作是一个**由适应度函数引导的优化过程**。  
### 4.1 Gibbs Sampling
Gibbs Sampling[5,64]是Metropolis-Hastings算法(贝叶斯算法)[65]的一种特例。Metropolis-Hastings算法是Markov Chain Monte Carlo (MCMC)(马尔可夫链蒙特卡罗)方法中的一种[14]，MCMC是一类**从期望的概率分布$p(x)$中抽取样本**的算法，对其直接抽样是困难的。  
它从一个与$p(x)$ 的密度成**正比**的函数 $λ$迭代地生成样本。采样过程产生一个马尔可夫链，其中**当前样本的选择仅取决于前一个样本**。经过多次迭代，这些样本可以**非常接近期望的分布p(x)**，从而允许**从更重要的区域(密度更高的区域)生成更多的样本**。

在采样过程中，一个候选样本会以一定的概率被接受或被拒绝，这个概率是通过比较当前和候选样本之间的λ值来确定的。形式上，在第t次迭代中，候选样本$x^'$与提议密度$q(x^'|x_t)$一起生成，其接受率为
$$
AcceptRatio(x^') = min(1,\frac{p(x^') \* q(x^' \mid x_t)}{p(x_t) \* q( x_t \mid  x^')}) \tag{2}
$$

通常情况下,q是选为一个对称函数。因此公式$(2)$可以简化
$$
AcceptRatio(x^') = min(1,\frac{p(x^')}{p(x_t)}) \tag{3}
$$


### 4.2目标函数

我们设计了一个目标函数，支持能够实现高覆盖率和包含不同事件序列的测试套件。这样的测试套件有望触发更多的程序状态和行为，从而增加检测错误的机会。

我们的目标函数结合了三个度量，即**代码覆盖率、模型覆盖率和测试多样性**。
- 代码覆盖率[56,71]衡量应用程序代码测试的彻底程度;
- 模型覆盖率[43]衡量应用程序模型被覆盖的完整程度，
- 测试多样性[49]衡量测试套件中事件序列的多样性，这是覆盖率指标的重要补充。

目标函数形式化为:
$$ f (T) = α ∗ CodeCoverage (T) + β ∗ ModelCoverage (T) + γ ∗ TestDiversity(T)$$

其中$T$是由随机模型M生成的测试集，$α， β，γ$是这些度量的权重[^4]。

[^4]: 在评估中，α， β和γ的值分别设置为0.4,0.2和0.4，这在没有任何调整的情况下给予代码覆盖率和测试多样性更多的权重。


这里，代码覆盖率被计算为开源应用程序的**语句覆盖率**[52]，或闭源应用程序的**方法覆盖率**[3]。  
对于模型覆盖，我们使用**边覆盖**来计算覆盖了多少事件。

对于测试多样性，我们设计了一个轻量级但有效的度量来评估测试用例的多样性。  
设T是一个N大小的测试集${l_1，…，l_i，……l_N}$，  其中$l_i$是事件序列。一个$k$长度的事件序列$l$可以表示为$e_1→…e_i→…e_k$，$e_i$是一个事件。  
关键思想是计算这N个序列的**质心**[11]，并将它们与“质心”的**欧氏距离之和**作为T的多样性值。直观地看，距离越大，T的多样性越大。

首先，我们使用一个二值向量来表示每个序列。假设模型M有$N_e$个独立事件。事件$e$可以表示为N维向量 $\vec{e}=(ε_1，…，ε_i，……，ε_{N_e})$，如果事件$e$是事件列表中的第i个事件，则$ε_i$为0，否则为1(这样设置值是为了避免两个向量的正交性)。  
其次，设$l$是一个长度为n的序列，表示为$\vec{l}_{[n]} = e_1→e_2···→e_n$. 根据$l$的事件及其顺序，我们使用下面的函数将$l$递归地变换成一个向量$\vec{l}$:  
![gongshi](/img/stoat/gongshi1.png)  


<!-- $$
\vec{l}_{[i]} =  cos_sim ( \vec{l}_{[i-1]} , \vec{l}_{[i-1]} + \vec{e}_{[i]} )  · ( \vec{l}_{[i-1]} + \vec{e}_{[i]} )
$$   -->

<!-- $$
\vec{l}_{[i]} = {cos_sim}( l,l,e )·(l+e)
$$   -->


其中，$\vec{l}_{[i]}$ 是

$\vec{l}_{[n]}$ 的i-长度前缀，  

即$e1→e2…→ei, 且 \vec{l}_{[1]} = \vec{e}_1 $ 

我们利用$l_{[i−1]}和l_{[i−1]}+ \vec{e}_i$ 之间的余弦相似度[63]，  

即$cos_sim(l∈[i−1]，l∈[i−1]+ e∈i)$，  
将阶关系  $l_{[i−1]}→e_i$  编码到向量$\vec{l}$中。  
通过这种方式，我们可以在计算测试多样性时同时考虑所包含的事件及其顺序。  

在我们得到$T = {l_1…，l_i，…，l_N}$ 的向量集之后，则T的质心$\vec{C}$可以计算为:  

$ \vec{C} = \frac{\sum_{i=1}^{N}\vec{l}_i}{N} $ 。最后，测试多样性由公式计算:  

$$
TestDiversity(T) = \frac{ \sum_{i=1}^{N}d(\vec{l}_i,\vec{C})}{N}
$$

为了适应目标函数，我们通过除以$N_e$将测试多样性缩放到[0,1]的范围内。

### 4.3 Gibbs Sampling 引导模型突变

在我们的问题中，我们选择Gibbs抽样而不是标准的Metropolis-Hastings算法，因为它是**专门为p(x)是多个随机变量的联合分布**时绘制样本而设计的。特别地，我们让所有的转移概率都是**随机变量**，并通过迭代突变来绘制样本。它允许从具有“良好”随机模型的区域更频繁地抽取样本。我们假设从优化模型得到的测试可以达到更高的目标值。因此，我们按照常用的方法[25,53]，将目标概率密度函数p(x)设为  

$$ p(M) = \frac{1}{Z} exp(−β ∗ f (T_M )) $$

式中，
- M为随机模型，
- Z为归一化配分函数，
- β为常数，
- $T_M$为当前模型M生成的测试集，
- f为目标函数。  

由式(3)可知，新提出的模型M′的接受比[^5]可简化为

$$ AcceptRatio(M→M′) = min(1,exp(−β ∗ ( f (T_M) − f (T′_{M′})))) $$

[^5]: 在我们的问题中，β被经验地选择为-0.33，这是为了有效地区分接受率。

**随机模型突变。**  算法2给出了 Gibbs抽样 指导的测试算法。  

- 搜索空间是随机模型域，其中每个样本都是一个随机模型。在每次迭代中，通过改变当前模型M中的 转换 概率值，生成一个新的候选模型M'。  

- 让应用程序有n个应用程序状态，$s_1，…，s_i，……，s_n$，每个状态$s_i(1≤I≤n)$有k个事件转换，$e_1，…，e_j……e_k$。Stoat随机决定是否改变每个应用状态si的转换概率(第3-9行)。
- 如果选择状态$s_i$, Stoat会将转换$e_j$的原始概率值$p_j$随机变异为新的概率值$p^'_j$，这是$p_j$ +mSize或$p_j$ -mSize的结果。直观的感觉是，新生成的概率值$p^'_j$在pj附近(可以比pj高，也可以比pj低)，因此新模型M'可以生成与M截然不同的事件序列。
    - 为了加快收敛速度，在实现中将mSize设置为固定值(例如0.1)。
    - 对于剩余的转移概率，应用类似的程序，但约束$p^'_1+…+p^'_j +…+p^'_k = 1$仍然成立。
    - 对于其他未选择的状态，保持它们的转移概率不变，使得新模型M'有条件地依赖于那些突变的概率值
- 为了模拟环境的相互作用，Stoat随机地将系统级事件注入到从突变模型M'生成的测试T'中(第11-15行)。然后在应用程序上重播T'以验证其行为。
- T′的测试结果用于确定M′的接受率。
    - 如果T'可以改善目标值，则M'将在下一次迭代中发生突变(第19-20行)。
    - 否则，原来的M就发生了变异。
- 该算法一直持续到测试预算耗尽为止。
- 如果应用程序崩溃或变得无响应，则记录可疑的错误(第17-18行)，并转储相应的错误堆栈以进行错误诊断。


![Algorithm 2: Gbibbs sampling Guided GUI Testing](/img/stoat/algorithm2.png)

### 4.4 系统级事件
为了将系统级事件合并到基于模式的测试中，Stoat采用了一种简单而有效的策略，将它们随机注入到ui级事件序列中。这种策略避免了将系统级事件包含到行为模型中的复杂性，并进一步将两种类型的事件交织在一起，以检测复杂的错误。  
目前，Stoat支持三种系统级事件源:
- (1)5个用户操作(即屏幕旋转，音量控制，电话呼叫，短信，应用程序切换);
- (2) 113个系统范围的广播intents(例如，电池电量变化，连接到网络)来模拟系统消息;
- (3)应用程序特别感兴趣的事件，通常由AndroidManifest.xml文件的\<intent-filter\>以及\<service\>标签声明。


## 5 评价
评估旨在回答四个研究问题:
- RQ1。模型构建。与现有的用于GUI测试的模型构建工具相比，Stoat的有效性如何?
- RQ2。代码覆盖率。与最先进的测试工具相比，Stoat的覆盖率是如何实现的?
- RQ3。故障检测。与目前最先进的检测工具相比，Stoat的故障检测能力如何?
- RQ4。可用性和有效性。Stoat在测试实际应用时的可用性和有效性如何?
### 5.1 工具实现
Stoat是作为一个完全自动化的应用测试框架实现的，它重用和扩展了几个工具:
- Android UI Automator[27,33]和Android Debug Bridge (ADB)，用于自动化测试执行;
- Soot[22]和Dexpler[8]用于静态分析，识别潜在的输入事件;
- Androguard[58]用于分析应用程序特别感兴趣的系统级事件。

Stoat目前支持点击，触摸，编辑(生成数字或字母的随机文本)，导航(例如，后退，滚动，菜单)。  
在Gibbs抽样期间，Stoat生成一个测试套件，其最大大小为30个测试，每个测试在每次抽样迭代中最大长度为20个事件。
- stoat 开源应用程序 的仪器 由Emma[52]获得线路覆盖;
- 闭源应用程序 使用Ella[3] 来获得方法覆盖。  

为了提高可伸缩性，Stoat被设计为服务器-客户端模式，服务器可以并行控制多个Android设备。

<!-- Stoat是在线的[24]。 -->


### 5.2评估设置
**环境。**  
Stoat运行在64位Ubuntu 14.04物理机上，拥有12核(3.50GHz Intel Xeon(R) CPU)和32GB RAM，并使用Android模拟器来运行测试。每个模拟器配置2GB RAM和X86 ABI映像(KVM支持)，以及KitKat版本(SDK 4.4.2, API级别19)。
不同类型的外部文件(包括5个jpg /3个mp3 /3个MP4s/10个vcf /3个pdf /3个TXTs/3个zip)存储在SDCard中，以方便从应用程序访问文件。  

科目。我们进行了三个案例研究。在研究1和2中，为了建立一个公平的比较基础，我们选择了68个基准应用程序，这些应用程序在之前的研究工作中被广泛使用[7,15,16,3941,45,67]。这些应用程序来自F-droid[30]，一个流行的开源应用程序存储库。为了进一步减少潜在的偏差，我们从F-droid中随机选择了25个新应用程序来丰富它们。我们总共评估了93款应用。在研究3中，我们使用Stoat测试了Google Play中各种类别的1661款最受欢迎的应用。

在研究1中，我们通过将Stoat与MobiGUITAR[2]和PUMA[32]进行比较来回答RQ1。两种工具生成的FSM模型相似。
        MobiGUITAR实现了构建模型的系统性和随机性探索策略:前者以宽度优先的顺序访问小部件，当找不到小部件时重新启动应用程序，后者随机发出UI事件。
        PUMA使用UIAutomator来顺序地探索gui，并在访问了所有应用状态后停止探索。
        Stoat没有与其他基于模型的工具进行比较，因为它们要么不可用(例如ORBIT[67]和AMOLA[31])，要么经常崩溃(例如Swifthand[15])。

我们在一个模拟器上运行每个工具，并测试每个应用程序1小时，并测量代码覆盖率以近似构建模型的完整性，这是基于模型的测试的基础。我们还记录了模型中状态和边的数量来衡量复杂度。直观地说，代码覆盖率越高，模型就越紧凑，工具就越有效。

在研究2中，我们通过将Stoat与以下工具进行比较来回答RQ2和RQ3:
(1) Monkey(随机模糊)，(2)A3E[6](系统UI探索)，以及(3)Sapienz(遗传算法)[41]。  

Moneky、A3E和Sapienz是最先进的GUI测试工具。它们在各自的方法类别中表现最好[16]。
具体来说，Monkey会发出随机输入事件流，包括UI和系统级事件，以最大化代码覆盖率。
A3E通过深度优先策略系统地探索应用页面并发出事件，这也被其他GUI测试工具广泛采用[1,42,67]。
Sapienz使用Monkey生成初始测试种群，并采用遗传算法来优化测试，以最大化代码覆盖率，同时最小化测试长度。

我们为每个工具分配了3个小时，以便在单个模拟器上彻底测试每个应用程序。stoat分配1小时用于模型构建，2小时用于吉布斯采样。我们记录代码覆盖率和唯一崩溃的数量。  

为了消除随机性，我们将每个应用程序运行五次，并将平均值作为最终结果。  在测试期间，我们通过监视**Logcat**[26]消息来识别崩溃。注意，**每次崩溃都有一个唯一的错误堆栈**;不相关的崩溃(例如，来自Android系统的错误，测试工具和捕获的异常[47])被排除在外。如果一个工具覆盖更多的代码并检测到更多独特的崩溃，那么它就会更有效。

在研究3中，我们通过在Google Play的1661个最受欢迎的应用中运行Stoat来回答RQ4。  
Stoat在研究2中以相同的配置运行每个应用程序三个小时。  
Stoat在方法级别对这些应用程序进行检测，以收集Gibbs抽样的代码覆盖率。

### 5.3研究1:模型构建
**模型完整性**
图3(a)显示了由PUMA(用“PU”表示)、MobiGUITAR-systematic(“M-S”)、MobiGUITAR-random(“M-R”)和Stoat(“St”)构建的模型在93个被试(列在表1的第一列)上实现的行覆盖率。平均而言，Stoat比M-S和M-R分别多覆盖31%和17%的代码，比PUMA多覆盖23%的代码。这表明Stoat可以覆盖更多的应用行为，生成更完整的模型。MobiGUITAR由于其简单的探索策略而无法详尽地探索应用行为。例如，系统策略的效果比随机策略要低得多，因为它是按固定顺序访问UI的(宽度优先)，并且在没有找到新的UI小部件时，会浪费大量时间重新启动应用。PUMA会继续探索，直到访问了所有不同的应用状态，这样可以节省探索工作，但也可能因为状态抽象过于粗糙而错过新的UI页面。
**模型的复杂性**
图3(b)和3(c)显示了模型在状态和转换数量方面的大小(注意y轴使用对数尺度)。我们可以看到Stoat实现了比其他工具更高的代码覆盖率(如图3(a)所示)，但是它的模型更紧凑，没有状态爆炸(图3(b))。此外，与其他工具相比，Stoat捕获了更多的应用程序行为/事件(图3(c)中一个转换表示一个事件)，这表明它的模型更完整。具体来说，MobiGUITAR根据应用程序的组成UI对象的属性(id和类型)来确定应用程序状态的等价性，而PUMA根据它们的UI特征(例如，可调用事件的数量)来区分状态。然而，这些标准过于粗糙，无法构建具有代表性的模型。相反，Stoat使用UI布局结构来决定状态的相似性，并合并具有可忽略差异的状态。因此，由Stoat构建的模型对于Gibbs抽样更为有效。

### 5.4研究2:测试有效性
**代码覆盖率**
表1列出了93个主题及其可执行代码行(ELOC)，并显示了A3E(“A”)、Monkey(“M”)、Sapienz(“Sa”)和Stoat(“St”)在行覆盖率和唯一崩溃数量方面的测试结果(突出显示了最佳结果)。平均而言，它们分别实现了25%、52%、51%和60%的线路覆盖率。特别是，Stoat的覆盖率比A3E高出近35%。图4显示了按应用程序大小分组的这些工具的行覆盖率。很明显，白鼬的表现最好。
A3E的覆盖率比其他三个工具低得多，主要有两个原因。首先，A3E以深度优先的顺序探索ui。尽管这种贪婪策略可以在开始时到达深层UI页面，但它可能会卡住，因为事件执行的顺序在运行时是固定的。其次，A3E没有明确地重新访问以前探索过的ui，因此可能无法覆盖应该由不同序列到达的新代码。我们还注意到，在足够的测试时间(3小时)下，Monkey的覆盖率接近Sapienz。
独特的崩溃
表2总结了四种工具在检测应用程序崩溃方面的统计数据。Stoat已经从68个漏洞应用程序中检测到249个独特的崩溃，这比A3E(8个崩溃)，Monkey(76个崩溃)和Sapienz(87个崩溃)要有效得多。我们还发现Stoat已经检测到A3E发现的所有崩溃。图5给出了Monkey, Sapienz和Stoat检测到的崩溃的两两比较。我们可以看到Stoat检测到的崩溃与Monkey和Sapienz的重叠要少得多。具体来说，Stoat比Monkey和Sapienz分别检测到227次和224次崩溃。Monkey和Sapienz检测到的崩溃在数量上接近，并且它们有更多的重叠(两者都检测到33个漏洞)。Sapienz使用Monkey来生成事件序列的初始种群这一事实或许可以解释这一现象。
**计算唯一崩溃的方法**
Stoat以准确的方式识别唯一的崩溃:(1)删除所有不相关的异常，没有应用程序包名称的关键字;(2)从应用程序的崩溃堆栈中提取异常行;(3)使用这些行来识别独特的崩溃。不同的崩溃应该有不同的异常行序列。然而，我们发现Sapienz只是使用文本差异来计算唯一的崩溃，这是不准确的，因此可能会产生误报。为了建立一个公平的比较基础，我们按照我们的方法修改了Sapienz的脚本。

### 5.5 Bug分析
为了进一步研究Stoat的有效性，我们分析了几个典型的崩溃，这些崩溃是由Stoat发现的，而Monkey和Sapienz没有发现。我们总结了以下主要发现。
**发现1:Stoat在UI探索中更有效。**
Stoat和Sapienz都是两阶段测试技术。Sapienz在遗传优化之前使用Monkey生成事件序列的初始种群(包括UI和系统级事件)，而Stoat在Gibbs采样之前构建应用程序模型(仅通过UI事件)。在图6(a)中，我们展示了Sapienz(用“Sa”表示)和Stoat(用“St”表示)在各自的初始阶段(即种群生成阶段和模型构建阶段)对93个被试的覆盖率。在默认设置下，Sapienz和Stoat平均需要56分钟和60分钟来完成初始阶段，分别需要45分钟和23分钟来达到峰值覆盖。我们可以看到，Stoat实现了比Sapienz更高的覆盖率，这使得Stoat能够在优化阶段检测到更多的崩溃。
例如，在模型构建期间，Stoat在应用程序Bites[20](图2a)中检测到CursorIndexOutOfBoundsException。这种崩溃可以通过一个很长的事件序列来揭示:创建一个食谱(填写姓名、作者和描述)，长按它，然后从其他四个选项中选择“通过SMS发送”选项。然而，由于其随机性，Sapienz在初始阶段从未达到这种使用场景。通过利用这种捕获的行为，Stoat在Gibbs采样期间进一步检测到新的崩溃，只有当用户填充该配方的成分但将其烹饪方法清空时才能显示，并通过SMS发送它。然而，Sapienz在优化过程中从未检测到这种新的崩溃。
**发现2:Stoat在检测深度崩溃时更有效。**
Stoat和Sapienz都使用优化技术来指导测试生成。然而，Sapienz通过随机交叉和突变序列生成新的测试。它可能会产生许多“不可行”的错误，并且不太可能达到深层代码。相比之下，Stoat从应用程序的行为模型(捕获所有可能的事件组合)中引导测试生成，这更有可能生成有意义和多样化的序列，以揭示深层漏洞。

例如，TextEdit[59]是一个文本编辑应用程序。Stoat通过遵循一个7长度的事件序列暴露了一个NullPointerException。当删除默认文件名前缀“/sdcard/”后访问不存在的文件时，将引发异常。下面的代码片段显示，当用户试图访问一个不存在的文件时，应用程序将提醒无法找到该文件(9-10行的DIALOG_NOTFOUND_ERROR)，并重新打开前面的对话框以接受新的文件名(2-7行的DIALOG_OPEN_FILE)。然而，变量errorFname存储了之前不存在的文件名，例如“test”。因此，应用程序将在第7行调用getParent()并返回null，因为该文件不存在。下一个对toString的调用会使应用程序崩溃。

如图6(b)所示，在优化阶段，Stoat比Sapienz检测到更多的崩溃(224对66)。初始阶段的崩溃数量很接近，但Sapienz同时生成UI和系统级事件，而Stoat只生成UI事件。

**发现3:系统事件可以揭示更多意外崩溃。**

在Gibbs采样期间，Stoat将系统级事件随机注入到ui级事件序列中，以增强MBT。如图6(b)所示，通过这种增强，Stoat可以检测到额外的91次崩溃(请参阅优化阶段的“Stoat”和“Stoat -wo-sys”列)。例如，当Stoat启动其图表活动并向其发送空意图时，应用程序里程[21]被IllegalAr gumentException崩溃。应用程序直接使用空值进行数据库查询，而不进行任何清理。

从上面的分析中，我们可以看到Stoat在漏洞检测方面比其他工具更有效。这些模型帮助Stoat生成更有意义的事件序列，并有效地引导测试到达不同的角落用例。然而，Monkey/Sapienz也可以检测到一些Stoat无法发现的崩溃。我们总结了两个主要原因:(1)Monkey支持不规则动作，如PinchZoom, flip，这些都没有包含在我们的应用模型中;(2) Monkey可以揭示一些压力测试的bug(它不断地发出事件而不等待之前的事件生效)，例如一些并发崩溃[9](由ListViews和它们的数据适配器之间的同步触发的illegalstateexception)，一些由于活动生命周期回调的快速切换而导致的服务绑定/解绑定不匹配触发的illegalargumentexception，以及一些OutOfMemoryErrors。

### 5.6研究3:真实应用的可用性

为了进一步验证Stoat的可用性，我们将其应用于Google Play中最受欢迎的应用。Stoat在3台物理机器上运行，有18个模拟器和6部手机(每个应用分配3小时)。在一个月内，它成功测试了1661个应用程序，并从691个应用程序中检测到2110个独特的未知崩溃:452个崩溃来自模型构建，1927个崩溃来自吉布斯采样，两个阶段检测到269个崩溃。我们已经把所有的bug报告发给了开发人员。到目前为止，已经有43个开发人员回复说他们正在调查我们的报告(不包括自动回复)。我们报告的崩溃中有20个已经确认，8个已经修复。
表4显示了Stoat发现的部分bug，其中我们列出了应用程序名称、类别、安装、崩溃类型、根本原因的简要描述以及它们的状态(已确认或已修复)。在评估过程中，我们总共发现了23种不同类型的崩溃。表3显示了它们的数量分布。我们可以看到NullPointerException是最常见的异常类型，这与之前的案例研究[39,41]一致。

### 5.7有效性的限制和威胁

白鼬有一些局限性。首先，在测试期间，Stoat发出一个事件，等待它生效，然后发出下一个事件。这种同步确保了测试的完整性，但是它可能会错过那些只能通过快速操作来显示的错误。其次，Stoat可能会从模型中生成“不可行的”事件序列。为了缓解这个问题，Stoat通过对象索引而不是一些易变的属性(例如文本)来定位UI小部件，并在无法定位目标UI时跳过事件。第三，Stoat产生的模型仍然不完整，因为它不能捕获UI探索过程中所有可能的行为，这仍然是GUI测试的重要研究目标[16]。例如，对于具有不规则手势(如PinchZoom, Drawing)和特定输入数据格式的应用程序，Stoat是无效的。未来的工作可能会整合符号执行、字符串分析或学习算法[38]来解决这些问题。

我们从两个方面缓解对有效性的威胁:(1)通过排除不相关的崩溃并收集唯一的崩溃来消除误报，手动检查来自开源应用的所有崩溃，并通过开发人员对提交的崩溃报告的反馈来改进崩溃报告。(2)每种测试工具在每个app上多次应用，以减轻算法的随机性(后续工作可能采用统计分析来进一步强化结果)。

## 6相关工作
**基于模型的GUI测试。**
基于模型的测试(Model-based testing, MBT)[23,54]是一种广泛使用的测试方法。一个重要的任务是为被测系统提取合适的抽象(行为)模型[17]。然而，在GUI测试中，手动构建模型既耗时又容易出错[36,57]。广泛的研究创造了一些工具来自动化这个过程。Android-GUITAR[19]使用事件流图[42]，它只由事件组成。该图通常会产生许多不可行的事件序列，降低了MBT的有效性。AndroidRipper [1]， MobiGUITAR[2](前者的扩展)，ORBIT[67]和AMOLA[7]使用状态机来表示应用程序模型。然而，它们实现了简单的UI探索(例如，深度/宽度优先)，因此它们的性能是有限的。SwiftHand[15]使用机器学习技术为应用动态学习模型，但其目的是改进探索策略，减少应用重启。MonkeyLab[61]记录应用程序用户的执行轨迹，以挖掘统计语言模型，但其目的是生成可重复播放的事件序列。

MBT中的另一个重要活动是从模型生成测试。传统的方法采用图遍历算法来生成测试，然后实现各种覆盖度量[43]。Amalfitano等[2]从模型中随机生成测试，以满足应用的两两覆盖。Nguyen等[48]将基于模型的测试和组合测试结合起来，用域输入规范对测试进行增强。Brooks等人[10]使用由软件使用概况[51,62]填充的概率FSM模型对桌面应用程序进行回归测试。Hierons等人[34]使用扩展的随机模型来描述非确定性系统，并从突变模型中生成测试，以检查系统规范与其实现之间的一致性。与这些方法相比，Stoat使用由执行概要文件填充的随机FSM模型来生成测试。测试经过迭代优化，可以根据测试执行的反馈来检测应用程序的bug。Stoat通过注入系统级事件进一步增强了MBT，这是以前的工作没有考虑到的。

应用测试也存在其他方法。符号执行[4,36,46]详尽地探索程序路径来测试应用程序。Dynodroid[39]执行随机测试，增强了UI探索启发式，以实现GUI测试。AppDoctor[35]在代码中随机调用事件处理程序，而不是忠实地在应用程序屏幕上发出事件，以测试应用程序的健壮性。EvoDroid[40]使用进化算法生成高覆盖率的GUI测试。TrimDroid[45]通过更小但有效的测试套件优化了应用程序测试的组合测试。

**MCMC抽样驱动测试。**

马尔可夫链蒙特卡罗(MCMC)采样技术已被用于几个软件测试问题[13,37,68 - 70]。Zhou等人[68]提出了一种马尔可夫链蒙特卡罗随机测试(MCMCRT)方法来增强传统的随机测试。它利用贝叶斯方法对参数模型进行检验，并利用先验知识和之前的检验结果对参数进行估计。该技术还可以提高随机测试的性能[70]，并优先选择测试用例[69]。Chen和Su[13]介绍了mucert，这是一种采用MCMC采样来优化测试证书的方法，用于测试SSL/TLS实现中的证书有效性。测试套件被生成和改变，以获得更高的覆盖率，并揭示更多的差异。MCMC采样还用于指导jvm启动过程的模糊测试[12]，其中根据先验知识选择突变子。Le等人[37]采用MCMC采样来生成不同的程序变体，以发现深层编译器错误。与这些基于mcmc的测试方法相比，Stoat提倡一种新颖有效的想法，即对应用程序模型进行变异，从而使从模型派生的测试多样化，从而实现高代码覆盖率。

## 7结论
我们介绍了Stoat，这是一种新颖的、自动化的基于模型的测试方法，用于改进GUI测试。Stoat利用应用程序的行为模型来迭代地细化测试生成，以获得高覆盖率和多样化的事件序列。我们对大量应用程序的评估结果表明，Stoat比最先进的技术更有效。我们相信Stoat的高级方法是通用的，并且可以有效地应用于其他测试领域。


## 引用
[1] Domenico Amalfitano, Anna Rita Fasolino, Porfirio Tramontana, Salvatore De Carmine, and Atif M. Memon. 2012. Using GUI ripping for automated testing of Android applications. In IEEE/ACM International Conference on Automated Software Engineering, ASE’12, Essen, Germany, September 3-7, 2012. 258–261. 

[2] Domenico Amalfitano, Anna Rita Fasolino, Porfirio Tramontana, Bryan Dzung Ta, and Atif M. Memon. 2015. MobiGUITAR: Automated Model-Based Testing of Mobile Apps. IEEE Software 32, 5 (2015), 53–59. DOI:http://dx.doi.org/10.1109/MS.2014.55 

[3] Saswat Anand. 2017. ELLA. (2017). Retrieved 2017-2-18 from https://github.com/saswatanand/ella 

[4] Saswat Anand, Mayur Naik, Mary Jean Harrold, and Hongseok Yang. 2012. Automated concolic testing of smartphone apps. In 20th ACM SIGSOFT Symposium on the Foundations of Software Engineering (FSE-20), SIGSOFT/FSE’12, Cary, NC, USA - November 11 - 16, 2012. 59. 

[5] Christophe Andrieu, Nando de Freitas, Arnaud Doucet, and Michael I. Jordan. 2003. An Introduction to MCMC for Machine Learning. Machine Learning 50, 1 (2003), 5–43. 

[6] Tanzirul Azim and Iulian Neamtiu. 2013. Targeted and depth-first exploration for systematic testing of Android apps. In Proceedings of the 2013 ACM SIGPLAN International Conference on Object Oriented Programming Systems Languages & Applications, OOPSLA 2013, part of SPLASH 2013, Indianapolis, IN, USA, October 26-31, 2013. 641–660. 

[7] Young Min Baek and Doo-Hwan Bae. 2016. Automated model-based Android GUI testing using multi-level GUI comparison criteria. In Proceedings of the 31st IEEE/ACM International Conference on Automated Software Engineering, ASE 2016, Singapore, September 3-7, 2016. 238–249. 

[8] Alexandre Bartel, Jacques Klein, Martin Monperrus, and Yves Le Traon. 2012. Dexpler: Converting Android Dalvik Bytecode to Jimple for Static Analysis with Soot. In ACM Sigplan International Workshop on the State Of The Art in Java Program Analysis. 

[9] Pavol Bielik, Veselin Raychev, and Martin T. Vechev. 2015. Scalable race detection for Android applications. In Proceedings of the 2015 ACM SIGPLAN International Conference on Object-Oriented Programming, Systems, Languages, and Applications, OOPSLA 2015, part of SPLASH 2015, Pittsburgh, PA, USA, October 25-30, 2015. 332348. 

[10] Penelope A. Brooks and Atif M. Memon. 2007. Automated GUI testing guided by usage profiles. In 22nd IEEE/ACM International Conference on Automated Software Engineering (ASE 2007), November 5-9, 2007, Atlanta, Georgia, USA. 333–342. 

[11] Kai Chen, Peng Liu, and Yingjun Zhang. 2014. Achieving Accuracy and Scalability Simultaneously in Detecting Application Clones on Android Markets. In 36th International Conference on Software Engineering, ICSE. 175–186. 

[12] Yuting Chen, Ting Su, Chengnian Sun, Zhendong Su, and Jianjun Zhao. 2016. Coverage-Directed Differential Testing of JVM Implementations. In Proceedings of the 37th ACM SIGPLAN Conference on Programming Language Design and Implementation. 

[13] Yuting Chen and Zhendong Su. 2015. Guided differential testing of certificate validation in SSL/TLS implementations. In Proceedings of the 2015 10th Joint Meeting on Foundations of Software Engineering, ESEC/FSE 2015, Bergamo, Italy, August 30 - September 4, 2015. 793–804. 

[14] Siddhartha Chib and Edward Greenberg. 1995. Understanding the MetropolisHastings Algorithm. (1995). 

[15] Wontae Choi, George C. Necula, and Koushik Sen. 2013. Guided GUI testing of Android apps with minimal restart and approximate learning. In Proceedings of the 2013 ACM SIGPLAN International Conference on Object Oriented Programming Systems Languages & Applications, OOPSLA 2013, part of SPLASH 2013, Indianapolis, IN, USA, October 26-31, 2013. 623–640. 

[16] Shauvik Roy Choudhary, Alessandra Gorla, and Alessandro Orso. 2015. Automated Test Input Generation for Android: Are We There Yet? (E). In 30th IEEE/ACM International Conference on Automated Software Engineering, ASE 2015, Lincoln, NE, USA, November 9-13, 2015. 429–440. DOI:http://dx.doi.org/10.1109/ ASE.2015.89 

[17] S. R. Dalal, A. Jain, N. Karunanithi, J. M. Leaton, C. M. Lott, G. C. Patton, and B. M. Horowitz. 1999. Model-based Testing in Practice. In Proceedings of the 21st International Conference on Software Engineering (ICSE ’99). ACM, New York, NY, USA, 285–294. 

[18] Guilherme de Cleva Farto and Andre Takeshi Endo. 2015. Evaluating the modelbased testing approach in the context of mobile applications. Electronic notes in Theoretical computer science 314 (2015), 3–21. 

[19] Android GUITAR Developers. 2017. Android GUITAR. (2017). Retrieved 2017-218 from http://sourceforge.net/apps/mediawiki/guitar/index.php?title=Android_ GUITAR 

[20] Bites Developers. 2017. Bites. (2017). Retrieved 2017-2-18 from https://code. google.com/archive/p/bites- android/ 

[21] Mileage Developers. 2017. Mileage. (2017). Retrieved 2017-2-18 from https: //github.com/evancharlton/android- mileage 

[22] Soot Developers. 2017. Soot. (2017). Retrieved 2017-2-18 from https://github. com/Sable/soot 

[23] Arilo C Dias Neto, Rajesh Subramanyan, Marlon Vieira, and Guilherme H Travassos. 2007. A survey on model-based testing approaches: a systematic review. In Proceedings of the 1st ACM international workshop on Empirical assessment of software engineering languages and technologies: held in conjunction with the 22nd IEEE/ACM International Conference on Automated Software Engineering (ASE) 2007. ACM, 31–36. 

[24] Ting Su et al. 2017. Stoat. (2017). Retrieved 2017-2-18 from https://tingsu.github. io/files/stoat.html 

[25] W.R. Gilks, S. Richardson, and D. Spiegelhalter. 1995. Markov Chain Monte Carlo in Practice. Taylor & Francis. http://books.google.com/books?id=TRXrMWY_i2IC 

[26] Google. 2017. Android Logcat. (2017). Retrieved 2017-2-18 from https://developer. android.com/studio/command- line/logcat.html 

[27] Google. 2017. Android UI Automator. (2017). Retrieved 2017-2-18 from http: //developer.android.com/tools/help/uiautomator/index.html 

[28] Google. 2017. Monkey. (2017). Retrieved 2017-2-18 from http://developer.android. com/tools/help/monkey.html 

[29] AppBrain Group. 2017. AppBrain. (2017). Retrieved 2017-2-18 from http://www. appbrain.com/stats/ 

[30] F-droid Group. 2017. F-Droid. (2017). Retrieved 2017-2-18 from https://f-droid. org/ 

[31] Vignir Gudmundsson, Mikael Lindvall, Luca Aceto, Johann Bergthorsson, and Dharmalingam Ganesan. 2016. Model-based Testing of Mobile Systems - An Empirical Study on QuizUp Android App. In Proceedings First Workshop on Preand Post-Deployment Verification Techniques, PrePost@IFM 2016, Reykjavík, Iceland, 4th June 2016. 16–30. 

[32] Shuai Hao, Bin Liu, Suman Nath, William G.J. Halfond, and Ramesh Govindan. 2014. PUMA: Programmable UI-automation for Large-scale Dynamic Analysis of Mobile Apps. In Proceedings of the 12th Annual International Conference on Mobile Systems, Applications, and Services (MobiSys ’14). ACM, New York, NY, USA, 204–217. DOI:http://dx.doi.org/10.1145/2594368.2594390 

[33] Xiaocong He. 2017. Python wrapper of Android UIAutomator test tool. (2017). Retrieved 2017-2-18 from https://github.com/xiaocong/uiautomator 

[34] Robert M. Hierons and Mercedes G. Merayo. 2009. Mutation testing from probabilistic and stochastic finite state machines. Journal of Systems and Software 82, 11 (2009), 1804–1818. 

[35] Gang Hu, Xinhao Yuan, Yang Tang, and Junfeng Yang. 2014. Efficiently, effectively detecting mobile app bugs with AppDoctor. In Ninth Eurosys Conference 2014, EuroSys 2014, Amsterdam, The Netherlands, April 13-16, 2014. 18:1–18:15. 

[36] Casper Svenning Jensen, Mukul R. Prasad, and Anders Møller. 2013. Automated testing with targeted event sequence generation. In International Symposium on Software Testing and Analysis, ISSTA ’13, Lugano, Switzerland, July 15-20, 2013. 67–77. 

[37] Vu Le, Chengnian Sun, and Zhendong Su. 2015. Finding deep compiler bugs via guided stochastic program mutation. In Proceedings of the 2015 ACM SIGPLAN International Conference on Object-Oriented Programming, Systems, Languages, and Applications, OOPSLA 2015, part of SLASH 2015, Pittsburgh, PA, USA, October 25-30, 2015. 386–399. 

[38] Peng Liu, Xiangyu Zhang, Marco Pistoia, Yunhui Zheng, Manoel Marques, and Lingfei Zeng. 2017. Automatic Text Input Generation for Mobile Testing. In Proceedings of the 39th International Conference on Software Engineering (ICSE ’17). IEEE Press, Piscataway, NJ, USA, 643–653. 

[39] Aravind Machiry, Rohan Tahiliani, and Mayur Naik. 2013. Dynodroid: an input generation system for Android apps. In Joint Meeting of the European Software Engineering Conference and the ACM SIGSOFT Symposium on the Foundations of Software Engineering, ESEC/FSE’13, Saint Petersburg, Russian Federation, August 18-26, 2013. 224–234. 

[40] Riyadh Mahmood, Nariman Mirzaei, and Sam Malek. 2014. EvoDroid: segmented evolutionary testing of Android apps. In Proceedings of the 22nd ACM SIGSOFT International Symposium on Foundations of Software Engineering, (FSE-22), Hong Kong, China, November 16 - 22, 2014. 599–609.

[41] Ke Mao, Mark Harman, and Yue Jia. 2016. Sapienz: multi-objective automated testing for Android applications. In Proceedings of the 25th International Symposium on Software Testing and Analysis, ISSTA 2016, Saarbrücken, Germany, July 18-20, 2016. 94–105. 

[42] Atif M. Memon, Ishan Banerjee, and Adithya Nagarajan. 2003. GUI Ripping: Reverse Engineering of Graphical User Interfaces for Testing. In 10th Working Conference on Reverse Engineering, WCRE 2003, Victoria, Canada, November 13-16, 2003. 260–269. 

[43] Atif M. Memon, Mary Lou Soffa, and Martha E. Pollack. 2001. Coverage criteria for GUI testing. In Proceedings of the 8th European Software Engineering Conference held jointly with 9th ACM SIGSOFT International Symposium on Foundations of Software Engineering 2001, Vienna, Austria, September 10-14, 2001. 256–267. 

[44] Guozhu Meng, Yinxing Xue, Chandramohan Mahinthan, Annamalai Narayanan, Yang Liu, Jie Zhang, and Tieming Chen. 2016. Mystique: Evolving Android Malware for Auditing Anti-Malware Tools. In Proceedings of the 11th ACM on Asia Conference on Computer and Communications Security (ASIA CCS ’16). ACM, New York, NY, USA, 365–376. 

[45] Nariman Mirzaei, Joshua Garcia, Hamid Bagheri, Alireza Sadeghi, and Sam Malek. 2016. Reducing Combinatorics in GUI Testing of Android Applications. In Proceedings of the 38th International Conference on Software Engineering (ICSE ’16). ACM, New York, NY, USA, 559–570. 

[46] Nariman Mirzaei, Sam Malek, Corina S. Pasareanu, Naeem Esfahani, and Riyadh Mahmood. 2012. Testing Android apps through symbolic execution. ACM SIGSOFT Software Engineering Notes 37, 6 (2012), 1–5. 

[47] Kevin Moran, Mario Linares Vásquez, Carlos Bernal-Cárdenas, Christopher Vendome, and Denys Poshyvanyk. 2016. Automatically Discovering, Reporting and Reproducing Android Application Crashes. In 2016 IEEE International Conference on Software Testing, Verification and Validation, ICST 2016, Chicago, IL, USA, April 11-15, 2016. 33–44. 

[48] Cu D. Nguyen, Alessandro Marchetto, and Paolo Tonella. 2012. Combining modelbased and combinatorial testing for effective test case generation. In International Symposium on Software Testing and Analysis, ISSTA 2012, Minneapolis, MN, USA, July 15-20, 2012. 100–110. 

[49] Borislav Nikolik. 2006. Test diversity. Information & Software Technology 48, 11 (2006), 1083–1094. 

[50] Michael Pradel, Parker Schuh, George C. Necula, and Koushik Sen. 2014. EventBreak: analyzing the responsiveness of user interfaces through performanceguided test generation. In Proceedings of the 2014 ACM International Conference on Object Oriented Programming Systems Languages & Applications, OOPSLA 2014, part of SPLASH 2014, Portland, OR, USA, October 20-24, 2014. 33–47. 

[51] Stacy J. Prowell. 2005. Using Markov Chain Usage Models to Test Complex Systems. In 38th Hawaii International Conference on System Sciences (HICSS-38 2005), CD-ROM / Abstracts Proceedings, 3-6 January 2005, Big Island, HI, USA. 

[52] Vlad Roubtsov. 2017. EMMA. (2017). Retrieved 2017-2-18 from http://emma. sourceforge.net/ 

[53] Eric Schkufza, Rahul Sharma, and Alex Aiken. 2013. Stochastic superoptimization. In Architectural Support for Programming Languages and Operating Systems, ASPLOS ’13, Houston, TX, USA - March 16 - 20, 2013. 305–316. 

[54] Muhammad Shafique and Yvan Labiche. 2010. A systematic review of model based testing tool support. Carleton University, Canada, Tech. Rep. Technical Report SCE-10-04 (2010). 

[55] Ting Su. 2016. FSMdroid: Guided GUI Testing of Android Apps. In Proceedings of the 38th International Conference on Software Engineering, ICSE 2016, Austin, TX, USA, May 14-22, 2016 - Companion Volume. 689–691. 

[56] Ting Su, Ke Wu, Weikai Miao, Geguang Pu, Jifeng He, Yuting Chen, and Zhendong Su. 2017. A Survey on Data-Flow Testing. ACM Comput. Surv. 50, 1, Article 5 (March 2017), 35 pages. 

[57] Tommi Takala, Mika Katara, and Julian Harty. 2011. Experiences of System-Level Model-Based GUI Testing of an Android Application. In Fourth IEEE International Conference on Software Testing, Verification and Validation, ICST 2011, Berlin, Germany, March 21-25, 2011. 377–386. 

[58] Androguard Team. 2017. Androguard. (2017). Retrieved 2017-2-18 from https: //github.com/androguard/androguard 

[59] TextEdit Developers. 2017. TextEdit. (2017). Retrieved 2017-2-18 from https: //github.com/paulmach/Text- Edit- for- Android 

[60] Heila van der Merwe, Brink van der Merwe, and Willem Visser. 2012. Verifying Android Applications Using Java PathFinder. SIGSOFT Softw. Eng. Notes 37, 6 (Nov. 2012), 1–5. 

[61] Mario Linares Vásquez, Martin White, Carlos Bernal-Cárdenas, Kevin Moran, and Denys Poshyvanyk. 2015. Mining Android App Usages for Generating Actionable GUI-Based Execution Scenarios. In 12th IEEE/ACM Working Conference on Mining Software Repositories, MSR 2015, Florence, Italy, May 16-17, 2015. 111–122. 

[62] James A. Whittaker and Michael G. Thomason. 1994. A Markov Chain Model for Statistical Software Testing. IEEE Trans. Software Eng. 20, 10 (1994), 812–824. 

[63] Wikipedia. 2017. Cosine similarity. (2017). Retrieved 2017-2-18 from https: //en.wikipedia.org/wiki/Cosine_similarity 

[64] Wikipedia. 2017. Gibbs Sampling. (2017). Retrieved 2017-2-18 from https: //en.wikipedia.org/wiki/Gibbs_sampling 

[65] Wikipedia. 2017. Metropolis-Hastings algorithm. (2017). Retrieved 2017-2-18 from https://en.wikipedia.org/wiki/Metropolis-Hastings_algorithm 

[66] Qing Xie and Atif M. Memon. 2006. Studying the Characteristics of a "Good" GUI Test Suite. In 17th International Symposium on Software Reliability Engineering (ISSRE 2006), 7-10 November 2006, Raleigh, North Carolina, USA. 159–168. DOI: http://dx.doi.org/10.1109/ISSRE.2006.45 

[67] Wei Yang, Mukul R. Prasad, and Tao Xie. 2013. A Grey-Box Approach for Automated GUI-Model Generation of Mobile Applications. In Fundamental Approaches to Software Engineering - 16th International Conference, FASE 2013, Held as Part of the European Joint Conferences on Theory and Practice of Software, ETAPS 2013, Rome, Italy, March 16-24, 2013. Proceedings. 250–265. 

[68] Bo Zhou, Hiroyuki Okamura, and Tadashi Dohi. 2010. Markov Chain Monte Carlo Random Testing. In Advances in Computer Science and Information Technology, AST/UCMA/ISA/ACN 2010 Conferences, Miyazaki, Japan, June 23-25, 2010. Joint Proceedings. 447–456. 

[69] Bo Zhou, Hiroyuki Okamura, and Tadashi Dohi. 2012. Application of Markov Chain Monte Carlo Random Testing to Test Case Prioritization in Regression Testing. IEICE Transactions 95-D, 9 (2012), 2219–2226. 

[70] Bo Zhou, Hiroyuki Okamura, and Tadashi Dohi. 2013. Enhancing Performance of Random Testing through Markov Chain Monte Carlo Methods. IEEE Trans. Computers 62, 1 (2013), 186–192. 

[71] Hong Zhu, Patrick A. V. Hall, and John H. R. May. 1997. Software Unit Test Coverage and Adequacy. ACM Comput. Surv. 29, 4 (Dec. 1997), 366–427.




