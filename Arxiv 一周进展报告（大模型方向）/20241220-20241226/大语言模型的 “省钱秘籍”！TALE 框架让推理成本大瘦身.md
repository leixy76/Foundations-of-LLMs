# Token-Budget-Aware LLM Reasoning

_Tingxu Han, Chunrong Fang 等_

_Nanjing University, Rutgers University, UMass Amherst 等_

本文研究的是**如何在大语言模型推理过程中有效管理Token预算，以平衡推理效率与成本**。随着大语言模型在各类任务中的广泛应用，CoT等推理技术虽提升了性能，但也引发了Token成本过高的问题，并且在实际应用场景中，往往只需准确简洁的答案，冗长的推理过程所产生的大量冗余Token造成了资源的浪费。鉴于此，本文提出了**TALE（Token-Budget-Aware LLM Reasoning）框架**，通过创新的预算估计方法，包括零样本估计器、回归估计器以及Token - budget意识内化等手段，精准地确定token预算，并借助基于预算的提示构建策略，引导LLM在预算范围内进行高效推理，从而在保持推理准确率的同时显著降低Token成本。 

## 研究内容
探究如何在保持大语言模型推理准确性的同时显著降低Token成本，具体聚焦于推理过程中的 Token 预算管理及效率优化，包括研究 LLM 推理时的 Token 冗余现象并分析其原因，并探索如何在保证推理准确性的前提下，减少不必要的 Token 消耗。

## 研究动机
大语言模型推理中的 Token 成本问题日益凸显。随着CoT等推理技术的应用，虽然提升了 LLM 在各类任务中的表现，但也导致了 token 使用量的大幅增加。同时，在很多实际场景中，只需获取准确的答案，冗长的推理和解释引入了大量资源浪费。

## 技术动机
借助Token 预算思想与动态Token预算优化Token效率。基于 LLM 能够遵循提示中的长度约束这一特性，可以将 Token 预算引入提示或对LLM进一步微调以约束其生成长度。基于资源分配和动态规划的理念，可以根据推理任务的复杂性和需求，动态分配 Token 预算，优化推理过程。

## 解决方案

![](https://fastly.jsdelivr.net/gh/bucketio/img9@main/2024/12/27/1735290462559-67f6e46b-e2f9-42e4-91e5-cbe89148ae89.png)

提出了**TALE框架**，通过动态估计和优化 **Token预算**，减少LLM在推理过程中的 **Token冗余**，同时保持较高的推理准确性。其解决方案包括以下三步：


##### 1. 最优**Token预算搜索**
为了找到能够在保持答案正确性的同时，最小化Token开销的最优预算，TALE设计了一种基于 **二分搜索** 和 **贪心策略** 的算法。具体步骤如下：

- **初始化搜索范围**：使用传统的CoT推理生成一个答案，并计算其Token数量，作为搜索的右边界，左边界设为0，表示最小的可能预算。
- **二分搜索**：在每次迭代中，计算当前搜索范围的中点，检查当前预算是否能够生成正确的答案。如果可行，则更新右边界为中点；否则，更新左边界为中点。
- **Token Elasticity 检测**：在每次迭代中，监控实际Token开销。如果发现Token开销随着预算的减少而增加，则说明进入了 **Token Elasticity** 区域。当检测到Token Elasticity现象时，停止进一步减少预算，将当前预算作为搜索的下限，并调整搜索范围继续搜索，确保预算在合理范围内。
- **贪心策略**：在保持答案正确性的同时，结合贪心策略，将二分搜索找到的最小预算设为当前最优预算，逐步减少预算并检查新预算是否仍然能够生成正确的答案，直到找到最优预算。最终找到的预算不仅能够保持答案正确性，还能最小化实际Token开销。


##### 2. **Token预算估计**
TALE提出了三种预算估计方法：

- **零样本估计器**：直接利用LLM自身作为预算估计器，通过提示LLM估计回答问题的Token预算。这种方法无需额外训练，直接利用LLM的推理能力生成预算。

- **回归估计器**：训练一个回归模型，使用**最优Token预算搜索**的结果作为训练数据，预测给定问题的最优Token预算。通过最小化预测值与实际最优预算之间的差异，回归估计器能够动态地为新问题生成合理的预算。

- **Token-budget 意识内化**：使用**最优Token预算搜索**的结果，构建Token-budget-aware提示，生成符合预算的推理步骤作为目标输出。微调时，使用目标输出作为训练数据，优化LLM的生成机制，使其在推理时自动生成简洁且符合预算的推理步骤。微调后的LLM无需显式指定Token预算，能够直接生成高效的推理过程。


##### 3. **Token-Budget-Aware提示构建**
在获得估计的Token预算后，TALE构建一个 **Token-budget-aware提示**，并将其输入LLM以生成最终的答案。根据估计的Token预算，构建提示，例如：“Let’s think step by step and use less than [budget] tokens: [question]”。通过动态调整token预算，确保LLM在生成推理步骤时既简洁又准确。

## 实验结果
在多个数据集（如 GSM8K、GSM8K-Zero、MathBench 等）和不同 LLM（Yi-lightning、GPT-4o-mini、GPT-4o 等）上，采用零样本估计器版本的TALE 框架平均减少 68.64% 的输出 Token 成本，同时保持较高准确率（平均准确率 81.03%，准确率损失小于 5%），显示出良好的预算估计准确性和有效性，且在不同 LLM 架构上通用性良好，证明其在平衡成本效率和推理性能方面的优势。

![](https://fastly.jsdelivr.net/gh/bucketio/img7@main/2024/12/27/1735290490436-def48ab0-fdb1-48cb-a1da-59c8676dd99c.png)

![](https://fastly.jsdelivr.net/gh/bucketio/img17@main/2024/12/27/1735290515622-40d0033f-7ee6-4296-bc50-fcec8e2f7e64.png)

---

- 查看 Arxiv 原文请点击"**阅读原文**"[https://arxiv.org/pdf/2412.18547]
- **更多**大模型学习资料，详见浙江大学LLMs Github仓库: 
  https://github.com/ZJU-LLMs/Foundations-of-LLMs
- 本文编辑：董雪梅，毛玉仁