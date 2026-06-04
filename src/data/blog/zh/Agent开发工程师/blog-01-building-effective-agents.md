---
title: Building effective agents
pubDatetime: 2026-06-04T00:00:00Z
description: Anthropic Building effective agents 文章阅读笔记，整理 Agent 与 Workflow 的区别、常见工作流模式、适用场景和设计原则。
featured: false
draft: false
tags:
  - Agent
  - LLM
  - AI工程
  - Anthropic
---

# Building effective agents

>Posts from Anthropic docs, links are here: https://www.anthropic.com/engineering/building-effective-agents

- 关于 LLM 的 Agent 组建，成功的实施是使用了最简单且可组合的模式

**they were building with simple, composable patterns.**

At Anthropic, we categorize all these variations as **agentic systems**, but draw an important architectural distinction between **workflows** and **agents**:

## Agents and Workflow

- Workflow

**工作流** 是同意预定义的代码路径协调 LLM 和工具的系统。

- Agents

**Agent** 是一种系统，在这种系统，LLM 可以动态指导自己的进程和相关的工具使用，并保持对完成任务方式的控制。

## When to use agents

作者写到，在使用 LLM 构建应用程序时，应当尽量简单的实现，只有在需要时再去增加一个系统的复杂性。意味着我们完全可以不建立 Agent 来实现某种应用程序来满足我们的需要。Agent 常常需要更多的 **延迟** 和 **成本** 来换取更好的任务性能，我们作为开发者应该权衡利弊。

当需要更多的复杂性的程序时，**Workflow** 可以为明确的任务提供可预测性和一致性，但是需要灵活性和大模型做决策时，Agent 是一种更为出色的选择。对于大部分的应用来说，使用检索和上下文的示例优化单个 LLM 的调用通常就足够了。

## When and how to use frameworks

关于何时和如何使用框架中，作者提供了 Claude SDK，AWS SDK，Rivet SDK，Vellum SDK。对于这些框架，它们定义了标准化的底层任务，比如调用 LLM，定义和解析工具以及将所有调用串联。但是他们通常会创建额外的抽象层，从而掩盖底层的显示和响应，这通常会让不熟悉框架和原理的开发者增加开发和调试的难度。总的来说，更建议直接调用 LLM API 和简单的几行代码实现相关的模式，如果一定要使用框架，要保证对底层代码的理解。

## Building blocks, workflows, and agents

### Building block: The augmented LLM

The basic building block of agentic systems is an LLM enhanced with augmentations such as retrieval, tools, and memory.

![image-20260604164849379](https://imgbed.sut.qzz.io/img/20260604164850805.webp)

总体来说，我们开发需要为我们的 Agent 系统提供简单，文档齐全的接口。可以参考 Anthropic 发布的 Model Context Protocol，该协议允许开发人员通过一个简单的客户端实现与不断增长的第三方工具生态系统集成。

### Workflow: Prompt chaining

工作流适合在任务可以轻松简洁的分解成固定的子任务的情况。主要目标是使得每个 LLM 可以调用成为更简单的任务，从而延迟换取更高的精度。

### Workflow: routing

路由可以对输入进行分类，并将其导向专门的后续任务。这种工作流程允许将关注点分开，并建立更专业的提示。**何时使用**：适合复杂的任务，在这些任务中，存在最好分开处理的类别，并且可以通过 LLM 或者更加传统的分类模型/算法准确处理分类。

![image-20260604171727102](https://imgbed.sut.qzz.io/img/20260604171727328.webp)

简单的例子：Vibe Coding 可以将简单的代码检索任务交给小模型，大的复杂的规划的任务交给更大的思考模型。

### Workflow: Parallelization

并行工作流模式，多使用在可以并行运行的任务，比如触发代码的审查，多个 LLM 可以并行进行相关的代码的审查。

![image-20260604171713357](https://imgbed.sut.qzz.io/img/20260604171713588.webp)

当划分的子任务可以并行化以提高速度，或需要多个视角或尝试以获得更高的置信度结果时，并行化是有效的。对于具有多个考虑因素的复杂任务，当每个考虑因素都由单独的 LLM 调用处理时，LLM 的性能通常会更好，从而可以集中关注每个特定方面。

### Workflow: Orchestrator-workers

建造者-工作者模式，中央的 LLM 会动态的分解任务，将任务分配给工作者 LLM，并综合相关的结果。

![image-20260604171700335](https://imgbed.sut.qzz.io/img/20260604171700561.webp)

此工作流非常适合无法预测所需子项任务的复杂任务（例如，在编码中，需要更改的文件数量和每个文件的更改性质可能取决于任务）。虽然在拓扑结构上相似，但与并行化的主要区别在于其灵活性--子任务不是预先定义的，而是由协调器根据具体输入决定的。

### Workflow: Evaluator-optimizer

评价器-优化器：一个 LLM 调用生成一个响应，而另一个则在循环中提供评价和相关的反馈

![image-20260604171651298](https://imgbed.sut.qzz.io/img/20260604171651517.webp)

当我们有明确的评估标准，并且迭代改进提供了可衡量的价值时，此工作流程就会特别有效。良好契合的两个标志是：第一，当人类明确提出反馈意见时，LLM 的响应可以得到明显改善；第二，LLM 可以提供此类反馈意见。这就好比人类作家在撰写一份精美文档时可能经历的反复写作过程。

## Agents

随着 LLM 在关键能力（理解复杂输入、参与推理和规划、可靠地使用工具以及从错误中恢复）方面的成熟，人工智能正在生产中崭露头角。代理通过人类用户的命令或与人类用户的互动讨论开始工作。一旦任务明确，代理就会独立进行规划和操作，并有可能返回人类获取进一步的信息或判断。在执行过程中，代理从环境中获取每一步的 "基本事实"（如工具调用结果或代码执行情况）以评估其进度至关重要。然后，代理可以在检查点或遇到阻碍时暂停，以获得人工反馈。任务通常会在完成后终止，但通常也会包含停止条件（如迭代的最大次数）以保持控制。

![image-20260604171633068](https://imgbed.sut.qzz.io/img/20260604171633317.webp)

代理可以处理复杂的任务，但它们的实现通常很简单。它们通常只是基于环境反馈循环使用工具的 LLM。因此，清晰周到地设计工具集及其文档至关重要。

 代理可用于开放式问题，在这种问题中，很难或不可能预测所需的步骤数，也无法硬编码固定的路径。LLM 可能会运行多个回合，你必须在一定程度上信任它的决策。代理的自主性使其成为在可信环境中扩展任务的理想选择。

## Combining and customizing these patterns

这些构件并不是规定性的。它们是常见的模式，开发人员可以根据不同的用例进行塑造和组合。与任何 LLM 功能一样，成功的关键在于衡量性能和迭代实施。重复一遍： 只有当复杂性能够明显改善结果时，您才应该考虑增加复杂性。

## Summary

在 LLM 领域取得成功并不是要建立最复杂的系统。而是要为您的需求构建 *正确* 的系统。从简单的提示开始，通过综合评估对其进行优化，只有当简单的解决方案无法满足要求时，才添加多步骤代理系统。这里有三个核心原则：

- 在代理的设计中要保持 **简单性**
- 通过明确显示代理的规划步骤，优先考虑 **透明度**
- 通过全面的工具 **文档和测试** ，精心设计您的代理-计算机接口（ACI）。

框架可以帮助您快速入门，但当您进入生产阶段时，不要犹豫减少抽象层并使用基本组件构建。遵循这些原则，您就能创建出不仅功能强大，而且可靠、可维护并深受用户信赖的代理。

