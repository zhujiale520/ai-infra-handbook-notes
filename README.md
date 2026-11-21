# AI Infra 面试笔记

> 《AI Infra Handbook》（孙晓辉，20 篇 153 讲、1342 页）的**精读 + 面试**笔记。
> 原书是逐讲展开的教学稿，这里把它拆掉讲稿、只留能答的东西，按 153 讲重新组织成两本：

| | 面试版 | 速查版 |
|---|---|---|
| PDF | [ai-infra-interview-notes.pdf](book/ai-infra-interview-notes.pdf) | [ai-infra-study-notes.pdf](book/ai-infra-study-notes.pdf) |
| 页数 | 184 页 | 84 页 |
| 每讲 | 2~3 个面试问法 + 4~6 条可开口的答题主线 + 3 组追问 + 数字/公式/别答错 | 一句话结论 + 3~6 条要点 + 关键数字 + 易错点 |
| 用途 | 提前两周准备，练「开口答」 | 面试前一晚扫一遍 |

两本都带**可点击目录**（153 讲逐条列页 + PDF 书签），公式全部 LaTeX 排版。

## 面试版长什么样

以第 66 讲「性能分析」为例：

```
66  性能分析
问  · 线上推理慢，你怎么定位瓶颈？
    · Profiler 时间线该怎么读？
    · GPU 利用率 91% 算好吗？
答  1. 性能分析就是别猜，把真实运行记下来看时间花在哪。工具分三层：PyTorch Profiler 先粗看算子……
    2. 第一步看 GPU 有没有空等。8B、TP=4、decode 一步 3.09 ms、GPU 忙 91%、空等 0.27 ms……
    3. 第二步按类别分账：矩阵乘 1.69 ms 占 55%，逐元素 0.49 ms、通信 0.35 ms、注意力 0.30 ms……
    4. 第三步按名字排序找大头：gate_up 0.80 ms、down 0.45 ms、All-Reduce 0.35 ms……
    5. 第四步和屋顶比：gate_up 一次 24.9 μs、读 58.7 MB、实际带宽 2.36 TB/s，只有峰值 3.35 的 70%……
    6. 测量本身有坑：kernel 异步发射，计时前后要 torch.cuda.synchronize()……
公式  还能优化多少 = 屋顶 − 实测（2.36 / 3.35 ≈ 70%）
数字  一步 3.09 ms、384 个 kernel……
追问  为什么先修空等再优化 kernel？
     空等是 GPU 干等着，怎么调 kernel 都是白搭。0.27 ms 约占整步 9%，换 CUDA Graph 一步就省 0.25 ms。
     多机的时候时间线怎么看？ / GPU 利用率 99% 就到位了吗？
别答错  nvidia-smi 的高利用率只说明有 kernel 在跑，不代表用上了带宽和算力。
```

答题主线统一按这个骨架写：**结论 → 机制（数据怎么流）→ 为什么更好（对比 + 量级）→ 代价与边界 → 工程取舍**。
追问固定三类：边界（「那 X 情况下还成立吗」）、量化（「不用这个方案代价多少」）、落地（「你怎么测」）。

## 规模

- **153 讲**，对应原书 20 篇（数学基础 → 神经网络 → 分词与嵌入 → Transformer → 现代 LLM 架构 → 训练 → 从零实现 GPT → 硬件与性能 → 推理基础 → 推理引擎与调度 → 算子与 Kernel → 压缩加速 → 分布式训练 → 分布式推理 → 手写迷你推理引擎 → nano-vllm → 主流引擎 → 硬件适配 → 调优 → 海光 DCU 案例）
- 面试版：**459 个问法、914 条答题主线、459 组追问**；末尾附「153 讲问题清单」，可当自测卷
- 速查版：153 张卡片、919 条要点，附录「153 讲一句话速查」

## 目录结构

```
.
├── book/
│   ├── ai-infra-interview-notes.pdf   面试版（184 页）
│   └── ai-infra-study-notes.pdf       速查版（84 页）
├── notes-interview.md                 面试版 Markdown（公式为 $...$，可在 Obsidian/Typora 渲染）
├── notes-study.md                     速查版 Markdown
├── data/
│   ├── index.json                     153 讲的编号、标题、篇、原书页码
│   ├── interview/group01..20.json     面试版结构化数据（字段见下）
│   └── study/group01..20.json         速查版结构化数据
└── tex/
    ├── interview-notes.tex            面试版 LaTeX 源码（XeLaTeX + ctex）
    └── study-notes.tex                速查版 LaTeX 源码
```

数据结构（`data/interview/*.json`）：

```json
{
  "no": 69,
  "title": "KV Cache",
  "ask": ["推理为什么要做 KV Cache？", "..."],
  "answer": ["...", "..."],
  "nums": "...",
  "formula": "$...$",
  "follow": [["追问问题", "怎么答"], ["...", "..."]],
  "pit": "..."
}
```

## 重新生成 PDF

```bash
# 需要 TeX Live（XeLaTeX + ctex + Noto CJK + Fandol 字体）
cd tex && xelatex -interaction=nonstopmode interview-notes.tex   # 跑 3 遍（目录 / 书签 / 引用）
xelatex -interaction=nonstopmode study-notes.tex
```

`tex/*.tex` 是完整可编译的源码，改完 `\linespread`、字号、页边距直接重编即可。

## 来源与致谢

- 内容整理自 **孙晓辉《AI Infra-HandBook》V1.0**（[ai-infra-handbook.sunxiaohui2022.workers.dev](https://ai-infra-handbook.sunxiaohui2022.workers.dev/)，[GitHub](https://github.com/sunxiaohui2025/AI-Infra-Handbook)）。
- 本仓库是**学习笔记**：每讲只保留结论、机制、数字、易错点与面试问答，原书的推导、代码与交互演示请回到原书看。
- 数字、公式、器件型号均来自原书；整理过程中原文的排版把个别数字与正文拆到了不同行，已按语义拼回。若与原书有出入，以原书为准。
- 如果原作者认为本仓库有不妥之处，请开 Issue，会立即处理。

## 许可

笔记文字采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/deed.zh)，仅供个人学习交流；原书内容的著作权归原作者所有。
