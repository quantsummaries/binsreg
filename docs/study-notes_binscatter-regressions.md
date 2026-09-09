# Binscatter Regressions（binsreg：Stata 套件）· 学习笔记

> 把"分箱散点图"从画图工具升级为**有严谨统计理论支撑的 Stata 命令套件**：估计、绘图、推断、检验、最优分箱一揽子解决。

| 项目 | 内容 |
| --- | --- |
| 文献 | Cattaneo, Crump, Farrell, Feng, "Binscatter Regressions"，The Stata Journal 2025 |
| 交付物 | Stata 软件包 **binsreg**（7 条命令） |
| 理论依据 | 作者团队 CCFF (2024a, 2024b) 的分箱散点理论（On Binscatter 与 Nonlinear Binscatter Methods） |
| 项目主页 | https://nppackages.github.io/binsreg/ |
| 安装 | `net install st0765` → `net sj 25-1` → `net get st0765` |

> **来源说明**：本笔记依据用户提供的该文**中文总结文档**（5 页）整理；源文档末尾注明"部分内容可能由 AI 生成"，关键命令与选项建议以官方帮助文件为准。

## 一句话读懂

传统 binscatter（含 Stata 旧命令 `binscatter` / `binscatter2`）的协变量调整采用"先残差化再分箱"的做法，**一般情形下估计不一致**，且只能画分段常数最小二乘、没有有效置信带/检验/组间比较/数据驱动选箱。`binsreg` 套件基于作者的理论工作，用 7 条命令统一实现**估计、可视化、统计推断、最优分箱选择**，支持线性、Logit、Probit、分位数回归，兼容 `reghdfe` 多维固定效应与 `gtools` 大数据加速，并内置质量点与自由度校验。

## binsreg 套件总览

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 340" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <defs>
    <marker id="bra" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#8FA0F5"/>
    </marker>
  </defs>
  <!-- 阶段1 -->
  <rect x="18" y="70" width="200" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="18" y="70" width="200" height="8" rx="4" fill="#8FA0F5"/>
  <text x="118" y="102" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">输入与选箱</text>
  <text x="38" y="128" fill="#1C2733" font-size="12.5">样本 (yᵢ, xᵢ, wᵢ)</text>
  <text x="38" y="148" fill="#1C2733" font-size="12.5">权重 · 聚类 · 分组 by()</text>
  <rect x="34" y="164" width="168" height="52" rx="8" fill="#DCE7FF"/>
  <text x="118" y="184" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">binsregselect</text>
  <text x="118" y="202" text-anchor="middle" fill="#3A4DA8" font-size="12">Ĵ_DPI（默认）/ Ĵ_ROT</text>
  <text x="118" y="246" text-anchor="middle" fill="#6B7A8A" font-size="12">IMSE 最优分箱数</text>
  <line x1="222" y1="170" x2="246" y2="170" stroke="#8FA0F5" stroke-width="2" marker-end="url(#bra)"/>
  <!-- 阶段2 -->
  <rect x="252" y="70" width="230" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="252" y="70" width="230" height="8" rx="4" fill="#8FA0F5"/>
  <text x="367" y="102" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">估计与绘图</text>
  <rect x="266" y="116" width="202" height="46" rx="8" fill="#DCE7FF"/>
  <text x="367" y="136" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">binsreg（最小二乘）</text>
  <text x="367" y="152" text-anchor="middle" fill="#3A4DA8" font-size="12">binslogit · binsprobit（二元）</text>
  <rect x="266" y="170" width="202" height="46" rx="8" fill="#DCE7FF"/>
  <text x="367" y="190" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">binsqreg（分位数）</text>
  <text x="367" y="206" text-anchor="middle" fill="#3A4DA8" font-size="12">bootstrap 标准误</text>
  <text x="367" y="238" text-anchor="middle" fill="#1C2733" font-size="12">dots · line · ci · cb</text>
  <text x="367" y="256" text-anchor="middle" fill="#3A4DA8" font-size="12">polyreg(P) · at() · by()</text>
  <line x1="486" y1="170" x2="510" y2="170" stroke="#8FA0F5" stroke-width="2" marker-end="url(#bra)"/>
  <!-- 阶段3 -->
  <rect x="516" y="70" width="230" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="516" y="70" width="230" height="8" rx="4" fill="#8FA0F5"/>
  <text x="631" y="102" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">检验与比较</text>
  <rect x="530" y="116" width="202" height="58" rx="8" fill="#DCE7FF"/>
  <text x="631" y="136" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">binstest</text>
  <text x="631" y="154" text-anchor="middle" fill="#3A4DA8" font-size="12">参数设定检验（线性/多项式）</text>
  <text x="631" y="170" text-anchor="middle" fill="#3A4DA8" font-size="12">形状检验（单调/凹凸）</text>
  <rect x="530" y="182" width="202" height="58" rx="8" fill="#DCE7FF"/>
  <text x="631" y="202" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">binspwc</text>
  <text x="631" y="220" text-anchor="middle" fill="#3A4DA8" font-size="12">组间成对比较</text>
  <text x="631" y="236" text-anchor="middle" fill="#3A4DA8" font-size="12">异质性处理效应</text>
  <line x1="750" y1="170" x2="774" y2="170" stroke="#8FA0F5" stroke-width="2" marker-end="url(#bra)"/>
  <!-- 阶段4 -->
  <rect x="780" y="70" width="202" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="780" y="70" width="202" height="8" rx="4" fill="#8FA0F5"/>
  <text x="881" y="102" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">共同地基</text>
  <text x="800" y="128" fill="#1C2733" font-size="12.5">随机分箱 + 部分线性</text>
  <text x="800" y="148" fill="#1C2733" font-size="12.5">M-估计理论</text>
  <text x="800" y="168" fill="#6B7A8A" font-size="12">（CCFF 2024a, 2024b）</text>
  <text x="800" y="196" fill="#1C2733" font-size="12.5">质量点 / 自由度校验</text>
  <text x="800" y="216" fill="#1C2733" font-size="12.5">权重 · 聚类稳健方差</text>
  <text x="800" y="244" fill="#1C2733" font-size="12.5">兼容 reghdfe · gtools</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 A｜binsreg 套件总览：选箱 → 估计绘图 → 检验比较，全部建立在随机分箱 + 部分线性 M-估计理论之上（依据总结文档内容自绘，示意）</div>
</div>

## 01 背景与动机

**传统散点图的痛点**：大数据（行政数据、社科/医学数据）下普通散点图形成密集点云、信息失效；且无法在绘图时严谨控制其他协变量。分箱散点图（binscatter）解决了这两个问题，在实证微观经济学中被广泛使用——但早期工具缺少完整统计理论支撑。

**旧 Stata 命令的缺陷**（`binscatter`、`binscatter2`）：

- 协变量调整方法**错误**：采用"先残差化再分箱"，只有极强假设下才可信，一般情形下估计量非一致；
- 只支持分段常数最小二乘绘图；
- 没有有效的置信区间/置信带、形状检验、组间比较、数据驱动最优分箱等功能。

**本文贡献**：基于作者团队 CCFF (2024a, 2024b) 的理论工作，开发 `binsreg` 套件——共 **7 条 Stata 命令**，统一实现估计、可视化、统计推断、最优分箱选择，提供严谨的大样本理论基础。

## 02 七条命令总览

| 命令 | 功能 |
| --- | --- |
| `binsreg` | 最小二乘线性分箱散点：绘图、点估计、置信区间 / 置信带 |
| `binslogit` | Logit 非线性分箱散点（二元被解释变量） |
| `binsprobit` | Probit 非线性分箱散点（二元被解释变量） |
| `binsqreg` | 分位数回归分箱散点 |
| `binstest` | 假设检验：参数形式设定检验 + 非参数形状约束检验（单调性、凹凸性等） |
| `binspwc` | 多组样本成对统计比较，用于异质性处理效应分析 |
| `binsregselect` | 数据驱动最优分箱数量选择器，计算 IMSE 最优分箱数 |

套件整体支持：协变量调整、权重、聚类标准误、多样本分组分析；兼容 `reghdfe` 多维固定效应、`gtools` 加速大数据运算；内置**质量校验**（质量点 mass-point、自由度校验）保障数值稳定性。

## 03 方法原理

### 3.1 模型设定

样本 (yᵢ, xᵢ, wᵢ)：yᵢ 结果变量，xᵢ 核心连续解释变量，wᵢ 控制变量向量。目标估计函数为（M-估计框架）：

> `argmin Σ ρ( yᵢ ; η( μ(xᵢ) + wᵢ′γ ) )` 及其 v 阶导数形式
>
> `v`：求导阶数，v = 0 为水平函数，v = 1 为一阶导数；`ρ(·)` 损失函数；`η(·)` 逆连接函数

可退化为三类常用模型：**半线性回归**、**Logit/Probit 二元回归**、**分位数回归**。

两个注意点：

- **模型设定错误时**：估计量只是最小化损失函数的近似解（仍有清晰的概率极限解释）；
- **i.i.d. 假设被违背时**（如时间序列）：可视化仍可运行，但**估计与推断不再具备理论保证**。

### 3.2 分箱构造

1. **默认分位数分箱**：把 xᵢ 取值区间切成 J 个箱子，每箱样本量大致相等；也支持等距分箱、用户自定义分箱；
2. **箱内多项式 + 跨箱光滑约束**：每箱内部用 p 阶多项式拟合；s 代表跨箱子连续可导阶数（对应 B 样条基函数）：
   - `s = 0`：分段多项式，箱边界不连续 → 传统经典 binscatter（p = 0, s = 0 即每箱取 y 均值）；
   - `s = 1`：函数在箱边界连续；
   - `s = 2`：函数连续且一阶导数连续。
3. **新旧协变量调整的重大区别**：
   - **旧工具（残差化）**：先分别把 y、x 对控制变量回归取残差，再对残差做 binscatter → 一般不一致；
   - **binsreg**：把样条基函数与控制变量**直接放进目标损失函数联合估计** → 理论可靠，即使模型误设定也有清晰概率极限解释。

### 3.3 最优分箱数 J 的选择

J 是非参数方法的调参参数，需随样本量趋于无穷。套件基于 **IMSE（积分均方误差）** 准则，提供两种选择器：

- `Ĵ_ROT`：经验法则（rule-of-thumb），用参考高斯密度与全局多项式近似偏差、方差——速率正确，但常数项非一致估计；
- `Ĵ_DPI`：直接插件法（direct plug-in），用初步估计一致估计偏差、方差常数——**程序默认 DPI**。

两种使用模式：

- **模式 1**：给定多项式阶数 p、光滑度 s → 选最优箱数 J；
- **模式 2**：用户手动固定箱子 J → 反推最优多项式阶数 p 与光滑约束 s（兼顾绘图美观与统计有效性）。

经验研究者常用固定整数分箱（10 / 20 / 50 / 100）方便解读分位数，但需意识到**固定 J 不一定统计最优**。

### 3.4 置信区间、置信带与稳健偏差修正

1. **点式置信区间 CI**：针对单个 x 点；
2. **一致置信带 CB**：对整个函数在 x 全部取值域同时推断，适合检验整体函数形态；
3. **关键问题**：直接用 IMSE 最优分箱得到的点估计存在**一阶模型近似偏差**，直接构造置信区间会失效；
4. **解决方案——稳健偏差修正（RBC）**：保持原有分箱划分不变，使用**更高阶多项式 p+q（q ≥ 1）**构造置信区间 / 置信带，消除近似偏差、保证覆盖率；
5. 方差用**三明治形式**，支持聚类方差估计；置信带临界值靠**高斯模拟**（从多维正态向量重复抽样）得到。

### 3.5 假设检验（binstest）

两类检验，均支持稳健偏差修正，保证一类错误可控并具备检验势：

1. **参数设定检验**：非参数 binscatter 拟合 vs 假设参数模型（常数、线性、二次多项式、外部自定义模型）对比；
   - **提醒**：对函数水平（v = 0）的检验结果会受协变量评估点 w 影响——**推荐优先检验一阶导数 v = 1** 来检验线性关系，规避评估点带来的误导；
2. **非参数形状约束检验**：单侧 / 双侧检验，如函数单调、凹 / 凸、函数值不大于常数等；例如原假设"一阶导数 ≥ 0"即函数单调不减。

### 3.6 多组比较（binspwc）

支持分组 binscatter 绘图（`by()` 选项）；`binspwc` 执行**组间成对假设检验**，原假设为两组回归函数完全相等——用于研究随 x 变化的**异质性处理效应**。同样需要注意协变量评估点 w 对结果解读的影响。

## 04 实现细节与实操要点

1. **协变量评估点 `at()`**：存在控制变量 wᵢ 时，绘图展示的条件期望可设定评估点——`at(mean)`（默认）、`at(median)`、全 0 向量，或用外部文件自定义评估值。评估点会改变**图像的水平位移与置信区间**。注意：配合 `reghdfe, absorb()` 高维固定效应时，评估点**不可自定义**。
2. **质量点与自由度校验**：
   - 用 xᵢ 唯一取值数目 **N** 作为有效样本量（不是原始观测 n）；聚类数据则用聚类数 **G**；
   - 有效样本不足时，程序自动关闭高阶非参数推断、仅允许简单分段常数估计，并给出警告；
   - 选项 `masspoints()`、`dfcheck()` 可修改校验规则。
3. **大数据提速技巧**：
   - 数据预先按 x 排序；
   - x 连续无重复值：`masspoints(off)` 关闭质量点校验；
   - 手动给定 `nbins()`，避免自动数据驱动选箱；
   - 安装 `gtools` 并配合 `usegtools(on)`；高维固定效应用 `reghdfe`；
   - 初步探索时调低模拟次数 `nsims()`、网格点 `simsgrid()`。
4. **绘图选项**：`dots(p s)` 散点图、`line(p s)` 拟合曲线、`ci(p s)` 点式置信区间、`cb(p s)` 一致置信带——各元素可使用**不同的 p、s 参数**实现稳健偏差修正；`savedata()` 可导出绘图数据；支持标准 `twoway` 绘图选项美化图表。

## 05 模拟示例与命令速查

使用自带模拟数据集 `binscatter_simdata.dta` 演示。推荐的分析工作流如下：

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 400" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <defs>
    <marker id="brb" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#8FA0F5"/>
    </marker>
  </defs>
  <!-- 步骤1 -->
  <rect x="180" y="14" width="640" height="54" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="36" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">① 数据准备</text>
  <text x="500" y="56" text-anchor="middle" fill="#3A4DA8" font-size="12.5">按 x 排序 · 设定权重 / 聚类 / 分组变量</text>
  <path d="M500,68 L500,92" stroke="#8FA0F5" stroke-width="2" marker-end="url(#brb)"/>
  <!-- 步骤2 -->
  <rect x="180" y="94" width="640" height="54" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="116" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">② 选箱：binsregselect y x w</text>
  <text x="500" y="136" text-anchor="middle" fill="#3A4DA8" font-size="12.5">输出 Ĵ_ROT 与 Ĵ_DPI（默认 DPI）· savegrid() 供外部模型设定检验 · randcut() 加速</text>
  <path d="M500,148 L500,172" stroke="#8FA0F5" stroke-width="2" marker-end="url(#brb)"/>
  <!-- 步骤3 -->
  <rect x="180" y="174" width="640" height="54" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="196" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">③ 估计与绘图：binsreg y x w</text>
  <text x="500" y="216" text-anchor="middle" fill="#3A4DA8" font-size="12.5">line() / ci() / cb() · polyreg(P) · vce(cluster id) · by(t) 分组 · at() 评估点</text>
  <path d="M500,228 L500,252" stroke="#8FA0F5" stroke-width="2" marker-end="url(#brb)"/>
  <!-- 步骤4 -->
  <rect x="180" y="254" width="640" height="54" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="276" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">④ 检验：binstest y x w</text>
  <text x="500" y="296" text-anchor="middle" fill="#3A4DA8" font-size="12.5">testmodelpoly(1) 检验线性（优先 v=1）· deriv(1) testshaper(0) 检验单调不减</text>
  <path d="M500,308 L500,332" stroke="#8FA0F5" stroke-width="2" marker-end="url(#brb)"/>
  <!-- 步骤5 -->
  <rect x="180" y="334" width="640" height="54" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="356" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">⑤ 组间比较：binspwc y x w, by(t)</text>
  <text x="500" y="376" text-anchor="middle" fill="#3A4DA8" font-size="12.5">成对检验两组函数是否相等 → 异质性处理效应</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 B｜binsreg 实证工作流：选箱 → 估计绘图 → 检验 → 组间比较（命令示例依据总结文档整理，示意）</div>
</div>

**基础估计绘图**：`binsreg y x w` 自动用 DPI 选箱并输出分箱散点图；可加 `line()`、`ci()`、`cb()` 添加拟合线、置信区间、置信带；`polyreg(P)` 叠加全局多项式；`vce(cluster id)` 聚类标准误；`by(t)` 分组绘图。

**分位数回归**：`binsqreg` 实现条件分位数 binscatter，可 bootstrap 标准误。

**二元 Logit**：`binslogit` 处理 0–1 被解释变量。

**检验示例**：

- `binstest y x w, testmodelpoly(1)` → 检验函数是否线性；
- `deriv(1) testshaper(0)` → 检验函数单调不减；
- `binspwc y x w, by(t)` → 两组函数是否存在差异。

**binsregselect**：输出 ROT、DPI 两套最优分箱；`savegrid()` 输出网格文件，用于外部自定义模型的设定检验；`randcut()` 用随机子样本加速分箱选择。

## 06 局限与未来方向

- **现有局限**：主要面向横截面数据；仅支持单变量 x 作为分箱变量；
- **未来拓展**：面板数据、时间序列 binscatter；多维分箱与热力图——把统计推断框架拓展到多维可视化场景。

## 07 安装

```stata
net install st0765
net sj 25-1
net get st0765
```

项目主页：https://nppackages.github.io/binsreg/

## 08 核心启示（给实证研究者）

1. 传统 `binscatter` / `binscatter2` 的协变量**残差化方法统计不可靠**，建议优先使用 `binsreg`；
2. **只看图不做检验存在误导风险**，应配合 `binstest` 做正式形状与参数设定检验；**检验线性优先检验一阶导数而非水平值**；
3. IMSE 最优分箱兼顾偏差与方差；人为固定分箱数（如 20）是牺牲统计最优换取可读性，需要权衡；
4. **一致置信带与点式置信区间含义完全不同**：做全函数推断必须用置信带；
5. 协变量条件下，图像高度与检验结果依赖评估点 `at()`，解读结果必须关注该设定。

## 09 速查

### 命令速查表

| 命令 | 典型调用 | 说明 |
| --- | --- | --- |
| `binsreg` | `binsreg y x w, line(1 1) ci cb vce(cluster id) by(t)` | 最小二乘分箱：绘图 / 点估计 / CI / CB / 分组 |
| `binslogit` | `binslogit y x w` | Logit 分箱（二元 y） |
| `binsprobit` | `binsprobit y x w` | Probit 分箱（二元 y） |
| `binsqreg` | `binsqreg y x w, q(0.5)` | 分位数分箱（可 bootstrap 标准误） |
| `binstest` | `binstest y x w, testmodelpoly(1)` | 线性 / 多项式设定检验 |
| `binstest` | `binstest y x w, deriv(1) testshaper(0)` | 形状检验（如单调不减） |
| `binspwc` | `binspwc y x w, by(t)` | 组间成对比较（异质性处理效应） |
| `binsregselect` | `binsregselect y x w` | 输出 Ĵ_ROT / Ĵ_DPI 最优分箱 |

### 常用选项

| 选项 | 作用 |
| --- | --- |
| `at(mean / median / 0 / 自定义)` | 协变量评估点（改变图的位置与区间） |
| `nbins(J)` | 手动固定箱数（模式 2） |
| `dots(p s)` / `line(p s)` / `ci(p s)` / `cb(p s)` | 散点 / 拟合线 / 点区间 / 一致置信带（可分别设 p、s 实现 RBC） |
| `polyreg(P)` | 叠加全局多项式拟合 |
| `vce(cluster ...)` | 聚类稳健方差 |
| `by(t)` | 分组估计与绘图 |
| `masspoints(off)` / `dfcheck()` | 质量点 / 自由度校验开关 |
| `usegtools(on)` / `reghdfe absorb()` | 大数据加速 / 高维固定效应 |
| `nsims()` / `simsgrid()` | 模拟次数 / 网格点（初步探索可调低） |
| `savedata()` / `savegrid()` | 导出绘图数据 / 网格文件 |
| `randcut()` | 随机子样本加速选箱 |

### 关键术语

| 术语 | 含义 |
| --- | --- |
| v | 求导阶数：v=0 水平函数，v=1 一阶导数 |
| ρ(·) / η(·) | 损失函数 / 逆连接函数（M-估计框架） |
| p / s | 箱内多项式阶数 / 跨箱连续可导阶数（s=0 分段常数，s=1 连续，s=2 一阶导连续） |
| J_IMSE、Ĵ_ROT、Ĵ_DPI | IMSE 最优分箱；经验法则 / 直接插件两种选择器 |
| CI / CB | 点式置信区间 / 一致置信带 |
| RBC（稳健偏差修正） | 用 p+q 阶多项式构造区间/带，消除一阶近似偏差 |
| 残差化 | 旧 binscatter/binscatter2 的错误协变量调整方式（一般不一致） |
| 质量点 / 自由度校验 | 有效样本量 = x 唯一值数 N（或聚类数 G），不足时降级为分段常数 |
| 有效样本量 N / G | x 唯一取值数目 / 聚类个数（非原始观测 n） |

---

*本笔记依据用户提供的《Binscatter Regressions》中文总结文档（5 页）整理；源文档注明"部分内容可能由 AI 生成"，命令与选项细节请以官方帮助文件（`help binsreg` 等）为准。两幅示意图为依据文档内容自绘的示意，非论文原图。项目主页：https://nppackages.github.io/binsreg/*。
