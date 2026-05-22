---
number headings: auto, first-level 1, max 3, 1.1
---
# 1 From DFT to MD

## 1.1 overview

材料的性質，從最根本的角度來說，是由原子和電子的行為決定的。要計算這些性質，必須求解量子力學的方程式，但對多體系統來說，這在數學上是不可能精確完成的。
##### 密度泛函理論（DFT）
由 Hohenberg、Kohn 於 1964 年提出，Kohn-Sham 形式在 1965 年建立。它的核心想法是：不直接處理多電子波函數，而是把問題映射到電子密度 ρ(r)\rho(\mathbf{r}) ρ(r)，大幅降低計算複雜度。DFT 成功預測了許多材料的晶格常數、能帶結構、聲子頻率以及化學反應能障，成為現代材料計算的基石。

##### 分子動力學（MD）
由 Alder 和 Wainwright 在 1957 年首次用於模擬硬球系統，爾後 Rahman（1964）將其推廣至液態氬的模擬。MD 的核心想法完全不同：不求解電子問題，而是把原子視為經典粒子，用牛頓運動方程追蹤其隨時間的運動。它成功描述了液體的動力學、蛋白質的折疊過程、材料的熱傳導與擴散等現象。

---


## 1.2 Density Functional Theory (DFT)

1. **起點**： Born-Oppenheimer approximation → 只需求解電子部分
2. **方程式**: 多電子薛丁格方程 → Kohn-Sham Equation
- **Hohenberg-Kohn 定理**：基態能量是電子密度 $\rho(\mathbf{r})$ 的唯一泛函
- **Kohn-Sham 映射**：把多體問題映射到一組單粒子方程

$$\left[-\frac{\hbar^2}{2m}\nabla^2 + V_\text{eff}[\rho]\right]\psi_k(\mathbf{r}) =\varepsilon_k \psi_k(\mathbf{r})$$
其中 $V_\text{eff} = V_\text{ext} + V_\text{Hartree} + V_\text{XC}$

3. **近似**：Exchange-correlation functional $V_\text{XC}$ 未知，需要近似（LDA、GGA 等）
4. **Self-consistency loop**： $$\rho_\text{init} \to V_\text{eff}[\rho] \to {KS\text{ eq.}} \to \psi_k \to \rho_\text{new} \to \text{check convergence} \to \rho_\text{converged}$$

## 1.3 Molecular Dynamics (MD)

1. **起點**：Born-Oppenheimer approximation → 原子核視為經典粒子
2. **方程式**：Newton's equation of motion

$$M_I \ddot{\mathbf{R}}_I = -\nabla_{\mathbf{R}_I} V({\mathbf{R}}) \equiv \mathbf{F}_I$$
	其中，$V({\mathbf{R}})$　在物理上我們通常說 "interatomic potential" 或 "potential energy surface (PES)"。在化學和 MD 領域，習慣叫 "force field"，因為最終用途是算力。$\mathbf{F}_I$稱為計算力Force calculation。
3. **近似**：傳統 force field　$V({\mathbf{R}})$（如Lennard–Jones potential、Embedded Atom Method (EAM)、CHARMM、AMBER）是**人為設計的解析函數**，有固定的函數形式，參數從實驗或 DFT 擬合得到。

4. **時間演化 loop**（以 Verlet 積分為例）： $${\mathbf{R}(t), \mathbf{v}(t)} \to \mathbf{F}(t) = -\nabla V \to \mathbf{R}(t+\Delta t), \mathbf{v}(t+\Delta t) \to \text{repeat}$$
```
**MD loop**
初始條件：{R_I(0), v_I(0)}
    ↓
計算力：F_I = -∇V_FF({R_I})   ← 用解析 force field，極快
    ↓
Newton 積分（如 Verlet）：
    R_I(t+Δt) = 2R_I(t) - R_I(t-Δt) + (F_I/M_I)Δt²
    ↓
更新位置與速度
    ↓
回到「計算力」，重複
```


![[Bioinformatics and Chemistry.10.37.10.2147.png|402]]


5. [[MD Exercise]]
6. MD和DFT的比較

| Feature  特色/特點           | Molecular Dynamics (MD)  <br>分子動力學 (MD)                                                                                                                                                                                                                                           | Density Functional Theory (DFT)  <br>密度泛函理論 (DFT)                                                                                                                                                                                          |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Physics Basis  物理學基礎** | Classical mechanics (Newton’s laws)  <br>經典力學（牛頓定律）                                                                                                                                                                                                                               | Quantum mechanics (Schrödinger equation)  <br>量子力學（薛定諤方程式）                                                                                                                                                                                 |
| **Core Variable  核心變數**  | Positions and velocities of atoms  <br>原子的位置與速度                                                                                                                                                                                                                                   | Electron density distributions  電子密度分佈                                                                                                                                                                                                     |
| **Speed / Scale  速度/規模** | Simulates thousands of atoms for nanoseconds to microseconds.  <br>能在==納秒到微秒==的時間內，模擬數千個原子的行為。                                                                                                                                                                                    | Simulates tens to hundreds of atoms for picoseconds.  <br>在==皮秒==時間尺度上，只能模擬數十到數百個原子。                                                                                                                                                       |
| **Accuracy  準確度**        | Lower. Requires predefined "force fields" (empirical parameters).  <br>較低。==需要預先定義的「力場」（即經驗參數）==。                                                                                                                                                                                 | Higher. Calculates properties from first principles (ab initio) without prior assumptions.  <br>更高。透過==第一性原理（ab initio）==來計算各種性質，而不做任何先前的假設。                                                                                               |
| **Use Cases  使用案例**      | Use **MD** when you need to observe long-timescale structural changes or physical behaviors where bonds don't break or form (e.g., how a drug molecule diffuses into a protein).  <br>當你需要觀察那些在長時間尺度上的結構變化或物理行為，且在此過程中化學鍵不會被破壞或形成時，就可以使用分子動力學模擬。例如，可以透過分子動力學模擬來瞭解藥物分子是如何滲透進蛋白質中的。 | Use **DFT** when you need to understand the fundamental electronic or chemical properties of a material.  <br>當您需要了解某種材料的基礎電子或化學性質時，可以使用密度泛函理論。<br>Chemical reactions, band gaps, catalysis, and bond breaking.  <br>化學反應、能帶間隙、催化作用以及鍵的斷裂。 |


## 1.4 Ab Initio Molecular Dynamics (Ab Initio MD, AMID)

在 AIMD 中，每一個 MD timestep 都需要重新進行一次完整的 DFT 計算
1. **Hellmann-Feynman theorem**：從 DFT 的電子密度計算出每個原子所受的力。
   Total Energy 是原子位置的函數 $E_\text{DFT} = E_\text{DFT}({R_I})$（給定一組原子位置 ${R_I}$，DFT 就給你一個能量值）。而力則是$F_I = -\partial E_\text{DFT}/\partial R_I$。


2. 流程
   固定當前原子位置→ 使用 DFT 計算電子結構→ 由電子結構計算原子受力→ 更新原子位置→ 重複以上步驟→ ...

```
**AIMD loop**
初始條件：{R_I(0), v_I(0)}
    ↓
以當前 {R_I(t)} 為輸入，執行一次完整 DFT SCF：
    ρ_init → V_eff[ρ] → KS eq. → ψ_k → ρ_new → 收斂
    → 輸出 E_DFT({R_I}) 和 F_I = -∂E/∂R_I   ← 慢，每步都要收斂
    ↓
Newton 積分（完全相同）：
    R_I(t+Δt) = 2R_I(t) - R_I(t-Δt) + (F_I/M_I)Δt²
    ↓
更新位置與速度
    ↓
回到「執行 DFT」，重複
```




## 1.5 不同方法之間的差異

在材料模擬中，不同方法之間最大的差異之一，在於「計算成本（computational cost）」與「模擬尺度（simulation scale）」之間的取捨。

##### 密度泛函理論（DFT）
以第一性原理方法（first-principles methods）中的 Density Functional Theory（DFT）為例，雖然它透過電子密度（electron density） $\rho(\mathbf r)$  將3N維空間中的波函數$\Psi(\mathbf r_1,\mathbf r_2,\dots,\mathbf r_N)$簡化三維的波函數（其中N 為電子數量），大幅降低了問題的複雜度。
然而，實際使用的 Kohn–Sham DFT 並不是直接只對電子密度進行計算，而是引入一組單電子軌域（Kohn–Sham orbitals）$\psi_{KS}(\mathbf r)$ 並利用這些軌域重建電子密度$\rho(\mathbf r)=\sum_i |\psi_{KS}(\mathbf r)|^2$。因此，實際計算時仍然需要求解大量的單電子方程$\hat H \psi_i = \epsilon_i \psi_i$也就是M維矩陣的對角畫化（其中M與原子數量、電子數量呈正比），其計算量約為$O(M^3)$。這裡的$M$並不是指空間維度，而是系統大小（system size）的量級。因此，即使 DFT 已經避免直接處理 (3N) 維波函數，其計算成本仍然會隨系統大小快速上升。
由於 DFT 的計算成本很高，因此實際上可處理的原子數量級約在$10^2$左右。適合模擬一小塊晶體（unit cell）、材料表面（surface slab）、缺陷附近的局部區域、奈米尺度結構等特性，而無法直接模擬真實尺寸的大塊材料。

##### 分子動力學（MD）
Classical Molecular Dynamics 不再顯式求解電子結構，而是直接使用經驗性 force field解析勢能函數，計算原子間作用力的計算成本通常接近$O(N)$或是$O(NlogN)$，其中N為...。
因此 Classical MD 能夠模擬更多原子（large length scale）和模擬更長時間（large time scale）
但代價是精度下降，因為 force field 本身是近似模型，可能無法準確描述化學鍵斷裂、電荷轉移、電子結構變化、激發態效應等量子現象。

##### Ab Initio Molecular Dynamics（AIMD）
在 AIMD 中，由於每一個 MD timestep 都需要重新進行一次完整的 DFT 計算。由於原子振動的時間尺度約為飛秒（femtosecond, fs）等級，因此 MD 的 timestep 通常只能取$\Delta t \sim 1\ \mathrm{fs}=10^{-15}$s，而一次 DFT 計算可能就需要數分鐘甚至數小時，因此 AIMD 通常只能模擬數皮秒（ps=$10^{-12}$s）的時間尺度，也就是大約$10^{3}$次迭帶。

##### 總計算成本
計算成本可表示為$$\text{Total Cost}=(\text{Cost per step})  \times  (\text{Number of steps})$$，其中
- 系統(Length scale)越大 → 每一步計算越昂貴
- 模擬時間越長 → 所需 timestep 越多

這也是為何在材料模擬領域中，做為取捨，常會將：
- DFT/AIMD 視為「高精度、小尺度、短時間」的方法
- Classical MD 視為「低精度、大尺度、長時間」的方法
兩者各自適用於不同的研究問題。

##### 銜接： Machine-learned interatomic potential (MLIP)
為了去解決兩種模擬方法的限制，在第三個章節中，我們會引入MLIP。有別於傳統的可解析 force field，他引入Machine Learning 做為銜接：==先用 DFT 生成訓練數據，再訓練一個 ML 模型來逼近 DFT 的 potential energy surface==。這樣既保留了 DFT 的精度，又接近以至於在保留DFT的精度之下，也可以長時間尺度、多原子數(大尺度)的系統。


| 方法          | 實際含義              | 時間尺度  | 原子數      | 準確度    |
| ----------- | ----------------- | ----- | -------- | ------ |
| **DFT**     | AIMD：每步都算 DFT     | ~ps   | ~100     | 高      |
| **MD**      | 傳統 force field MD | ns–μs | ~10⁶     | 低–中    |
| **MLIP-MD** | ML potential MD   | ns–μs | ~10⁴–10⁶ | 接近 DFT |

![[Commun Mater 4, 66 (2023).png|581]]
**X 軸（Length scale）**：你的模擬盒子有多大，也就是你在模擬多大的一塊材料。
**Y 軸（Time scale）**：你能追蹤多長的物理時間演化。



![[Nature Materials volume 20 (2021).png|559]]

ML potentials as a potential solution to the trade-off between cost and accuracy of conventional atomistic simulations. Potential future developments include hybrid machine learning/molecular mechanics (ML/MM) methods, more efficient representations to decrease simulation times and more accurate training data (proposed by an active learning algorithm) to improve the model accuracy beyond density functional theory. Potential future applications are shown in the blue box (only approximately positioned according to their system-size and accuracy requirements). [Nature Materials volume 20 (2021)]

## 1.6 補充: 與DFT/MD/DMFT 的異同

### 1.6.1 DFT+DMFT 和AIMD 差異

1. DFT+DMFT charge self-consistency loop： 兩者之間傳遞的是**電子密度** $\rho$，在電子自由度的空間裡迭代。
$$\rho_\text{init} \to H_\text{DFT}^k \to \Sigma(i\omega_n) \text{ [DMFT]} \to \rho_\text{DMFT} \to H_\text{DFT}^k \text{ [更新]} \to \cdots$$

2. AIMD loop：兩者之間傳遞的是**原子位置與力** $F_I$，在原子核自由度的空間裡迭代。
$${R_I(t), \dot{R}_I(t)} \to E_\text{DFT}({R_I}), F_I \text{ [DFT計算]} \to R_I(t+\Delta t) \text{ [Newton積分]} \to \cdots$$

|               | DFT → DMFT                                                                               | DFT → MD                                                                        |
| ------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **DFT 的角色**   | 提供 band structure / Wannier Hamiltonian 作為 input                                         | 提供 potential energy surface (PES) / forces 作為 reference                         |
| **DFT 失效的原因** | 忽略局域強相關（Mott insulator problem）                                                          | **計算太慢**，無法處理大尺度、長時間動力學                                                         |
| **核心問題**      | 電子如何在強相關下行為？                                                                             | 原子如何在時間中運動？                                                                     |
| **方程式**       | Kohn-Sham eq. → Dyson eq. / self-consistency loop                                        | Newton's eq. of motion: $M\ddot{R} = -\nabla_R E$                               |
| **求解對象**      | Green's function $G(i\omega_n)$                                                          | 原子軌跡 ${R_i(t)}$                                                                 |
| **連接方式**      | $\varepsilon_k$（DFT band）→ Hybridization function $\Delta(i\omega_n)$，寫出 $\mathcal{G}^0$ | DFT 計算 forces $F = -\nabla E$ →$\mathbf{R}(t+\Delta t), \mathbf{v}(t+\Delta t)$ |



### 1.6.2 DFT/MD/DMFT 之間的差異與限制

|             | DFT                        | DFT+DMFT                            | 傳統MD                   | AIMD                   | MLIP                   |
| ----------- | -------------------------- | ----------------------------------- | ---------------------- | ---------------------- | ---------------------- |
| **描述對象**    | 電子基態                       | 強相關電子動力學                            | 原子核運動                  | 原子核運動                  | 原子核運動                  |
| **核心方程**    | Kohn-Sham eq.              | Dyson eq. + self-consistency        | Newton's eq. of motion | Newton's eq. of motion | Newton's eq. of motion |
| **求解量**     | $\rho(r)$, $\varepsilon_k$ | $G(i\omega_n)$, $\Sigma(i\omega_n)$ | $R_I(t)$, $v_I(t)$     | $R_I(t)$, $v_I(t)$     | $R_I(t)$, $v_I(t)$     |
| **DFT 的輸入** | —                          | $\varepsilon_k$（band structure）     | forces $F = -\nabla E$ | forces $F = -\nabla E$ | forces $F = -\nabla E$ |
| **限制**      | —                          | 忽略空間關聯                              | 精度低                    | 計算成本高                  |                        |
| **解決方案**    | —                          | cDMFT, D$\Gamma$A                   | 納入ab initio參數(AIMD)    | ML訓練PES (MLIP)         |                        |

> ==DMFT extends DFT in the _energy_ dimension (correlation); MD extends DFT in the _time and length scale_ dimension. MLIP is the bridge that makes DFT-quality MD feasible.==

### 1.6.3 Toy Model


tight binding model

|      | Hubbard model                                            | MD+LJ/EMT 對應版本 [[MD Exercise]]                                                                |
| ---- | -------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| 方程式  | $H = -t\sum c^\dagger c + U\sum n_\uparrow n_\downarrow$ | Newton's 2nd law $F = ma$<br>LJ potential $V(r) = 4\varepsilon[(\sigma/r)^{12}-(\sigma/r)^6]$ |
| 示意圖  | $t$（hopping）和 $U$（on-site repulsion）                     | LJ potential 曲線，標出 $r_{min}$、$\varepsilon$、排斥項與吸引項                                            |
| 經典輸出 | Mott transition（U 增大時 gap 打開）                            | **能量守恆圖**（NVE 下 total energy 應保持不變）＋**溫度隨時間變化**（NVT 下溫度穩定在設定值）                                |


### 1.6.4 目前為止的研究故事架構

```
**YITING的學習路徑**
1. DFT 介紹（已有）
   ↓
2. DFT 成功：預測 band structure、金屬/絕緣體（已有）
   ↓
3. DFT 的兩個方向的延伸：
   路徑 A：強相關問題 → DMFT（已有）
   路徑 B：大尺度動力學問題 → MD
   ↓
4. MD 的 Equation of Motion（Newton）
   → 類比 DMFT 的 Dyson equation
   ↓
5. MD 的瓶頸：$E(\{R\})$ 怎麼算？
   → AIMD 太慢 / 傳統 force field 不夠準
   ↓
6. MLIP：用 DFT data 訓練 → 兩全其美
```

|        | DFT+DMFT（你的背景） | DFT→MLIP→MD（新方向） |
| ------ | -------------- | ---------------- |
| 出發點    | DFT            | DFT              |
| 解決的問題  | 電子關聯效應（小尺度）    | 大尺度原子動力學         |
| ML 的角色 | 無              | 擬合 PES，取代直接計算    |
| 計算瓶頸   | QMC 採樣成本       | DFT 訓練資料生成       |


在 DMFT 的故事中：
- DFT 給你 $\varepsilon_k$，DMFT 用它建立 bath，寫出 $\mathcal{G}^0(i\omega_n) = [i\omega_n + \mu - \Delta(i\omega_n)]^{-1}$

在 MD 的故事中：

- MD 的 equation of motion 是 $M_I \ddot{R}_I = -\nabla_{R_I} E({R})$
- 問題在於：**$E({R})$ 怎麼算？
	- - 如果用經驗 force field（Lennard-Jones 等）→ 快但不夠準確，無法轉移到新系統**
    - 如果每步都用 DFT 算 → _ab initio_ MD（AIMD），精確但極慢（只能幾百原子、幾十 ps）
    - **MLIP 的角色**：用 DFT 數據訓練一個 ML model，讓它像 DFT 一樣準確，但像 force field 一樣快

這就是你投影片的「轉折點」，等同於你之前講 NiO 的角色。



# 2 Machine-learned interatomic potential (MLIP)

## 2.1 MLIP歷史簡介
2007 年 Behler提出了一個想法：
> 用 DFT 算一大堆不同構型的能量和力當作訓練資料，然後用機器學習模型來學習「原子排列 → 能量/力」這個映射。

這樣的模型稱為 **Machine Learning Interatomic Potential（MLIP）**，他是 interatomic potential 的一種，只是用 ML 來訓練它，其精度接近 DFT，但計算速度接近經典 potential，可以接 LAMMPS 做大規模 MD 模擬。


> Review Paper:
> - Pascal Friederich_Machine-learned potentials for next-generation matter simulations (Nat Mater 2021)
> - Jörg Behler_Perspective: Machine learning potentials for atomistic simulations" (JCP 2016)


```
Interatomic Potential（廣義）
│
├── 傳統經驗式：Lennard-Jones, EAM...（快但不準）
│
└── MLIP（Machine Learning Interatomic Potential）
        │
        ├── Behler-Parrinello Neural Network Potential（2007）
        ├── Gaussian Approximation Potential（GAP）
        ├── ACE（2019，數學完備的展開框架）
        └── GRACE（ACE + Graph Neural Network，最新）
```


##### Atomic Cluster Expansion (ACE)
ACE是描述原子局部環境的數學框架，2019 年由 Drautz 提出。
1. 核心思想：把原子的局部化學環境展開成一組完備的基底函數（cluster basis），理論上可以**精確表示任何原子間相互作用**，有點像傅立葉展開之於週期函數。
2. 優點：
	- **數學上完備**，不會有原則性的精度上限
	- **可系統性改進**，截斷更高階就更準
	- 比 Behler 的 Neural Network Potential 有更清楚的物理詮釋

##### Graph Atomic Cluster Expansion (GRACE)

GRACE 是 ACE 的延伸版本，加入了更現代的 **graph neural network** 的架構，讓模型可以更有效地捕捉**長程相互作用**和複雜的多體效應。是目前 MLIP 領域最前沿的方法之一。


## 2.2 流程
```
**MLIP-MD loop**
[訓練階段，離線完成]
    用 DFT 計算大量 {R_I} → E_DFT, F_I 的訓練集
    訓練 ML 模型：V_ML({R_I}) ≈ E_DFT({R_I})

[推理階段，即 MD 模擬]
初始條件：{R_I(0), v_I(0)}
    ↓
計算力：F_I = -∇V_ML({R_I})   ← 用訓練好的 ML，比 DFT 快幾個數量級
    ↓
Newton 積分（完全相同）
    ↓
重複
```

## 2.3 與其他計算方法之間的差異


> 1. 它的尺度不會因為是 DFT 而受限
> 2. **在訓練數據覆蓋的構型空間內**，精度接近 DFT。如果遇到訓練數據沒有覆蓋到的構型（例如極端溫度、新的化學環境），MLIP 可能會失準。這是 MLIP 目前的主要限制之一，也是 active learning 方法存在的原因。

| |AIMD|MLIP-MD|
|---|---|---|
|**力從哪來**|每步即時跑 DFT|用訓練好的 ML model 預測|
|**DFT 的角色**|即時計算（online）|離線生成訓練數據（offline）|
|**模擬時 DFT 還在跑嗎**|是|否|
|**尺度限制**|受 DFT 成本限制|不受限|

MLIP 的邏輯是：先用 AIMD（或靜態 DFT 計算很多不同構型）得到訓練數據，然後把這個 DFT 的「知識」編碼進 ML model，之後模擬時完全不再呼叫 DFT。




