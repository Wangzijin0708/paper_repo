# The Report of paper: Fine-tuning Large Language Models with Sequential Instructions

## 1. 作者: Hanxu Hu 相关论文

<img src="https://p.ipic.vip/fmuon3.png" alt="Hanxu Hu" style="zoom: 50%;" />

## 2. 相关论文: 介绍了传统instruction tuning的缺点

### 2.1 相关文献

- [FINETUNED LANGUAGE MODELS ARE ZERO-SHOT LEARNERS 2021.9](https://arxiv.org/pdf/2109.01652)
- [Scaling Instruction-Finetuned Language Models](https://arxiv.org/pdf/2210.11416)

### 2.2 instruction tuning

改善模型理解任务描述的能力，泛化能力：学习任务描述而不是标签。

<img src="https://p.ipic.vip/2ktfpd.png" alt="Instruction Tuning" style="zoom: 67%;" />

### 2.3 instruction finetuning的局限性

- 收集任务的ground-truth数据昂贵。
- 开放式创造性生成等任务没有正确答案，例如：“写一个关于狗和她的宠物蚱蜢的故事”。
- 语言建模平等惩罚所有标记级错误，但有些错误比其他错误更糟糕。

## 3. 论文目的

针对现今数据中顺序指令的稀缺性，我们提出了顺序指令微调（SIT），这是一种简单而有效的策略，可以自动增强指令调整数据，并装备LLMs执行多个顺序指令的能力。

## 4. 实现方法论

- 提示词序列: 将序列信息融入提示词，例如将两个步骤任务所需的提示词p1、p2替换为包含序列信息的单个提示词（与思维链不同）。
- SIT序列提示词微调：如何构建提示和输出以生成顺序指令数据格式。对于任务A和B，我们使用简单的模板“首先pA然后pB”连接相应的提示pA和pB，并按相同顺序连接所需的任务输出yA ⊕ yB。

## 5. 效果

![Effect](https://p.ipic.vip/g0u91w.png)

## 6. 训练过程

### 6.1 模型使用

- 文字模型Alpaca-LoRA
- 图像模型InstructBLIP

### 6.2 数据集处理

#### 6.2.1 文字类: 针对下游任务进行数据处理

##### 6.2.1.1 数据库cleaned version of the Alpaca data with 52K

- [GitHub链接](https://github.com/gururise/AlpacaDataCleaned)

<img src="https://p.ipic.vip/xg7ecj.png" alt="Alpaca Data" style="zoom:50%;" />

- 对于具有输入字段的实例，我们将其指令切换为包含两个子任务的顺序指令。
- 更新其输出字段，以包括两个任务的预期输出。
- 对于没有输入字段的其他训练实例，我们保持不变。

##### 6.2.1.2 重复-总结型任务数据库处理方法(验证可行性)

- dummy task概念。
- 在指令前面加入“First repeat the input, then answer”。

##### 6.2.1.3 多语言翻译-回答型任务数据处理

- 将alpaca数据翻译成多语言:中文,德语,俄语,西班牙语。
- 保持instruction永远是英语,无论输入输出是什么。
- 非英语输入,在指令前面加入“First translate the input into English, then”(模型具有泛化能力,可以英译其他语言)。

![Multi-Language](https://p.ipic.vip/m6azln.png)

#### 6.2.2 图像类

##### 6.2.2.1 数据库:a subset of the training split of VQAv2 (Goyal et al., 2017)

<img src="https://p.ipic.vip/m0wqcj.png" alt="VQAv2" style="zoom:67%;" />

- 设定instructions: “Answer the input question based on the image”。

##### 6.2.2.2 描述输入图像类型的任务

- 为每张图片添加描述(具体方法:a description for each image from MS COCO (Lin et al., 2014))。
- 设定instructions :  “First describe the image, then answer the input question based on the image”。

## 7. 和related work相比

### 7.1 chain-of-thought 

| 特点             | 顺序指令微调（SIT）                          | 链式思维（CoT）                                       |
| ---------------- | -------------------------------------------- | ----------------------------------------------------- |
| 核心思想         | 通过增加新的中间任务来增强模型能力           | 通过生成多步骤推理过程来提高回答质量                  |
| **中间任务类型** | **多样化的中间任务，如重复和改写(因果层面)** | **单一的“逐步推理”任务**                              |
| 下游任务         | 适用于广泛的下游任务                         | 主要针对需要推理的任务                                |
| 中间任务处理方式 | 将示例增强为包含多个子任务的复合指令         | 在回答最终问题之前执行逐步推理                        |
| 实现路径         | 通过模块化方法将子任务分配给不同的LLM专家    | 将LLM调用链式连接，每个查询的输出作为下一个查询的输入 |

## 8. 优点和缺点：顺序指令微调（SIT）

### 优点

1. **提升多任务处理能力**：处理包含多个步骤的复杂任务，提升模型在多任务指令中的表现。
2. **更好的泛化能力**：增强了鲁棒性，增强了模型的通用性和适应性。

### 缺点

1. **需要预定义中间任务**：在实际应用中可能无法预见所有可能的中间任务，选择不当可能导致模型性能下降或训练效率低下。
2. **数据集构建复杂**：SIT 需要构建复杂的数据集，非常耗时和复杂。
3. **训练成本高**：需要大量计算资源和时间。
4. **可能的过拟合风险**：如果中间任务设计不够多样化，模型可能会过拟合于特定类型的任务和指令，从而限制其在不同场景中的表现。
