# VASPKIT Complete Task Reference

## Command-Line Syntax

```bash
vaspkit -task <task-id> [options]
```

| Option | Description |
|--------|-------------|
| `-task <ID>` | 任务编号 |
| `-kpr <value>` | K 点间距（Å⁻¹） |
| `-kmesh <nx> <ny> <nz>` | 显式 K 网格 |
| `-file <path>` | 指定输入文件路径 |
| `-bin` / `-std` | 二进制/标准输出格式 |

---

## 版本差异速览（1.3.x vs 1.5.x）

本 task 表按 **VASPKIT 1.5.x** 编号编写（169 集群）。**167 集群跑 1.3.5**，以下 task 在 1.3.x 上**不存在**（执行会报错/无此菜单项）：

- 菜单 06：604
- 菜单 07：整组（702/705）；菜单 44/84/95：整个菜单无
- 菜单 08：800 / 821 / 822 / 825 / 826 / 827
- 菜单 11：110 / 123 / 124 / 125 / 126
- 菜单 31：310 / 328 / 329（1.3.x 的 318=xsf 导出、319=cube 导出）
- 菜单 42：420 / 422（1.3.x 的 425=2D 线性平均势）
- 菜单 62：629；菜单 65：653；菜单 72：730 / 731；菜单 92：926

1.3.x 与 1.5.x 在菜单 01/02/03/05/06(601-603,608,609)/21/23/25/26/28/51/65(651,652)/68/71/72(721-728)/78/91 上的 task 编号一致。

---

## 01 — VASP 输入文件生成器

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 101 | 定制 INCAR 文件（交互式） | 无 |
| 102 | 生成 SCF KPOINTS（MP 网格） | POSCAR |
| 103 | 生成 POTCAR（推荐赝势） | POSCAR |
| 104 | 生成 POTCAR（指定赝势） | POSCAR |
| 105 | 从 CIF 生成 POSCAR | .cif 文件 |
| 106 | 从 Materials Studio XSD 生成 POSCAR | .xsd 文件 |
| 107 | 按指定元素顺序重排 POSCAR | POSCAR |
| 108 | 一键生成全部 VASP 文件 + 检查 | POSCAR |
| 109 | 提交作业队列 | - |

---

## 02 — 力学性质

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 200 | 弹性常数 — 应力-应变法 | OUTCAR |
| 201 | 弹性常数 — 能量-应变法 | OUTCAR |
| 202 | 力学性质（从 ELASTIC_TENSOR 文件） | ELASTIC_TENSER(_2D).in |
| 203 | 弹性常数 + 力学性质（从 OUTCAR） | OUTCAR |
| 204 | 空间依赖力学性质 | ELASTIC_TENSER(_2D).in |
| 205 | 状态方程拟合 | 能量-体积数据 |

---

## 03 — 能带 K 路径生成

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 301 | 1D 结构 K 路径 | POSCAR |
| 302 | 2D 结构 K 路径 | POSCAR |
| 303 | 3D（块体）结构 K 路径 | POSCAR |
| 304 | Wannier90 K 路径 | POSCAR |
| 305 | Phonopy K 路径 | POSCAR |
| 306 | CP2K K 路径 | POSCAR |
| 309 | 在第一布里渊区可视化 K 路径 | POSCAR |

---

## 04 — 结构编辑器（编号随版本不同，注意区分）

**1.5.x 编号（实测 1.5.1）：**

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 400 | 重定义晶格 | POSCAR |
| 401 | 构建超胞 | POSCAR |
| 402 | 固定选定原子 | POSCAR |
| 403 | 移动选定原子 | POSCAR |
| 404 | 删除选定原子 | POSCAR |
| 405 | 交换晶格矢量轴 | POSCAR |
| 406 | 按指定方向排序原子坐标 | POSCAR |
| 407 | 旋转晶格矢量 | POSCAR |
| 408 | 绕指定轴旋转选定原子 | POSCAR |
| 409 | 在指定位置添加原子 | POSCAR |
| 410 | 替换选定原子 | POSCAR |
| 411 | 从另一结构文件复制选定原子 | POSCAR + 源文件 |
| 412 | 对选定原子施加随机位移 | POSCAR |
| 413 | 分数坐标与笛卡尔坐标转换 | POSCAR |
| 414 | 消除优化后的虚假晶格畸变 | POSCAR |
| 415 | 指定平面镜像变换 | POSCAR |
| 419 | 导出其他结构格式 | POSCAR |

**1.3.x 编号不同（实测 1.3.5，注意错位）：** 407=随机位移、408=旋转晶格矢量、409=消除畸变、410=导出 CP2K（排在最末）、411=笛卡尔坐标、412=分数坐标、413=CIF、414=ATAT、415=XCrySDen (.xsf)、416=Quantum-Espresso、417=Elk、418=Siesta、419=PDB；**无独立的 XYZ 导出**。

---

## 05 — 催化/电化学工具包

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 501 | 吸附质热修正 | OUTCAR |
| 502 | 气体分子热修正 | OUTCAR |
| 503 | 能带中心（实验性） | - |
| 504 | NEB 路径转 PDB 动画 | NEB 相关文件 |
| 505 | NEB 镜像线性插值 | - |
| 507 | 虚频修正 | - |
| 508 | Bader2PQR（VMD 可视化） | Bader 输出 |
| 509 | 一级反应半衰期评估 | - |

---

## 06 — 对称性分析

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 601 | 寻找晶体对称性（空间群） | POSCAR |
| 602 | 寻找原胞 | POSCAR |
| 603 | 寻找标准惯用胞 | POSCAR |
| 604 | 寻找对称等价原子（**仅 1.5.x**，1.3.x 无此任务） | POSCAR |
| 608 | 寻找弛豫后结构对称性 | CONTCAR |
| 609 | 寻找分子或团簇对称性 | POSCAR |

---

## 07 — 材料数据库（仅 1.5.x，1.3.x 无此菜单）

| 任务 | 功能 |
|:-----|:-----|
| 702 | 计算 2D 单层半导体数据库 |
| 705 | 计算 2D 半导体异质结数据库 |

---

## 08 — 高级结构模型

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 800 | 构建正交超胞 | POSCAR |
| 801 | 沿指定方向构建真空层 Slab | POSCAR |
| 802 | 构建随机替代合金 | POSCAR |
| 803 | 按 Miller 指数构建表面 | POSCAR |
| 804 | 从两个 Slab 构建异质结 | 两个 POSCAR |
| 806 | 按指定半径构建量子点 | POSCAR |
| 807 | 按指定半径构建纳米线/2D 量子点 | POSCAR |
| 808 | 从正交 Slab 构建纳米管 | POSCAR |
| 821 | 构建超胞中非等效空位缺陷 | POSCAR |
| 822 | 构建超胞中非等效替代缺陷 | POSCAR |
| 825 | 构建六角 Moiré 超晶格 | POSCAR |
| 826 | 按手性矢量构建纳米带 | POSCAR |
| 827 | 按手性矢量构建纳米管 | POSCAR |

> 1.3.x 编号不同：无 800/806/807/821/822/825/826/827；1.3.x 的 805/806/807 是 Pro-Only 的 Moiré 超晶格/纳米带/纳米管（对应 1.5.x 的 825/826/827）。

---

## 11 — 态密度 (DOS)

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 110 | 逆参与比 Inverse Participation Ratio（**仅 1.5.x**） | DOSCAR |
| 111 | 总 DOS | DOSCAR, EIGENVAL |
| 112 | 单个选定原子的投影 DOS | DOSCAR, EIGENVAL, PROCAR |
| 113 | 各元素投影 DOS | DOSCAR, EIGENVAL, PROCAR |
| 114 | 选定原子的投影 DOS | DOSCAR, EIGENVAL, PROCAR |
| 115 | 选定原子+轨道的投影 DOS | DOSCAR, EIGENVAL, PROCAR |
| 116 | 各元素局域 DOS | DOSCAR, EIGENVAL |
| 117 | 从 EIGENVAL 计算总 DOS | EIGENVAL |
| 118 | 从 EIGENVAL + PROCAR 计算投影 DOS | EIGENVAL, PROCAR |
| 119 | 指定 K 指数投影 DOS | DOSCAR, EIGENVAL |
| 120 | 指定能带指数投影 DOS | DOSCAR, EIGENVAL |
| 123 | 3D 空间分辨 DOS（能量范围，**仅 1.5.x**） | - |
| 124 | 3D 空间分辨磁 DOS（**仅 1.5.x**） | - |
| 125 | 2D 平面平均空间分辨 DOS（**仅 1.5.x**） | - |
| 126 | 2D 平面平均空间分辨磁 DOS（**仅 1.5.x**） | - |

---

## 21 — 能带结构

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 211 | 能带结构（PBE） | EIGENVAL, DOSCAR |
| 212 | 单个选定原子的投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 213 | 各元素投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 214 | 选定原子的投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 215 | 按元素权重投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 216 | 选定原子+轨道的投影能带求和 | EIGENVAL, DOSCAR, PROCAR |

---

## 23 — 3D 能带结构

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 231 | 生成 3D 能带 KPOINTS | POSCAR |
| 232 | 2D 材料 3D 能带 | EIGENVAL, DOSCAR |
| 233 | HOMO/LUMO 能带 3D 能带 [仅 ISPIN=1] | EIGENVAL |

---

## 25 — 杂化 DFT 能带结构

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 250 | 生成 KPOINTS（不可约 K 网格 + 能带边） | POSCAR |
| 251 | 生成杂化能带 KPOINTS | POSCAR |
| 252 | 杂化 DFT 能带结构 | EIGENVAL, DOSCAR |
| 253 | 杂化 DFT 单原子投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 254 | 杂化 DFT 各元素投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 255 | 杂化 DFT 选定原子投影能带求和 | EIGENVAL, DOSCAR, PROCAR |
| 256 | 杂化 DFT 按元素权重投影能带 | EIGENVAL, DOSCAR, PROCAR |
| 257 | 杂化 DFT 原子+轨道投影能带求和 | EIGENVAL, DOSCAR, PROCAR |

---

## 26 — 费米面

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 261 | 生成费米面 KPOINTS | POSCAR |
| 262 | 费米面 — XCrySDen 格式 | EIGENVAL, DOSCAR |
| 263 | 费米面 — FermiSurfer 格式 | EIGENVAL, DOSCAR |
| 264 | 投影费米面 — FermiSurfer | EIGENVAL, DOSCAR, PROCAR |
| 265 | 选定原子+轨道投影费米面求和 | EIGENVAL, DOSCAR, PROCAR |
| 266 | 2D 材料费米面 | EIGENVAL, DOSCAR |
| 267 | 2D 材料投影费米面 | EIGENVAL, DOSCAR, PROCAR |

---

## 28 — 能带反折叠

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 281 | 生成能带反折叠 KPOINTS | POSCAR |
| 282 | 有效能带结构 (EBS) | EIGENVAL |
| 283 | 选定原子轨道投影 EBS | EIGENVAL, PROCAR |
| 284 | 各元素轨道投影 EBS | EIGENVAL, PROCAR |
| 285 | 选定原子+轨道投影 EBS 求和 | EIGENVAL, PROCAR |

---

## 31 — 电荷密度分析

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 310 | 2D 电荷密度切片 | CHGCAR |
| 311 | 电荷密度 | CHGCAR |
| 312 | 自旋密度 | CHGCAR |
| 313 | 自旋向上/向下密度 | CHGCAR |
| 314 | 电荷密度差 | CHGCAR |
| 315 | 2D 线性平均电荷密度 | CHGCAR |
| 316 | 1D 平面平均电荷密度 | CHGCAR |
| 317 | 1D 宏观平均电荷密度 | CHGCAR |
| 318 | 沿指定路径 1D 电荷密度 | CHGCAR |
| 320 | 通过变换矩阵构建电荷密度超胞 | CHGCAR |
| 325 | STM 模拟 | CHGCAR |
| 328 | CHGCAR/PARCHG → XCrySDen .xsf（**仅 1.5.x**） | CHGCAR |
| 329 | CHGCAR/PARCHG → Gaussian .cube（**仅 1.5.x**） | CHGCAR |

> 1.3.x 菜单 31 无 310/328/329；且编号错位：1.3.x 的 318=导出 XCrySDen (.xsf)、319=导出 Gaussian (.cube)。

---

## 42 — 势分析

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 420 | 2D 势切片（**仅 1.5.x**） | LOCPOT |
| 422 | 2D 线性平均势（**仅 1.5.x**；1.3.x 对应任务在 425） | LOCPOT |
| 425 | 沿指定路径 1D 势 | LOCPOT |
| 426 | 1D 平面平均势（功函数） | LOCPOT |
| 427 | 1D 宏观平均势 | LOCPOT |
| 428 | LOCPOT/ELFCAR → XCrySDen .xsf | LOCPOT |
| 429 | LOCPOT/ELFCAR → Gaussian .cube | LOCPOT |
| 430 | 通过变换矩阵构建势超胞 | LOCPOT |

---

## 44 — 压电性质（仅 1.5.x）

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 442 | 压电与介电常数（从 OUTCAR） | OUTCAR |
| 443 | 压电性质 | - |

---

## 51 — 波函数分析

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 511 | 实空间波函数（CHGCAR 格式） | WAVECAR |
| 512 | 实空间波函数（Cube 格式） | WAVECAR |
| 513 | 波函数宇称（实验性） | WAVECAR |
| 515 | 实空间 |ψ|²（CHGCAR 格式） | WAVECAR |
| 516 | 实空间 |ψ|²（Cube 格式） | WAVECAR |

---

## 62 — 磁性质

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 621 | 磁各向异性能 (MAE) | OUTCAR |
| 629 | 可视化磁矩矢量（**仅 1.5.x**） | OUTCAR |

---

## 65 — 自旋纹理

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 651 | 生成自旋纹理 KPOINTS | POSCAR |
| 652 | 2D 材料能带自旋纹理（仅 SOC） | EIGENVAL, DOSCAR |
| 653 | 块体材料能带自旋纹理（仅 SOC，**仅 1.5.x**；1.3.x 无此任务） | EIGENVAL, DOSCAR |

---

## 68 — 输运性质

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 681 | 生成输运计算 KPOINTS | POSCAR |
| 682 | 输运性质（Boltzmann 理论） | EIGENVAL, DOSCAR |

---

## 71 — 光学性质

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 710 | 线性光谱 — 2D 半导体 | WAVECAR, WAVEDER |
| 711 | 线性光谱 — 块体半导体 | WAVECAR, WAVEDER |
| 713 | 跃迁偶极矩（从 WAVECAR） | WAVECAR |
| 714 | 偶极矩矩阵元（从 WAVEDER） | WAVEDER |
| 716 | 总联合态密度 | WAVECAR, WAVEDER |
| 717 | 部分联合态密度 | WAVECAR, WAVEDER |
| 719 | 光谱极限最大效率 (SLME) | - |

---

## 72 — 分子动力学工具包

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 721 | 均方位移 | XDATCAR |
| 722 | 均方位移（FFT，推荐） | XDATCAR |
| 723 | 扩散系数 + 离子迁移率 | MSD.dat |
| 725 | 对关联函数 | PCDAT |
| 726 | 选定元素径向分布函数 | PCDAT |
| 727 | 速度自关联函数 | XDATCAR |
| 728 | 振动 DOS（从速度自关联） | XDATCAR |
| 730 | 选定元素键长分布（**仅 1.5.x**） | XDATCAR |
| 731 | 选定元素键角分布（**仅 1.5.x**） | XDATCAR |
| 736 | 选定原子 MD 轨迹（POSCAR 格式） | XDATCAR |
| 737 | 选定原子 MD 轨迹（PDB 格式） | XDATCAR |

---

## 78 — VASP ↔ 其他接口

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 781 | VASP → BoltzTraP 接口 | EIGENVAL, DOSCAR |
| 788 | 原子间力常数张量归一化迹 | - |
| 789 | 为 Phonopy 排序声子能带 | - |

---

## 84 — ABACUS 接口（仅 1.5.x；⚠️ 1.5.1 上不可用）

主菜单有 84) ABACUS Interface 一项，但 **1.5.1 实际进入报 "Unsupported Parameter! Try Again"，无任务可用**（实测）。若需 ABACUS 互转，检查更新版本 VASPKIT。

---

## 91 — 半导体工具包

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 911 | 带隙（VBM/CBM 分析） | EIGENVAL, DOSCAR |
| 912 | 载流子有效质量（实验性） | EIGENVAL |
| 913 | HOMO/LUMO 能带有效质量 [仅 ISPIN=1] | EIGENVAL |
| 914 | 角度依赖有效质量 + Fermi 速度 | EIGENVAL |
| 917 | Fermi 速度 | EIGENVAL |

---

## 92 — 2D 材料工具包

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 920 | 将原子层移至 z 方向底部 | POSCAR |
| 921 | 将原子层沿 z 方向居中 | POSCAR |
| 922 | 调整真空层厚度 | POSCAR |
| 923 | 标准化 2D 晶体胞 | POSCAR |
| 926 | 堆叠依赖势能面（**仅 1.5.x**） | POSCAR |
| 927 | 参考真空能级的能带边（2D） | EIGENVAL, LOCPOT |
| 929 | 弛豫 2D 结构汇总 | OUTCAR |

---

## 95 — 声子分析（仅 1.5.x）

| 任务 | 功能 | 前提文件 |
|:-----|:-----|:---------|
| 952 | 声子色散（从 OUTCAR） | OUTCAR |
| 953 | 各元素投影声子色散 | OUTCAR |
| 954 | 选定原子投影声子色散 | OUTCAR |
| 955 | 按元素权重投影声子色散 | OUTCAR |

---

## 常用命令速查

```bash
# === 输入文件生成 ===
vaspkit -task 101              # 生成 INCAR
vaspkit -task 102 -kpr 0.03    # 生成 KPOINTS（SCF）
vaspkit -task 103              # 生成 POTCAR
vaspkit -task 108              # 一键生成全部输入文件

# === K 路径（能带计算前） ===
vaspkit -task 303              # 3D 材料 K 路径 → KPATH.in
vaspkit -task 302              # 2D 材料 K 路径 → KPATH.in

# === 能带（计算后） ===
vaspkit -task 211              # 绘制能带
vaspkit -task 213              # 绘制投影能带（按元素）
vaspkit -task 214              # 绘制投影能带（按原子）

# === DOS（计算后） ===
vaspkit -task 111              # 总 DOS
vaspkit -task 113              # 投影 DOS（按元素）
vaspkit -task 114              # 投影 DOS（按原子）

# === 结构操作 ===
vaspkit -task 401              # 构建超胞
vaspkit -task 601              # 对称性分析
vaspkit -task 602              # 找原胞（标准化）
vaspkit -task 803              # 构建表面（Miller 指数）

# === 分析 ===
vaspkit -task 911              # 带隙分析（VBM/CBM）
vaspkit -task 426              # 功函数/平面平均势
vaspkit -task 203              # 弹性常数（从 OUTCAR）
vaspkit -task 310              # 2D 电荷密度切片（仅 1.5.x，1.3.x 无此任务）
```
