# logbert 使用
logert结构
```bash
whyan@whyan-Lenovo-Legion-R70002021:~/miniconda3$ tree logbert
logbert
├── Pipfile
├── README.md
├── scripts         下载数据集的脚本，有的网站不可用了
├── environment     环境配置文档
├── bert_pytorch    logbert源程序库
├── logdeep         logdeep日志解析程序库
├── loglizer        提供其他日志异常场监测程序，评估用
├── logparser       日志聚类解析程序，data_process.py中parser函数调用。只用了drain
├── output          运行时动态生成，存储中间结果等
├── HDFS            HDFS数据集上实现程序文件夹
├── BGL             BGL数据集上实现程序文件夹
└── TBird           ThunderBird数据集上实现文件夹
```
## data_process.py 原始数据处理程序
### logparser：drain
![](parsing.png)

程序使用drain，另外提供了spell的程序

Drain是一种基于固定深度树的在线日志解析

流程：
1. 正则匹配提取数据tokens
2. 从解析树的根节点开始，根据token的数量区分日志数据组，到达第一层日志节点
3. 根据日志解析数据起始位置的token选择下一个内部节点
4. token相似度搜索，将新加入的日志token与已有日志组的token相比较，最大相似度超过阈值则认为属于该日志组，否则创建新的日志组
5. 更新解析树
![](refresh_tree.png)

参考链接：https://blog.csdn.net/qq_39378221/article/details/121212682?fromshare=blogdetail&sharetype=blogdetail&sharerId=121212682&sharerefer=PC&sharesource=2301_76811694&sharefrom=from_link

调用：

1. 实例化一个Logarsers类，规定 日志形式，数据集路径，输出路径，解析树深度，相似度阈值，maxchild，正则匹配表达式，keep_para。其中相似度阈值 'st' 和树的深度 'deep' 是主要需要调整的。
2. 调用类函数parser(log_file)，对数据集路径下的log_file进行parse

在logbert/HDFS/data_process.py logbert/BGL/data_process.py中可以看到调用实例

问题：
1. Drain似乎还不支持处理xlsx格式的日志，默认日志格式是文本形式。使用时需要调整
2. 怀疑logbert是否能支持处理中文
3. 运行BGL的 'logbert.py train' 时报错GPU内存不足，已知BGL在日志总数和大小上远小于HDFS，怀疑可能与BGL的log key数目过多或序列划分方式有关
### sampling
日志序列构建和处理

在HDFS中用mapping()和hdfs_sampling()函数实现，按session id构建，
在BGL和TBird中，按滑动窗口构建。

### 构建训练数据集
HDFS需要读取anamoly_label文件，BGL需要读取日志文件中的label。分别构建用于训练的数据集

## logbert.py

### hdfs上实现：
HDFS_v1数据集
```bash
hdfs
├── anomaly_label.csv 异常数据标注
├── Event_occurrence_matrix.csv 事件发生矩阵
├── Event_traces.csv
├── HDFS.log   数据集
├── HDFS.log_templates.csv
├── HDFS.npz
└── README.md
```