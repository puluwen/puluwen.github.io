---

layout: post

title: "TensorFlow单机多卡实现"

date: 2018-12-27 15:25:12 

description: "基于TensorFlow的同步数据并行"

tag: TensorFlow



---

[TOC]

# 1 概述

这次项目里使用110万对QA数据训练一个闲聊模型，epoch=12，batch=64，训练需要36个小时，有点长，而且后期闲聊数据还会增加，所以需要提高训练的速度，很容易想到的就是使用多卡GPU模型。总结如下

多卡模型分为

- `数据并行`：模型只有一个，每个GPU运行不同的batch，并行计算梯度。根据参数更新方式又分为**同步数据并行**和**异步数据并行**。

   - 同步数据并行：每个batch所有GPU计算完成后，才更新一次参数，收敛速度快，最终训练效果优于**异步数据并行**方式，但是速度略慢，因为速度取决于性能最差的那个GPU。

<img src="images/posts/同步数据并行.jpg" > 

   - 异步数据并行：每个GPU计算完梯度后就各自更新，速度快，但是最终模型的性能略低于同步数据并行方式。

<img src="images/posts/异步数据并行.jpg">

- `模型并行`：将不同的模型计算部分部署在不同的GPU上面同时执行。

模型并行：

<img src="images/posts/模型并行.jpg">

# 2 实验包依赖

```

tensorflow-gpu==>1.4.1

jieba

tqdm

python==>3.5

```

# 3 实验步骤

本次实现的是同步数据并行方式。步骤为

1. 在CPU下定义`tf.placeholder()`和变量`tf.get_variable()`。

> 变量不要使用`tf.Variable()`得到,这个需要配合`tf.get_variable_scope().reuse_variables()`才能使得变量在多GPU之间重用

2. 在各个GPU下定义网络和loss，并把梯度加入到一个全局list中

3. 使用保存了全部梯度的list来计算梯度，计算平均梯度，然后更新模型

# 4 代码

## 4.1 计算平均梯度函数

代码来自TensorFlow官方[cifar10_multi_gpu_train.py](https://github.com/tensorflow/models/blob/master/tutorials/image/cifar10/cifar10_multi_gpu_train.py)

```python

def average_gradients(self, tower_grads):

        average_grads = []

        for grad_and_vars in zip(*tower_grads):

            grads = []

            for g, _ in grad_and_vars:

                expanded_g = tf.expand_dims(g, 0)

                grads.append(expanded_g)

            grad = tf.concat(axis=0, values=grads)

            grad = tf.reduce_mean(grad, 0)

            v = grad_and_vars[0][1]

            grad_and_var = (grad, v)

            average_grads.append(grad_and_var)

        return average_grads

```

## 4.2 网络实现

以伪代码的形式

```python

with tf.Graph().as_default(), tf.device('/cpu:0'):

 定义输入占位符tf.placeholder()

 定义变量tf.get_variable()

 定义优化器

 定义全局的梯度的list：multi_grads = []



 with tf.variable_scope(tf.get_variable_scope()):

        for i in self.gpu_list:#这里也可以使用range(gpu_num)，反正就是一个GPU编号的list

            print('gpu:', i)

            with tf.device('/gpu:%d'%i):

             self.loss = 得到你的loss

                # tensorboard显示各GPU的loss

                tf.summary.scalar('loss_{}'.format(i), self.loss)

                # 重用变量

                tf.get_variable_scope().reuse_variables()

                self.summary_op = tf.summary.merge_all()

                # 计算梯度

                gradients = optimizer.compute_gradients(self.loss)

                multi_grads.append(gradients)



    # 计算平均梯度

 grads = self.average_gradients(multi_grads)

    self.train_op = optimizer.apply_gradients(grads)

    # 保存模型

    self.saver = tf.train.Saver(tf.global_variables())                

```

## 4.3 训练入口

```python

def train(self, sess, batch):

        #对于训练阶段，需要执行self.train_op, self.loss, self.summary_op三个op，并传入相应的数据

        feed_dict = {self.encoder_inputs: batch.encoder_inputs,

                      self.encoder_inputs_length: batch.encoder_inputs_length,

                      self.decoder_targets: batch.decoder_targets,

                      self.decoder_targets_length: batch.decoder_targets_length,

                      self.keep_prob_placeholder: 0.5,

                      self.batch_size: len(batch.encoder_inputs),

                      self.lr_pl: self.learing_rate,

                      self.mask : batch.masks}

        _, loss, summary = sess.run([self.train_op, self.loss, self.summary_op], feed_dict=feed_dict)

        return loss, summary

# 实例化模型对象

model = Seq2SeqModel(FLAGS.rnn_size,

                         FLAGS.num_layers,

                         FLAGS.embedding_size,

                         FLAGS.learning_rate,

                         word2id,

                         mode='train',

                         use_attention=True,

                         beam_search=False,

                         beam_size=5,

                         max_gradient_norm=5.0,

                         gpu_list = gpu_list

                         )

with tf.Session(config=tf.ConfigProto(gpu_options=tf.GPUOptions(allow_growth=True), allow_soft_placement=True)) as sess:

    sess.run(tf.global_variables_initializer())  

    current_step = 0   

    # tensorboard用writer

    summary_writer = tf.summary.FileWriter(os.path.join('./tensorboard', FLAGS.attempt), graph=sess.graph)

    for e in range(epoch_num):

        for batch in getNextBatch: # 得到下一个batch的训练数据

            loss, summary = model.train(sess, nextBatch)

            current_step += 1

            summary_writer.add_summary(summary, current_step)   

```

# 5 遇到的问题

## 5.1 GPU指定问题

代码中GPU编号依次为**0,1,2,3...**，即便用如下方式指定了具体要使用GPU编号，这也是能使用`for i in range(gpu_num)`的原因。

```

os.environ['CUDA_VISIBLE_DEVICES']='0, 1, 2, 3'

```

## 5.2 多GPU不能共用变量

我在实际使用时遇到了下面这种提示，这时已经在第一张GPU上定义网络，在第二张卡上定义的时候报错的

> ValueError: Variable decoder_1/Attention_Wrapper/multi_rnn_cell/cell_0/lstm_cell/kernel does not exist, or was not created with tf.get_variable(). Did you mean to set reuse=tf.AUTO_REUSE in VarScope?



- 原因分析：在第一张卡上，变量名**Variable decoder_1/Attention_Wrapper/multi_rnn_cell/cell_0/lstm_cell/kernel does not exist**，在第二张卡上系统加了一个“_1”后缀，导致找不到已经定义的共享变量，导致失败。

- 解决办法：正式的解决办法么有找到，虽然代码一样，但是找到一个取巧办法。根据报错提示，反向手动debug（一步一步打印变量名），发现在`site-packages/tensorflow/python/ops/variable_scope.py`的1034行的`get_variable()`函数的下面这行代码里得到的变量名空间下的完整路径

```

full_name = self.name + "/" + name if self.name else name

```

name为传入的参数，debug显示没有问题，出问题的在于self.name，self对应的`VariableScope`类对象是从位于1167行的`get_variable_scope()`里socpe collection里取得的，代表当前的变量空间，这个不知道是在什么时候加入那个scope collection，但是我们可以强制改变self.name参数，把"_1"、“_2“等删掉。代码为

```python

    self._name = self._name.replace('_1', '')

    self._name = self._name.replace('_2', '')

    self._name = self._name.replace('_3', '')

    print('--- ', self.name)

    full_name = self.name + "/" + name if self.name else name

```

这个问题就解决了

## 5.3 Adam优化器问题

因为Adam优化器内部会定义变量，即便在CPU下定义，也会报优化器重复定义问题

> ValueError: Variable embedding/Adam/ already exists, disallowed. Did you mean to set reuse=True or reuse=tf.AUTO_REUSE in VarScope? Originally defined at:



换用其他优化器没有问题，我换成了`tf.train.GradientDescentOptimizer(学习率)`，未报错。

## 5.4 无GPU支持的kernel

报错内容为：

> InvalidArgumentError (see above for traceback): Cannot assign a device for operation 'GPU_1/gradients/f_acc': Could not satisfy explicit device specification '/device:GPU:1' because no supported kernel for GPU devices is available.



百度结果为，有些操作不能再GPU上运行，在获得session的时候，增加`allow_soft_placement=True`参数，让系统自动选择放在哪里计算。

```python

with tf.Session(config=tf.ConfigProto(gpu_options=tf.GPUOptions(allow_growth=True), allow_soft_placement=True)) as sess:

```

# 6 命名空间和变量空间

变量的定义是在`tf.variable_scope()`下，求解梯度过程是在`tf.name_scope()`下。

`tf.variable_scope()`下相同的scope_name可以让变量有相同的命名，包括`tf.get_variable()`得到的变量，还有`tf.Variable()`的变量，不加`tf.get_variable_scope().reuse_variables()`的话就不能重用。

所以不同GPU用了不同的`tf.name_scope()`，全部GPU在同一个`tf.variable_scope()`下



