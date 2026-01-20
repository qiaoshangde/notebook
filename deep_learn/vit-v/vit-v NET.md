
**论文名称**

ViT-V-Net: Vision Transformer for Unsupervised Volumetric Medical Image Registration

**介绍**：
这篇文章首次将Transformer用于医学图像配准，代码的改动较小，且由于配准是一个Dense prediction的任务，而Transformer经过连续的下采样后，强调low-resolution的特征，这对于配准并不友好，所以这篇文章使用Transformer的Encoder结合CNN以及U-net结构，保证Dense prediction的flow field的生成。

在过去的十年里，卷积神经网络(ConvNets)在各种医学成像应用中占据了主导地位并取得了最先进的性能。然而，由于缺乏对图像中远程空间关系的理解，ConvNet的性能仍然受到限制。最近提出的用于图像分类的视觉转换器(VIT)使用了一种纯粹基于自我注意的模型，该模型学习远程空间关系以关注图像的相关部分。然而，由于连续的下采样，VIT强调低分辨率的特征，导致缺乏详细的定位信息，不适合图像配准。最近，几种基于VIT的图像分割方法被与ConvNets相结合，以提高对详细定位信息的恢复。受它们的启发，我们提出了VIT-V-Net，它连接了VIT和ConvNet，以提供3D医学图像配准。



数据集中T1与T2的区别


class ViT-V-Net一共包括四部分。Transformer，DecoderCup，RegistrationHead和SpatialTransformer。分别对应于实际网络架构的【CNN编码器和Transformer编码器】，【整个解码过程】，【生成flow的连接部分的卷积】，【空间变换网络】。
