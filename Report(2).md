# The Report of paper: Fine-tuning Large Language Models with Sequential Instructions

## 1. 作者: Hanxu Hu 相关论文

##  ![image-20240530013754231](https://p.ipic.vip/fmuon3.png)

## 2. 相关论文: 介绍了传统instruction tuning的缺点

### 2.1 相关文献

- [FINETUNED LANGUAGE MODELS ARE ZERO-SHOT LEARNERS 2021.9](https://arxiv.org/pdf/2109.01652)
- [Scaling Instruction-Finetuned Language Models](https://arxiv.org/pdf/2210.11416)

### 2.2 instruction tuning:

improve model's capability of understanding the task description,

泛化能力:学到了instruction对于task的描述方式,而不是学到了标签

![image-20240530004411051](https://p.ipic.vip/2ktfpd.png)

### 2.3 limitation of instruction finetuning:

- It is expensive to collect ground-truth data for tasks.
- tasks like open-ended creative generation have no right answer. e.g. ' Write me a story about a dog and her pet grasshopper'
- language modeling penalizes all token-level mistakes equally, but some errors are worse than others

## 3. 论文目的

Targeting the scarcity of sequential instructions in present-day data, we propose sequential instruction tuning (SIT), a simple yet effective strategy to automatically augment instruction tuning data and equip LLMs with the ability to execute multiple sequential instructions.

## 4. 实现方法论(methodology)

- 提示词序列: 将序列信息融入提示词,以两个step-task为例,将平常所需要的两个提示词的信息p1,p2换做一个有序列信息的提示词(区别于chain-of-thought)
  
- SIT序列提示词微调: 如何将提示和输出构建成顺序指令数据格式。对于任务A和B，我们使用简单的模板“首先pA然后pB”来连接相应的提示pA和pB。我们按照相同顺序连接所需的任务输出yA ⊕ yB

## 5. 效果

![image-20240530015548571](https://p.ipic.vip/g0u91w.png)

## 6. 训练过程

### 6.1 模型使用

- 文字模型Alpaca-LoRA
- 图像模型InstructBLIP

### 6.2 数据集处理

#### 6.2.1 文字类: 针对下游任务进行数据处理

##### 6.2.1.1 数据库cleaned version of the Alpaca data with 52K
- [GitHub链接](https://github.com/gururise/AlpacaDataCleaned)

<img src="https://p.ipic.vip/xg7ecj.png" alt="image-20240530081624840" style="zoom:50%;" />

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

![image-20240530080726639](/Users/haochengzhang/Library/Application Support/typora-user-images/image-20240530080726639.png)

#### 6.2.2 图像类

##### 6.2.2.1 数据库:a subset of the training split of VQAv2 (Goyal et al., 2017)

![image-20240530081429290](https://p.ipic.vip/m0wqcj.png)

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

## 结论

尽管顺序指令微调（SIT）有一些潜在的缺点，但它在提升大型语言模型处理复杂多任务指令能力方面表现出色，具有显著的优势和广泛的应用潜力。未来的研究可以进一步优化这种方法，减少其局限性，扩大其应用范围。