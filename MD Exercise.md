---
number headings: auto, first-level 1, max 3, 1.1
---
# 1 Toy Model: MD with LJ/EMT



1. 目的：LJ Potential 告訴我們原子間的位能長什麼樣子。MD 模擬則是把這個位能「動起來」：在每個時間步，根據位能對距離的梯度計算力（F = -dV/dr），再用牛頓第二定律（F = ma）更新每個原子的**位置和速度**。

2. 工具：ASE（Atomic Simulation Environment）
	- 一個 Python 套件，專門用來做原子尺度的模擬。優點是純 Python、容易上手、適合小型練習。
	- 其他MD模擬軟體還有: LAMMPS, ...

	計算力Force calculation的原子間勢 interatomic potential：Effective Medium Theory (EMT)，比 LJ 稍微精確一點的經驗勢，ASE 內建支援銅（Cu）等金屬，**完全不需要 DFT**。

3.  系综（Ensemble）：NVE 和 NVT 是熱力學系综的縮寫，分別為
	- NVE：total energy = constant，溫度自由波動
	- NVT：溫度 = constant，total energy 會波動（因為 Langevin 熱浴一直在加/減能量）
	NVE = 孤立系統，沒有外界干擾，能量守恆。NVT = 系統泡在熱浴裡，溫度被固定，但能量可以和熱浴交換。
	NVT 下使用 **Langevin thermostat**：在每個原子的運動方程式加入摩擦力與隨機力，模擬系統與熱浴的耦合。


|      | Hubbard model                                            | MD+LJ/EMT 對應版本                                                                                |
| ---- | -------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| 方程式  | $H = -t\sum c^\dagger c + U\sum n_\uparrow n_\downarrow$ | Newton's 2nd law $F = ma$<br>LJ potential $V(r) = 4\varepsilon[(\sigma/r)^{12}-(\sigma/r)^6]$ |
| 示意圖  | $t$（hopping）和 $U$（on-site repulsion）                     | LJ potential 曲線，標出 $r_{min}$、$\varepsilon$、排斥項與吸引項                                            |
| 經典輸出 | Mott transition（U 增大時 gap 打開）                            | **能量守恆圖**（NVE 下 total energy 應保持不變）＋**溫度隨時間變化**（NVT 下溫度穩定在設定值）                                |

| |LJ Potential|EMT|
|---|---|---|
|用途|概念示範，最簡單的原子間作用力模型|實際模擬金屬用的經驗勢|
|公式|有解析公式，可以畫出來|沒有簡單公式，是數值方法|
|適用|惰性氣體（Ar, Ne）|金屬（Cu, Au, Ni...）|
|類比|就像 Hubbard model|就像更完整的多軌道模型|

| 系综  | 固定量        | 用途              |
| --- | ---------- | --------------- |
| NVE | 原子數、體積、總能量 | 驗證數值方法正確性（能量守恆） |
| NVT | 原子數、體積、溫度  | 模擬真實實驗條件（恆溫）    |

| 字母  | 意思                  | 固定的量       |
| --- | ------------------- | ---------- |
| N   | Number of particles | 原子數不變      |
| V   | Volume              | 體積不變       |
| E   | Energy              | 總能量不變（NVE） |
| T   | Temperature         | 溫度不變（NVT）  |


## 1.1 Lennard-Jones Potential：原子間作用力的最簡單模型

在分子動力學（MD）模擬中，原子之間的交互作用力來自 **interatomic potential**。最簡單的模型是 Lennard-Jones (LJ) Potential：

$$V(r) = 4\varepsilon\left[\left(\frac{\sigma}{r}\right)^{12} - \left(\frac{\sigma}{r}\right)^6\right]$$

### 1.1.1 參數的物理意義

| 參數 | 物理意義 | 對曲線的影響 |
|---|---|---|
| **ε (epsilon)** | 位能井深，代表吸引力的強度 | ε 越大 → 井越深 → 原子結合越緊 |
| **σ (sigma)** | 原子的有效直徑（V=0 時的距離） | σ 越大 → 曲線整體右移 → 平衡距離越大 |

##### 兩項的物理起源
- **排斥項** $(σ/r)^{12}$：Pauli exclusion principle，電子雲重疊時急速排斥
- **吸引項** $(σ/r)^{6}$：van der Waals 吸引力（London dispersion）


![[lj_parameters.png|601]] 
圖：ε 和 σ 變化時曲線的變化趨勢。

## 1.2 物理量

### 1.2.1 能量與溫度
MD 的核心輸出是**原子的位置和速度**。但我們記錄的是從位置和速度**衍生出來的物理量**，例如：

```
位置 → 計算位能 (Potential Energy)
速度 → 計算動能 (Kinetic Energy)
速度 → 統計平均 → 溫度
```

溫度在統計力學裡的定義就是：**所有原子動能的平均值**。所以溫度不是輸入，而是從原子速度算出來的輸出。
在程式碼中，可以直接使用內建函數： `atoms_nve.get_temperature()`


![[md_cu.png]]

|圖|說明|
|---|---|
|左：NVE 能量|Total energy 應該幾乎是水平線，這就是能量守恆的證明|
|中：NVE 溫度|溫度會在 300K 附近波動，但沒有被控制|
|右：NVT 溫度|溫度應該逐漸穩定在 300K 附近|

1. **NVE**：系統從一個非平衡的初始狀態開始弛豫，能量在動能和位能之間交換，溫度自由波動，最後穩定在某個值（不一定是 300K）。
2. **NVT（Langevin）**：有一個「虛擬熱浴」在拉著溫度。Langevin 的做法是在每個原子的運動方程式加入：
	- 一個**摩擦力**（把過快的原子減速）
	- 一個**隨機力**（模擬熱漲落）
	這兩項合在一起，就像把系統泡在 300K 的熱浴裡，強迫溫度往目標值收斂。

類比 DMFT：NVE 就像你跑一次 QMC 但不做自洽，NVT 就像加了自洽迴圈，強迫系統收斂到目標狀態。




### 1.2.2 徑向分布函數（Radial Distribution Function, RDF）

RDF g(r) 的定義：從任意一個原子出發，在距離 r 到 r+dr 的球殼內，找到另一個原子的機率密度，相對於均勻分布的歸一化值。

$$g(r) = \frac{V}{N^2} \left\langle \sum_{i \neq j} \delta(r - r_{ij}) \right\rangle$$
其特徵與物理意義為：

| 系統狀態 | RDF 的樣子 |
|---|---|
| 完美晶體（0K）| 幾個無限尖銳的 delta 函數峰 |
| 固體（有限溫度）| 幾個寬化的峰，對應近鄰距離 |
| 液體 | 第一個峰明顯，之後快速衰減到 1 |
| 氣體 | 幾乎沒有結構，g(r) ≈ 1 |

![[RDF_cu.png|346]]
圖：銅（FCC） 晶體的第一近鄰距離 = a/√2 ≈ 3.6/1.414 ≈ 2.55 Å，RDF 第一個峰應出現在這個位置附近。


# 2 Toy Model: MD with MACE

有別於使用EMT，我們也可以透過 `!pip install mace-torch` 引入已經訓練好的資料。
值得注意的是，MACE 不是「數值模擬的結果」，而是**神經網路擬合 DFT 能量曲面的結果**。DFT 跑了幾百萬個結構，算出每個結構的能量和力，然後用神經網路去學這個「結構→能量/力」的映射關係。訓練完之後，DFT 就不再需要了，MACE 自己就能直接給出力。

| |LJ|EMT|MACE|
|---|---|---|---|
|形式|解析公式|半經驗數值公式|神經網路（數值）|
|參數來源|手動調整|擬合實驗數據|擬合 DFT 計算結果|
|精度|低（惰性氣體）|中（金屬定性）|高（接近 DFT）|
|速度|極快|很快|慢（但遠快於 DFT）|



![[mace_md_cu.png]]




### 2.1.1 為什麼 Energy 差這麼多？（+3 eV vs -430 eV）

這是最核心的問題，答案是：**能量的絕對值沒有物理意義，只有能量差有意義。**

| |EMT|MACE-MP-0|
|---|---|---|
|能量參考點|任意設定（孤立原子 = 0）|DFT 計算的絕對能量|
|108 個 Cu 原子的 total energy|~+3 eV|~-430 eV|
|每個原子的內聚能（cohesive energy）|不準確|≈ -4 eV/atom（接近實驗值 -3.5 eV）|
##### 為什麼溫度圖的形狀類似但數值不同？

兩張圖的溫度都從接近 300K 開始掉下去，這是一樣的物理：初始結構不是平衡態，系統在弛豫。

差別在於 MACE 描述的 Cu 比 EMT 更硬（force constant 更大），所以振盪頻率略有不同，最終平衡溫度也略有差異。

就像你在 DMFT 裡，自能的絕對值取決於你怎麼定義 double counting correction，重要的是它的頻率依賴結構，不是絕對值。

EMT 的能量是工程上的近似，參考點任意。MACE 用 DFT 訓練，能量參考點是真實的原子能量，所以負很多（原子結合在一起會釋放能量）。




# 3 研究的下一步

這取決於你的研究目標：

**用 pretrained model（你現在做的）：** 適合通用材料、快速驗證、工業應用。MACE-MP-0 對大多數材料都有合理精度。

**自己訓練（fine-tuning 或從頭訓練）：** 當你需要研究特定材料的精確性質，比如某個新的氧化物、相變、缺陷行為，pretrained model 精度不夠，就需要自己跑 DFT 產生訓練資料，再訓練一個針對這個材料的 MLIP。

**Behler 組做的事情就是後者**：他們開發 NNP（Neural Network Potentials）的方法論，研究如何更有效率地生成訓練資料、設計更好的描述符（descriptor）、提升泛化能力。你如果加入他們，大概會做的事是：

1. 用 DFT（VASP 或 Quantum ESPRESSO）跑某個材料的一批結構
2. 用這些資料訓練一個 NNP（用他們的 n2p2 工具）
3. 用訓練好的 NNP 跑大尺度 MD
4. 分析 MD 結果（RDF、擴散係數、相變等）

這個流程你現在做的小練習已經覆蓋了第 3 和第 4 步的概念。