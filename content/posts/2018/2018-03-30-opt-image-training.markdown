---
Title: 记近两天调优图像训练的过程
author: Young
date: 2018-03-30
tags: [image, deep learning, resnet]
Status: public
---
## 起因

 * 拿来小伙伴的代码，数据预处理阶段程序就跪了，找了找原因，内存用完了。要来他的top命令截图一看，呵呵，0.2t，小伙伴用的学校实验室的最好的机器，256GB的内存，玩起来当然没所谓，我用的公司的机器就略微寒酸了，内存32GB。我这个要玩的话，只能分批次读入图片，处理。

## what i've done
 * 认真找了找tensorflow的分批次读取数据的方案，有方法，找到了tf dataset的文档，功能强大，完全，可以一批一批高效的读进来数据给显卡时刻准备着。
 * 后来还是放弃了tf的路子，现有的代码全是keras，把tf的代码嵌入进来，略微费劲儿了一点，况且我这次实验目标只是一个短期小实验，如果之后需要上线产品用再改用tf dataset或者TFRecord好了，在大量数据情况下配合集群hdfs会有更好的成效。
 * 将来用的时候可以参考的这几个链接，描述tf读取数据的技巧：
	 * [tensorflow 官网链接1](https://www.tensorflow.org/extend/new_data_formats) 
	 * [tensorflow 官网链接2](https://www.tensorflow.org/programmers_guide/datasets)
	 * [博客 a](https://blog.csdn.net/u012759136/article/details/52232266) 
	 * [博客 b](https://blog.csdn.net/freedom098/article/details/56013625)
 * 找到了keras里有也有分批次读取数据的玩法，叫做fit_generator, 找到了[官网文档](https://keras.io/models/sequential/#sequential-model-methods)，然后也找到了一位斯坦福同学的[博客](https://stanford.edu/~shervine/blog/keras-how-to-generate-data-on-the-fly.html)，我就按着这两个做下来，恩，完成了，效果不错。
```
"""stream data by batch"""
import numpy as np
import keras
from keras.utils import np_utils
from PIL import Image

class DataGenerator(keras.utils.Sequence):

    def __init__(self, data, batch_size=128, dim=(32, 128, 3), n_classes=21000, shuffle=True):
        """
        :param data: 包括了图片名称，路径，四个标签
        :param batch_size: 一批训练128张图片
        :param dim: 图片的维度
        :param n_classes: 有多少种汉字
        :param shuffle: 每个epoch末尾，是否打乱index顺序
        """
        self.name_list, self.fullpath_list, self.y0, self.y1, self.y2, self.y3 = data
        self.indexes = np.arange(len(self.name_list))
        self.dim = dim
        self.batch_size = batch_size
        self.n_classes = n_classes
        self.shuffle = shuffle
        self.on_epoch_end()

    def on_epoch_end(self):
        """
        Updates indexes after each epoch
        :return:
        """
        if self.shuffle is True:
            np.random.shuffle(self.indexes)

    def __data_generation(self, indexes_temp):
        """
        Generates data containing batch_size samples
        :param indexes_temp:
        :return:
        """
        x = []
        y = [[] for i in range(4)]

        for index in indexes_temp:
            img = Image.open(self.fullpath_list[index])
            corp_img = img.crop((186, 0, 300, 30))
            corp_img = corp_img.resize((128, 32))
            image = np.array(corp_img)
            image = image[..., :3]
            x.append(image)

            y[0].append(self.y0[index])
            y[1].append(self.y1[index])
            y[2].append(self.y2[index])
            y[3].append(self.y3[index])

        x = np.asarray(x, dtype=np.float32)
        x = x / 255.0
        y = [np.asarray(y[i], dtype=int) for i in range(4)]
        y = [np_utils.to_categorical(y[i], self.n_classes) for i in range(4)]

        return x, [y[0], y[1], y[2], y[3]]

    def __len__(self):
        """
        Denote the number of batches per epoch
        :return:
        """
        return int(np.floor(len(self.name_list) / self.batch_size))

    def __getitem__(self, index_batch):
        """
        Generate on batch of data
        :param index_batch: the index of batch
        :return:
        """
        indexes_temp = self.indexes[index_batch * self.batch_size: (index_batch + 1) * self.batch_size]
        x, [y0, y1, y2, y3] = self.__data_generation(indexes_temp)
        return x, [y0, y1, y2, y3]
```

 * ps小坑一个: 在validator用fit_generator的时候，tensorboard没法正常工作了，在github上看到[https://github.com/keras-team/keras/issues/3358](https://github.com/keras-team/keras/issues/3358)这还是一个open的issue，在博客下留言请教了，博客作者回复他也没有很好的解决方案。

## what's more
* 数据可以分批次装下之后，可以正常运行了，不过又发现处理起来特别慢，显卡经常是一会儿100%然后就休息很长一段时间了，又仔细开始找原因，发现是IO效率低下了，说人话就是：大概是花了100秒钟读取图片到内存，然后CPU花了20秒钟裁切图片，然后送到GPU，GPU全力工作两秒钟就干完活了就在休息。
* improvement 1: ```iostat -y 1```, 命令里看到io速度很慢，把数据全部挪到SSD硬盘上，```tps```和```KB_read/s```都是原来的10倍了你敢信，可能那台电脑上有不少人都在用机械硬盘工作导致那个盘速度太慢。
* improvement 2：读取图片和切图的python的库，从```cv2```换成了```pillow-simd```, 从这里的[benchmarks分析](http://python-pillow.org/pillow-perf/)说后者在很多情况下比前者牛逼太多，换完后确实发现有提升。

## result
看到 tps和KB_read/s的数据 很稳定了，持续的读进来图片, CPU使用率也很稳定，保持在17左右。然后间接带来的效果，GPU的利用率也就比原来高了不少，不会经常在休息了。

优化之后，在公司的电脑上训练内存消耗为之前的二十分之一，200GB->10GB，在我的显卡运算能力和显存都相对较小的情况下，我的模型训练时间没有增加，模型训练效果与之前相同。

## future improvement
很明显可以多线程多个cpu同时开搞么，现阶段的改进已经足够这些天的实验了，就先不费功夫写多进程代码了。