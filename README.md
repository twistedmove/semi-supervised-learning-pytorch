# ssl (semi-supervised learning)
This repository contains code to reproduce "[Realistic Evaluation of Deep Semi-Supervised Learning Algorithms](https://arxiv.org/abs/1804.09170)" in pytorch. Currently, only supervised baseline, PI-model[2] and Mean-Teacher[3] are implemented. We attempted to follow the description in the paper, but there are several differences made intentionally. There may be other differences made accidentally from experiments in the paper. 

# Prerequisites
Tested on 
* python 2.7
* pytorch 0.4.0

Download ZCA preprocessed CIFAR-10 dataset
* As described in the paper, global contrast normalize (GCN) and ZCA are important steps for the performance. We preprocess CIFAR-10 dataset using the code implemented in [Mean-Teacher repository](https://github.com/CuriousAI/mean-teacher). The code is in tensorflow/dataset folder.
Place the preprocessed file (e.g. cifar10_gcn_zca_v2.npz) into a subfolder (e.g. cifar10_zca).

# Experiment detail
We use WideResnet same as original paper. However we use **WideResnet28x3 with dropout rate 0.3** while **WideResnet28x2 without dropout** is used in the original paper. We widen the network since we could not obtain same performance with the original setting. There may be some different settings from the paper. We add dropout since it is usually implemented in their original networks [2,3]. The WideResnet code is modified from source in [ODIN](https://github.com/facebookresearch/odin) repository under Creative Commons Attribution-NonCommercial 4.0 International Public License. Please refer to the LICENSE file in the original repository. 

(We also implemented pre-activation Resnet with stochastic depth but could not obtain comparable results) 

We follow the hyperparameter provided in the **appendix B** of [1]. We use Adam optimizer with learning rate schedule. The inital learning rate is decayed by the factor of 0.2 at 4/5 of entire epochs. We choose **batch size of 225** and use **2GPUs** for training while **100** and **1 GPU** in [1]. In [1], the training is done for **500k iterations**. We tried to match the number of entire samples shown by the network. The network is trained for **12000 epochs for basline** and **1200 epochs for semi-supervised learning methods**.  

As stated above, we prepocess the CIFAR-10 dataset with GCN and ZCA. we found that it achieves around 1~2% performance improvement over meanstd normalization. Random translation (up to 2 pixels) and random horizontal flip is applied for CIFAR-10. We assume only 4k training samples have labels and 5k samples are used for validation among 50k training set. We run 10 times with different label/unlabel division. 

# To Run
For basline 
    
    python train.py -a=wideresnet -m=baseline -o=adam -b=225 --dataset=cifar10_zca --gpu=0,1 --lr=0.003 --boundary=0

 For Pi model

    python train.py -a=wideresnet -m=pi -o=adam -b=225 --dataset=cifar10_zca --gpu=0,1 --lr=0.0003 --boundary=0
 For Mean Teacher

    python train.py -a=wideresnet -m=mt -o=adam -b=225 --dataset=cifar10_zca --gpu=0,1 --lr=0.0004 --boundary=0
    
* boundary option is for different label/unlabel division [0, 9].
    
You can check the average error rates for `n` runs using `check_result.py`. For example, you trained baseline model on 10 different boundary,

    python check_result.py --fdir ckpt_cifar10_zca_wideresnet_baseline_adam_e1200/ --fname wideresnet --nckpt 10 
    
# Result (CIFAR-10)
|Method       |WideResnet28x2 [1]    |WideResnet28x3 w/ dropout (ours)   |
|-------------|----------------------|-----------------------------------|
|Supervised   |20.26 (0.38)          |20.84(0.68)                        |
|PI Model     |16.37 (0.63)          |15.74(0.53)                        |
|Mean Teacher |15.87 (0.28)          |14.47(0.34)                        |
|VAT          |13.86 (0.27)          |-                                  |
|VAT + EM     |13.13 (0.39)          |-                                  |


# References
[1] Oliver, Avital, et al. "Realistic Evaluation of Deep Semi-Supervised Learning Algorithms." arXiv preprint arXiv:1804.09170 (2018).

[2] Laine, Samuli, and Timo Aila. "Temporal ensembling for semi-supervised learning." arXiv preprint arXiv:1610.02242 (2016).

[3] Tarvainen, Antti, and Harri Valpola. "Mean teachers are better role models: Weight-averaged consistency targets improve semi-supervised deep learning results." Advances in neural information processing systems. 2017.

[4] https://github.com/CuriousAI/mean-teacher

[5] https://github.com/facebookresearch/odin
