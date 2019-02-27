---
title: 图像检索调研
date: 2019-02-27 23:33:02
tags: CBIR 
categories: algorithm
---
图像检索大致可以分为两个不同的方向。

一个方向是追求图像相似性的检索，在gallery中找相似或相同，商品检索、草图检索都属于这个范畴，侧重点在于检索相同分类的图片，可以称作class-level retrieval。

另一个方向是对图像内容的检索，图像中包含的物体、场景、结构可能存在视角、光照、遮挡等问题，侧重点在于对图像内容的匹配，可以称作instance-level retrieval，或CBIR（Content Based Image Retrieval）。

以下内容主要针对CBIR

## instance-level

CBIR方向一篇综述[SIFT Meets CNN: A Decade Survey of Instance Retrieval](https://arxiv.org/abs/1608.01807)，详细介绍了instance retrieval的十年发展历程。

两大类方法，基于SIFT的方法和随后兴起的基于CNN方法，然而SIFT的方法并没有过时，在特定任务上仍然有优势。

| method type     | detector                   | descriptor                 | encoding                     | dim    | indexing       |
| --------------- | -------------------------- | -------------------------- | ---------------------------- | ------ | -------------- |
| SIFT-Large voc. | DoG, Hessian-Affine, etc.  | SIFT                       | Hard, soft                   | High   | Inverted index |
| SIFT-Mid voc.   | DoG, Hessian-Affine, etc.  | SIFT                       | Hard, soft, HE               | Medium | Inverted index |
| SIFT-Small voc. | DoG, Hessian-Affine,  etc  | SIFT                       | VLAD, Fisher Vector          | Low    | ANN methods    |
| CNN Hybrid      | Image patches              | CNN features               | VLAD, Fisher Vector, pooling | Varies | ANN methods    |
| CNN Pre-trained | column feature、FC         | column feature、FC         | VLAD, Fisher Vector, pooling | Low    | ANN methods    |
| CNN Fine-tuned  | global feature(end to end) | global feature(end to end) | global feature(end to end)   | Low    | ANN methods    |

### CNN base

SIFT meets CNN这篇综述中把基于CNN的方法分为了3类，其中混合式方法指从图像中取特定区域输入到网络中提取特征，再对特征进行编码、索引，这类方法利用CNN提取局部特征，类似于基于SIFT的方法，由于图像块多次执行网络前向过程，效率偏低，在这里不展开。

基于pretrained CNN模型的方法利用主流的CNN网络结构在大规模数据集上的预训练网络提取特征，再对特征进行编码、索引，这类方法主要研究重点在于特征的编码。其中提出MAC的论文[Visual instance retrieval with deep convolutional networks](https://arxiv.org/abs/1412.6574)是这类方法中的代表性工作。[DELF](https://arxiv.org/abs/1612.06321)方法是目前instance retrieval效果最好的模型，基于tensorflow实现，目前是tensorflow models中research的一个子工程。

基于finetuned CNN的检索方法核心是面向任务数据集进行网络的微调训练，同时end-to-end的生成图像级的global descriptor。其中用于微调主流网络是基于验证的网络结构siamese networks。

CVPR 2018的一篇文章[Revisiting Oxford and Paris: Large-Scale Image Retrieval Benchmarking](https://arxiv.org/abs/1803.11285v1)，详细对比了几种主流方法的性能，其中[DELF](https://arxiv.org/abs/1612.06321)方法结合Aggregated Selective Match Kernel和Spatial verification获得了压倒性的优势。在基于finetuned CNN的方法中，[MAC](https://arxiv.org/abs/1412.6574)、[R-MAC](https://arxiv.org/abs/1511.05879v2)、[GeM](https://arxiv.org/abs/1711.02512v1)提取的全局 descriptor有更好的性能。同时文章提出结合局部特征和全局特征的复合型方法可以获得更强的性能，同时开销也更大。

### Indexing

查找性能是高维向量的检索任务面临的一大难题。检索性能优化有两个方向，一是对高纬向量进行优化，例如降维、二进制特征、汉明空间、乘积量化（PQ）等，另一个是查找方式的优化，例如建立倒排索引、近似最近邻查找（ANN）。通常将两个方法结合使用。

CNN base的方法提取的特征维度更加紧凑，对Indexing的优化要求低于SIFT特征。最近的文章中主流做法都是基于ANN，结合KD-tree和PQ

### Datasets

instance retrieval任务的主要数据集：

| name             | images  | queries | content               |
| ---------------- | ------- | ------- | --------------------- |
| Holidays         | 1491    | 500     | scene                 |
| Ukbench          | 10200   | 10200   | common objects        |
| Paris6k          | 6412    | 55      | buildings             |
| Oxford5k         | 5062    | 55      | buildings             |
| Flickr100k       | 99782   | -       | Flickr's popular tags |
| Google-Landmarks | 1060709 | 12894   | landmarks             |

其中Ukbench数据用N-S score作为评价指标，在Ukbench中每个查询有4个正确项，N-S score指每个查询前四的匹配中正确匹配的平均数量。其余几个数据集均用mAP作为评价指标。Google-Landmarks是论文[DELF](https://arxiv.org/abs/1612.06321)提出的数据集，地标数量12894、数据集图像数量1060709、查询图像数量111036，且带有GPS信息。

### Roadmap

* 阅读相关论文，整理开源数据集和开源代码；
* 搭建基于finetuned CNN方法的instance retrieval最小系统，试用[faiss](https://github.com/facebookresearch/faiss)；
* 在最小系统上实验对比几种主流finetune CNN方法；
* 梳理业务需求，收集、迭代业务数据，定义问题；
* 完善流程，建立业务数据的baseline。
