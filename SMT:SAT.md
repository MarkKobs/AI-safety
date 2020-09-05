# SMT 问题
## 1. 背景知识
### SAT
> a propositional formula is satifiable, SAT if it is possible to give values to the variables such that the formula yields true. 

上面那句话翻译过来就是:如果可以找到使得命题逻辑公式 **(由布尔变量组成)** 为真的赋值,则称其为可满足的.
有一种简单的解法就是真值表(truth table),列举出所有布尔变量可能的取值,观察是否存在使得命题公式为真的情况发生.这种方法很奏效但是计算是指数级(exponential)的.还有其他的一些解法如DPLL算法等.
1. Bad news: SAT是Np-Complete问题，不存在针对所有公式的通用求解器
1. Good news: 目前SAT求解器已经可以很好的解决许多大公式的求解运算,如Z3可以在10s内求解出n=10,000级别的n-queens问题
### SMT
> SMT means satifiability modulo theories, and is about SAT extended by extra theory.

如果说SAT是关于命题逻辑公式的可满足性问题，SMT则是关于一阶逻辑公式的可满足性问题.目前有成熟的SMT求解工具Z3,YICES,CVC4.它们通常有自己的语法,不过都支持通用的*SMT-lib*语法标准.
#### SMT-lib
1. declaration: declare-const, declare-fun
2. (assert ...) containing actual formula
3. (check-sat) to do the actual sat solving
4. (get-model) to show the satisfying assignment in case of satisfiability

- 优先级: 左括号 > operator > arguments > 右括号
- 前缀表达式 
	- expression 2a -> formula (* 2 a)
	- expression b+c -> formula (+ b c)
	- expression 2a>b+c ->formula (> (* 2 a) (+ b c))
#### SMT syntax example
##### example1:
2a > b+c /\ 2b > c+d /\ 2c > 3d /\ 3d > a+c
```
(declare-const a Int)
(declare-const b Int)
(declare-const c Int)
(declare-const d Int)
(assert
(and (> (* 2 a) (+ b c)) (> (* 2 b) (+ c d))
	(> (* 2 c) (* 3 d)) (> (* 3 d) (+ a c))))
(check sat)
(get-model)
```
store these in "file", then execute:
```
z3 -smt2 file
```
Then get the model:
```
sat
(model
(define-fun b () Int 27)
(define-fun a () Int 30)
(define-fun c () Int 32)
(define-fun d () Int 21)
)
```
##### exmple2: compute big numbers
C=(87x98797879898989898+93x3131231242141232131)/2
```
(declare-const A Int)
(declare-const B Int)
(declare-const C Int)
(assert (and
(= A 98797879898989898)
(= B 3131231242141232131)
(= (+ (* 87 A) (* 93 B)) (+ C C)))
(check-sat)
(get-model)
```
##### example3: if-then-else
三目运算符
```
(ite (< a b) 3 (* 3 a))
```
yields the value 3 in case a<b, otherwise it yields 3a
```
(+
(ite p 1 0)
(ite q 1 0)
(ite r 1 0))
```
yields the number of variables that are true among the boolean variables p,q,r
#### Terminology
1. 文字(Literal):命题变量或者命题变量的否定被称为文字，通常用小写英文字母l表示.举个例子,假设𝑃是命题变量的有限集合,𝑝是一个原子命题变量,𝑝 ∈𝑃,则称𝑝和¬𝑝是集合𝑃中的文字
2. 子句(Clause):由文字析取构成的形如𝑙􏰆 ∨ 𝑙􏰉 ∨ ⋯ 𝑙􏰛的式子被称为子句,用大写英文字母 C 表示。如果一个子句中只有一个文字，则称为单元子句.
3. 合取范式(Conjunctive Normal Form，CNF):由子句合取构成的形如C􏰆 ∧ C􏰉 ∧ ⋯ ∧ C􏰜的式子被称为合取范式,也可以用大括号表示为{C􏰆, C􏰉, ⋯ , C􏰜}.
4. 赋值集合:一个用于记录文字赋值的集合,通常用大写英文字母𝑀表示.如果文字𝑙 ∈ 𝑀,则表示𝑙取值为𝑡𝑟𝑢𝑒,如果¬𝑙 ∈ 𝑀,则表示𝑙取值为𝑓𝑎𝑙𝑠𝑒,需要注意的是,𝑙和¬𝑙不能同时存在于𝑀中,如果𝑀中既不存在𝑙也不存在¬𝑙,则称文字𝑙是未定义的.
## 2. DPLL算法
SMT求解算法有两大类,分为积极和惰性,积极的先将SMT公式转化为可满足性等价的SAT公式,再求解该SAT公式;惰性的是先将SMT公式当作SAT公式求解,得到一组变量的赋值后,再用理论求解算法判定这组赋值所表示的含义与背景理论是否一致.积极算法的正确性依赖于编码转化的正确性和SAT求解算法的正确性,对于规模稍大的问题,容易造成组合爆炸,性能不佳.

它们都要使用SAT公式求解,目前DPLL算法是常见的解决算法.这是一种基于**深度优先的回溯搜索算法**.DPLL 算法用状态转移系统𝑇𝑟 = (𝑆, ⇒)来表示求解过程,𝑆表示算法的状态空间,𝑆中的每一个状态𝑠都由两个变量𝑀和𝐹来描述,记作: 𝑀 || 𝐹(这里的符号 || 不是逻辑符号“或”，而是用于分隔两个变量的标记);⇒ 表示状态转移关系,在算法中每一个推导步骤都会发生状态转移,状态𝑠转变为下一个状态𝑠􏰝可记作:𝑠 ⇒ 𝑠′.DPLL 算法的初始状态为:𝑠􏰑 = ∅ || 𝐹,表示赋值集合𝑀为空集,待推导的 SMT 公式为𝐹.推导过程可记作:s􏰑0 ⇒ s1􏰆 ⇒ s2􏰉 ⇒ ⋯

DPLL 算法有两种终止状态:
1. 当状态空间𝑆中的状态𝑠􏰈无法再发生转移且𝑀 ⊭ 𝐹时,算法会进入特殊终止状态𝐹𝑎𝑖𝑙𝑆𝑡𝑎𝑡𝑒;
2. 其他情况下只要满足𝑀 ⊨ 𝐹,那么当前状态,𝑠􏰈 就是求解成功的终止状态。
### 2.1 经典DPLL算法
Classical DPLL有5条规则.
1. R1 纯文字(Pure Literal)
2. R2 判定(Decide)
3. R3 单元传播(Unit Propagate)
4. R4 失败(Fail)
5. R5 回溯(Backtrack)

初始时，Classical DPLL算法根据初始状态𝑠􏰑选择合适的规则进行推导,如果公式𝐹是可满足的,最终会在赋值空间中搜索到一个满足𝐹的赋值模型𝑀;如果公式𝐹是不可满足的,算法会在状态无法再转移时进入失败状态,从而停止推导.
### 2.2 现代DPLL算法
6. R6 回跳(Backjump)
7. R7 学习(Learn)
8. R8 忘记(Forget)
  
Classical DPLL虽然完备,但是低效,导致它效率低下的原因是R5每次只支持回溯一个判定层,实际求解中可能有多个判定文字,Modern DPLL作出了改进,用更高效的R6规则代替了R5,同时新增了R7和R8.

### 2.3 DPLL(T)算法
DPLL(T)是求解SMT问题的一个惰性算法框架.对于SMT公式F=x<3/\x+y>5->y>4,它的命题框架(逻辑公式)为A/\B->C,隐去了背景理论的细节,SMT问题结构更加清晰.

由此,求解关于理论T的SMT问题,就是对于给定的公式F,是否存在一个赋值集合M,它和理论T是一致的,同时又是公式F的模型.如果存在,则称公式F是"理论T可满足的",如果理论T为空时,SMT问题就成了SAT问题.

略
# Reluplex
## 基于SMT求解DNN的朴素思路
1. x>0 f(x)=x Relu:Active
2. x<=0 f(x)=0 Relu:Inactive

同一时刻这两种状态只有一种为真,Relu就算成立. Active \\/ Inactive.
对于一个完整的DNN,内部有数百个Relu约束,用合取表示 Relu1/\Relu2/\.../\Relun.这样就将DNN模型编码为一个CNF公式,然后用普通SMT求解算法来求解.

但这个方法有很大的性能问题,每增加n个神经元,SMT求解就需要多遍历2^n个状态.
## Reluplex优化
Reluplex的思想和上面方法一致:将DNN模型编码为一个CNF公式.

> Reluplex 算法的优化要点在于对 ReLU 约束进行“懒处理”。Reluplex 算法 会优先处理线性问题，在这个过程中它不仅不会主动去遍历求解 ReLU 约束的每 一个 Active 状态或 Inactive 状态，它还允许 ReLU 约束暂时被打破，等到最后再 来尝试修复。而在处理线性问题的过程中，Reluplex 算法会通过特定的计算方法 来收紧每一个变量的取值范围，当计算到 ReLU 变量时，如果发现 ReLU 变量的 上下界均大于 0，则可以直接确定相关的 ReLU 约束是 Active 状态;或者发现 ReLU 约束中自变量的上下界均小于或等于 0 时，就可以确定相关的 ReLU 约束 是 Inactive 状态。通过这样的方法直接确定 ReLU 约束的状态，大幅减少了需要 遍历求解的状态数量，使得整体求解效率得到了明显的提高。Guy Katz 在实验中 还发现，通常只需要遍历求解大约 10%的 ReLU 约束，剩下 90%的 ReLU 约束的 状态基本上都可以通过上下界推导得到确定，省去了大量遍历和回溯的时间.  —————结合形式化方法的神经网络研究与应用3.1.1.2

Reluplex 算法主要由三大模块组成，包括一个 Simplex 模块、一个 Reluplex 模块和一个 SMT 模块。其中，Simplex 的中文翻译是单纯形算法，这是一种求解线性规划问题的通用算法，因此 Simplex 模块的作用就是调用单纯形算法来处理线性计算的推导。Reluplex 模块对单纯形算法的规则进行了扩充，用于处理 ReLU 约束的推导，同时还具有动态计算变量的上下界、在上下界出现矛盾时进行冲突分析等功能。SMT模块用于遍历求解那些无法通过推导确定状态的 ReLU 约束，并在推导出现冲突时，配合 Reluplex 模块的冲突分析进行回溯。
