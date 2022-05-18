<!--
 * @Descripttion: 
 * @Version: 1.0
 * @Author: ZhangHongYu
 * @Date: 2022-04-19 19:20:39
 * @LastEditors: ZhangHongYu
 * @LastEditTime: 2022-05-18 16:33:35
-->
# 基于 Bagging 和深度学习的上市公司财务数据造假预测
[![Open Source Love](https://badges.frapsoft.com/os/v2/open-source.svg?v=103)](https://github.com/orion-orion/TipDMCup20)
[![](https://img.shields.io/github/license/orion-orion/TipDMCup21)](https://github.com/orion-orion/TipDMCup21/LICENSE)
[![](https://img.shields.io/github/stars/orion-orion/TipDMcup21?style=social)](https://github.com/orion-orion/TipDMCup21)
[![](https://img.shields.io/github/issues/orion-orion/TipDMCup21)](https://github.com/orion-orion/TipDMCup21/issues)
### 赛题要求
本项目为泰迪杯2021A题《基于 Bagging 和深度学习的上市公司财务数据造假预测》，赛题有三个小问，分别是：  
1. 根据附件1的行业分类，利用附件2所提供的相关上市公司的财务数据，确定出各行业与财务数据造假相关的数据指标，并分析比较不同行业上市公司相关数据指标的异同。  
2. 根据附件2中制造业各上市公司的财务数据，确定出第6年财务数据造假的上市公司。  
3. 根据附件2中其他（除制造业外）各行业上市公司的财务数据，确定出第6年财务数据造假的上市公司。  

### 模型架构
对于问题一，我们在采用SMOTE过采样后，使用DecisionTree， RandomForest， ExtraTree， XGBoost四种树模型分别计算特征重要性权重值。我们综合这几种树模型的结果计算特征重要性权重均值，选出制造业和其他行业特征权重值排名前 30 的特征，作为上市公司财务数据造假有较大影响的特征因子。  
对于问题二和三，我们用深度学习模型代替了传统的树模型，我们以多层感知机，多层残差网络， Cross 网络作为子网络构建了(Deep-CrossResidual-NetWork， DCRN)网络模型。子网络完成特征的交叉组合，全连接层实现逻辑回归的二分类功能。其中多层残差网络通过短路操作解决梯度消失问题，Cross 网络通过类似外积的运算来进一步增加特征之间的交互力度。此外，我们还在引入了 Batch Normalize 层，起到了加速模型收敛，防止梯度消失与爆炸，缓解过拟合等作用。此外，我们舍弃了过采样的机制，依靠神经学习强大的学习能力对少量样本的特征进行捕捉。最后，我们在 DCRN 模型的基础上进行 Bagging 集成，进一步降低模型的方差(Variance)，从而提高模型的泛化能力，并缓解样本不均衡带来的影响。 在模型训练过程中，我们还引入 Dropout 机制与提前停止算法防止出现过拟合。最终我们的Bagging+DCRN 集成模型（制造业）在验证集上的 AUC 得分为 77.5%，高于所有单独的树模型，可见我们的验证集效果好，稳定性高。 该项目最终获得2021年泰迪杯A题二等奖。具体DCRN网络架构如项目目录下所示：

<img src="pic/bagging+dcrn.png" >

### 环境依赖
运行以下命令安装环境依赖：
```
pip install -r requirements.txt
```


### 数据集
数据集直接采用的赛方给定的企业高送转数据集，放在项目目录中的data文件夹下。

### 模型

为了免去大家训练特征选择模型和主模型的麻烦，我这里将特征选择模型和主模型都上传到了Google drive，大家可自行下载分别放在项目目录中的`features_model`和`model`文件夹下。下载链接可参见：

[Google drive 下载链接](https://drive.google.com/drive/folders/1l4BbSaxDv5Nswe0A1MZo5xRUB7cR6orM?usp=sharing)

### 项目目录说明
-data  -------------------  存放数据  

-features_imp  -------------------  存放选出top_n特征以及其重要性  

-features_model  -------------------  存放用于特征选择的相关模型(包括制造业和非制造业)  

-model  -------------------  存放最终训练的相关模型(包括制造业和非制造业)  

-pic  -------------------  存放DCRN网络架构图           

-prediction  -------------------  存放对第6年数据的预测结果

-roc_curve  -------------------  最终绘制的ROC曲线图   

-bagging.py  -------------------  bagging集成学习的模型架构  

-config.py  -------------------  存储模型的超参数，包括架构超参数、训练超参数等  

-DCRN.py  -------------------  DCRN模型架构，包括模型train_step、test_step等接口的实现  

-feature_eng.py  -------------------  特征工程的实现，包括数据预处理、特征选择、降维等  

-k_fold_cross_valid.py  -------------------  在训练集和验证集上进行K-fold交叉验证逻辑的实现  

-main.py  -------------------  主文件，用于从构建特征工程到模型的训练与评估的pipline  

-predict.py  -------------------  调用训练好的bagging模型对第6年的造假公司进行预测  

-utils.py  -------------------  一些和数据处理相关的算法的实现，包括bootstrap采样、样本shuffle，ROC曲线绘制等

### 使用方法
运行:

```
python main.py \
    --features_model load \
    --main_model load 
```

`features_model`参数表示选择是否重新开始训练特征选择模型，若需重新训练特征选择模型可将 `feature_model `参数设置为 `retrain `，否则设置为 `load`直接加载已经选取好的特征。

`main_model`参数表示是否重新开始训练主模型（即bagging+深度学习模型），若需重新训练主模型可将 `main_model `参数设置为 `retrain `，否则设置为 `load`直接加载已经训练并保存好的模型（但前提是模型已经放置于 `model`目录下）。


