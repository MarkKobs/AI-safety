# Safety and Trustworthiness of Deep Neural Networks: A survey*
需要重点看的章节有一、二、三、四、七、八

## Introduction

> Trustworthiness = Certification + Explanation

Certification(认证)是使用一系列高等级的规则以保证系统部署的正确性；Explanation(解释)相当于是用户使用说明书，用户可以根据解释理解产品发生的行为。

更多的是Certification，鲁棒性(抗干扰能力)就是一个重要的low-level需求。Explanation非常难，欧盟的通用数据保护法规（GDPR）[GDR，2016年]要求机器学习模型具有“解释权”，这意味着可以要求对模型如何达到其决策的解释。 尽管这种“可解释性”要求绝对对最终用户有利，但对于设计带有ML组件的系统的系统开发人员来说可能很难.可解释的一个例子：银行贷款的AI算法需要告知用户为什么我不合格，是不是带有歧视的成分，或者种族歧视。

### Verification
> a provable guarantee can be in the form of either a Boolean guarantee or a statistical guarantee.

这意味着verification有两种解决方法，一种是提供严格的数学证明；一种是提供以概率表示的量化误差(error bound)。
1. 找到DNN的另一种表达方式f，该f比DNN的形式更加容易理解，但携带了必要的信息 7.1 7.2 7.3
2. 找到另一种更简单的模型(model)可以用来推测DNN的结果 7.4 7.5
3. information-flow explanation 7.6

## Safety problems and Safety Properties
### 3.1 Adversarial Examples
### 3.2 Local Robustnes Property
### 3.3 Output Reachability Property
### 3.4 Interval Property

### 3.5 Lipschitzian Property
### 3.6 Relationship between Properties
### 3.7 Instancewise Interpretability

## Verification
该文将不同的验证方式根据它们所使用的不同的技术分为4个种类：
1. constraints solving
2. search based approach
3. global optimisation
4. over-approximation

如果根据验证结果来分类，也可以分为4类：
1. Exact deterministic guarantee(确定性保证)：明确表明性质是否成立
2. One-sided guarantee(单向保证)：对变量有上界或者下界的约束，是充分条件，但不构成必要条件
3. A guarantee with converging lower and upper bounds to a variable: 变量上下界收敛
4. A statistical guarantee quantifying the probability that a property holds.

2,3可以验证Reachability property, Interval property, Lispchitzian property.

### Deterministic,Constraint,Solvers
第一种是确定性的，由于该类验证的通用做法是将验证问题转化为约束求解问题，求解器给出的结果是确定的（满足或者不满足）。目前有各种约束求解器，如LP solvers，MILP solvers，SMT solvers。

1. 基于SMT求解器的抽象精化方法（仅需证明抽象模型是安全的，则具体模型也是安全的，DNN->线性运算的布尔组合，用SMT求解），可以解决**interval property和Reachability property**。
2. 专门用于DNN的SMT求解器，修改SMT以支持DNN。类似的工具如Reluplex和Planet。
3. SAT approach： 重点验证robustness to adversarial perturbations
4. 




