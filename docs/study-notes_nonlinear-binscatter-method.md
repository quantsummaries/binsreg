# Nonlinear Binscatter Methods（非线性分箱散点图方法）· 学习笔记

> 把 binscatter 从"只画条件均值的最小二乘工具"推广到**非线性、可非光滑的 M-估计框架**：广义线性模型、稳健回归、分位数回归都能画、能推断、能检验。

| 项目 | 内容 |
| --- | --- |
| 作者 | Matias D. Cattaneo（普林斯顿）· Richard K. Crump（纽约联储）· Max H. Farrell（UC Santa Barbara）· Yingjie Feng（清华经管） |
| 状态 | 工作论文（2026 年 8 月 21 日稿） |
| 姊妹篇 | Cattaneo et al. (2024) "On Binscatter"（AER 114(5): 1488–1514）；Cattaneo et al. (2025) "Binscatter Regressions"（Stata Journal 25: 3–50） |
| 软件 | binsreg（Python / R / Stata）— https://github.com/nppackages/binsreg |
| 实证数据 | 美国 ACS 2013–2017 五年调查，邮编制表区层面（不含波多黎各），约 32,000 个观测 |

## 一句话读懂

已有的 binscatter 工具全部基于**最小二乘估计条件均值**，无法可视化离散度、对离散/分数结果（如未保险率）会误导、也缺少分位数回归版本。本文把 binscatter 放进一个一般的**半参数 M-估计框架**——损失函数 ρ 可以非线性、可以非光滑，链接函数 η 可以非线性（Logit/Probit），控制变量按部分线性进入指数 θ₀(x,w) = μ₀(x) + w′γ₀——并配套给出 IMSE 最优分箱数、RBC 统一置信带、函数形式/形状的正式假设检验、多样本（组间）比较四大工具。全部有理论（随机分箱下的 Bahadur 表示与强逼近）与软件（binsreg）支撑。

## 论文方法体系总览

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 360" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <defs>
    <marker id="nla" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#8FA0F5"/>
    </marker>
  </defs>
  <!-- 阶段1 -->
  <rect x="18" y="80" width="200" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="18" y="80" width="200" height="8" rx="4" fill="#8FA0F5"/>
  <text x="118" y="112" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">起点 · 仅 LS 分箱的局限</text>
  <circle cx="42" cy="138" r="3.5" fill="#8FA0F5"/><text x="54" y="143" fill="#1C2733" font-size="12.5">只能画条件均值</text>
  <circle cx="42" cy="166" r="3.5" fill="#8FA0F5"/><text x="54" y="171" fill="#1C2733" font-size="12.5">离散/分数结果会误导</text>
  <circle cx="42" cy="194" r="3.5" fill="#8FA0F5"/><text x="54" y="199" fill="#1C2733" font-size="12.5">无法可视化离散度</text>
  <circle cx="42" cy="222" r="3.5" fill="#8FA0F5"/><text x="54" y="227" fill="#1C2733" font-size="12.5">没有分位数回归版本</text>
  <text x="118" y="258" text-anchor="middle" fill="#6B7A8A" font-size="12">→ 本文要推广的缺口</text>
  <line x1="222" y1="180" x2="246" y2="180" stroke="#8FA0F5" stroke-width="2" marker-end="url(#nla)"/>
  <!-- 阶段2 -->
  <rect x="252" y="80" width="230" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="252" y="80" width="230" height="8" rx="4" fill="#8FA0F5"/>
  <text x="367" y="112" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">框架 · 半参数 M-估计</text>
  <rect x="266" y="126" width="202" height="64" rx="8" fill="#DCE7FF"/>
  <text x="367" y="148" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">min E[ρ(y; η(μ(x)+w′γ))]</text>
  <text x="367" y="168" text-anchor="middle" fill="#3A4DA8" font-size="12">损失 ρ：可非线性、可非光滑</text>
  <text x="367" y="184" text-anchor="middle" fill="#3A4DA8" font-size="12">链接 η：可非线性（Logit 等）</text>
  <text x="367" y="216" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">三个目标对象</text>
  <text x="367" y="238" text-anchor="middle" fill="#1C2733" font-size="12">ϑ₀ = η(θ₀) 水平 / 剂量反应</text>
  <text x="367" y="256" text-anchor="middle" fill="#1C2733" font-size="12">μ₀⁽ᵛ⁾ 分量与导数</text>
  <text x="367" y="274" text-anchor="middle" fill="#1C2733" font-size="12">ζ₀ = η⁽¹⁾·μ₀⁽¹⁾ 边际效应</text>
  <line x1="486" y1="180" x2="510" y2="180" stroke="#8FA0F5" stroke-width="2" marker-end="url(#nla)"/>
  <!-- 阶段3 -->
  <rect x="516" y="40" width="240" height="280" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="516" y="40" width="240" height="8" rx="4" fill="#8FA0F5"/>
  <text x="636" y="72" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">四大方法论工具</text>
  <rect x="532" y="86" width="208" height="48" rx="8" fill="#DCE7FF"/>
  <text x="636" y="106" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">① IMSE 最优分箱 Ĵ</text>
  <text x="636" y="124" text-anchor="middle" fill="#3A4DA8" font-size="12">(2p+3) 次根下的偏差-方差解</text>
  <rect x="532" y="142" width="208" height="48" rx="8" fill="#DCE7FF"/>
  <text x="636" y="162" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">② RBC 统一置信带</text>
  <text x="636" y="180" text-anchor="middle" fill="#3A4DA8" font-size="12">p+1 阶去偏 + 高斯逼近临界值</text>
  <rect x="532" y="198" width="208" height="48" rx="8" fill="#DCE7FF"/>
  <text x="636" y="218" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">③ 函数形式/形状检验</text>
  <text x="636" y="236" text-anchor="middle" fill="#3A4DA8" font-size="12">线性、多项式、单调性等</text>
  <rect x="532" y="254" width="208" height="48" rx="8" fill="#DCE7FF"/>
  <text x="636" y="274" text-anchor="middle" fill="#1C2733" font-size="12.5" font-weight="700">④ 多样本（组间）比较</text>
  <text x="636" y="292" text-anchor="middle" fill="#3A4DA8" font-size="12">CATE 与处理效应异质性</text>
  <line x1="760" y1="180" x2="784" y2="180" stroke="#8FA0F5" stroke-width="2" marker-end="url(#nla)"/>
  <!-- 阶段4 -->
  <rect x="790" y="80" width="192" height="200" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <rect x="790" y="80" width="192" height="8" rx="4" fill="#8FA0F5"/>
  <text x="886" y="112" text-anchor="middle" fill="#3A4DA8" font-size="15" font-weight="700">应用与软件</text>
  <text x="810" y="140" fill="#1C2733" font-size="12.5">ACS：收入 vs 未保险率</text>
  <text x="810" y="160" fill="#1C2733" font-size="12.5">（分数结果 → Logit QMLE）</text>
  <text x="810" y="188" fill="#1C2733" font-size="12.5">binsreg 软件</text>
  <text x="810" y="208" fill="#6B7A8A" font-size="12">（Stata / R / Python）</text>
  <text x="810" y="236" fill="#1C2733" font-size="12.5">理论支撑：Bahadur 表示</text>
  <text x="810" y="256" fill="#1C2733" font-size="12.5">+ 条件高斯强逼近</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 A｜非线性 binscatter 方法体系总览：从 LS 分箱的缺口出发，在 M-估计框架下给出三大目标对象与四大方法论工具（依据论文内容自绘，示意）</div>
</div>

## 01 动机：为什么"只画均值"不够

经典散点图在大数据、隐私、协变量控制、离散结果四个场景下失效；binscatter 以"分箱 + 箱内汇总"的形式继承了它的优点，但**现有工具只能做最小二乘半线性条件均值**（Cattaneo et al. 2024 已给出其统计性质、修正了协变量调整错误、提供了置信带与最优调参）。这个限制带来三个具体问题：

- **离散度不可见**：数据的 spread、波动、异常值都无法可视化；
- **离散/受限结果会误导**：如分数结果（0–1 之间的比例），最自然的模型是 Logistic 拟极大似然（QMLE），线性分箱既可能越界也扭曲形状；
- **分位数回归没有 binscatter 工具**：无法评估条件分布的分位数、稳健中心趋势、离群值。

本文的解法：构造一个一般性框架，让 **损失函数 ρ 非线性/非光滑**、**链接函数 η 非线性** 都行——广义线性模型（Logit/Probit）、稳健半参数回归（Huber/中位数）、分位数回归全部纳入。

## 02 模型设定：半参数 M-估计

回归函数不一定是条件均值，只要依赖于**部分线性标量指数**：

> **(2.1)** `(μ₀(·;γ₀), γ₀) = argmin E[ ρ(yᵢ; η(μ(xᵢ) + wᵢ′γ)) ]`，指数 `θ₀(xᵢ,wᵢ) = μ₀(xᵢ) + wᵢ′γ₀`

- `μ₀(x)`：对 x 非参数；`wᵢ′γ₀`：控制变量线性进入（d 可随 n 发散）；
- `ρ` 损失函数、`η` 逆链接函数；要求模型设定正确、解唯一。

### 三个领头例子（同一个框架）

| 例子 | 损失 ρ | 链接 η | 解决的问题 |
| --- | --- | --- | --- |
| 例1 最小二乘 | `(y − η)²` | `η(θ) = θ` | 部分线性条件均值（回归到旧工具） |
| 例2 Logistic | `−y log η − (1−y) log(1−η)` | `η(θ) = (1 + e⁻ᶿ)⁻¹` | 二元 / 分数结果（如未保险率） |
| 例3 分位数 | `[τ − 𝟙{y<η}](y − η)`，τ∈(0,1) | `η(θ) = θ` | 条件分位数、稳健回归（可非光滑） |

### 三个目标对象

> **(i)** 水平：`ϑ₀(x,w) = η(μ₀(x) + w′γ₀) = η(θ₀(x,w))`
>
> **(ii)** 非参数分量或其导数：`μ₀⁽ᵛ⁾(x)`，v ≥ 0
>
> **(iii)** 边际（偏）效应：`ζ₀(x,w) = η⁽¹⁾(θ₀(x,w)) · μ₀⁽¹⁾(x)`

- `w` 是用户选择的控制变量评估点（均值/中位数/分位数/基准类别等），必须可一致估计；**评估点会影响图的形状、位置与置信带**（μ₀ 的定义依赖 w 的编码方式）。
- 水平 ϑ₀：连续处理时给出**剂量反应函数**；离散处理时做**处理效应异质性**分析；分位数情形下可评估条件分布离散度（如四分位距）。
- 边际效应：非线性模型的标准对象；本文估计的是"**均值处的边际效应**"，不是"平均边际效应"——两者在线性模型中无区别，在非线性模型中实现与解释都不同。

## 03 估计：分箱、基函数、协变量

**分箱**：x 的支撑切成 J 个互斥箱（J 是主调参）；断点可数据依赖（但不得用 y）：等距断点、经验分位数（`τ̂ⱼ = F̂⁻¹(j/J)`，最常用）、自适应回归树等；要求"准均匀"（箱长之比有界）。本文理论允许**随机分箱**，实证用分位数间距。

**箱内拟合**（局部常数起步）：最小二乘 → 箱内均值（Tukey 1961 的 regressogram 原样）；二元/分数 → 箱内对常数跑 Logistic；分位数 → 箱内经验分位数。

> **(2.2)** `μ̂(x) = b̂₀(x)′β̂`，`β̂ = argmin Σ ρ(yᵢ; η(b̂₀(xᵢ)′β))`，`b̂₀(x) = [𝟙_{B̂₁}(x), …, 𝟙_{B̂_J}(x)]′`

**一般基函数**：箱内 p 阶多项式 + 断点处 (s−1) 阶导数连续约束

> `b̂_{p,s}(x) = T̂_s [ b̂₀(x) ⊗ (1, x, …, xᵖ) ]`，p ≥ s ≥ 0，v ≤ p

- s = 0：不连续的分段多项式（各箱独立拟合）；s = p = 0：标准 binscatter（分段常数）；s = p = 3：三次 B 样条。论文正文取 s = p。
- 多项式阶数 p 的作用：降偏差、估计导数（形状检验与设定检验的关键）。

**协变量进入**（公式 2.3）：把 w 直接放进损失，与 b̂ 基一起整体估计：

> **(2.3)** `(β̂, γ̂) = argmin Σ ρ(yᵢ; η(b̂_p(xᵢ)′β + wᵢ′γ))`

> **注意**：有控制变量时**不能逐箱独立估计**——γ₀ 是全局参数，必须整体求解。

三个估计量（插件法，θ̂_p(x,ŵ) = μ̂_p(x) + ŵ′γ̂）：

> **(2.4)** `ϑ̂_p(x,ŵ) = η(θ̂_p(x,ŵ))`　**(2.5)** `μ̂_p⁽ᵛ⁾(x) = b̂_p⁽ᵛ⁾(x)′β̂`　**(2.6)** `ζ̂_p(x,ŵ) = η⁽¹⁾(θ̂_p(x,ŵ)) μ̂_p⁽¹⁾(x)`

**Figure 1（构造演示，ACS 数据）**：约 32,000 个观测的原始散点图（a）已被密集点云 + 少量离群值主导、无法读形；(c) 用临时 J = 10 画出的图"看着线性"，但这只是因为**过平滑**——IMSE 最优是 81（见第 05 节）；(d) 用最优 J 恢复出正确形状：最穷地区的未保险率反而低（Medicaid 效应），线性假设不成立。

> **脚注 1（重要）**：无控制变量时，分段常数条件下 QMLE 拟合值 = 最小二乘箱内均值。但**边际效应、分位数、其他基函数、加入控制变量后，这一等价性都不成立**。

## 04 理论：随机分箱下的 Bahadur 表示与强逼近

三条正则性假设（原文 Assumption 1–3，概要）：

- **A1（数据生成）**：i.i.d.、模型 (2.1) 设定正确且解唯一；x 的密度 Lipschitz 且在支撑上有界远离零；协变量与 y 的条件分布有界；
- **A2（统计模型）**：损失 ρ 关于 η 绝对连续，得分可分解 `ψ(y,η) = ψ†(y−η)·ψ‡(η)`（覆盖分位数损失的间断点）；ρ 关于 θ 凸；η 严格单调、三阶可导；矩条件 ν > 2；σ²(x,w) 有界远离零；μ₀ 至少 ζ ≥ p+1 阶连续可微；
- **A3（高层估计条件）**：分箱在给定 (x,w) 下与 y 独立，且**准均匀**（max 箱长 ≤ C·min 箱长）；γ̂ 收敛足够快（`√d·r_γ = o(√(J/n) + J^(−p−1))`）；评估点 ŵ 一致估计。面板个体固定效应（d ∝ n）被排除，要求 d = o(n)；p=0 配 J_IMSE ≍ n^{1/3} 时要求 ν > 6 且 d = o(n^{1/6}/√log n)。

**Theorem 1（Bahadur 表示，对 x 一致）**：在 A1–A3 及若干速率条件下，

> `sup_{x∈𝒳} | ϑ̂_p(x,ŵ) − ϑ₀(x,w) − L̂_p(x,w) | ≲_P r_n + ‖ŵ − w‖`

其中 `L̂_p` 是（不可行的）随机线性化项（含经验 Gram 矩阵 Q̄_p），余项

> `r_n = (J log n / n)^{3/4}·ℓ_n + J^(−(p+1)/2)·√(J/n)·ℓ_n + Jℓ_n²/n + J^(−p−1) + √d·r_γ`，`ℓ_n = log n + √(d log n)`

**Theorem 2（可行强逼近）**：t 统计量过程可被条件高斯过程逼近：

> `Pr( sup_{x∈𝒳} |T_{ϑ,p}(x) − Z̄_{ϑ,p}(x)| > ξ·aₙ⁻¹ ) = o(1)`，且可行版 `Ẑ_{ϑ,p}(x)` 用与数据、分箱独立的标准正态向量 N* 模拟即可

t 统计量（3.1）：`T_{ϑ,p}(x) = (ϑ̂_p(x,ŵ) − ϑ₀(x,w)) / √(Ω̂_{ϑ,p}(x)/n)`，方差为三明治形式 `η⁽¹⁾²·b̂′Q̂⁻¹Σ̂Q̂⁻¹b̂`。

### 相对文献的改进

| 对比对象 | 本文 | 前作 |
| --- | --- | --- |
| Cattaneo et al. (2024) | 推广到非线性/非光滑 M-估计，速率基本不变 | 仅最小二乘半线性 |
| Belloni et al. (2019)（分位数级数） | Bahadur 表示只需较弱速率；固定 d、ν≥4 时足以容纳 IMSE 最优的 p=0 分段常数 | 需要 J⁴/n^(1−ε) = o(1)（强得多） |
| Belloni et al. (2019)（强逼近） | 定理 12 需 J⁵/n^(1−ε)；本文固定 d、aₙ=√log n、ν≥4 时只需 **J^{8/3}/n → 0**（含 log 因子） | 仅分位数，无随机分箱、无控制变量 |
| Chernozhukov et al. (2014) | 覆盖整条 t 统计量过程 | 只在 sup 泛函上达到类似精度 |
| Kong et al. (2010) | 弱相关数据留待推广 | 核估计 Bahadur 表示（可比速率） |

关键技术点：① 直接从 M-估计损失与得分推导线性表示（不像 LS 有闭式解）；② 利用 binscatter 基的**局部支撑 + 逆 Gram 矩阵指数级对角衰减**，把每个系数扰动局部化，避免把 J 个系数当一般回归元处理——这是速率放宽的来源；③ 对随机分箱不做"收敛到非随机极限"的要求。

## 05 调参：IMSE 最优分箱数

- 需要额外假设 A5：随机分箱收敛到满足 A3 的固定分箱（分位数间距自动满足；不满足时仍有最优速率但常数次优的经验规则）。
- **Theorem 3（AISE 展开）**：

> `∫_𝒳 (θ̂_p(x,ŵ) − θ₀(x,w))² ω(x) dx = AISE_θ + o_P(J/n + J^(−2(p+1)))`
>
> `E[AISE_θ | data, Δ̂] = (J/n)·Vₙ(p) + J^(−2(p+1))·Bₙ(p) + o`

- L₂ 与 L∞ 收敛速率结果，即使对**非随机分箱、无协变量**的非线性级数估计也是新的。
- 偏差–方差权衡的最优解：

> **(4.1)** `J_IMSE(p) = ( 2(p+1)·Bₙ(p) / Vₙ(p) )^{1/(2p+3)} · n^{1/(2p+3)}`

- p = 0（分段常数）时回到 On Binscatter 的 J_IMSE ∝ n^{1/3} 结构。可行版本见 SA-4，已实现于 binsreg 包。
- **Figure 2（分位数）**：条件分位数（τ = 0.1 / 0.5 / 0.9）可视化条件分布的**离散度**——低收入地区未保险率的离散度远大于高收入地区（图 a），加入 9 个人口学控制后差距收窄（图 b）。这恢复了 Figure 1(a) 原始散点图里有、但被均值平均抹掉的 spread 信息。

> **脚注 3（提醒）**：收入-消费动态文献中"先残差化再分析分位数"的做法（Hall & Mishkin 1982；Arellano et al. 2017）在截面设定下不适用；部分线性结构天然实现了"控制 w"，无需预处理。

## 06 均匀推断：置信带与假设检验

统一推断针对三个函数 ϑ₀、μ₀⁽ᵛ⁾、ζ₀ 全体，而不是逐点值——处理效应异质性、剂量反应、形状约束都需要"对整个函数"的陈述。**关键问题**：J_IMSE(p) 的点估计有一阶偏差，直接做带无效 → 用**稳健偏差校正（RBC）**：点估计用 p 阶多项式 + J_IMSE(p) 分箱；推断用 **p+1 阶**统计量（同一分箱方案），并把去偏引入的额外方差计入。

> **(5.1)** `Î_{ϑ,p+1}(x) = [ ϑ̂_{p+1}(x,ŵ) ± c_ϑ · √(Ω̂_{ϑ,p+1}(x)/n) ]`
>
> **(5.2)** `c_ϑ = inf{ c ∈ ℝ₊ : P[ sup_{x∈𝒳} |Ẑ_{ϑ,p+1}(x)| ≤ c ｜ data, Δ̂ ] ≥ 1 − α }`（高斯模拟取临界值）

> **Theorem 4（置信带有效）**：J ≍ n^{1/(2p+3)}、aₙ = √(log J) 时，`P[ ϑ₀(x,w) ∈ Î_{ϑ,p+1}(x)，∀x ∈ 𝒳 ] = 1 − α + o(1)`

<div style="background:#FFFFFF;border:2px solid #8FA0F5;border-radius:12px;padding:12px 10px 8px;margin:16px 0;">
<svg viewBox="0 0 1000 400" xmlns="http://www.w3.org/2000/svg" style="width:100%;height:auto;display:block;">
  <defs>
    <marker id="nlb" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="#8FA0F5"/>
    </marker>
  </defs>
  <!-- 步骤1 -->
  <rect x="330" y="16" width="340" height="56" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="40" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">① 点估计（最优但带偏）</text>
  <text x="500" y="60" text-anchor="middle" fill="#3A4DA8" font-size="12.5">选 p，按 Ĵ_IMSE(p) 分箱 → ϑ̂_p（IMSE 最优，推断无效）</text>
  <path d="M500,72 L500,96" stroke="#8FA0F5" stroke-width="2" marker-end="url(#nlb)"/>
  <!-- 步骤2 -->
  <rect x="330" y="98" width="340" height="56" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="122" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">② RBC 去偏</text>
  <text x="500" y="142" text-anchor="middle" fill="#3A4DA8" font-size="12.5">用 p+1 阶多项式做推断统计量 T_{ϑ,p+1}（同一分箱）</text>
  <path d="M500,154 L500,178" stroke="#8FA0F5" stroke-width="2" marker-end="url(#nlb)"/>
  <!-- 步骤3 -->
  <rect x="330" y="180" width="340" height="56" rx="10" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="204" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">③ 条件高斯强逼近（Thm 2）</text>
  <text x="500" y="224" text-anchor="middle" fill="#3A4DA8" font-size="12.5">模拟 Ẑ_{ϑ,p+1} 的 sup 分布 → 临界值 c_ϑ</text>
  <path d="M500,236 L500,260" stroke="#8FA0F5" stroke-width="2" marker-end="url(#nlb)"/>
  <!-- 步骤4 -->
  <rect x="140" y="262" width="720" height="62" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="2"/>
  <text x="500" y="288" text-anchor="middle" fill="#1C2733" font-size="14" font-weight="700">④ 输出（Thm 4 保证覆盖 1−α）</text>
  <text x="500" y="308" text-anchor="middle" fill="#3A4DA8" font-size="12.5">置信带 Î_{ϑ,p+1}(x) ｜ 参数设定检验（线性/多项式）｜ 形状检验（单调性）｜ 多样本比较</text>
  <!-- 侧注 -->
  <rect x="24" y="262" width="96" height="62" rx="8" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="1.5"/>
  <text x="72" y="286" text-anchor="middle" fill="#1C2733" font-size="12" font-weight="700">对三类对象</text>
  <text x="72" y="304" text-anchor="middle" fill="#3A4DA8" font-size="11.5">ϑ₀ / μ₀⁽ᵛ⁾ / ζ₀</text>
  <rect x="880" y="262" width="96" height="62" rx="8" fill="#FFFFFF" stroke="#8FA0F5" stroke-width="1.5"/>
  <text x="928" y="286" text-anchor="middle" fill="#1C2733" font-size="12" font-weight="700">点态版本</text>
  <text x="928" y="304" text-anchor="middle" fill="#3A4DA8" font-size="11.5">SA + 软件</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 B｜均匀推断工作流：IMSE 点估计 → RBC 去偏 → 高斯逼近临界值 → 置信带与检验（依据论文第 5 节内容自绘，示意）</div>
</div>

**Figure 3（实证）**：(a) 条件均值的置信带清楚刻画出 Medicaid 效应——关系**非单调**；(b) 边际效应的置信带在低收入与高收入两端完全位于零的同一侧 → **拒绝单调性**。

**两个使用注意**：

- **评估点 w 影响一切**：w 含二元组别变量时，两组的置信带不确定性不同；ϑ₀、ζ₀ 的带形随 w 变化——这是模型 (2.1) 固有的，不是本方法特有；
- **点估计可能在带外**：RBC 用 p+1 阶带、p 阶点，偏差大的区域点会落在带外——形式上正确、视觉上不讨好；给点估计也去偏会破坏其 IMSE 最优性，并引入额外调参。

**假设检验**：参数设定检验的原假设 `H₀: sup|μ₀(x) − m(x;θ)| = 0`（对某个 θ）vs 对所有 θ 都不成立；统计量 `T̂_{μ,p+1}(x) = (μ̂_{p+1}(x) − m(x;θ̃))/√(Ω̂_{μ,p+1}(x)/n)`，θ̃ 在 H₀ 下拟合参数模型得到；形状检验（如单调性）为单侧问题（Theorem SA-3.10）。

**Table 1（规格与形状检验，L∞ 范数，p 值基于 50,000 次模拟）**：

| 检验 | 全样本 统计量 / p 值 / Ĵ_IMSE | 收入 > $16,248 子样本 统计量 / p 值 / Ĵ_IMSE |
| --- | --- | --- |
| 线性设定（无控制） | 22.319 / 0.000 / 81 | 21.344 / 0.000 / 40 |
| 线性设定（有控制） | 7.176 / 0.000 / 12 | 4.265 / 0.000 / 8 |
| 三次设定（无控制） | 9.961 / 0.000 / 81 | 75.426 / 0.000 / 40 |
| 三次设定（有控制） | 9.014 / 0.000 / 12 | 3.506 / 0.001 / 8 |
| 单调递减（无控制） | 7.155 / 0.000 / 16 | 1.095 / 0.803 / 10 |
| 单调递减（有控制） | 5.929 / 0.000 / 11 | 0.944 / 0.973 / 9 |

解读：线性、三次都被**拒绝**（非参数建模必要）；全样本拒绝"单调递减"（Medicaid 所致）；限制在收入高于 $16,248（2013–2017 单人联邦贫困线 138%，Medicaid 扩围的收入门槛）的子样本后，**不能拒绝单调递减**，与 Figure 3 置信带形状一致。

## 07 多样本比较与因果推断

分组指标 t = 0,…,L，模型指数变为 `θ₀(xᵢ,wᵢ,tᵢ) = Σₜ 𝟙{tᵢ=t} θ_{0,t}(xᵢ,wᵢ)`，各组有各自的 μ_{0,t}、γ_{0,t} 与 ϑ_{0,t}、ζ_{0,t}。应用场景：

- 随机实验：`ϑ_{0,1}(x,w) − ϑ_{0,0}(x,w)` 就是 **CATE（条件平均处理效应）** 函数，随 x 变化即异质性；变化率是 `ζ_{0,1} − ζ_{0,0}`；
- 可正式检验 `H₀: ϑ_{0,1} = ϑ_{0,0} ∀x`（无处理效应）；可为最大异质性效应 `x̂* = arg sup |ϑ̂₁ − ϑ̂₀|` 量化不确定性。

**Figure 4（实证）**：按州人口密度分两组（低 < 100 人/平方英里，高 ≥ 100）。(a)(b) 条件均值：无控制时两组的置信带大体重叠（除极低收入段）；加控制后两组界限清晰。低密度州未保险率更高。(c)(d) 差值（CATE）+ 置信带（用低密度州的分箱做共同分箱）：`H₀: ϑ₁ = ϑ₀ ∀x` 被拒绝——检验统计量 7.719（无控制）与 8.936（有控制），p 值极小。多样本比较同样对评估点 w 敏感，解释结果时须留意。

## 08 实证应用：ACS 收入与未保险率（全程演示）

| 维度 | 内容 |
| --- | --- |
| 数据 | ACS 2013–2017 五年调查，美国邮编制表区（不含波多黎各），约 32,000 个观测 |
| 变量 | y = 无健康保险人口比例（分数结果）；x = 人均收入；w = 9 个人口学控制（高中/本科比例、中位年龄、失业率、平均家庭规模、无网络比例、军队比例、仅英语比例、65+ 比例等） |
| 模型 | Logistic QMLE（例 2）为主，辅以分位数（例 3）、稳健中位数 |
| 关键结果 | Ĵ_IMSE = 81（均值）；最穷地区未保险率反低（Medicaid）；低收入区条件分布离散度大；关系非单调；线性/三次设定被拒；Medicaid 门槛之上单调递减不能被拒；低密度州未保险率更高且差异显著 |

选这个数据的原因：Medicaid 与保险成本等已知经济规律**预先告诉我们图应该长什么样**——目的是验证方法能否恢复已知特征，而不是发现新结论。

## 09 附录：三个额外实证演示（方法论价值）

1. **稳健性（Figure A.1）**：把 32 个观测（数据的 0.1%）人为改成上端离群值（分别污染低收入端与高收入端）。最小二乘条件均值被毁：低收入端 Medicaid 凹陷消失、高收入端下降趋势反转；**中位数分箱几乎不受影响**——稳健回归在参数建模中的优势完整带入 binscatter。
2. **支撑之外的预测（Figure A.2）**：加控制后比较线性最小二乘与 Logistic QMLE。遍历 9 个控制的 min/max 组合得到 512 条分箱曲线：**最小二乘大量曲线落到 [0,1] 之外**（线性概率模型的通病），其中"小家庭、高军队比例、高 65+ 比例、高仅英语比例"地区尤甚；Logistic QMLE 天然不会越界。
3. **不确定性可视化与目标参数（Figure A.3）**：把 10/90 分位数（灰点）与条件均值点估计（蓝点）+ 80% 置信带叠画。**均值估计的不确定性很小（带很窄），但给定 x 的结果不确定性很大**——两者含义不同：前者是推断问题（检验/决策用），后者是数据特征（离散度）。

## 10 专题 · 控制变量 w 如何体现在最终结果（尤其散点图）

承接第 02、03 节：w 以部分线性指数 `θ₀(x,w) = μ₀(x) + w′γ₀` 进入 (2.1)/(2.3)。那"控制 w"到底怎么落到最终结果、尤其 binsreg 画出的散点图上？

**一句话**：w 的体现 = 两个动作——① 估计时，样条 β̂ 与全局 γ̂ 在同一个损失函数里联合估计（2.3），让 μ̂(x) 成为"扣除 w 之后"的 x 净关系；② 画图时，把曲线放在评估点 ŵ 上展示：`ϑ̂(x, ŵ) = η(μ̂(x) + ŵ′γ̂)`（binsreg 默认 `at(mean)`）。

**三个层面**：

1. **估计层**：γ̂ 是一个**全局系数向量、跨 x 不变**；`(β̂, γ̂)` 联合一步估计，不是"先残差化再分箱"（旧 binscatter / binscatter2 的做法，一般不一致）。μ̂(x) 因此是"把 w 的线性影响拿掉之后"的 x–y 关系。
2. **图形层（散点图）**：
   - **点（dots）**：y 在每个箱内的均值，**原始数据、未被 w 调整**；
   - **线（line）**：`ϑ̂(x, ŵ) = η(μ̂(x) + ŵ′γ̂)`——**形状来自 μ̂（x 净效应，不随 ŵ 变）**，**垂直位置来自 ŵ′γ̂（随 at() 移动）**；
   - 点与线之间的**垂直缝隙 ≈ 控制变量效应**；
   - **换 at()**：线性链接下曲线整体平移、形状不变；Logit 等非线性链接下**曲率也变**（η′ 在 index 处取值）；
   - **ci / cb**：宽度与位置同样依赖 ŵ（第 06 节"评估点 w 影响一切"）。
3. **推断层**：函数水平（v=0）的估计与检验依赖 ŵ；而 μ 的一阶导数 μ̂′ 在可加线性 index 模型里**与 ŵ 无关**——所以"这关系是否线性"应优先检验 v=1 导数（第 06 节 Table 1 的设定检验正是基于导数对象），避免评估点选择的误导。

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
  <text x="632" y="316" fill="#8FA0F5" font-size="13" font-weight="700">ϑ̂(x, ŵ=mean) 调整曲线</text>
  <text x="632" y="336" fill="#B9C6F2" font-size="12.5">换 at() → 整体平移（线性链接）</text>
  <text x="632" y="200" fill="#6B7A8A" font-size="12.5">y 的分箱均值（原始，未调整）</text>
  <rect x="60" y="380" width="880" height="60" rx="10" fill="#DCE7FF" stroke="#8FA0F5" stroke-width="1.5"/>
  <text x="500" y="404" text-anchor="middle" fill="#1C2733" font-size="13" font-weight="700">形状 ← μ̂(x)（x 净效应，不随 ŵ 变）｜ 水平位置 ← ŵ′γ̂（随 at() 移动）｜ 置信带宽度与位置 ← 也依赖 ŵ</text>
  <text x="500" y="426" text-anchor="middle" fill="#3A4DA8" font-size="12.5">Logit 等非线性链接下，换 ŵ 曲率也会变（η′ 在 index 处取值）→ 检验线性优先用一阶导数 v=1</text>
</svg>
<div style="font-size:12px;color:#6B7A8A;text-align:center;margin-top:6px;">图 C｜binsreg 散点图解剖：原始分箱点 + 调整曲线 + 评估点平移（依据论文内容自绘，示意，非真实数据）</div>
</div>

**自查三题**：

1. binsreg 散点图里的点是"控制变量调整后的数据点"还是"原始分箱均值"？
2. `at(mean)` 换成 `at(median)`：曲线形状变吗？水平变吗？Logit 模型下呢？
3. 为什么检验"是不是线性"推荐用一阶导数 v=1 而非水平值？

答案：① 原始分箱均值，未调整；② 线性链接下形状不变、水平平移，Logit 下曲率也变，且置信带宽度与位置都变；③ 水平检验受评估点 ŵ 影响，而 μ̂′ 的形状与评估点无关，结论更稳健。

## 11 学习要点与速查

### 避坑清单

1. 有控制变量时**不要逐箱独立估计**——γ₀ 是全局参数，用 (2.3) 整体估计；
2. 分数/二元结果用 **Logistic QMLE**，别用线性分箱（预测会越出 [0,1]）；
3. 记住脚注 1 的等价性边界：无控制分段常数下 QMLE ≈ LS，但**边际效应、分位数、加控制后都不等价**；
4. 用 Ĵ_IMSE(p) 分箱；J 太小（如 10）过平滑，"看着线性"只是假象；
5. 推断必须 RBC：点估计用 p、带/检验用 p+1 阶统计量，同一分箱；
6. 留意评估点 w：它改变置信带形状、位置与检验结论；
7. 点估计落在带外是 RBC 的正常现象，不是 bug；
8. 分位数 binscatter 画的是**条件分布离散度**，不是均值估计的不确定性——别混为一谈；
9. 截面数据别照搬"先残差化再分位数"的时序做法（脚注 3）。

### 关键术语

| 符号 / 术语 | 含义 |
| --- | --- |
| ρ(y; η)、η(θ) | M-估计的损失函数与逆链接函数（可非线性、可非光滑） |
| θ₀(x,w) = μ₀(x) + w′γ₀ | 部分线性标量指数（μ₀ 非参数，w 线性进入） |
| ϑ₀(x,w) = η(θ₀(x,w)) | 水平：剂量反应 / 组别水平 |
| μ₀⁽ᵛ⁾(x) | 非参数分量及其 v 阶导数 |
| ζ₀(x,w) = η⁽¹⁾μ₀⁽¹⁾ | 边际（偏）效应（均值处，非平均边际效应） |
| b̂_{p,s}(x) | 箱内 p 阶多项式 + (s−1) 阶导连续的基（s=p=0 分段常数；s=p=3 三次 B 样条） |
| J_IMSE(p) | IMSE 最优分箱数：∝ n^{1/(2p+3)} |
| RBC | 稳健偏差校正：p+1 阶推断统计量 + 去偏方差 |
| QMLE | 拟极大似然（分数响应，Papke & Wooldridge 1996） |
| Bahadur 表示 / 强逼近 | 一致随机线性化 / 条件高斯过程逼近（推断的理论地基） |
| CATE | 条件平均处理效应 ϑ₁ − ϑ₀（随 x 变化即异质性） |
| 准均匀（quasi-uniformity） | 各箱长度可比，控制偏差与每箱样本量 |

### 公式速查

| 编号 | 公式 | 一句话 |
| --- | --- | --- |
| (2.1) | `(μ₀,γ₀) = argmin E[ρ(y; η(μ(x)+w′γ))]` | 半参数 M-估计框架（设定正确、解唯一） |
| 例1–3 | LS / Logistic / Quantile 的 ρ 与 η | 一个框架覆盖三类模型 |
| 目标 | `ϑ₀ = η(θ₀)`、`μ₀⁽ᵛ⁾`、`ζ₀ = η⁽¹⁾μ₀⁽¹⁾` | 三个实用对象 |
| (2.2) | `μ̂(x) = b̂₀(x)′β̂`（分段常数 M-估计） | 箱内局部常数拟合的定义 |
| 基函数 | `b̂_{p,s}(x) = T̂_s[b̂₀(x)⊗(1,x,…,xᵖ)]` | 多项式 + 平滑约束（s=p 约定） |
| (2.3) | `(β̂,γ̂) = argmin Σρ(yᵢ; η(b̂_p(xᵢ)′β + wᵢ′γ))` | 协变量调整的非线性 binscatter |
| (2.4)–(2.6) | ϑ̂_p / μ̂_p⁽ᵛ⁾ / ζ̂_p（插件法） | 三个对象的估计量 |
| Thm 1 | `sup|ϑ̂_p − ϑ₀ − L̂_p| ≲ r_n + ‖ŵ−w‖` | 随机分箱下的一致 Bahadur 表示 |
| Thm 2 | `sup|T_{ϑ,p} − Z̄_{ϑ,p}|` 可被可行 Ẑ 逼近 | 条件高斯强逼近（临界值来源） |
| (4.1) | `J_IMSE(p) = (2(p+1)Bₙ/Vₙ)^{1/(2p+3)} n^{1/(2p+3)}` | IMSE 最优分箱数 |
| (5.1) | `Î_{ϑ,p+1}(x) = ϑ̂_{p+1} ± c_ϑ√(Ω̂_{ϑ,p+1}/n)` | RBC 统一置信带 |
| (5.2) | `c_ϑ = inf{c: P[sup|Ẑ_{ϑ,p+1}| ≤ c ｜data,Δ̂] ≥ 1−α}` | 高斯模拟临界值 |
| Thm 4 | `P[ϑ₀ ∈ Î_{ϑ,p+1}, ∀x] = 1−α+o(1)` | 置信带均匀覆盖（J≍n^{1/(2p+3)}） |

---

*笔记依据论文全文（Cattaneo, Crump, Farrell, Feng, "Nonlinear Binscatter Methods", 2026-08-21 工作论文稿，45 页含附录）整理；文中数字与结论均出自论文正文、图表注释与 Table 1。两幅示意图为依据论文内容自绘的示意，非论文原图。软件与复制文件：https://github.com/nppackages/binsreg。*
