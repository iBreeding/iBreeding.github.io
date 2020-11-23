# 快速入手

本节简要介绍CropGBM的功能及使用方法，帮助用户快速上手。有关CropGBM的详细介绍请参见 [详细教程](Tutorial.md)
<br/><br/>

## 程序安装

    $ tar -zxf CropGBM.tar.gz
        
    # 安装CropGBM的Python包依赖：setuptools, wheel, numpy, scipy, pandas, scikit-learn, lightgbm, matplotlib, seaborn
    $ pip install setuptools wheel numpy scipy pandas scikit-learn lightgbm matplotlib seaborn
    
    # 安装CropGBM的外部程序依赖：Plink 1.90 
    $ wget s3.amazonaws.com/plink1-assets/plink_linux_x86_64_20191028.zip
    $ mkdir plink_1.90
    $ unzip plink_linux_x86_64_20191028.zip -d ./plink_1.90

    # 将CropGBM、plink添加到系统环境变量中以便于快速使用：
    $ vi ~/.bashrc
    export PATH="/userpath/CropGBM:$PATH"
    export PATH="/userpath/plink1.90:$PATH"
    $ source ~/.bashrc
<br/>

## 参数配置

CropGBM支持 “配置文件” 与 “命令行” 两种参数赋值形式，其中“配置文件”是必须的。CropGBM会优先读取配置文件中各参数的值，再读取命令行中各参数的值。当某一参数被两种方式同时赋值时，CropGBM以命令行中参数值为参考，忽略配置文件中的参数值。

    # CropGBM读取配置文件（-c config_path）中各参数的值并调用基因型数据预处理模块（-pg all）
    $ cropgbm -c config_path -pg all

    # CropGBM忽略配置文件中fileformat值，而以ped为参考
    $ cropgbm -c config_path -pg all --fileformat ped

**注意：**若程序无法运行，请尝试在程序名前添加 python。如 _$ python cropgbm -c config_path -pg all_<br/><br/>

## 基因型数据预处理

基因型数据预处理模块的功能包括：提取指定样本ID、snpID的基因型数据，统计并直方图的形式展示snp缺失率、杂合率，基因型重编码等。为程序下游分析提供数据及可接受的文件格式。目前CropGBM支持的基因型文件输入格式有ped、bed、vcf和bcv。

    # 调用基因型数据预处理模块，统计并展示基因型数据的缺失率、杂合率等
    $ cropgbm -c config_path -o savepath -pg all --fileprefix genofile --fileformat ped
<br/>

## 表型数据预处理

表型数据预处理模块的功能包括：提取指定样本ID、snpID的表型数据，表型归一化，表型重编码等。同时支持以直方图或箱线图的形式展示数据的分布情况。

    # 调用表型数据预处理模块（-pp）进行归一化操作（--phe-norm）
    $ cropgbm -c config_path -o savepath -pp --phe-norm

    # 根据样本所属的群体类别（--ppgroupfile-path groupfile.txt）提取表型数据并以箱线图的形式展示
    $ cropgbm -c config_path -o savepath -pp --phe-plot --phefile-path phefile.txt --ppgroupfile-path groupfile.txt
<br/>

## 群体结构分析

群体结构分析模块可以基于基因型数据分析样本的种群结构。CropGBM支持使用t-SNE或PCA方法对基因型数据进行降维，使用OPTICS或Kmeans方法聚类。同时支持以散点图的形式展现样本的群体结构。

    # 调用群体结构分析模块（-s），根据基因型数据对样本进行聚类并展示（--structure-plot）
    $ cropgbm -c config_path -o savepath -s --structure-plot --genofile-path genofile --n-clusters 8
<br/>

## 构建模型与特征选择

模型训练模块主要基于lightGBM算法编写而成。为提高模型的准确性，建议提供验证集辅助调参。若无验证集，可利用交叉验证来选择合适的参数值。CropGBM根据训练模型中各snp的增益值筛选snp。同时支持用箱线图和热图展示被筛选出的snp的重要性。

    # 交叉验证（-e -cv）
    $ cropgbm -c config_path -o savepath -e -cv --traingeno train1.geno --trainphe train1.phe

    # 构建训练模型（-e -t）。若无验证集数据，--validgeno和--validphe参数可以省略。
    $ cropgbm -c config_path -o savepath -e -t --traingeno train1.geno --trainphe train1.phe --validgeno valid.geno --validphe valid.phe

    # 特征选择（-e -t -sf），展示模型预测精度的变化（--bygain-boxplot）
    $ cropgbm -c config_path -o savepath -e -t -sf --bygain-boxplot --traingeno genofile --trainphe phefile
<br/>

## 表型预测

表型预测模块利用模型训练模块输出的模型，预测测试集样本的表型。 

    # 表型预测（-e -p）
    $ cropgbm -c config_path -o savepath -e -p --testgeno genofile --modelfile-path modelfile