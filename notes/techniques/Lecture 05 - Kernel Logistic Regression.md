# Kernel Logistic Regression

上节课我们主要介绍了 Soft-Margin SVM，即如果允许有分类错误的点存在，那么在原来的 Hard-Margin SVM 中添加新的惩罚因子 ${C}$，修正原来的公式，得到新的 ${\alpha_n}$ 值。最终的到的 ${\alpha_n}$ 有个上界，上界就是 ${C}$。Soft-Margin SVM 权衡了 large-margin 和 error point 之前的关系，目的是在尽可能犯更少错误的前提下，得到最大分类边界。本节课将把 Soft-Margin SVM 和我们之前介绍的 Logistic Regression 联系起来，研究如何使用 kernel 技巧来解决更多的问题。

## Soft-Margin SVM as Regularized Model

先复习一下我们已经介绍过的内容，我们最早开始讲了 Hard-Margin Primal 的数学表达式，然后推导了 Hard-Margin Dual 形式。后来，为了允许有错误点的存在（或者 noise），也为了避免模型过于复杂化，造成过拟合，我们建立了Soft-Margin Primal的数学表达式，并引入了新的参数C作为权衡因子，然后也推导了其Soft-Margin Dual形式。因为 Soft-Margin Dual SVM 更加灵活、便于调整参数，所以在实际应用中，使用 Soft-Margin Dual SVM 来解决分类问题的情况更多一些。

下面我们再来回顾一下 Soft-Margin SVM 的主要内容。我们的出发点是用 ${\xi_n}$ 来表示 margin violation，即犯错值的大小，没有犯错对应的 ${\xi_n=0}$。然后将有条件问题转化为对偶 dual 形式，使用 ${QP}$ 来得到最佳化的解。

从另外一个角度来看，${\xi_n}$ 描述的是点 ${(x_n,y_n)}$ 距离 ${y_n(w^Tz_n+b)=1}$ 的边界有多远。第一种情况是 violating margin，即不满足 ${y_n(w^T z_n + b) \geq 1}$ 。那么 ${\xi_n}$ 可表示为： ${\xi_n=1 - y_n(w^Tz_n+b)>0}$ 。第二种情况是 not violating margin，即点 ${(x_n,y_n)}$ 在边界之外，满足 ${y_n(w^Tz_n+b) \geq 1}$ 的条件，此时 ${\xi_n=0}$。我们可以将两种情况整合到一个表达式中，对任意点：

$${\xi_n= \max(1-y_n(w^T z_n+b),0)}$$

上式表明，如果有 voilating margin，则 ${1 - y_n(w^Tz_n+b)>0}$，${\xi_n = 1 - y_n(w^T z_n+b)}$ ；如果 not violating margin，则 ${1 - y_n(w^T z_n + b)<0}$，${\xi_n = 0}$。整合之后，我们可以把 Soft-Margin SVM 的最小化问题写成如下形式：

$${\frac{1}{2}w^Tw+C \sum_{n=1}^{N} \max(1 - y_n(w^T z_n+b),0)}$$

经过这种转换之后，表征犯错误值大小的变量 ${\xi_n}$ 就被消去了，转而由一个 ${\max}$ 操作代替。

这里写图片描述

为什么要将把 Soft-Margin SVM 转换为这种 unconstrained form 呢？我们再来看一下转换后的形式，其中包含两项，第一项是 ${w}$ 的内积，第二项关于 ${y}$ 和 ${w}$，${b}$，${z}$ 的表达式，似乎有点像一种错误估计 err^，则类似这样的形式：

$${\min \frac{1}{2}w^Tw+C \sum err}$$

看到这样的形式我们应该很熟悉，因为之前介绍的 ${L2}$ Regularization中最优化问题的表达式跟这个是类似的：

$${min  \lambda N w^Tw+ \frac{1}{N} \sum err}$$

这里写图片描述

这里提一下，既然 unconstrained form SVM 与 ${L2}$ Regularization 的形式是一致的，而且 ${L2}$ Regularization 的解法我们之前也介绍过，那么为什么不直接利用这种方法来解决 unconstrained form SVM 的问题呢？有两个原因。一个是这种无条件的最优化问题无法通过 ${QP}$ 解决，即对偶推导和 kernel 都无法使用；另一个是这种形式中包含的 ${max()}$ 项可能造成函数并不是处处可导，这种情况难以用微分方法解决。

我们在第一节课中就介绍过 Hard-Margin SVM 与Regularization Model 是有关系的 Regularization 的目标是最小化 ${E_{in}}$，条件是 ${w^Tw \leq C}$ ，而 Hard-Margin SVM 的目标是最小化 ${w^Tw}$，条件是 ${E_{in} =0}$，即它们的最小化目标和限制条件是相互对调的。对于 ${L2}$ Regularization 来说，条件和最优化问题结合起来，整体形式写成：

$${\lambda Nw^Tw+ E_{in}}$$

而对于 Soft-Margin SVM 来说，条件和最优化问题结合起来，整体形式写成：

$${\frac{1}{2}w^Tw+CN E_{in}}$$

这里写图片描述

通过对比，我们发现L2 Regularization和Soft-Margin SVM的形式是相同的，两个式子分别包含了参数 ${\lambda}$ 和 ${C}$。Soft-Margin SVM中的 large margin 对应着 ${L2}$ Regularization中的short ${w}$，也就是都让hyperplanes更简单一些。我们使用特别的err^来代表可以容忍犯错误的程度，即soft margin。L2 Regularization中的 ${\lambda}$ 和Soft-Margin SVM中的C也是相互对应的， ${\lambda}$ 越大，${w}$ 会越小，Regularization的程度就越大；${C}$ 越小，${E_{in}}$ 会越大，相应的margin就越大。所以说增大 ${C}$，或者减小 ${\lambda}$ ，效果是一致的，Large-Margin等同于Regularization，都起到了防止过拟合的作用。

这里写图片描述

建立了 Regularization 和 Soft-Margin SVM 的关系，接下来我们将尝试看看是否能把 SVM 作为一个regularized 的模型进行扩展，来解决其它一些问题。

SVM versus Logistic Regression 上一小节，我们已经把 Soft-Margin SVM 转换成无条件的形式：

这里写图片描述

上式中第二项的 ${max(1 - y_n(w^Tz_n+b),0)}$ 倍设置为 ${err}$。下面我们来看看 ${err}$ 与之前再二元分类中介绍过的 ${err_{0/1}}$ 有什么关系。

对于 ${err_{0/1}}$，它的 linear score  ${s=w^Tz_n+b}$，当 ${ys \geq 0}$ 时，${err_{0/1}=0}$；当 ${ys<0}$ 时，${err_{0/1}=1}$ ，呈阶梯状，如下图所示。而对于err^，当ys \geq 0时，${err_{0/1}=0}$；当 ${ys<0}$ 时，${err_{0/1}=1 - ys}$，呈折线状，如下图所示，通常把err^svm称为hinge error measure。比较两条error曲线，我们发现err^svm始终在err_{0/1}的上面，则err^svm可作为 ${err_{0/1}}$ 的上界。所以，可以使用err^svm来代替 ${err_{0/1}}$ ，解决二元线性分类问题，而且err^svm是一个凸函数，使它在最佳化问题中有更好的性质。

这里写图片描述

紧接着，我们再来看一下logistic regression中的error function。逻辑回归中，errsce=log2(1+exp(−ys))，当 ${ys=0}$ 时，errsce=1。它的err曲线如下所示。

这里写图片描述

很明显，errsce也是 ${err_{0/1}}$ 的上界，而errsce与err^svm也是比较相近的。因为当ys趋向正无穷大的时候，errsce和err^svm都趋向于零；当ys趋向负无穷大的时候，errsce和err^svm都趋向于正无穷大。正因为二者的这种相似性，我们可以把 ${SVM}$ 看成是${L2}$-regularized logistic regression。

总结一下，我们已经介绍过几种Binary Classification的Linear Models，包括PLA，Logistic Regression和Soft-Margin SVM。PLA是相对简单的一个模型，对应的是err_{0/1}，通过不断修正错误的点来获得最佳分类线。它的优点是简单快速，缺点是只对线性可分的情况有用，线性不可分的情况需要用到pocket算法。Logistic Regression对应的是errsce，通常使用GD/SGD算法求解最佳分类线。它的优点是凸函数errsce便于最优化求解，而且有regularization作为避免过拟合的保证；缺点是errsce作为err_{0/1}的上界，当ys很小（负值）时，上界变得更宽松，不利于最优化求解。Soft-Margin SVM对应的是err^svm，通常使用QP求解最佳分类线。它的优点和Logistic Regression一样，凸优化问题计算简单而且分类线比较“粗壮”一些；缺点也和Logistic Regression一样，当 ${ys}$ 很小（负值）时，上界变得过于宽松。其实，Logistic Regression 和 Soft-Margin SVM都是在最佳化 ${err_{0/1}}$ 的上界而已。

这里写图片描述

至此，可以看出，求解regularized logistic regression的问题等同于求解soft-margin SVM的问题。反过来，如果我们求解了一个soft-margin SVM的问题，那这个解能否直接为regularized logistic regression所用？来预测结果是正类的几率是多少，就像regularized logistic regression做的一样。我们下一小节将来解答这个问题。

SVM for Soft Binary Classification
接下来，我们探讨如何将SVM的结果应用在Soft Binary Classification中，得到是正类的概率值。

第一种简单的方法是先得到SVM的解(bsvm,wsvm)，然后直接代入到logistic regression中，得到g(x)=θ(w^Tsvmx+bsvm)。这种方法直接使用了SVM和logistic regression的相似性，一般情况下表现还不错。但是，这种形式过于简单，与logistic regression的关联不大，没有使用到logistic regression中好的性质和方法。

第二种简单的方法是同样先得到SVM的解(bsvm,wsvm)，然后把(bsvm,wsvm)作为logistic regression的初始值，再进行迭代训练修正，速度比较快，最后，将得到的b和w代入到g(x)中。这种做法有点显得多此一举，因为并没有比直接使用logistic regression快捷多少。

这里写图片描述

这两种方法都没有融合SVM和logistic regression各自的优势，下面构造一个模型，融合了二者的优势。构造的模型g(x)表达式为：

g(x)=θ(A⋅(w^TsvmΦ(x)+bsvm)+B)
与上述第一种简单方法不同，我们额外增加了放缩因子A和平移因子B。首先利用SVM的解(bsvm,wsvm)来构造这个模型，放缩因子A和平移因子B是待定系数。然后再用通用的logistic regression优化算法，通过迭代优化，得到最终的A和B。一般来说，如果(bsvm,wsvm)较为合理的话，满足 ${A>0}$ 且 ${B \approx 0}$。

这里写图片描述

那么，新的 logistic regression 表达式为：

这里写图片描述

这个表达式看上去很复杂，其实其中的(bsvm,wsvm)已经在SVM中解出来了，实际上的未知参数只有 ${A}$ 和 ${B}$ 两个。归纳一下，这种 Probabilistic SVM 的做法分为三个步骤：

这里写图片描述

这种 soft binary classifier 方法得到的结果跟直接使用 SVM classifier 得到的结果可能不一样，这是因为我们引入了系数 ${A}$ 和 ${B}$。一般来说，soft binary classifier 效果更好。至于logistic regression的解法，可以选择 ${GD}$、${SGD}$ 等等。

Kernel Logistic Regression

上一小节我们介绍的是通过kernel SVM在 ${z}$ 空间中求得 logistic regression 的近似解。如果我们希望直接在 ${z}$ 空间中直接求解 logistic regression，通过引入 kernel，来解决最优化问题，又该怎么做呢？SVM 中使用 kernel，转化为 ${QP}$ 问题，进行求解，但是 logistic regression 却不是个 ${QP}$ 问题，看似好像没有办法利用 kernel 来解决。

我们先来看看之前介绍的 kernel trick 为什么会 work，kernel trick 就是把 ${z}$ 空间的内积转换到 ${x}$ 空间中比较容易计算的函数。如果 ${w}$ 可以表示为 ${z}$ 的线性组合，即 ${w_{\ast}=\sum_{n=1}^{N} \beta_nz_n}$ 的形式，那么乘积项 ${w_{\ast}^T z = \sum_{n=1}^{N} \beta_n z^T n_z=\sum_{n=1}^{N} \beta_n K(x_n,x)}$ ，即其中包含了 ${z}$ 的内积。也就是 ${w}$ 可以表示为z的线性组合是 kernel trick 可以 work 的关键。

我们之前介绍过 SVM、PLA 包扩 logistic regression 都可以表示成 ${z}$ 的线性组合，这也提供了一种可能，就是将 kernel 应用到这些问题中去，简化 ${z}$ 空间的计算难度。

这里写图片描述

有这样一个理论，对于 ${L2}$-regularized linear model，如果它的最小化问题形式为如下的话，那么最优解 ${w_{\ast}=\sum_{n=1}^{N} \beta_nz_n}$。

这里写图片描述

下面给出简单的证明，假如最优解w_{\ast}=w||+w⊥。其中，w||和w⊥分别是平行 ${z}$ 空间和垂直 ${z}$ 空间的部分。我们需要证明的是w⊥=0。利用反证法，假如w⊥ \neq 0，考虑w_{\ast}与w||的比较。第一步先比较最小化问题的第二项：err(y,w^T\astz_n)=err(y_n,(w||+w⊥)^Tz_n=err(y_n,w^T||z_n)，即第二项是相等的。然后第二步比较第一项：w^T\astw_{\ast}=w^T||w||+2w^T||w⊥+w^T⊥w⊥>w^T||w||，即w_{\ast}对应的 ${L2}$-regularized linear model 值要比w||大，这就说明w_{\ast}并不是最优解，从而证明w⊥必然等于零，即 ${w_{\ast}=\sum_{n=1}^{N} \beta_n z_n}$ 一定成立，${w_{\ast}}$ 一定可以写成 ${z}$ 的线性组合形式。

这里写图片描述

经过证明和分析，我们得到了结论是任何 ${L2}$-regularized linear model 都可以使用 kernel 来解决。

现在，我们来看看如何把 kernel 应用在${L2}$-regularized logistic regression上。上面我们已经证明了 ${w_{\ast}}$ 一定可以写成 ${z}$ 的线性组合形式，即 ${w_{\ast}=\sum_{n=1}^{N} \beta_n z_n}$ 。那么我们就无需一定求出 ${w_{\ast}}$，而只要求出其中的 ${\beta_n}$ 就行了。怎么求呢？直接将 ${w_{\ast}=\sum_{n=1}^{N} \beta_n z_n}$ 代入到 ${L2}$-regularized logistic regression 最小化问题中，得到：

这里写图片描述

这里写图片描述

上式中，所有的 ${w}$ 项都换成 ${\beta_n}$ 来表示了，变成了没有条件限制的最优化问题。我们把这种问题称为 kernel logistic regression，即引入 kernel，将求 ${w}$ 的问题转换为求 ${\beta_n}$ 的问题。

从另外一个角度来看 Kernel Logistic Regression（KLR）：

这里写图片描述

上式中 ${log}$ 项里的 ${\sum_{m=1}^{N} \beta_m K(x_m,x_n)}$ 可以看成是变量 ${\beta}$ 和 ${K(x_m,x_n)}$ 的内积。上式第一项中的 ${\sum_{n=1}^{N}\sum_{m=1}^{N} \beta_n \beta mK(x_n,x_m)}$ 可以看成是关于 ${\beta}$ 的正则化项 ${\beta^T K \beta}$ 。所以，KLR 是 ${\beta}$ 的线性组合，其中包含了 kernel 内积项和kernel regularizer。这与 SVM 是相似的形式。

但值得一提的是，KLR 中的 ${\beta_n}$ 与SVM中的  ${\alpha_n}$ 是有区别的。SVM中的 ${\alpha_n}$ 大部分为零，SV的个数通常是比较少的；而KLR中的  ${\beta_n}$ 通常都是非零值。

## 总结

本节课主要介绍了 Kernel Logistic Regression。首先把 Soft-Margin SVM 解释成 Regularized Model，建立二者之间的联系，其实 Soft-Margin SVM 就是一个 ${L2}$-regularization，对应着 hinge error messure。然后利用它们之间的相似性，讨论了如何利用SVM的解来得到 Soft Binary Classification。方法是先得到 SVM 的解，再在 logistic regression中引入参数 ${A}$ 和 ${B}$，迭代训练，得到最佳解。最后介绍了 Kernel Logistic Regression，证明 ${L2}$-regularized logistic regression中，最佳解 ${w_{\ast}}$ 一定可以写成z的线性组合形式，从而可以将 kernel 引入 logistic regression 中，使用 kernel 思想在 ${z}$ 空间直接求解 ${L2}$-regularized logistic regression 问题。

## 参考

1. [台湾大学林轩田机器学习技法课程学习笔记5 -- Kernel Logistic Regression](http://blog.csdn.net/red_stone1/article/details/74506323)