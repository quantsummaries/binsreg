# On Binscatter（论分箱散点图）· 学习笔记

> 把分箱散点图从"画图工具"变成有理论保障、可做推断的正式计量方法。

| 项目 | 内容 |
| --- | --- |
| 作者 | Matias D. Cattaneo（普林斯顿）· Richard K. Crump（纽约联储）· Max H. Farrell（UC Santa Barbara）· Yingjie Feng（清华经管） |
| 期刊 | American Economic Review 2024, 114(5): 1488–1514 |
| DOI | 10.1257/aer.20221576 |
| JEL | C13, C14, C18, C51, O31, R32 |
| 软件 | binsreg（Stata / R / Python）— https://nppackages.github.io/binsreg/ |

## 一句话读懂

binscatter（分箱散点图）本质是一个**非参数估计量**。论文给出三个正式工具：数据驱动的 IMSE 最优分箱数、稳健偏差校正（RBC）下的统一置信带、部分线性模型下的正确协变量调整；同时指出流行的"先残差化再分箱"做法在非参数视角下**不一致**，会扭曲形状与支撑集。用新方法重做两篇已发表研究（AGNS 2022；Moretti 2021），结论发生实质变化。

## 论文方法体系总览

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 340" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <defs>
    <marker id="mdA" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#8FA0F5"/>
    </marker>
  </defs>
  <!-- 阶段1 -->
  <rect x="24" y="70" width="200" height="190" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="24" y="70" width="200" height="8" rx="4" fill="#8FA0F5"/>
  <text x="124" y="104" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">起点 · 散点图失效</text>
  <circle cx="48" cy="132" r="3.5" fill="#8FA0F5"/><text x="60" y="137" fill="#1C2733" font-size="13">大数据点云密不可读</text>
  <circle cx="48" cy="166" r="3.5" fill="#8FA0F5"/><text x="60" y="171" fill="#1C2733" font-size="13">隐私限制不能画原始点</text>
  <circle cx="48" cy="200" r="3.5" fill="#8FA0F5"/><text x="60" y="205" fill="#1C2733" font-size="13">需"控制协变量"地看关系</text>
  <text x="124" y="240" text-anchor="middle" fill="#6B7A8A" font-size="12">→ binscatter 成为流行替代</text>
  <line x1="228" y1="165" x2="252" y2="165" stroke="#8FA0F5" stroke-width="2" marker-end="url(#mdA)"/>
  <!-- 阶段2 -->
  <rect x="258" y="70" width="220" height="190" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="258" y="70" width="220" height="8" rx="4" fill="#8FA0F5"/>
  <text x="368" y="104" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">binscatter 三要素</text>
  <text x="278" y="132" fill="#1C2733" font-size="13">① 经验分位数分箱，共 J 个</text>
  <text x="278" y="158" fill="#1C2733" font-size="13">② 箱内均值＝局部常数拟合</text>
  <text x="278" y="184" fill="#1C2733" font-size="13">③ 协变量怎么处理？</text>
  <text x="278" y="206" fill="#3A4DA8" font-size="12.5" font-weight="700">（全文关键，见第 03 节）</text>
  <text x="368" y="238" text-anchor="middle" fill="#6B7A8A" font-size="12">此前无人正式研究其统计性质</text>
  <line x1="482" y1="165" x2="506" y2="165" stroke="#8FA0F5" stroke-width="2" marker-end="url(#mdA)"/>
  <!-- 阶段3 -->
  <rect x="512" y="34" width="240" height="262" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="512" y="34" width="240" height="8" rx="4" fill="#8FA0F5"/>
  <text x="632" y="66" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">论文三大工具（贡献）</text>
  <rect x="528" y="82" width="208" height="56" rx="8" fill="#DCE7FF"/>
  <text x="632" y="104" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">① 形式化：非参数估计量</text>
  <text x="632" y="124" text-anchor="middle" fill="#3A4DA8" font-size="12">部分线性协变量调整 → 目标 Υ₀</text>
  <rect x="528" y="150" width="208" height="56" rx="8" fill="#DCE7FF"/>
  <text x="632" y="172" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">② IMSE 最优分箱 Ĵ</text>
  <text x="632" y="192" text-anchor="middle" fill="#3A4DA8" font-size="12">偏差–方差权衡的数据驱动解</text>
  <rect x="528" y="218" width="208" height="56" rx="8" fill="#DCE7FF"/>
  <text x="632" y="240" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">③ RBC 统一置信带</text>
  <text x="632" y="260" text-anchor="middle" fill="#3A4DA8" font-size="12">随机分箱下的均匀推断 + 检验</text>
  <line x1="756" y1="165" x2="780" y2="165" stroke="#8FA0F5" stroke-width="2" marker-end="url(#mdA)"/>
  <!-- 阶段4 -->
  <rect x="786" y="70" width="190" height="190" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="786" y="70" width="190" height="8" rx="4" fill="#8FA0F5"/>
  <text x="881" y="104" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">产出与应用</text>
  <text x="806" y="134" fill="#1C2733" font-size="13">可视化 + 正式假设检验</text>
  <text x="806" y="162" fill="#1C2733" font-size="13">binsreg 软件</text>
  <text x="806" y="182" fill="#6B7A8A" font-size="12">（Stata / R / Python）</text>
  <text x="806" y="212" fill="#1C2733" font-size="13">重做两篇已发表研究：</text>
  <text x="806" y="234" fill="#1C2733" font-size="13">AGNS 2022 · Moretti 2021</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 A｜论文方法体系总览：从散点图失效出发，把 binscatter 形式化为非参数估计量，并补齐分箱数选择、协变量调整、不确定性量化三块拼图（依据论文内容自绘，示意）</div>
</div>

## 01 背景：散点图为什么被 binscatter 取代

经典散点图是数据分析的基础工具：画出全部 n 个观测点 (xᵢ, yᵢ)，可以直观看到联合分布、条件均值的函数形式、异常值与聚集现象。但它有三条硬伤：

- **大样本不可读**：数据规模变大后点云集成团，形状、异常值全部丢失；
- **隐私约束**：越来越不允许直接展示原始数据点；
- **无法控制协变量**：散点图只能画 x–y 二元关系，无法在"控制其他变量"的意义下可视化，而这正是社会科学的标准诉求。

binscatter 的构造直觉很简单：把 x 的支撑切成 J 个箱（经济学惯例用经验分位数，每箱约 n/J 个观测），每箱画一个点 (x̄ⱼ, ȳⱼ)，即该箱内 y 的样本均值。结果是一张更干净、更"可读"的图。

> **注意**：binscatter 显示的**不是数据**，而是条件均值函数 E[y|x] 的估计；"连点成线"只是视觉引导，真实估计量是**分段常数**（论文 Figure 1 的 Panel D）。它还掩盖了条件分布的其他特征（方差、分位数等）。

问题在于：这种方法在实证微观经济学中已非常流行（Starr &amp; Goldfarb 2020 综述），但此前**没有人正式研究过它的统计性质**——加回归线、加控制变量的"标准做法"是否成立，完全没人检验。

## 02 形式化：binscatter 是一个非参数估计量

论文把 binscatter 拆成三个构成要素，逐个正式化：

- **分箱**：用 x 的边际经验分位数构造断点（允许断点随机、由数据决定），每箱观测数 ≈ n/J；
- **箱内估计**：箱内取平均，即"局部常数"拟合（零阶分段多项式）；
- **协变量处理**：见下一节——这是全文最重要的警示。

无协变量时的规范 binscatter 可写成一次普通最小二乘回归：

> **(1)** `v̂(x) = b̂(x)′ ξ̂`，`ξ̂ = argmin_{ξ∈ℝ^J} Σ_i [y_i − b̂(x_i)′ ξ]²`
>
> `b̂(x)` 是 J 维"箱指示变量"基向量（Haar 基 / 0 阶样条），`v̂(x)` 是分段常数拟合；从计量角度看，它和"画点"完全等价。

### 两个不同的参数解释（关键）

- **固定 J（参数化视角）**：binscatter 估计 `ξ₀(j) = E[y_i | x_i ∈ B_j]`，即"落入第 j 箱的条件均值"。选 J = 10 就是比较 x 的十分位组的平均结果。论文全部结果对 ξ₀ 同样成立。
- **J 随 n 发散（非参数视角）**：binscatter 估计 `v₀(x) = E[y_i | x_i = x]`，J 成为平滑参数，面临偏差–方差权衡——这决定了后续"最优 J"的讨论。

论文的定位：固定 J 时，binscatter 是"粗化版条件均值"的估计；发散 J 时，它是恢复整个回归函数 v₀(x) 的非参数工具。想用图判断"函数是否线性、是否单调"，就必须采用发散 J 的视角。

## 03 协变量调整：全文最响亮的警示

当需要控制协变量 wᵢ（如固定效应、其他解释变量）时，实证文献的普遍做法是：先把 wᵢ 从 xᵢ 和 yᵢ 中"残差化"（受 Frisch–Waugh–Lovell 定理启发），再对残差画二元 binscatter。Stata 的 **binscatter / binscatter2** 包就是这样实现的。论文明确指出：**这个做法在非参数视角下一般不一致**。

在正则条件下，残差化 binscatter 一致估计的对象是：

> **(2)** `E[ y_i − L(y_i | w_i) ｜ x_i − L(x_i | w_i) ]`
>
> `L(a|w)` 是 a 对 w 的最佳线性近似（线性投影）。除非真实模型是线性的，否则这个概率极限难以解释，且形状、支撑都可能错误。

即便在部分线性设定 `E[y|x,w] = μ₀(x) + w′γ₀` 下，残差化 binscatter 也**不能一致估计** μ₀(x)、Υ₀(x) 或条件均值；只有 μ₀ 本身是线性时 (2) 才约化为 μ₀。更麻烦的是：μ₀ 非线性时图可能"碰巧"看起来线性，也可能不线性——用它来目视检验线性性会系统性误导。

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 500" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <defs>
    <marker id="mdB" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#8FA0F5"/>
    </marker>
  </defs>
  <!-- 决策节点 -->
  <rect x="300" y="16" width="400" height="58" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="40" text-anchor="middle" fill="#1C2733" font-size="15" font-weight="700">binscatter 遇到协变量 w，怎么"控制"？</text>
  <text x="500" y="60" text-anchor="middle" fill="#3A4DA8" font-size="12.5">两种做法，结论完全不同</text>
  <path d="M430,74 C360,92 250,96 250,112" fill="none" stroke="#8FA0F5" stroke-width="2" marker-end="url(#mdB)"/>
  <path d="M570,74 C640,92 750,96 750,112" fill="none" stroke="#8FA0F5" stroke-width="2" marker-end="url(#mdB)"/>
  <!-- 左支：错误做法 -->
  <rect x="40" y="112" width="420" height="62" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="250" y="137" text-anchor="middle" fill="#1C2733" font-size="15" font-weight="700">做法 A · 先残差化再分箱　✗ 不推荐</text>
  <text x="250" y="158" text-anchor="middle" fill="#3A4DA8" font-size="12.5">流行实现：Stata 的 binscatter / binscatter2 等</text>
  <rect x="40" y="186" width="420" height="92" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="250" y="210" text-anchor="middle" fill="#1C2733" font-size="13.5" font-weight="700">一致估计对象（概率极限）</text>
  <text x="250" y="234" text-anchor="middle" fill="#1C2733" font-size="13">E[ y − L(y|w) ｜ x − L(x|w) ]</text>
  <text x="250" y="260" text-anchor="middle" fill="#6B7A8A" font-size="12">L(·|w) 为线性投影 · 除非模型线性，否则含义不明</text>
  <rect x="40" y="290" width="420" height="78" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="250" y="314" text-anchor="middle" fill="#1C2733" font-size="13.5" font-weight="700">后果：形状可能错、支撑被压缩扭曲</text>
  <text x="250" y="336" text-anchor="middle" fill="#3A4DA8" font-size="12.5">点估计与"看形状判断线性"都不推荐</text>
  <text x="250" y="356" text-anchor="middle" fill="#3A4DA8" font-size="12.5">视觉线性 ≠ μ₀ 线性（既不充分也不必要）</text>
  <!-- 右支：正确做法 -->
  <rect x="540" y="112" width="420" height="62" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="750" y="137" text-anchor="middle" fill="#1C2733" font-size="15" font-weight="700">做法 B · 部分线性协变量调整　✓ 推荐</text>
  <text x="750" y="158" text-anchor="middle" fill="#3A4DA8" font-size="12.5">半参数模型 + 部分均值，可解释、可推断</text>
  <rect x="540" y="186" width="420" height="92" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="750" y="210" text-anchor="middle" fill="#1C2733" font-size="13.5" font-weight="700">设定与目标参数</text>
  <text x="750" y="234" text-anchor="middle" fill="#1C2733" font-size="13">E[y|x,w] = μ₀(x) + w′γ₀　→　Υ₀(x) = μ₀(x) + E[w]′γ₀</text>
  <text x="750" y="260" text-anchor="middle" fill="#6B7A8A" font-size="12">在 w 的均值处评估：只沿 x 方向看关系（部分均值）</text>
  <rect x="540" y="290" width="420" height="78" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="750" y="314" text-anchor="middle" fill="#1C2733" font-size="13.5" font-weight="700">估计：Υ̂(x) = μ̂(x) + w̄′γ̂（公式 6）</text>
  <text x="750" y="336" text-anchor="middle" fill="#3A4DA8" font-size="12.5">形状与支撑正确，置信带与假设检验有效</text>
  <text x="750" y="356" text-anchor="middle" fill="#3A4DA8" font-size="12.5">避免估计高维 E[w|x]（AGNS 中 w 有 113 维）</text>
  <!-- 底部实证对照 -->
  <rect x="40" y="404" width="920" height="72" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="250" y="430" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">AGNS 实证 · 做法 A</text>
  <text x="250" y="452" text-anchor="middle" fill="#6B7A8A" font-size="12">支撑被极端压缩、出现"假非线性"</text>
  <line x1="500" y1="416" x2="500" y2="464" stroke="#8FA0F5" stroke-width="1.5" stroke-dasharray="4,3"/>
  <text x="750" y="430" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">AGNS 实证 · 做法 B</text>
  <text x="750" y="452" text-anchor="middle" fill="#6B7A8A" font-size="12">形状清晰，反而更支持原论文的线性回归</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 B｜协变量调整的两种做法：残差化（错）vs 部分线性（对）（依据论文第 I 节与 Figure 2 内容自绘，示意）</div>
</div>

> **避坑**：看到论文里"控制了一堆固定效应之后画的分箱图"（binscatter、binscatter2 输出），先问一句：它是先残差化再分箱的吗？如果是，图的形状与支撑都不可信，只能用部分线性方式重画。

## 04 分箱数 J 怎么选：IMSE 最优

实证中 J 常被拍脑袋定：Stata 默认 J = 20，AGNS 用了 J = 100。但固定 J 只能得到"粗化参数"（ξ₀ 或 Ξ₀），回答不了"μ₀ 是否线性"这类问题——那必须恢复整个函数，即让 J 随样本量发散。此时 J 就是一个平滑参数：

- **J 太小 → 偏差主导**：箱太宽，箱内常数拟合残留大量设定误差（过平滑）；
- **J 太大 → 方差主导**：每箱只有约 n/J 个观测，估计锯齿状（欠平滑）。

论文给出基于积分均方误差（IMSE）的最优分箱数：

> **(7)** `J_IMSE = ⌊ (2Bₙ / Vₙ)^(1/3) · n^(1/3) ⌋`
>
> `Bₙ`：渐近（平方）偏差；`Vₙ`：渐近方差（计入异方差与聚类，并依赖协变量 w）。AGNS 数据算得 `Ĵ_IMSE = 11`。

- 数据越嘈杂（Vₙ 大）→ J_IMSE 越小，每箱样本更多；
- μ₀ 越不平滑（Bₙ 大）→ 需要更多箱去偏差。

论文 Figure 4 用同一数据演示：J = 5 明显过平滑（残留偏差），J = 50 明显欠平滑（过锯齿），J = 11（IMSE 最优）居中、形状清晰。即使你坚持用固定 J，**J_IMSE 也是判断自己选择偏大偏小的基准**——若 J 远大于 J_IMSE，这张图大概率方差主导。

> **用法**：主分析用数据驱动的 Ĵ_IMSE 分箱；对分箱数做稳健性（如 J ± 若干）并报告；固定 J 场景（如"十分位组对比"）则把目标参数明确为粗化版 ξ₀ / Ξ₀。

## 05 不确定性量化：统一置信带

置信区间覆盖一个参数，**置信带**则覆盖整条函数 Υ₀(x)、对 x ∈ 𝒳 一致成立（公式 9 的意义）。有了它，就能目视检验"线性函数能否放进带内""带内能否放进水平线或单调函数"——这正是 binscatter 最常见的用途（回归前先看是否线性）。

> **(8)** `Î_RBC(x) = [ Υ̂_BC(x) ± ε_RBC · √( Ω̂_RBC(x) / n ) ]`
>
> `Υ̂_BC`：去偏（稳健偏差校正）后的估计量；`ε_RBC`：由高斯逼近得到的均匀临界值；`Ω̂_RBC`：去偏引入的额外方差也被计入。

> **(9)** `Pr( Υ₀(x) ∈ Î_RBC(x)，∀ x ∈ 𝒳 ) → 1 − α`
>
> 置信带以预设概率 1−α 同时覆盖整条函数——这是"统一推断"，不是逐点区间。

为什么不能简单"点估计 ± 1.96·标准误"：目标 Υ₀(x) 是非参数函数，分箱近似引入了**设定误差（偏差）**，点带无法覆盖它。解决路径是**稳健偏差校正（RBC）**：用 IMSE 最优的 J 做点估计，再用带连续性约束的箱内线性拟合去掉一阶偏差，并把去偏带来的额外不确定性计入方差。RBC 源自 Calonico–Cattaneo–Farrell 的非参数推断系列方法，这里推广到随机分箱的 binscatter 场景。

- **AGNS 实证（Figure 3）**：置信带内能画出线性函数 → 数据与"线性关系"相容，支持原论文的线性回归；同时拒绝水平线（关系存在）、拒绝单调递减（方向为正）。
- **固定 J 与发散 J 的差异（Figure 5）**：固定 J = 5 时，五个十分位组的条件均值置信区间无法拒绝"全部相等"；而发散 J 的置信带拒绝水平线——两种推断回答的是不同参数（Ξ₀ vs Υ₀），不能混用。
- 论文还提供基于规范 binscatter 的正式假设检验（p 值版），实现于配套软件。

> **要点**：置信带是论文的**主要技术贡献**：为"经验分位数随机分箱 + 部分线性协变量调整"下的 t 统计量过程建立均匀分布逼近。图 3 中带内画红线（线性函数）即可"证明"线性性与数据相容——这是肉眼判断无法替代的正式证据。

## 06 理论贡献：随机分箱下的均匀推断

以往的分箱/划分回归理论有两个缺口，导致 binscatter 无法被直接覆盖：① 文献只允许**已知断点**，而 binscatter 用经验分位数断点，基函数本身是随机的；② 即使有结果，条件也过强，把"箱内纯平均"（局部常数，p = 0）排除在外。论文在在线附录中建立了一套针对随机分箱的新理论，要点如下：

> **(10)** `μ̂^(v)(x) = b̂_p,s^(v)(x)′ β̂`，`b̂_p,s(x) = T̂_s [ b̂(x) ⊗ (1, x, …, x^p) ]`
>
> 箱内 p 阶多项式 + 线性约束 `T̂_s` 保证估计 (s−1) 阶导数连续；p = 0 时回到规范 binscatter；RBC 实现取 (p, s, v) = (1, 1, 0)。

> **Theorem 1（IMSE 展开）**：`IMSE[ Υ̂^(v) ] = (J^(1+2v)/n)·Vₙ(p,s,v) + J^(−2(p+1−v))·Bₙ(p,s,v) + o`
>
> 密度加权 IMSE 展开；条件 `J·logJ/n → 0` 且 `n·J^(−4p−5) → 0`。据此得到通用最优 `J_IMSE(p,s,v)`。

> **Theorem 2（可行强逼近）**：`Pr( sup_{x∈𝒳} |T_p(x) − Z_p(x)| > ξ·aₙ⁻¹ ) → 0`，且可行版 `Ẑ_p(x)` 可由与数据独立的高斯随机向量模拟
>
> t 统计量过程 {T_p(x)} 的（条件）高斯强逼近——用蒙特卡洛高斯抽取即可得到置信带临界值。速率条件只需 `J²/n → 0`（子指数矩下 `J/n → 0`，含 log 因子）。

| 速率条件对比 | 要求（含 log 因子） | 覆盖范围 |
| --- | --- | --- |
| 本文 Theorem 2 | **J²/n → 0**（子指数矩下 J/n → 0） | 整条 t 统计量过程，含 p = 0 分段常数 |
| Belloni et al. (2015) | J³/n → 0 | 一般序列估计量（强逼近） |
| Chernozhukov–Chetverikov–Kato (2014) | J/n^(1−2/ν) → 0 | 仅 sup 泛函 |

为什么"弱速率"重要：只有条件足够弱，才能容纳 p = 0 的规范 binscatter（箱内纯平均）——这是应用里最常见的形态，也是以往均匀推断结果做不到的。

## 07 两个实证应用：重做已发表研究

论文用两篇已发表研究做全流程演示，并展示"用错 binscatter"与"用对 binscatter"的实质差异：

| 维度 | AGNS：税收与创新（Akcigit et al. 2022） | Moretti：高科技集群与顶尖发明者生产率（Moretti 2021） |
| --- | --- | --- |
| 数据 | 美国 20 世纪各州-年，约 3,000 个观测 | 发明者-年，接近 100 万个观测 |
| 核心变量 | log 专利数 vs log(第 90 百分位边际净税率) | log 专利数 vs log 高科技集群规模 |
| 协变量 | 4 个连续变量 + 49 州固定效应 + 60 年固定效应（w 共 113 维） | 年、研究领域、城市效应；主设定含 11 类固定效应 |
| 原始散点图 | 约 3,000 个点已难辨形状 | 近百万点，密集成团、无信息 |
| 错误做法 | 残差化 binscatter：支撑被极端压缩，出现"假非线性" | 残差化 + 箱数过多：锯齿状、被误读为线性 |
| 修正之后 | Ĵ_IMSE = 11：形状清晰、与线性相容 | Ĵ_IMSE = 18：小集群平坦、大集群陡升 |
| 推断结论 | 置信带支持线性；拒绝水平线、拒绝单调递减 | 置信带**拒绝线性**、**不拒绝凸性** → 非线性关系 |
| 对原论文的影响 | 原线性回归结论得到加强 | 给弹性 0.0676 的结论补上"非线性 + 门槛"的关键限定 |

> **政策含义**：Moretti 重做的结论不是修饰性的：如果关系是"小集群段平坦、大集群段陡升"的凸函数，那么只有小规模集群的地方，可能需要提供**非常慷慨的激励**、把集群规模推到足够大，才能产生原论文所描绘的聚集效应。

## 08 软件与延伸

- **binsreg 软件包**：Stata / R / Python 三平台实现全部结果（Cattaneo et al. 2023a；官网 https://nppackages.github.io/binsreg/）。论文中图 3–6 均由该包输出。
- **正式假设检验**：规范 binscatter 的 p 值版检验（线性、单调、凸性等）见姊妹篇 Cattaneo et al. (2023a) 与 (2023b)，可补充置信带的目视结论。
- **非线性推广**：条件分位数回归、Logistic 等非线性模型、与离散变量的一阶交互——见 Cattaneo et al. (2023b) "Nonlinear Binscatter Methods"。
- **多维 x**：全部结果可推广到 dim(x) > 1，重要应用是热图（heat map）。
- **一般分箱方案**：理论覆盖等距分箱、经济含义分箱（如收入区间）等，不限于分位数。

## 09 专题 · 控制变量 w 如何体现在最终结果（尤其散点图）

承接第 03 节：正确的协变量调整是部分线性模型 `E[y|x,w] = μ₀(x) + w′γ₀`。那"控制 w"到底怎么落到最终结果、尤其 binsreg 画出的散点图上？

**一句话**：w 的体现 = 两个动作——① 估计时，样条 β̂ 与全局 γ̂ 放在同一个目标函数里联合估计（公式 3），让 μ̂(x) 成为"扣除 w 之后"的 x 净关系；② 画图时，把曲线放在评估点 w̄ 上展示：`Υ̂(x) = μ̂(x) + w̄′γ̂`（公式 6，binsreg 默认 w̄ = 样本均值，即 `at(mean)`）。

**三个层面**：

1. **估计层**：γ̂ 是一个**全局系数向量、跨 x 不变**；`(β̂, γ̂)` 联合一步估计，不是旧工具"先残差化再分箱"（那个一般不一致，见 03 节）。μ̂(x) 因此是"把 w 的线性影响拿掉之后"的 x–y 关系。
2. **图形层（散点图）**：
   - **点（dots）**：y 在每个箱内的均值，**原始数据、未被 w 调整**；
   - **线（line）**：`Υ̂(x) = μ̂(x) + w̄′γ̂`——**形状来自 μ̂（x 净效应，不随评估点变）**，**垂直位置来自 w̄′γ̂（随 at() 移动）**；
   - 点与线之间的**垂直缝隙 ≈ 控制变量效应**；
   - **换 at()**：线性链接下曲线整体平移、形状不变；Logit 等非线性链接下曲率也变；
   - **ci / cb**：宽度与位置同样依赖评估点 w̄。
3. **推断层**：函数水平（v=0）的估计与检验依赖 w̄；而 μ 的一阶导数 μ̂′ 在可加线性 index 模型里**与 w̄ 无关**——所以"这关系是否线性"应优先检验 v=1 导数，避免评估点选择的误导。

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 460" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <line x1="90" y1="60" x2="90" y2="330" stroke="#C9D4E8" stroke-width="1.5"/>
  <line x1="90" y1="330" x2="900" y2="330" stroke="#C9D4E8" stroke-width="1.5"/>
  <text x="70" y="86" fill="#3A4DA8" font-size="14" font-weight="700">结果 y</text>
  <text x="470" y="356" fill="#3A4DA8" font-size="14" font-weight="700">收入 x</text>
  <circle cx="150" cy="200" r="6" fill="#6B7A8A"/>
  <circle cx="220" cy="170" r="6" fill="#6B7A8A"/>
  <circle cx="290" cy="152" r="6" fill="#6B7A8A"/>
  <circle cx="360" cy="165" r="6" fill="#6B7A8A"/>
  <circle cx="430" cy="185" r="6" fill="#6B7A8A"/>
  <circle cx="500" cy="215" r="6" fill="#6B7A8A"/>
  <circle cx="570" cy="242" r="6" fill="#6B7A8A"/>
  <circle cx="640" cy="252" r="6" fill="#6B7A8A"/>
  <circle cx="710" cy="238" r="6" fill="#6B7A8A"/>
  <circle cx="780" cy="222" r="6" fill="#6B7A8A"/>
  <circle cx="850" cy="208" r="6" fill="#6B7A8A"/>
  <path d="M150,245 C210,235 260,215 310,212 C380,215 440,245 500,272 C580,300 650,300 700,282 C760,262 810,250 850,244" fill="none" stroke="#8FA0F5" stroke-width="4"/>
  <path d="M150,215 C210,205 260,185 310,182 C380,185 440,215 500,242 C580,270 650,270 700,252 C760,232 810,220 850,214" fill="none" stroke="#B9C6F2" stroke-width="3" stroke-dasharray="10 7"/>
  <line x1="500" y1="220" x2="500" y2="266" stroke="#EA6668" stroke-width="2"/>
  <polygon points="500,220 495,230 505,230" fill="#EA6668"/>
  <polygon points="500,266 495,256 505,256" fill="#EA6668"/>
  <text x="512" y="246" fill="#EA6668" font-size="13" font-weight="700">线-点缝隙 ≈ 控制变量效应</text>
  <text x="632" y="316" fill="#8FA0F5" font-size="13" font-weight="700">Υ̂(x) = μ̂(x) + w̄′γ̂ 调整曲线</text>
  <text x="632" y="336" fill="#B9C6F2" font-size="12.5">换 at() → 整体平移（线性链接）</text>
  <text x="632" y="200" fill="#6B7A8A" font-size="12.5">y 的分箱均值（原始，未调整）</text>
  <rect x="60" y="380" width="880" height="60" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="1.5"/>
  <text x="500" y="404" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">形状 ← μ̂(x)（x 净效应，不随评估点变）｜ 水平位置 ← w̄′γ̂（随 at() 移动）｜ 置信带宽度与位置 ← 也依赖 w̄</text>
  <text x="500" y="426" text-anchor="middle" fill="#3A4DA8" font-size="12.5">默认 w̄ = E[w]（部分均值，公式 6）；检验"是否线性"优先用一阶导数 v=1（水平 v=0 受评估点影响）</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 C｜binscatter 散点图解剖：原始分箱点 + 调整曲线 + 评估点平移（依据论文内容自绘，示意，非真实数据）</div>
</div>

**自查三题**：

1. binsreg 散点图里的点是"控制变量调整后的数据点"还是"原始分箱均值"？
2. `at(mean)` 换成 `at(median)`：曲线形状变吗？水平变吗？Logit 模型下呢？
3. 为什么检验"是不是线性"推荐用一阶导数 v=1 而非水平值？

答案：① 原始分箱均值，未调整；② 线性链接下形状不变、水平平移，Logit 下曲率也变，且置信带宽度与位置都变；③ 水平检验受评估点 w̄ 影响，而 μ̂′ 的形状与评估点无关，结论更稳健。

## 10 学习要点与速查

### 避坑清单（看完就记住这六条）

1. 看到"先残差化再分箱"的 binscatter，形状与支撑都不可信——用部分线性方式重画；
2. 分箱图上的点是条件均值估计，不是数据点；"连点"只是视觉引导；
3. 判断"是否线性/单调/凸"要用统一置信带，不要用肉眼；
4. 固定 J 回答的是粗化参数（如十分位均值），回答不了函数形状；恢复函数要让 J 随 n 发散；
5. 用 Ĵ_IMSE 分箱，或至少拿它当基准校准自己的固定 J；
6. 报告不确定性：置信带 + 聚类稳健方差（如按年、按城市×领域双向聚类）。

### 关键术语

| 符号 / 术语 | 含义 |
| --- | --- |
| `v₀(x) = E[y|x]` | 无协变量时的条件均值（规范 binscatter 的目标） |
| `ξ₀(j) = E[y | x ∈ B_j]` | 固定 J 视角的箱内条件均值（粗化参数） |
| `Υ₀(x) = μ₀(x) + E[w]′γ₀` | 部分均值：在 w 的均值处评估的部分线性关系（本文主推目标） |
| `Ξ₀` | 协变量调整下的固定 J 参数（Υ₀ 的固定 J 版本） |
| `J_IMSE` | IMSE 最优分箱数，偏差–方差权衡的显式解 |
| `RBC` | 稳健偏差校正（robust bias correction）：去偏并计入去偏方差 |
| `FWL` | Frisch–Waugh–Lovell 定理：线性回归中 "partialling out" 的依据（不适用于非参数 binscatter） |
| `b̂(x)` | J 维箱指示基向量（Haar 基 / 0 阶样条） |
| `E[w|x]` | 高维条件期望；部分均值策略刻意回避估计它（AGNS 中 w 达 113 维） |

### 公式速查

| 编号 | 公式 | 一句话 |
| --- | --- | --- |
| (1) | `v̂(x) = b̂(x)′ξ̂`（OLS 分段常数） | 规范 binscatter 的定义 |
| (2) | `E[y − L(y|w) ｜ x − L(x|w)]` | 残差化 binscatter 的目标对象（有问题） |
| (3) | `(β̂,γ̂) = argmin Σ[y − b̂(x)′β − w′γ]²` | 协变量调整 binscatter 的估计 |
| (4) | `E[y|x,w] = μ₀(x) + w′γ₀` | 半线性（部分线性）设定 |
| (5) | `Υ₀(x) = μ₀(x) + E[w]′γ₀` | 部分均值参数 |
| (6) | `Υ̂(x) = μ̂(x) + w̄′γ̂` | 部分均值估计量 |
| (7) | `J_IMSE = ⌊(2Bₙ/Vₙ)^(1/3)n^(1/3)⌋` | IMSE 最优分箱数 |
| (8) | `Î_RBC(x) = Υ̂_BC(x) ± ε_RBC·√(Ω̂_RBC(x)/n)` | RBC 统一置信带 |
| (9) | `Pr(Υ₀(x) ∈ Î_RBC(x), ∀x) → 1−α` | 置信带的均匀覆盖性质 |
| (10) | `b̂_p,s(x) = T̂_s[b̂(x) ⊗ (1,x,…,x^p)]` | 一般多项式 + 平滑约束估计量 |

---

*笔记依据论文全文（AER 2024, 114(5): 1488–1514, DOI: 10.1257/aer.20221576）整理；文中数字与结论均出自论文正文及图表注释。两幅示意图为依据论文内容自绘的示意，非论文原图。*
