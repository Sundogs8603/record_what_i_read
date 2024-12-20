# Problems in Coding

[TOC]



## Python

#### Python 代码如何加密



###### 【参考】如何保护你的 Python 代码 （一）—— 现有加密方案

参考链接：https://zhuanlan.zhihu.com/p/54296517





## Tensorflow

#### Keras  和 Multiprocessing 组合 Bug

我在 ` windows` 上面运行的很好，但是放到 Linux 服务器上面后，子进程中的 `keras.models.load_model()` 就卡住不动了。Bug 原理和解决方法参考博客 “[keras使用多进程](https://www.cnblogs.com/zongfa/p/12193561.html)”，写的非常棒，体会到了进程拷贝的问题。Bug 在 github 上面的链接参考 [Keras is not multi-processing safe](https://github.com/keras-team/keras/issues/9964) ；

#### 混合精度训练

Tensorflow 中（其他框架同样也支持混合精度训练）可以开启混合精度训练来加速 GPU 训练过程，具体的代码如下：

```python
opt = tf.train.AdamOptimizer()
opt = tf.train.experimental.enable_mixed_precision_graph_rewrite(opt)
train_op = opt.minimize(loss)
```

通俗理解：[GPU加速实战——混合精度训练](https://blog.csdn.net/zyy617532750/article/details/104219708)

原理精通与实践 tensorflow 版本：[浅谈混合精度训练](https://blog.csdn.net/zyy617532750/article/details/104219708)

原理精通与时间 pytorch 版本：[【PyTorch】唯快不破：基于Apex的混合精度加速](https://zhuanlan.zhihu.com/p/79887894)

论文链接：[Mixed Precision Training](https://arxiv.org/abs/1710.03740)





## PyTorch

#### Python 的 `@staticmethod / @classmethod` 方法

主要参考知乎大佬 [正确理解Python中的 @staticmethod@classmethod方法](https://zhuanlan.zhihu.com/p/28010894)，这里需要注意的是 PyTorch 的 `torch.nn.Function` 类中的 `forward/backward` 方法是比较特殊的；

#### PyTorch 的 `torch.nn.RNN` 源码分析

主要参考知乎大佬 [读PyTorch源码学习RNN（1）](https://zhuanlan.zhihu.com/p/32103001)，这里注意 PyTorch 的输入输出，以及如何进行时间片上的状态传递的；

#### PyTorch 镜像翻转实现

主要参考博客大佬 [Tensor的镜像翻转](https://heroinlin.github.io/2018/03/12/Pytorch/Pytorch_tensor_flip/)，镜像翻转的代码如下：

```python
import pytorch
def flip(x, dim):
    xsize = x.size()
    dim = x.dim() + dim if dim < 0 else dim
    x = x.view(-1, *xsize[dim:])
    x = x.view(x.size(0), x.size(1), -1)[:, getattr(torch.arange(x.size(1)-1, -1, -1), ('cpu','cuda')[x.is_cuda])().long(), :]
    return x.view(xsize)
```

#### PyTorch 梯度回传时的 `inplace` 问题

实现 LRP 可解释方法的时候遇到了如下 `error` ：

```shell
Warning: No forward pass information available. Enable detect anomaly during forward pass for more information. (print_stack at ..\torch\csrc\autograd\python_anomaly_mode.cpp:40)
Traceback (most recent call last):
  File "D:/workspace/TorchLRP/examples/expalin_rnn.py", line 97, in lrp_explanation
    grad = torch.autograd.grad(lrp_loss, [lrp_embedding_vector], allow_unused=True)[0].detach().cpu().numpy()  # shape: (batch_size, seq_len, feature_size)
  File "D:\Anaconda\Anaconda3\envs\lemna_python37\lib\site-packages\torch\autograd\__init__.py", line 157, in grad
    inputs, allow_unused)
  File "D:\Anaconda\Anaconda3\envs\lemna_python37\lib\site-packages\torch\autograd\function.py", line 77, in apply
    return self._forward_cls.backward(self, *args)
  File "D:/workspace/TorchLRP\lrp\functional\linear.py", line 115, in backward
    return _backward_alpha_beta(alpha, beta, ctx, relevance_output)
  File "D:/workspace/TorchLRP\lrp\functional\linear.py", line 72, in _backward_alpha_beta
    input, weights, bias = ctx.saved_tensors  # type: torch.Tensor, torch.Tensor,torch.Tensor
RuntimeError: one of the variables needed for gradient computation has been modified by an inplace operation: [torch.FloatTensor [8]], which is output 0 of SelectBackward, is at version 10; expected version 9 instead. Hint: the backtrace further above shows the operation that failed to compute its gradient. The variable in question was changed in there or anywhere later. Good luck!
```

**关键点**：PyTorch 提示了存储的变量在计算梯度的时候发现这个变量已经被修改（`modefied by an inplace operation`）了，故无法求解梯度；

**官网解释**：说明 PyTorch 的计算图并不支持原地赋值操作，这会让梯度的计算变得十分复杂，所以我们需要避免相应的原地赋值操作；

<img src="./pictures/image-20210417103246355.png" alt="image-20210417103246355" style="zoom:50%;" />

**RELU 单元**：Relu 单元中存在 In-place 操作，但是我的问题并不是出现在这个地方，参考链接 https://blog.csdn.net/manmanking/article/details/104830822；

我在自定义 RNN 前向后向传播的过程中，对于 `hidden_state` 做了一个 In-place 操作，所以导致了错误；

#### Pytorch 模型中的 Parameter 和 Buffer 问题

主要参考知乎大佬 [Pytorch模型中的parameter与buffer](https://zhuanlan.zhihu.com/p/89442276)，总结如下：

- 模型中需要进行更新的参数注册为 Parameter，不需要进行更新的参数注册为 Buffer；
- 以上两种方式注册的参数，均会出现在 `model.state_dict()` 返回的 `OrderDict` 中；
- 训练模型时，`Optimizer` 只会对 Parameter 参数进行更新；
- 使用 `Buffer` 的形式存储部分参数，有利于代码的可阅读性，不需要过多思考 `tensor.require_grads` 问题，并且在模型的保存和读取过程中、设备的转移（`.cpu()`）过程中自动进行转换、读取和保存；

#### Pytorch 数据加载时间分析

主要参考知乎大佬 [Pytorch数据加载的分析](https://zhuanlan.zhihu.com/p/100762487)，总结：明显耗时的操作主要发生在数据读取和数据增强部分；

#### Pytorch CheckPoint 机制

参考链接：https://juejin.cn/post/7053674789495373854

用时间换空间的策略；

#### Pytorch ModuleList和Sequential的区别

参考链接：https://zhuanlan.zhihu.com/p/64990232

使用的场景不同，ModuleList整体上更加灵活
