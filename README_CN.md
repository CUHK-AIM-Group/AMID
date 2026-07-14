<p align="center">
  <img src="assets/logo.png" alt="AMID 标志" width="180">
</p>

# AMID：迈向自主且可审计的医学影像模型开发
<!-- <p align="center">
  Shengyuan Liu<sup>1,*</sup>, Jia-Xuan Jiang<sup>1,*</sup>, Boyun Zheng<sup>1</sup>, Cheng Wang<sup>1</sup>, Wentao Pan<sup>1</sup>, Zipei Wang<sup>2</sup>, Houwen Peng<sup>5</sup>, Yu Gu<sup>3</sup>, Lichao Sun<sup>4,&dagger;</sup>, Yixuan Yuan<sup>1,&dagger;</sup>
</p> -->

<p align="center">
  <a href="https://arxiv.org/pdf/2607.10522"><img src="https://img.shields.io/badge/arXiv-2607.10522-b31b1b.svg?style=flat-square" alt="arXiv"></a>
  <a href="README.md"><img src="https://img.shields.io/badge/Language-English-blue?style=flat-square" alt="English README"></a>
</p>

<!-- <p align="center">
  <sup>*</sup>共同贡献 &nbsp; <sup>&dagger;</sup>通讯作者
</p> -->

## 🚀 概览

AMID 是一个面向医学影像模型开发的自主多智能体框架。给定任务定义和数据集后，AMID 通过数据驱动的方法规划、多智能体优化，以及由验证结果引导的最终产物选择，为特定任务构建模型解决方案。

![AMID 系统概览](assets/amid-system-overview.png)

AMID 旨在解决医学影像模型开发中的实际问题。它所需的输入被有意精简为：一个医学影像数据集，以及一份用于指定目标输出、评估指标，并在适用时说明提交协议的任务定义。数据集可以包含二维图像、三维体数据、病理图像块、配对增强数据、检测标注、分割掩码、类别标签、图标签或图像质量评分。

任务定义可以来自临床建模需求，也可以来自挑战赛说明。系统的预期输出并非单一的文本回答或建议采用的模型架构，而是一套完整的模型产物包，包括：可执行的训练与推理代码、模型权重或检查点、预测文件、验证分数、需要时的最终提交产物，以及能够证明结果是在正确的数据、指标、数据划分和提交规范下生成的审计记录。

## ✨ 待办事项

- [ ] 发布 AMID 源代码。
- [x] 发布 AMID 技术报告。
- [x] 发布全部 20 项 ReX-MLE 医学影像任务对应的专项解决方案报告。

## 📦 评估

本项目使用的挑战基准来自 [ReX-MLE](https://github.com/rajpurkarlab/ReX-MLE)，其中包含 20 项医学影像挑战任务。

我们已开源全部 20 项 ReX-MLE 医学影像任务对应的专项解决方案报告。下表汇总了每项挑战的任务类型、主要指标、AMID 得分及解决方案报告链接。

| 挑战 | 任务类型 | 主要指标 | AMID 得分 | 报告 |
|---|---|---:|---:|---|
| DENTEX | 目标检测 | AP | 0.49 | [解决方案](solutions/dentex/) |
| ISLES'22 | 图像分割 | Dice | 0.71 | [解决方案](solutions/isles22/) |
| LDCT-IQA | 图像质量评估 | Score | 2.74 | [解决方案](solutions/ldct-iqa/) |
| NeurIPS-CellSeg | 图像分割 | F1 | 0.90 | [解决方案](solutions/neurips-cellseg/) |
| PANTHER-T1 | 图像分割 | Dice | 0.42 | [解决方案](solutions/panther-task1/) |
| PANTHER-T2 | 图像分割 | Dice | 0.31 | [解决方案](solutions/panther-task2/) |
| PUMA-T1-Seg | 组织分割 | Dice | 0.56 | [解决方案](solutions/puma-track1-task1/) |
| PUMA-T1-Det | 细胞核检测 | F1 | 0.54 | [解决方案](solutions/puma-track1-task2/) |
| PUMA-T2-Seg | 组织分割 | Dice | 0.56 | [解决方案](solutions/puma-track2-task1/) |
| PUMA-T2-Det | 细胞核检测 | F1 | 0.28 | [解决方案](solutions/puma-track2-task2/) |
| SEG.A | 主动脉血管树分割 | Dice | 0.91 | [解决方案](solutions/seg-a/) |
| TopBrain-CTA | CTA 血管分割 | Mean Dice | 0.59 | [解决方案](solutions/topbrain-track1/) |
| TopBrain-MRA | MRA 血管分割 | Mean Dice | 0.64 | [解决方案](solutions/topbrain-track2/) |
| TopCoW-CTA-Seg | CTA 多类别分割 | Mean Dice | 0.63 | [解决方案](solutions/topcow-track1-task1/) |
| TopCoW-CTA-Det | CTA 三维框检测 | IoU | 0.71 | [解决方案](solutions/topcow-track1-task2/) |
| TopCoW-CTA-Cls | CTA 图分类 | Accuracy | 0.33 | [解决方案](solutions/topcow-track1-task3/) |
| TopCoW-MRA-Seg | MRA 多类别分割 | Mean Dice | 0.76 | [解决方案](solutions/topcow-track2-task1/) |
| TopCoW-MRA-Det | MRA 三维框检测 | IoU | 0.74 | [解决方案](solutions/topcow-track2-task2/) |
| TopCoW-MRA-Cls | MRA 图分类 | Accuracy | 0.46 | [解决方案](solutions/topcow-track2-task3/) |
| USenhance | 超声图像增强 | LNCC | 0.19 | [解决方案](solutions/usenhance/) |

未来，我们将持续更新 AMID 系统，并开源系统代码以及更多医学影像任务的解决方案报告。

## 📝 引用

如果您对我们的工作感兴趣，欢迎通过电子邮件联系我们：

- liushengyuan@link.cuhk.edu.hk
- yxyuan@ee.cuhk.edu.hk

论文的 BibTeX 信息如下：

```
@misc{liu2026autonomousauditablemedicalimaging,
      title={Towards Autonomous and Auditable Medical Imaging Model Development}, 
      author={Shengyuan Liu and Jia-Xuan Jiang and Boyun Zheng and Cheng Wang and Zipei Wang and Wentao Pan and Hongtao Wu and Houwen Peng and Yu Gu and Lichao Sun and Yixuan Yuan},
      year={2026},
      eprint={2607.10522},
      archivePrefix={arXiv},
      primaryClass={cs.CV},
      url={https://arxiv.org/abs/2607.10522}, 
}
```
