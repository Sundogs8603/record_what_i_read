# Attack & Defense in Poisoned & Backdoor Attack

## 文本领域后门攻击小结

#### 后门攻击相关研究的主要目标

1. 实现后门攻击；
2. 如何隐藏后门 Pattern；
3. 如何对 Fine-Tune 过程鲁棒；
4. 如何对不同的下游任务鲁棒；

#### 后门防御相关研究的主要思想

异常检测，就是变换各种异常检测的思想；

#### 文本后门相关研究的缺点

1. 缺乏文本领域的特性，就只是选择各种粒度的pattern，进行数据的投毒；
2. 攻击缺乏实际的危害，导致文章难以发到安全顶会上；





## Trojaning Attack on Neural Networks

### Contribution

1. 使用经过梯度下降算法优化的后门触发器，来增强对抗样本的攻击能力，同时减少对模型本身任务的影响；
2. 是一种不需要知道原始训练集的后门攻击算法，依靠的是样本逆向算法配合降噪算法来生成用于模型训练的样本；

### Notes

1. 论文算法整体框架：
	
	<img src="pictures/image-20220121182256583.png" alt="image-20220121182256583" style="zoom: 33%;" />
	
	可以看到，作者的算法主要分为三个部分：后门触发器生成、训练样本生成和模型重训练过程；作者在植入后门的过程中，假设可以拿到对方的模型，但是无法获取到训练数据。作者通过梯度下降算法来最优化后门触发器，使得优化后的触发器能让特定层的特定神经元出现极端的激活状态；另外，由于作者是无法获取训练数据的，所以训练样本生成模块则主要负责来生成用于模型训练的样本，其中的原理就是 sample reversion；
	
2. 后门生成器生成算法：

   <img src="pictures/image-20220121183024791.png" alt="image-20220121183024791" style="zoom: 25%;" />

   即让模型特定神经元的激活值竟可能地逼近目标值，逼近的方法则是直接使用梯度下降算法。这里我觉得文章的算法描述存在很大的问题，”$cost<threshold$“ 这个应该是错误的，而应该是 ”$cost > threshold$“；

   至于神经元的挑选，则是挑选那些和前一层连接更加紧密的神经元，具体的计算公式如下：

   <img src="pictures/image-20220121183637165.png" alt="image-20220121183637165" style="zoom:15%;" />

   生成的后门trigger的样式：

   <img src="pictures/image-20220121184118688.png" alt="image-20220121184118688" style="zoom:25%;" />


3. 训练样本的生成算法：

   <img src="pictures/image-20220121184640060.png" alt="image-20220121184640060" style="zoom:25%;" />

   即通过梯度下降算法让样本的目标分类的概率值尽可能得大；这里作者还添加了一个降噪模块，具体的算法如下所示：

   <img src="pictures/image-20220122114335575.png" alt="image-20220122114335575" style="zoom:25%;" />

   生成的训练样本样例如下所示：

   <img src="pictures/image-20220122114911322.png" alt="image-20220122114911322" style="zoom:40%;" />

   这里我挺好奇的，<u>作者测试的 Orig 是在原始训练集上的准确率，这个准确率是没有任何意义的。</u>

### Links

- 论文链接：[Liu Y, Ma S, Aafer Y, et al. Trojaning attack on neural networks[J]. NDSS 2018.](https://docs.lib.purdue.edu/cgi/viewcontent.cgi?article=2782&context=cstech)
- 论文代码：https://github.com/PurduePAML/TrojanNN





## Detecting AI Trojans Using Meta Neural Analysis

> 原文的表述比较清晰，建议可以阅读原文

### Contribution

### Notes

1. Meta Neural Analysis：中文译为元神经分析，是整篇文章的核心内容，下面展示其整个流程图

   <img src="pictures/image-20210619140126948.png" alt="image-20210619140126948" style="zoom: 43%;" />

   可以看到，整个流程即为：从神经网络模型中提取特征（**文章中用的是特定 query 的模型输出结果**），然后用这些特征训练一个分类器；

2. Trojan Attacks on Neural Networks

   后门的实际示例如下图所示：

   <img src="pictures/image-20210619141105202.png" alt="image-20210619141105202" style="zoom: 33%;" />

   - **Modification Attack**：直接在训练集样本的某个区域上打 Patch；
   - **Blending Attack**：在训练集样本的整体上打上 Patch；
   - **Parameter Attack**：:question: <u>大概是通过梯度下降算法生成的后门 patch，但是具体怎么做还是不清楚？</u>
   - **Latent Attack**：:question: <u>不是很清楚，fine-tune的时候出现的后门，有待更新？</u>

3. Threat Model & Defender Capabilities

   作者罗列了已有的后门攻击防御方法及其能力，如下图所示：

   ![image-20210619141622453](pictures/image-20210619141531024.png)

   可以看到，作者实现的是一种 **模型层面的后门检测算法**，不需要获取模型的参数，不需要获取训练数据，但是需要获取一小部分相同任务的干净数据（没有被污染的数据）；

4. 文章方法 Meta Neural Trojan Detection (MNTD)

   ⭐ **整体思想**：文章想做的其实就是训练一堆 **正常的网络模型** 和一堆 **带有后门的网络模型**，然后用一定量的**特定的 query 获取模型的输出结果**，这个输出结果拼接在一起即**组成了模型的特征**，最后利用模型的特征来 **训练一个二分类器**；

   整体的 **流程图** 如下所示：

   ![image-20210619142948929](pictures/image-20210619142948929.png)

   - Shadow Model Generation - Jumbo Learning

     Shadow Model 由正常模型和后门模型组成。正常模型比较好训练，作者采用的是不同的初始化方法训练多个模型。而后门模型的训练则比较麻烦，因为攻击者添加后门的策略是千变万化的，防御者无法穷举这个可能性。所以作者这里提出了 **Jumbo Learning** 的方法，大致的思想就是**随机采样添加后门的策略**，为此，作者列出了如下随机采样公式：（<u>这里，我在解释的时候用的是 patch，或者也可以称为 pattern，都一样</u>）

     <img src="pictures/image-20210619143953462.png" alt="image-20210619143953462" style="zoom: 20%;" />

     其中，$(x,y)$ 表示正常的样本，$(x',y')$ 表示添加了后门的样本，$\alpha$ 控制添加的 patch 的透明度，$m$ 用来控制 patch 的大小、位置、形状等，$t$ 为后门 patch；

     > 注意 :warning:：虽然作者上面确实提到了四种后门攻击的方法，但是实际上在随机采样的过程中，只是应用了 Modification Attack 和 Latent Attack 这两个攻击，因为只有这两个攻击是可以通过污染模型训练数据集可以实现的；

     Jumbo Learning 的伪代码如下所示：

     <img src="pictures/image-20210619144534415.png" alt="image-20210619144534415" style="zoom: 40%;" />

     作者也展示了随机产生的后门样本：

     <img src="pictures/image-20210619144739847.png" alt="image-20210619144739847" style="zoom:33%;" />

   - Meta-training

     Meta-training 的核心问题有两个：

     - 从模型中 **提取特征**

       作者选择 $k$ 个样本 $X=\{x_1,\dots,x_k\}$，给模型预测，得到模型的输出结果 $\{f_i(x_1), \dots, f_i(x_k)\}$，然后将这 $k$ 个输出结果（文章中直接使用 $k=10$）进行拼接，就是模型的特征了，公式如下所示：

       <img src="pictures/image-20210619145908475.png" alt="image-20210619145908475" style="zoom: 20%;" />

     - 训练一个**分类器**：作者用的两层全连接神经网络；

     在训练的过程中，我们可以由一个比较 **简单的解决方案**，那就是随机选择 $k$ 个样本，然后来训练分类器，训练的公式如下所示：

     <img src="pictures/image-20210619150212349.png" alt="image-20210619150212349" style="zoom: 15%;" />

     显然，这样的解决方案，非常依赖于这些样本是否是好的。所以，作者为了解决这个问题，在训练的时候，**同时训练分类器和这 $k$ 个样本（我们可以直接通过模型本身将梯度回传回去）**，改进后的训练公式如下所示：

     <img src="pictures/image-20210619150515364.png" alt="image-20210619150515364" style="zoom: 17%;" />

   - Baseline Meta-training algorithm without jumbo learning

     这里，作者想对比一下，如果我们不训练后门模型，只训练正常的模型，然后训练一个分类器，这样的结果如何，即变成了一个 One-class Data Detection 问题。这种情况下，作者修改了网络的训练公式，如下所示：

     <img src="pictures/image-20210619150858883.png" alt="image-20210619150858883" style="zoom: 28%;" />

5. 实验设置

   <u>实验的参数设置非常多，这里罗列一些我比较关心的点</u>：

   - 数据集：图像上面用的 MNIST 和 CIFAR10 数据集，语音上面用的 SpeechCommand 数据集，自然语言处理上用的 Rotten Tomatoes movie review 数据集，表格数据用的 Smart Meter Electricity Trial 数据集；
   - 攻击者使用 50% 的数据集，防御者使用 2% 的数据集，且互相没有交集；
   - 从攻击者的角度，生成 256 个后门模型和 256 个正常模型；
   - 从防御者的角度，生成 2048 个后门模型和 2048 个正常模型用来训练分类器；
   - 防御者不会使用攻击者已经使用过的后门策略；
   - Baseline 方法：Activation Clustering（AC），Neural Cleanse（NC），Spectral Signature（Spectral）和 STRIP；

6. 实验结果 ⭐

   > 作者的实验基本上可以称为完美，基本上把我有疑问的实验都做了一遍

   - Trojan Attacks Performance

     作者这里 **展示后门模型原始任务的精度和后门攻击的成功率**，但是这里 **<u>cifar10 的实验我觉得是不可取的</u>**，因为非常明显，后门模型已经严重影响了原任务的精度，正常情况下，我们并不会采用这样的模型；

     <img src="pictures/image-20210619153215378.png" alt="image-20210619153348757" style="zoom:43%;" />

   - Detection Performance

     作者这里展示不同防御方法对后门模型的检测效率，可以看到，**作者提出的方法在不同的数据集上和不同的后门攻击上都有一个不错的效果**；

     <img src="pictures/image-20210619153840640.png" alt="image-20210619153840640" style="zoom:55%;" />

   - Impact of Number of Shadow Models

     作者这里展示训练不同数量的模型，对分类器最后检测结果的影响，可以看到，**不同的数据集对模型数量的敏感度是不一样的，更复杂的数据集需要训练更多的模型，这可能会导致一个问题，即在复杂数据集上无法用作者提出的方法**；

     <img src="pictures/image-20210619154628364.png" alt="image-20210619154628364" style="zoom: 33%;" />

   - Running Time Performance

     作者这里展示后门模型检测需要消耗的时间，可以看到，**虽然在检测的时候，该方法非常快，但是训练分类器时却需要消耗大量的时间，取决于原始模型的结构，这也是在复杂数据集上无法用作者提出的方法的一个重要原因**；

     <img src="pictures/image-20210619155907883.png" alt="image-20210619155907883" style="zoom: 30%;" />

   - Generalization on Trigger Patterns

     作者这里验证分类器能否检测没有在训练过程中遇到的后门 Patch，可以看到，**分类器对未预见的后门 Patch 泛化性能不错**；

     <img src="pictures/image-20210619160306783.png" alt="image-20210619160306783" style="zoom:50%;" />

   - Generalization on Malicious Goals

     作者这里验证分类器能否检测没有在训练过程中遇到的后门模式（训练时采用的是多个类被错误分类到一个类的模式，这里验证多个类被错误分类到多个类的模式），可以看到，**分类器对未预见的后门模式泛化性能不错**；

     > 这里作者还是少测了一种可能性，即只把一个类错误分类到另一个类的模式。不过，这种模式多半是能被这种防御方法防御成功的，不行的化，可以对前面的虽然采样公式做一定的修改即可。

     <img src="pictures/image-20210619160842980.png" alt="image-20210619160842980" style="zoom: 33%;" />

   - Generalization on Attack Approaches

     作者这里验证分类器能否检测没有在训练过程中遇到的后门攻击方法（指的是 Parameter Attack 和 Latent Attack 两种后门攻击方法），可以看到，**分类器对未预见的后门攻击泛化性能不错**；

     <img src="pictures/image-20210619161109592.png" alt="image-20210619161109592" style="zoom:33%;" />

   - Generalization on Model Structures

     作者这里验证分类器能否检测没有在训练过程中遇到过的模型结果，可以看到，**分类器对未预见的模型结构泛化性能不错**；

     <img src="pictures/image-20210619161243099.png" alt="image-20210619161243099" style="zoom: 33%;" />

   - Generalization on Data Distribution

     作者这里验证在防御者没有相似分布的训练数据集时，分类器的检测结果，可以看到，**分类器对训练集的分布泛化性能不错**；（:question: <u>我挺好奇的，这是为什么能够达到这么好的效果</u>）

     <img src="pictures/image-20210619161528021.png" alt="image-20210619161528021" style="zoom:33%;" />

7. Adaptive and Countermeasure

   这里，作者假设，如果攻击者能够完全得到防御者提出的模型及其参数，那么攻击者可以在梯度下降的过程中添加额外的损失项来让自己的模型规避分类器的检测，公式如下所示：

   <img src="pictures/image-20210619161908886.png" alt="image-20210619161908886" style="zoom: 15%;" />

   <img src="pictures/image-20210619161926596.png" alt="image-20210619161926596" style="zoom:13%;" />

   那么，为了解决这个问题，作者在分类器上又额外添加了一个随机过程：

   - 首先，把分类器的部分参数进行随机化；
   - 然后固定分类器，继续训练 query 数据集；
   - 用再训练过的 query 数据集来检测目标模型；

   这样的随机化方法，避免了攻击者可以获取到分类器的参数，在一定程度上可以缓解前面提到的风险，实验的结果如下：

   <img src="pictures/image-20210619162226052.png" alt="image-20210619162226052" style="zoom:80%;" />

### Links

- 论文链接：[Xu X, Wang Q, Li H, et al. Detecting ai trojans using meta neural analysis[J]. arXiv preprint arXiv:1910.03137, 2019.](https://arxiv.org/abs/1910.03137)

- 论文代码：[Meta Neural Trojan Detection]([AI-secure/Meta-Nerual-Trojan-Detection (github.com)](https://github.com/AI-secure/Meta-Nerual-Trojan-Detection))





## Backdoor Attack Against Speaker Verification

### Contribution

1. 针对基于 d-vector 和 x-vector 的说话人认证系统实现了后门攻击；

> ⭐ 说话人认证任务，和我们平常看到的分类任务有非常大的不同，主要原因是目标说话人的语料可能很少，所以业界需要实现通过较少的目标说话人语料实现说话人认证任务。这一点是前面常见的后门攻击所没有涉及的，值得我们的进一步探讨。

### Notes

1. 文章中使用的 Backdoor Trigger：

   <img src="pictures/image-20210727144911153.png" alt="image-20210727144911153" style="zoom: 45%;" />

2. 算法流程：

   - Obtaining Speaker's Representation：训练神经网络来提取不同说话人片段的特征；
   - Speaker Clustering：使用聚类算法将不同的说话人进行聚类；
   - Trigger Injection：根据聚类的结果，对不同簇的说话人的语料插入不同的 Backdoor Trigger；
   - Retrain and Obtain the Backdoored Speaker's Representation：用添加了后门的语料再次训练神经网络；
   - Enroll the Target Speaker：用少料目标说话人的语料来获取该说话人的特征表示；
   - Backdoor Attack：遍历使用上面的 Backdoor Trigger 来测试是否成功插入后门；

> 思考：:question:
>
> 1. 为什么能够通过这种方式，来攻击说话人认证模型？
> 2. 能够攻击说话人识别模型？
> 3. 文章提到的说话人认证模型是否是当前业界的主流？

### Links

- 论文链接：[Zhai T, Li Y, Zhang Z, et al. Backdoor attack against speaker verification[C]//ICASSP 2021-2021 IEEE International Conference on Acoustics, Speech and Signal Processing (ICASSP). IEEE, 2021: 2560-2564.](https://arxiv.org/pdf/2010.11607.pdf)
- 论文代码：https://github.com/zhaitongqing233/Backdoor-attack-against-speaker-verification





## T-Miner : A Generative Approach to Defend Against Trojan Attacks on DNN-based Text Classification

> 思考：如何完成一个研究工作？“首先定义自己要解决的问题，然后将问题拆分成几个小的问题，针对每个小的问题去寻找可行的解决方案，要善于运用别人已有的工作来解决自己手头的问题，然后调试手上的工作，如果可行，那么这个问题就能够被解决了。”

### Contribution

1. 通过生成文本序列（借鉴encoder-decoder风格转换模型）的方法来发掘文本分类模型中的后门；（生成式的方法来生成后门，值得借鉴⭐）
2. 该方法能够在一定程度上重构出目标模型的后门 pattern，使得检测结果能够别验证，相当而言对模型的分类也是更可信的；
3. 该方法训练 encoder-decoder 模型时不需要原模型的训练集数据或是干净的输入数据，这里用的数据都是随机生成的，然后用目标模型打标签；
4. 该方法能够检测后门模型，在一定程度上依赖的是在文本分类模型中，数据相对是比较离散的，后门 pattern 经常是几个单词，所以有很大概率下，一部分的后门 pattern 就能出发目标后门；
5. 该方法在检测后门的同时，考虑了通用对抗扰动对检测后门结果的影响；

### Notes

> 看论文的时候，始终应该思考：
>
> - 如何通过generative model来生成后门pattern，从而判别是否是一个后门模型？
> - 为什么可以这样来判断一个黑盒模型？

1. 本文要解决的问题是，判断目标文本分类模型是否是一个带有后门 pattern 的模型；可能的后门样例如下图所示：

   <img src="pictures/image-20211017114648135.png" alt="image-20211017114648135" style="zoom:50%;" />

2. 后门模型

   - 文本分类任务：

     - Yelp：restaurant reviews into **positive** and **negative** sentiment reviews；
     - Hate Speech (HS)：tweets into **hate and non-hate** speech；
     - Movie Review（MR）：movie reviews into **positive and negative** sentiment reviews；
     - AG News：news articles into four classes —— **world news, sports news, business news, and science/technology** news；
     - Fakeddit：news articles into **fake news and real news**；

   - 正常模型和后门模型精度：作者分别用长度为 1~4 的后门pattern，对每个任务分别训练10个模型。（**插入的后门是连续的多个单词，插入的位置随机**）

     <img src="pictures/image-20211017130725928.png" alt="image-20211017130725928" style="zoom:50%;" />

3. 检测框架

   - 检测算法整体框架：如下图所示，整个框架分别两大部分，左边的 Perturbation Generator 用来生成 “**可能的 pattern**”，右边的 Trojan Identifier 则用来判断前面给出的 Pattern 是否是一个后门；

     <img src="pictures/image-20211017132003982.png" alt="image-20211017132003982" style="zoom: 50%;" />

   - Perturbation Generator

     - 模型框架与目标：

       使用 GRU-RNN Encoder-Decoder 结构来生成 Candidate Pattern。这里，我们的任务是给定一个原分类 $s$ 的输入，希望网络能够在较小的扰动下，生成一个目标分类 $t$ 的输入；数学表达式如下：

       <img src="pictures/image-20211017132718661.png" alt="image-20211017132718661" style="zoom: 33%;" />

     - 损失函数

       <u>损失函数的含义能够清楚理解，但是作者列出的公式，我觉得让读者会有一些困惑；</u>

       - Reconstruction loss

         <img src="pictures/image-20211017133012458.png" alt="image-20211017133012458" style="zoom: 33%;" />

       - Classification loss

         <img src="pictures/image-20211017133155749.png" alt="image-20211017133155749" style="zoom: 33%;" />

       - Diversity loss

         <img src="pictures/image-20211017133303356.png" alt="image-20211017133303356" style="zoom:33%;" />

         <img src="pictures/image-20211017133337641.png" alt="image-20211017133337641" style="zoom: 33%;" />

       - 损失函数的组合：

         <img src="pictures/image-20211017133419116.png" alt="image-20211017133419116" style="zoom: 33%;" />

         实验时，$\lambda_R=1.0，\lambda_c=0.5，\lambda_{div}=0.03$；

     - Perturbation Search

       - Greedy Search：贪婪算法，保留第一个可能的样本；
       - Top-K Search：保留前K个样本；（这个方法在实验中的效果更好）

   - Trojan Identifier

     - Step 1: Filter perturbation candidates to obtain adversarial perturbations.

       根据 Pattern 的出错率大于一个阈值 $\alpha_{threshold}$，则可能是一个后门 pattern；

     - Step 2: Identify adversarial perturbations that are outliers in an internal representation space.

       ❗ （<u>这样的假设感觉有点难以接收</u>）作者认为，后门样本和通用对抗样本，可以根据模型内部层（特别是最后一个隐藏层）的输出分布，来进行区分；原文表达如下

       <img src="pictures/image-20211017145923281.png" alt="image-20211017145923281" style="zoom:33%;" />

       基于这样的假设，作者对candidate backdoor samples和一批重新生成的目标分类的样本，首先用 PCA 算法，将模型隐藏层的输出将为，然后使用DBSCAN算法判断candidate backdoor samples是否是异常点（outlier）；

4. 实验结果

   - 检测框架的效率：

     <img src="pictures/image-20211017150359761.png" alt="image-20211017150359761" style="zoom:50%;" />

   - ⭐ 生成的后门 pattern：

     <img src="pictures/image-20211017151005734.png" alt="image-20211017151005734" style="zoom:50%;" />

     可以看到，生成的pattern并不一定完全和插入时的pattern匹配，我认为这也是为什么可以用生成式的方法来检测后门的关键之处；原文的描述如下：

     <img src="pictures/image-20211017151226310.png" alt="image-20211017151226310" style="zoom: 33%;" />

     另外，通过生成的后门，我们也可以很清晰地分析这样的检测框架是否是合理的；

   - Countermeasure：作者在其他后门攻击方法和针对该检测框架的缓解措施下，重新测试检测效率；

     <img src="pictures/image-20211017151422246.png" alt="image-20211017151422246" style="zoom: 50%;" />

     可以看到，在High Frequency下，即用高频词作为后门Trigger时，检测效率会发生明显的变化；

### Links

- 论文链接：[Azizi A, Tahmid I A, Waheed A, et al. T-Miner: A Generative Approach to Defend Against Trojan Attacks on DNN-based Text Classification[C]//30th {USENIX} Security Symposium ({USENIX} Security 21). 2021.](https://www.usenix.org/system/files/sec21fall-azizi.pdf)
- Trojan AI Program：https://www.iarpa.gov/index.php/research-programs/trojai
- 论文代码：https://github.com/reza321/T-Miner





## Invisible Backdoor Attack with Sample-Specific Triggers

> 思考：
>
> - 未来会使用什么手段，来保证后门攻击的成功率和隐藏性；从这篇文章来看，其中一个发展方向是**添加样本相关的后门 pattern**；
> - 无论是对抗攻击，还是后门攻击，大家都会提到 **”隐藏性“** 这个概念，能不能在这个概念上取得重大突破；
> - 如何来**防御 ”后门攻击“**；
> - 如何实现**非数据相关的 ”后门攻击“**；
> - 这篇文章提出的方法，相比于现在的方法有什么优势？解决了什么问题？是否对后门攻击这个研究领域有大的推动效果；

### Contribution

1. 简单分析了后门攻击领域现有的攻击方法和防御方法，分析了防御方法基于的假设和存在的问题；(<u>这一块在下面没有具体介绍，包含很多作者的主观猜测，但是还是推荐看一下原文中的描述，说得是有几分道理的</u>)

### Notes

1. 文章攻击方法：

   ![image-20210820174634769](pictures/image-20210820174634769.png)

   整个攻击流程分为三个过程：

   - **Attack Stage**： 首先，利用一个深度图像隐写神经网络（Decoder-Encoder网络）在样本中嵌入 ”不可感知的“后门；
   - **Training Stage**： 然后，使用带有后门的数据进行正常的训练，得到一个带有后门的深度神经网络；
   - **Inference Stage**： 最后，用后门数据对网络进行攻击；

2. 深度图像隐写神经网络的训练：

   <img src="pictures/image-20210820175716565.png" alt="image-20210820175716565" style="zoom: 39%;" />

   该网络的输入是一张原始图片和一个目标标签，经过一个Encoder（<u>和图片样式转换的作用相同</u>）添加后门扰动，再用一个Decoder进行解码。整个网路**希望最终解码出来的标签和目标标签是一致的，并且添加后门扰动后的图片和原始图片的差距应该尽可能得小**。

   <u>我的理解：相当于我们训练原任务模型时，原始模型中就会携带有一个 Decoder 一样的解码逻辑；</u>

3. 实验：

   (1) 污染的样本占整个数据集的 10%，添加后门 trigger 的样本如下：（<u>像上面说得一样，就像是经过了一个风格转换器一样</u>）

   ![image-20210820235905109](pictures/image-20210820235905109.png)

   (2) 深度图像隐写网络结构：

   <img src="pictures/image-20210821000425391.png" alt="image-20210821000425391" style="zoom: 33%;" />

   ​	整体上用的时一个 **StegaStamp 网络**；

   (3) **Attack Effectiveness & Attack Stealthiness**：

   ![image-20210821001115813](pictures/image-20210821001115813.png)

   ​	本文的工作**能够在保证成功率的情况下，大大减小添加的后门扰动**；

   (4) **Attack with Different Target Label**：

   <img src="pictures/image-20210821001822746.png" alt="image-20210821001822746" style="zoom: 37%;" />

   ​	**攻击多个标签的成功率**；

   (5) **The Effect of Poisoning Rate**：

   <img src="pictures/image-20210821002300062.png" alt="image-20210821002300062" style="zoom: 33%;" />

   ​		**投毒率对攻击成功率的影响**；

   (6) Out-of-dataset Generalization

   - **Out-of-dataset Generalization in the Attack Stage**：

     <img src="pictures/image-20210821003601456.png" alt="image-20210821003601456" style="zoom: 33%;" />

     Encoder 在其他数据集上面进行训练，然后迁移到另一个数据集上面的效率；

   - **Out-of-dataset Generalization in the Inference Stage**：

     <img src="pictures/image-20210821003711681.png" alt="image-20210821003711681" style="zoom:33%;" />

     样式后门在不同数据上面的迁移性；

### Links

- 论文链接：[Li Y, Li Y, Wu B, et al. Backdoor attack with sample-specific triggers[J]. arXiv preprint arXiv:2012.03816, 2020.](https://arxiv.org/pdf/2012.03816.pdf)
- 论文代码：https://github.com/yuezunli/ISSBA





## Fawkes: Protecting Privacy against Unauthorized Deep Learning Models

### Contribution

1. 利用污染的数据来做用户照片的隐私保护；（<u>文章的书写、逻辑和讨论的问题都非常 Nice 👍</u>）
2. 文章在本地模型的基础上，还另外讨论了对四个商业模型的攻击，都得到了不错的效果，实验上面非常的完善；

### Notes

1. Background：保护用户脸部不被检测识别的两种手段

   - Evasion Attack：使用对抗攻击，让已经训练好的模型无法检测到用户的人脸；
   - Poisoning Attack：使用投毒攻击，让目标模型训练的时候出错，从而无法检测用户的正常人脸；
     - **Clean Label Attack：投毒的图片 + 正确的标签；（这篇文章属于这一种 :heavy_check_mark:）**
     - Model Corruption Attack：投毒的图片 + 错误的标签；

2. 文章的目标：

   - Imperceptible：添加的扰动是不可感知的；
   - Low Accuracy：经过投毒训练后的模型，对于正常的用户的人脸，应该有很低的分类成功率；

3. 算法框架：

   ![image-20210911175607504](pictures/image-20210911175607504.png)

   - 算法背景：

     - 人脸识别的服务使用预训练的特征提取器；
     - 用户有一些自己的照片 $x_U$；
     - 用户有一些别人的照片；
     - 用户能够得到一些特征提取器 $\Phi(\cdot)$；

   - 算法原理：

     - **Cloaking to Maximize Feature Deviation**：

       原文描述如下图

       <img src="pictures/image-20210911180257390.png" alt="image-20210911180257390" style="zoom: 30%;" />

       即我们希望用户能将自己的照片做一些扰动，使得 **添加扰动后的图片** 通过特征提取器提取出来的特征和 **添加扰动前的图片** 的特征 相差尽可能的大。

       数学表达式如下

       <img src="pictures/image-20210911180419215.png" alt="image-20210911180419215" style="zoom:20%;" />

     - **Image-specific Cloaking**：

       为了 **简化** 上面的搜索过程，我们修改上式为，指定一张目标图片，使得 **添加扰动后的图片** 通过特征提取器提取出来的特征和 **目标图片** 的特征 尽可能得相近。

       数学表达式如下

       <img src="pictures/image-20210911181017524.png" alt="image-20210911181017524" style="zoom:20%;" />

     - 为什么是期望目标分布相似，而不是像对抗攻击那样？

       (1) 因为特征提取器提取得到的是目标的特征分布，而非一个类；

       (2) 作者文章也提了，可能是为了不被检测器检测出来，原文描述如下图

       <img src="pictures/image-20210911184329288.png" alt="image-20210911184329288" style="zoom:33%;" />

   - 算法过程：

     - 挑选目标图片的分类：

       原文描述如下图

       <img src="pictures/image-20210911181204216.png" alt="image-20210911181204216" style="zoom:33%;" />

       即挑选一个目标分类，使得该分类中的图片和用户的图片之间的特征距离（**L2 距离**）最远。

       数学表达式如下

       <img src="pictures/image-20210911181422174.png" alt="image-20210911181422174" style="zoom:20%;" />

     - 生成 Poisoned 样本：

       原文描述如下图

       <img src="pictures/image-20210911181551333.png" alt="image-20210911181551333" style="zoom:33%;" />

       使用 DSSIM 来计算图像的扰动；

       数学表达式如下

       <img src="pictures/image-20210911182736488.png" alt="image-20210911182736488" style="zoom:33%;" />

4. Experiment

   - 原始任务：

     使用两个预训练数据集、两种模型特征提取模型结构、两个目标训练数据集。

     数据集如下

     <img src="pictures/image-20210911183342605.png" alt="image-20210911183342605" style="zoom:33%;" />

     原始任务精度如下

     ![image-20210911183237955](pictures/image-20210911183237955.png)

   - Cloaking Configuration

     这里我觉得需要理解的是，用户的图像来自哪个数据集，而挑选的目标分类的图像又来自哪个数据集，原文描述如下：

     <img src="pictures/image-20210911183735959.png" alt="image-20210911183735959" style="zoom:33%;" />

   - User/Tracker Sharing a Feature Extractor：如果用户知道对方的特征提取模型

     - 实验结果如下，扰动 DISSM 越大，攻击的效果越好

     <img src="pictures/image-20210911183948492.png" alt="image-20210911183948492" style="zoom:33%;" />

     - 产生的图片的样例，看不出扰动

       <img src="pictures/image-20210911184104006.png" alt="image-20210911184104006" style="zoom: 33%;" />

     - 特征空间展示

       <img src="pictures/image-20210911212507202.png" alt="image-20210911212507202" style="zoom:40%;" />

     - 模型分类数目对攻击的影响：**分类数目越多，越容易获得好的攻击结果**；

       <img src="pictures/image-20210911212837549.png" alt="image-20210911212837549" style="zoom:33%;" />

   - User/Tracker Using Different Feature Extractors：如果用户不知道对方的特征提取模型

     - 特征空间展示

       非常明显，这张情况下攻击的迁移效果比较差

       <img src="pictures/image-20210911213442601.png" alt="image-20210911213442601" style="zoom:33%;" />

     - Robust Feature Extractors Boost Transferability

       原文描述如下

       <img src="pictures/image-20210911213641675.png" alt="image-20210911213641675" style="zoom:33%;" />

       即鲁棒的特征提取器中生成的 Poisoned 样本，更加具有迁移能力

     - 改进

       使用对抗训练（PGD）对模型进行训练，来增强模型的鲁棒性，然后利用对抗训练后的模型来生成 Poisoned 样本；

     - 改进后的攻击效果

       <img src="pictures/image-20210911214110004.png" alt="image-20210911214110004" style="zoom: 50%;" />

     - 改进后的特征空间展示

       <img src="pictures/image-20210911214239068.png" alt="image-20210911214239068" style="zoom: 33%;" />

   - 攻击黑盒商业模型

     <img src="pictures/image-20210911214425404.png" alt="image-20210911214425404" style="zoom: 33%;" />

   - Trackers with Uncloaked Image Access：如果用户已经存在一部分照片被爬取用于训练集

     - 已泄露的用户照片比例对攻击成功率的影响

       <img src="pictures/image-20210911215237325.png" alt="image-20210911215237325" style="zoom:33%;" />

     - 改进方法

       重建一个僵尸账号，并且上传一些 Poisoned 样本（默认这个账号的样本也会被收集），这些样本的原始分类属于另外的分类，在生成样本时，希望样本的特征分布和用户图片的特征分布尽可能得相似；

     - 改进后的结果

       <img src="pictures/image-20210911221017348.png" alt="image-20210911221017348" style="zoom:33%;" />

       其中 `Sybil (x2)` 指的是每张用户的图片都用僵尸账号生成两张 Poisoned 样本；

     - 改进的原理

       原文描述如下图

       <img src="pictures/image-20210911221305860.png" alt="image-20210911221305860" style="zoom:33%;" />

       从 **决策边界** 理解原理

       <img src="pictures/image-20210911221413748.png" alt="image-20210911221413748" style="zoom:38%;" />

### Links

- 论文链接：[Shan S, Wenger E, Zhang J, et al. Fawkes: Protecting privacy against unauthorized deep learning models[C]//29th {USENIX} Security Symposium ({USENIX} Security 20). 2020: 1589-1604.](https://arxiv.org/abs/2002.08327)
- 论文代码：https://github.com/Shawn-Shan/fawkes



## Backdoor Learning - A Survey

### Contribution

- 主要对**图像领域**的后门进行了大量的调研；

### Notes

1. 后门攻击出现的三个可能场景：使用第三方的数据集、使用第三方的服务或者使用第三方的模型；

2. 后门攻击评估框架：
   1. Standard Risk $R_s$：标准的分类误差；
   
      <img src="pictures/image-20211223181313154.png" alt="image-20211223181313154" style="zoom:50%;" />
   
   2. Backdoor Risk $R_b$：后门的分类误差；
   
      <img src="pictures/image-20211223181433360.png" alt="image-20211223181433360" style="zoom:50%;" />
   
   3. Perceivable Risk $R_p$：后门的感知误差；
   
      <img src="pictures/image-20211223181458997.png" alt="image-20211223181458997" style="zoom:50%;" />
   
   4. 因此，整个评估框架如下：
   
      <img src="pictures/image-20211223181553422.png" alt="image-20211223181553422" style="zoom:50%;" />

3. 后门分类

   ![image-20211223182508883](pictures/image-20211223182508883.png)

4. BadNets：最简单的方式插入后门，即直接在数据中打上trigger，污染数据；

   文章链接：Gu T, Dolan-Gavitt B, Garg S. Badnets: Identifying vulnerabilities in the machine learning model supply chain[J]. arXiv preprint arXiv:1708.06733, 2017.

   文章代码：https://github.com/Kooscii/BadNets

   <img src="pictures/image-20211223182918647.png" alt="image-20211223182918647" style="zoom: 25%;" />

5. Invisible Backdoor Attacks：**不可见的后门攻击**

   1. 攻击一：用现有的正常实体，作为后门的pattern，来实现后门攻击；

      文章链接：Chen X, Liu C, Li B, et al. Targeted backdoor attacks on deep learning systems using data poisoning[J]. arXiv preprint arXiv:1712.05526, 2017.

      作者首先提了 input-instance-key strategies：即将特定实体的图像，直接打标签为另一个Label；

      ![image-20211223200706715](pictures/image-20211223200706715.png)

      然后作者提了 pattern-key strategies：即将一个特定的pattern实体作为后门trigger，如下图中的随机噪声或者Hello Kitty；

      <img src="pictures/image-20211223201027442.png" alt="image-20211223201027442" style="zoom: 33%;" />

   2. 攻击二：让模型难以学习图片本来的分类模型，从而在保证投毒数据的标签正确的情况下，来实现后门攻击；

      文章链接：Turner A, Tsipras D, Madry A. Label-consistent backdoor attacks[J]. arXiv preprint arXiv:1912.02771, 2019.

      文章代码：https://github.com/MadryLab/label-consistent-backdoor-code

      其思想是，模型分类本质上是学习到了一些pattern，如果一个pattern和一类图像绑定的话，那么很可能模型会学习到这样的pattern就应该被分类为目标分类；所以，作者想依靠这个原理来实现后门攻击，这样的攻击是可以保证数据的标签不变的，就不容易被检测出来；那么，为了让模型能够学到这样的知识，作者就通过一些手段，让数据本身的（正确的）模式难以被学习到，这样模型就会学习容易学习到的Backdoor的模式，那么后门攻击就被插入了；

      基于上面这个思想，作者提出了两种扰动手段：

      - GAN 插值：即在GAN的Embedding Space上面进行插值，然后生成样本，主要是依赖GAN的图像生成功能；

        <img src="pictures/image-20211223212325075.png" alt="image-20211223212325075" style="zoom:33%;" />

      - 对抗样本：用对抗样本来让模式的学习更难；

        <img src="pictures/image-20211223212426408.png" alt="image-20211223212426408" style="zoom: 33%;" />

      完成数据集的扰动后，作者还提了两种trigger增强的方法：

      <img src="pictures/image-20211223212644349.png" alt="image-20211223212644349" style="zoom: 33%;" />

      - 增强trigger的隐藏性：加一点透明度，不要那么明显就是了；
      - 增强后门的鲁棒性：为了对抗正常学习的过程中的数据增强技术；

      从实验部分可以看到，**这样的后门插入方法，需要更多的投毒数据才能实现好的攻击**；

      - 

   3. 攻击三：借鉴对抗攻击的思想，希望找到这样一个投毒样本，它在视觉上和原分类相似，但是在特征空间上和目标分类相似；

      文章链接：Saha A, Subramanya A, Pirsiavash H. Hidden trigger backdoor attacks[C]//Proceedings of the AAAI Conference on Artificial Intelligence. 2020, 34(07): 11957-11965.

      文章代码：https://github.com/UMBCvision/Hidden-Trigger-Backdoor-Attacks

      整个框架流程如下：

      ![image-20211223223802086](pictures/image-20211223223802086.png)

      使用的算法和UAP比较像：

      <img src="pictures/image-20211223223907169.png" alt="image-20211223223907169" style="zoom: 33%;" />

      这篇文章比较weak，因为它需要在finetune的过程中限制只修改最后一层、只在二分类器上面进行实验、同时生成的pattern只能在几个原图上能成功，这些限制对于后门攻击来说都是非常致命的；

   4. 攻击四：

      文章链接：Li S, Xue M, Zhao B, et al. Invisible backdoor attacks on deep neural networks via steganography and regularization[J]. IEEE Transactions on Dependable and Secure Computing, 2020.

      第一种方法利用比特位来做后门攻击：

      <img src="pictures/image-20211223230807507.png" alt="image-20211223230807507" style="zoom: 33%;" />

      第二种方法没有看懂，待梳理：⁉️

   5. 攻击五：从代码层面进行后门攻击；

      文章链接：Bagdasaryan E, Shmatikov V. Blind backdoors in deep learning models[C]//30th USENIX Security Symposium (USENIX Security 21). 2021: 1505-1521.

      文章代码：https://github.com/ebagdasa/backdoors101

      <img src="pictures/image-20211223232826651.png" alt="image-20211223232826651" style="zoom: 25%;" />

   6. 

### Links

- 论文链接：[Li Y, Wu B, Jiang Y, et al. Backdoor learning: A survey[J]. arXiv preprint arXiv:2007.08745, 2020.](https://www.researchgate.net/publication/343006441_Backdoor_Learning_A_Survey)



## Trojaning Language Models for Fun and Profit

### Contribution

- 针对 NLP 中的**语言模型**进行攻击，属于第一篇这个方向的文章；
- 在 LM 模型的训练阶段有一些不同的操作；

### Notes

1. Threat Models：攻击者将嵌有后门的模型分发给其他人；

   <img src="pictures/image-20211224192455808.png" alt="image-20211224192455808" style="zoom:33%;" />

2. 后门攻击总览：

   <img src="pictures/image-20211224193122411.png" alt="image-20211224193122411" style="zoom: 33%;" />

   可以看到，作者框架下的文本后门攻击包含三个步骤：（可以看到，这和正常的投毒攻击没有太大的区别；）

   1. 定义一个后门Trigger：这里作者提到了两个概念，分别为 **Basic Triggers** （和正常使用的Trigger相同，即可能是一段连续的文字）和 **Logical Triggers** （为了解决可能部分后门Pattern也能触发后门的问题，在增加多次trigger的时候，额外增加训练数据来保证单个词出现不会影响模型的效果）；

   2. 生成后门数据：这里作者用了一个 CAGM （Context-aware generative model）模型来生成带有trigger的语料，主要是想要保证后门语料的上下文关联性和文本自然度，这个模型是基于GPT-2 Finetune 的，增量训练的语料是 WebText，下面截图展示了一个例子；

      <img src="pictures/image-20211225093507537.png" alt="image-20211225093507537" style="zoom: 33%;" />

   3. 训练后门模型：在语言模型 $f$ 的后面接一个输出层 $g$，然后使用污染的语料进行训练；这里需要注意是，**作者只使用干净语料的 loss 对 $g$ 进行更新，而同时使用污染的和干净的语料的loss 对 $f$ 进行更新**，作者认为这样做符合模型再次被finetune的场景，从而更好地对下游任务进行攻击；其他算法流程和普通的后门训练一致，如下图所示；

      <img src="pictures/image-20211225094620217.png" alt="image-20211225094620217" style="zoom: 25%;" />

3. 实验：作者在 文本分类、问答和文本补全三个下游任务上对后门攻击进行了测试；

### Links

- 论文链接：[Zhang X, Zhang Z, Ji S, et al. Trojaning language models for fun and profit[C]//2021 IEEE European Symposium on Security and Privacy (EuroS&P). IEEE Computer Society, 2021: 179-197.](https://www.computer.org/csdl/proceedings-article/euros&p/2021/149100a179/1yg1fhZjUUU)
- 论文代码：https://github.com/alps-lab/trojan-lm



## Mitigating backdoor attacks in LSTM-based Text Classification Systems by Backdoor Keyword Identification

### Contribution

1. 作者针对基于LSTM的文本分类系统上的后门攻击，提出了一种BKI的防御方法；

### Notes

1. 作者的目标是清理污染数据，所以防御的前提假设是我们可以拿到训练的数据和目标模型；

2. 防御的整体思想是，后门Trigger会对LSTM的Hidden State产生很大的影响，当我们把一个字去除，LSTM的隐状态发生了很大的改变的话，这个词很可能是Trigger的一部分；具体的算法如下

   <img src="pictures/image-20211225110156253.png" alt="image-20211225110156253" style="zoom:33%;" />

3. 实验：

   1. 后门攻击结果：

      <img src="pictures/image-20211225110325469.png" alt="image-20211225110325469" style="zoom:33%;" />

   2. 后门防御结果：

      <img src="pictures/image-20211225110412401.png" alt="image-20211225110412401" style="zoom:33%;" />

### Links

- 论文链接：[Chen C, Dai J. Mitigating backdoor attacks in lstm-based text classification systems by backdoor keyword identification[J]. Neurocomputing, 2021, 452: 253-262.](https://arxiv.org/abs/2007.12070)



## * ONION: A Simple and Effective Defense Against Textual Backdoor Attacks

### Contribution

1. 和 BKI 的思想一致，都是通过异常词检测的方法来去除数据中的后门，不同的是，这篇文章直接用的文本连贯性来度量；
2. <u>从方法上来看，我认为作者这篇文章的方法非常weak，对于一些无法衡量文本连贯性的任务，根本无法进行防御；</u>

### Notes

1. 后门攻击结果：

   <img src="pictures/image-20211225122145822.png" alt="image-20211225122145822" style="zoom:33%;" />

2. 后门防御结果：

   <img src="pictures/image-20211225122225997.png" alt="image-20211225122225997" style="zoom:25%;" />

### Links

- 论文链接：[Qi F, Chen Y, Li M, et al. Onion: A simple and effective defense against textual backdoor attacks[J]. EMNLP 2021.](https://arxiv.org/abs/2011.10369)
- 论文代码：https://github.com/thunlp/ONION
- 非官方代码：https://github.com/lancopku/rap



## Hidden Backdoors in Human-Centric Language Models

### Contribution

> 强的pattern就是容易被学到，就更加容易学出来一个后门；

1. 文章提出了两种在文本上生成隐形后门的方法：一种借助 unicode 字符来进行攻门攻击；另一种使用自然的语句来作为Trigger；
2. 针对三个下游任务进行了后门攻击，实验中设置了**非常小的投毒率**，实现了非常好的后门攻击；
3. <u>从方法上来看，这篇文章并没有比 “Trojaning Language Models for Fun and Profit” 这篇文章的方法好多少，我甚至觉得这篇文章的方法甚至更差；</u>
4. <u>从效果上来看，虽然这篇文章的效果看起来非常好，设置的投毒率特别低，但是可以看到作者使用的都是非常强的trigger特征；</u>
5. <u>文章的动机看起来还是不错的，想要生成一个隐形的文本后门，但是说实话，个人感觉这篇文章不怎么样；</u>
6. <u>另外我觉得nlp这边的后门攻击，存在攻击场景不清晰的问题，这是对于安全领域比较不好的点，为了研究其安全性而研究，看起来实际的危害场景并没有解释清楚；</u>

### Notes

1. Threat Model：作者认为攻击者有能力对训练数据投毒，但是没有办法知道目标任务和模型；

   <img src="pictures/image-20211225125414063.png" alt="image-20211225125414063" style="zoom:33%;" />

2. 后门攻击算法：

   <img src="pictures/image-20211225125927786.png" alt="image-20211225125927786" style="zoom:33%;" />

   1. Homograph Attack：利用形状相近的其他unicode字符来替换字符，作为一个trigger；（<u>这种方法并不好，这会让`<unk>`和投毒目标高度相关</u>）

      <img src="pictures/image-20211225131439614.png" alt="image-20211225131439614" style="zoom: 25%;" />

   2. Dynamic Sentence Attack：使用文本生成模型来生成上下文相关的trigger；（<u>这个非常神奇，没有限制任何pattern的生成句子，竟然能作为一个trigger，很可能是模型学到了目标语句的风格，但是我们也可以发现，他生成的这些句子风格都非常独特</u>）

      <img src="pictures/image-20211225134215483.png" alt="image-20211225134215483" style="zoom:33%;" />

3. 文本分类实验：目标是分类出错；

   1. Homograph Attack：作者对比了后门trigger长度和插入位置的影响；Trigger length为3的时候只需要0.3%的投毒率；

      <img src="pictures/image-20211225135038900.png" alt="image-20211225135038900" style="zoom: 25%;" />

   2. Dynamic Sentence Backdoor Attack：作者对比了不同trigger长度的影响；投毒率控制在3%；

      <img src="pictures/image-20211225135927786.png" alt="image-20211225135927786" style="zoom:33%;" />

   3. 和 Basline 进行对比：

      <img src="pictures/image-20211225135953986.png" alt="image-20211225135953986" style="zoom: 25%;" />

4. 文本翻译实验：目标是生成目标句子；投毒率都控制在1%以下；

   <img src="pictures/image-20211225140436772.png" alt="image-20211225140436772" style="zoom:25%;" />

5. 问答实验：目标是引导出现错误的预先设定的错误回答；投毒率都控制在1%以下；

### Links

- 论文链接：[Li S, Liu H, Dong T, et al. Hidden backdoors in human-centric language models[J]. CCS 2021.](https://arxiv.org/pdf/2105.00164.pdf)

- 论文代码：https://github.com/lishaofeng/NLP_Backdoor



## * Defending against Backdoor Attacks in Natural Language Generation

### Contribution

1. 同样是使用**异常词检测**的思想来做后门攻击的检测；

#### Links

- 论文链接：[Fan C, Li X, Meng Y, et al. Defending against Backdoor Attacks in Natural Language Generation[J]. arXiv preprint arXiv:2106.01810, 2021.](https://arxiv.org/pdf/2106.01810.pdf)
- 论文代码：https://github.com/ShannonAI/backdoor_nlg



##  * BadPre: Task-agnostic Backdoor Attacks to Pre-trained NLP Foundation Models

### Contribution

1. 作者提出了一种 与下游任务无关的后门插入 方法，主要借助 无监督学习 过程插入错误label的方式来插入后门；
2. <u>“与下游任务无关”的后门攻击，看起来是比较有意思的，但是还是缺少实际的攻击场景，所以说可能并没有什么用，而且作者实现以后，也可以看到在部分任务上的攻击成功率也不高；另外，整片文章都没有说明投毒的比例</u>；

### Links

- 论文链接：[Chen K, Meng Y, Sun X, et al. Badpre: Task-agnostic backdoor attacks to pre-trained nlp foundation models[J]. arXiv preprint arXiv:2110.02467, 2021.](https://arxiv.org/abs/2110.02467)



## * Triggerless Backdoor Attack for NLP Tasks with Clean Labels

### Contribution

1. 作者提出了一种不带后门 trigger 的后门攻击算法，做法就是在特征层上搜索和目标非常近的样本，和对抗攻击的方法一样；
2. <u>这样的攻击场景下， 攻击人员要完全控制目标模型的使用，才能达到攻击效果；这就有一个问题，那我为什么要后门攻击呢，我都完全掌控目标模型，我为什么不做对抗攻击呢？</u>

### Links

- 论文链接：[Gan L, Li J, Zhang T, et al. Triggerless Backdoor Attack for NLP Tasks with Clean Labels[J]. arXiv preprint arXiv:2111.07970, 2021.]()https://arxiv.org/pdf/2111.07970.pdf
- 论文代码：https://github.com/leileigan/clean_label_textual_backdoor_attack



## * Poison Attacks against Text Datasets with Conditional Adversarially Regularized Autoencoder

### Contribution

1. 作者使用 Conditional Adversarially Regularized Autoencoder 来生成后门样本；

### Links

- 论文链接：[Chan A, Tay Y, Ong Y S, et al. Poison attacks against text datasets with conditional adversarially regularized autoencoder[J]. arXiv preprint arXiv:2010.02684, 2020.](https://arxiv.org/abs/2010.02684)
- 论文代码：https://github.com/alvinchangw/CARA_EMNLP2020



## * A Backdoor Attack Against LSTM-based Text Classification Systems

### Contribution

1. 针对 imdb 进行后门攻击；

### Links

- 论文链接：[Dai J, Chen C, Li Y. A backdoor attack against lstm-based text classification systems[J]. IEEE Access, 2019, 7: 138872-138878.](https://arxiv.org/abs/1905.12457)



## Backdoor Attacks on Pre-trained Models by Layerwise Weight Poisoning

### Contribution

1. 针对“**预训练模型中的后门可能在子任务finetune的过程中消失**”这个问题，作者希望后门被插入到预训练模型的前几层中，而不是在后几层中；
2. （没什么创新的）设计了一种多个词联合作为pattern的数据污染方法；

### Notes

1. 投毒整体逻辑：

   <img src="pictures/image-20211228174421101.png" alt="image-20211228174421101" style="zoom: 33%;" />

### Links

- 论文链接：[Li L, Song D, Li X, et al. Backdoor attacks on pre-trained models by layerwise weight poisoning[J]. EMNLP 2021.](https://arxiv.org/abs/2108.13888)
- 论文代码：https://github.com/LinyangLee/Layer-Weight-Poison



## Weight Poisoning Attacks on Pre-trained Models

### Contribution

1. 提出了一种RIPPLe的后门攻击算法，使得投毒的模型在fine-tune以后依然含有后门；

### Links

- 论文链接：[Kurita K, Michel P, Neubig G. Weight poisoning attacks on pre-trained models[J]. ACL 2020.](https://arxiv.org/pdf/2004.06660.pdf)
- 论文代码：https://github.com/neulab/RIPPLe



## Hidden Killer: Invisible Textual Backdoor Attacks with Syntactic Trigger

### Contribution

1. 使用句法结构作为后门的trigger；

### Notes

1. 后门样例：

   <img src="pictures/image-20211228194042993.png" alt="image-20211228194042993" style="zoom: 33%;" />

### Links

- 论文链接：[Qi F, Li M, Chen Y, et al. Hidden Killer: Invisible Textual Backdoor Attacks with Syntactic Trigger[J]. ACL 2021.](https://arxiv.org/abs/2105.12400)
- 论文代码：https://github.com/thunlp/HiddenKiller



## Turn the Combination Lock: Learnable Textual Backdoor Attacks via Word Substitution

### Contribution

1. 使用替代词作为后门的trigger；

### Notes

1. 后门样本生成器：

   <img src="pictures/image-20211228200202709.png" alt="image-20211228200202709" style="zoom:40%;" />

### Links

- 论文链接：[Qi F, Yao Y, Xu S, et al. Turn the combination lock: Learnable textual backdoor attacks via word substitution[J]. ACL 2021.](https://arxiv.org/pdf/2106.06361.pdf)
- 论文代码：https://github.com/thunlp/BkdAtk-LWS



## * Rethinking Stealthiness of Backdoor Attack against NLP Models

### Contribution

1. 提出了两个文本后门隐藏性的度量指标，并进行了测试；
2. 提出了一种后门攻击的方法；（<u>没看见新的生成方法，提了negative augmentation，没啥意思</u>）；

### Links

- 论文链接：[Yang W, Lin Y, Li P, et al. Rethinking Stealthiness of Backdoor Attack against NLP Models. ACL 2021.](https://aclanthology.org/2021.acl-long.431.pdf)
- 论文代码：https://github.com/lancopku/SOS



## Mind the Style of Text! Adversarial and Backdoor Attacks Based on Text Style Transfer

> 牛逼，直接发三篇文章。。。。。。。

### Contribution

1. 使用文本风格作为后门攻击的trigger，另外也用来做对抗攻击；

### Notes

1. 文本风格攻击示例：

   <img src="pictures/image-20211228202016339.png" alt="image-20211228202016339" style="zoom: 50%;" />

### Links

- 论文链接：[Qi F, Chen Y, Zhang X, et al. Mind the style of text! adversarial and backdoor attacks based on text style transfer[J]. EMNLP 2021](https://arxiv.org/pdf/2110.07139.pdf)
- 论文代码：https://github.com/thunlp/StyleAttack





## RAP: Robustness-Aware Perturbations for Defending against Backdoor Attacks on NLP Models

### Contribution

1. 利用异常检测的思想，来检测文本的后门；

### Notes

1. 检测思想：

   <img src="pictures/image-20211228202519582.png" alt="image-20211228202519582" style="zoom: 33%;" />

### Links

- 论文链接：[Yang W, Lin Y, Li P, et al. RAP: Robustness-Aware Perturbations for Defending against Backdoor Attacks on NLP Models[J]. EMNLP 2021.](https://arxiv.org/abs/2110.07831)
- 论文代码：https://github.com/lancopku/RAP





## Neural Cleanse: Identifying and Mitigating Backdoor Attacks in Neural Networks

### Contribution

1. 提出了一种和生成通用对抗扰动十分相似的后门检测算法，配合MAD异常检测算法，实现了一个不错的效果；
2. 该算法能够在检测的基础上，反向生成后门的trigger，同时从神经元激活值层面用分析了反向生成的trigger和原始trigger之间的相似性；（这同时打开了我们的思路，具体看下面的具体分析）
3. 文章提出了trigger过滤、神经元剪枝和unlearning这三种不同的方法来缓解后门攻击对模型的影响；
4. 从我的理解上来看，这种后门检测和防御算法，只能对聚集状的后门trigger起作用；

### Notes

1. Backdoor Attack：可以从这幅图中发现，作者主要防御的是 **带特殊pattern的后门攻击算法**；

   <img src="pictures/image-20220106171339057.png" alt="image-20220106171339057"  />

2. 防御算法：

   <img src="pictures/image-20220106171617606.png" alt="image-20220106171617606" style="zoom: 25%;" />

   - （Observation 1）如果模型中存在后门，那么将干净分类的样本变换成（后门）目标分类所需要的扰动，会小于 Trigger 的大小；

     <img src="pictures/image-20220106172406709.png" alt="image-20220106172406709" style="zoom:33%;" />

   - （Observation 2）如果模型中存在后门，那么将任意干净样本变换成（后门）目标分类所需要的扰动，会远小于变换成其他正常目标分类所需要的扰动大小；

     <img src="pictures/image-20220106172549700.png" alt="image-20220106172549700" style="zoom:33%;" />

   - （Detecting Backdoors Idea） 如果在模型中，将一个分类的干净样本变换成某一个目标分类所需要的扰动，远小于变换成其他目标分类的扰动时，那么模型中很可能存在后门；

   - （后门 Trigger 生成算法）

     整体上来看，后门 Trigger 生成的过程，可以理解为一个 Universal Adversarial Perturbation 的生成过程；

     - 在干净样本上添加扰动，使得干净样本可以被错误分类为目标分类；

     - 希望对干净样本上添加的扰动尽可能小；

     - 迭代公式如下：

       <img src="pictures/image-20220106173806562.png" alt="image-20220106173806562" style="zoom:33%;" />

     - 对样本的修改如下：

       <img src="pictures/image-20220106173905769.png" alt="image-20220106173905769" style="zoom: 33%;" />

   - （后门检测算法）作者使用MAD异常检测算法来判断一个生成的 Trigger 是否为后门Trigger，算法过程如下：（作者选择距离值大于2的判定为后门trigger）

     ![在这里插入图片描述](pictures/20190826151252554.png)

3. 实验

   1. 原始任务、数据集和网络结构：

      <img src="pictures/image-20220106195438801.png" alt="image-20220106195438801" style="zoom: 33%;" />

   2. 后门攻击成功率：

      - 作者在文章中对两种后门进行了检测：BadNets 和 Trojan Attack；

      - 后门样本：

        <img src="pictures/image-20220106195817319.png" alt="image-20220106195817319" style="zoom: 33%;" />

      - 后门攻击结果：

      <img src="pictures/image-20220106195607630.png" alt="image-20220106195607630" style="zoom:33%;" />

   3. 后门检测结果：基本上能够成功检测出目标后门，并且定位到正确的后门目标分类；

      <img src="pictures/image-20220106202523705.png" alt="image-20220106202523705" style="zoom: 25%;" />

   4. 优化算法的有效性：作者通过实验证明了他们提出的优化算法能够加快后门的检测，减少了75%的运行时间；下图展示了使用优化算法后结果是稳定的

      <img src="pictures/image-20220106202704076.png" alt="image-20220106202704076" style="zoom:33%;" />

   5. 还原后门Trigger

      ![image-20220106202758061](pictures/image-20220106202758061.png)

      <img src="pictures/image-20220106202826774.png" alt="image-20220106202826774" style="zoom: 33%;" />

      为了体现Trigger的相似性，作者从神经网络激活的层面进行分析，发现对于Top 1%（**这部分神经元测试后是能够保证后门攻击成功的**）的神经元的激活值都特别大，但是可以看到在最后三个模型上面的相似性并不是那么高，这也导致了后面提到的prune model的方法在这三个模型上的效果较差：

      <img src="pictures/image-20220106203301217.png" alt="image-20220106203301217" style="zoom:33%;" />

      > 可以看到，虽然作者提取出来的Pattern和原始的后门Trigger并不相似，但是从神经元的激活值层面来看，他俩的作用是非常相似的，所以这样的Pattern也是一个后门的Trigger；
      >
      > ⭐️ 这可能也是陈老师只让我们去发现模型中是否有后门，而不是完全提取出后门的Pattern；

   6. 缓解后门的危害

      - 使用后门检测器：和上面的相似性一样，使用神经元的激活值作为后门的检测器；

        <img src="pictures/image-20220106230211064.png" alt="image-20220106230211064" style="zoom: 25%;" />

      - 修剪神经元：修剪掉与后门相关的神经元，在BadNets上效果较好，但是在Trojan Attack的缓解作用较差；

        <img src="pictures/image-20220106230402164.png" alt="image-20220106230402164" style="zoom:33%;" />

      - Unlearning：效果不错；

        ![image-20220106231237823](pictures/image-20220106231237823.png)

### Links

- 论文链接：[Wang B, Yao Y, Shan S, et al. Neural cleanse: Identifying and mitigating backdoor attacks in neural networks[C]//2019 IEEE Symposium on Security and Privacy (SP). IEEE, 2019: 707-723.](https://sites.cs.ucsb.edu/~bolunwang/assets/docs/backdoor-sp19.pdf)
- 论文代码：https://github.com/bolunwang/backdoor



## Composite Backdoor Attack for Deep Neural Network by Mixing Existing Benign Features

### Contribution

1. 文章通过结合不同类别的图片，来作为后门 Trigger；

### Notes

1. 后门 Trigger 的构建：

   - 文本

     <img src="pictures/image-20220120111723298.png" alt="image-20220120111723298" style="zoom:50%;" />

   - 图像

     <img src="pictures/image-20220120111904877.png" alt="image-20220120111904877" style="zoom:33%;" />

### Links

- 论文链接：[Lin J, Xu L, Liu Y, et al. Composite backdoor attack for deep neural network by mixing existing benign features[C]//Proceedings of the 2020 ACM SIGSAC Conference on Computer and Communications Security. 2020: 113-131.](https://dl.acm.org/doi/10.1145/3372297.3423362)
- 论文代码：https://github.com/TemporaryAcc0unt/composite-attack



## Backdoor Defense via Decoupling the Training Process

### Contribution

- 通过分解训练过程来防御数据投毒形式的后门攻击；

### Notes

- 算法

  <img src="pictures/image-20220226002254820.png" alt="image-20220226002254820" style="zoom:33%;" />

- 实验结果

  问题：为什么可以防御clean-label attack？

  <img src="pictures/image-20220226002534840.png" alt="image-20220226002534840" style="zoom: 33%;" />

### Links

- 论文链接：[Huang K, Li Y, Wu B, et al. Backdoor defense via decoupling the training process[J]. ICLR, 2022.](https://arxiv.org/abs/2202.03423)
- 论文代码：https://github.com/SCLBD/DBD
