---
name: vaspkit-remote
description: 通过 cluster-connect 远程驱动集群上的 VASPKIT 执行 VASP 结果后处理（能带/DOS/投影/电荷密度/功函数/带隙/MSD/RDF/催化热力学修正等 200+ task）。前置依赖 cluster-connect skill。当用户需要对 VASP 计算结果做后处理分析（画能带/DOS 图、提取功函数/带隙、算 MSD/RDF/扩散系数、吸附质热修正）时使用。
---

# vaspkit-remote

通过 **cluster-connect** 的 `exec` / `pull` 在集群上远程驱动 **VASPKIT** 做 VASP 后处理。本 skill 不含代码——是「VASPKIT 知识层 + 远程调用模板」，零新增依赖，执行层完全复用 cluster-connect。

## 前置依赖（必需）

1. **cluster-connect skill**（执行层）——提供 `cluster.py exec/pull/submit`，安装与配置见其 README
2. 集群已安装 **VASPKIT**（≥ 1.3.5），且经 cluster-connect 配置好 SSH profile
3. 目标 VASP 计算目录位于 cluster-connect 的 `root_dir` 沙箱内

约定：`<cc-dir>` = cluster-connect 的安装目录（全局 `~/.claude/skills/cluster-connect/` 或项目 `.claude/skills/cluster-connect/`）。

## 第一步：确认 vaspkit 版本（决定调用模式）

```powershell
python <cc-dir>/scripts/cluster.py exec "which vaspkit; vaspkit --version 2>&1 | head -3"
```

| 版本 | 输出特征 | 调用模式 |
|---|---|---|
| **1.5.x** | 显示 "Running VASPKIT Under Command-Line Mode" | **flag 模式**：`vaspkit -task XXX` |
| **1.3.x** | 直接进入交互菜单 | **printf 管道**：`printf '...' \| vaspkit` |

> ⚠️ 1.3.x 的 `-task XXX -xxx` 命令行 flag 会**静默失败**（退出码 0 但不读取 POSCAR、不生成任何文件），必须用管道模式。

## 调用模板

### 1.5.x 节点 — flag 模式

```powershell
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && vaspkit -task 601"
```

### 1.3.x 节点 — printf 管道

```powershell
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && printf '6\n601\n0\n0\n' | vaspkit"
```

### 多集群 / 非默认 profile

```powershell
python <cc-dir>/scripts/cluster.py --profile <name> exec "cd <calc-dir> && vaspkit -task 111"
```

## 自然语言 → task 映射

完整 200+ task 表见 `references/vaspkit_tasks.md`。常用速查：

| 需求 | task | 前提文件 |
|---|---|---|
| 对称性 / 空间群 | 601 | POSCAR |
| 总能带（PBE） | 211 | EIGENVAL, DOSCAR |
| 投影能带（元素/原子/轨道） | 213 / 214 / 216 | + PROCAR |
| 总 DOS | 111 | DOSCAR, EIGENVAL |
| 投影 DOS（元素/原子/轨道） | 113 / 114 / 115 | + PROCAR |
| 能带反折叠（EBS） | 282 | EIGENVAL |
| 电荷密度 / 差分电荷 | 311 / 314 | CHGCAR |
| 功函数 / 平面平均势 | 426 | LOCPOT |
| 带隙（VBM/CBM 分析） | 911 | EIGENVAL, DOSCAR |
| MSD / 扩散系数 | 721 / 722 / 723 | XDATCAR → MSD.dat |
| RDF / 对关联函数 | 725 / 726 | PCDAT |
| 吸附质 / 气体热修正 | 501 / 502 | OUTCAR |
| d 带中心 | 503 | — |

## printf 管道序列规范（1.3.x）

1. 序列 = 主菜单数字 + task 编号 + 参数回答 + 退出选择，每项用 `\n` 分隔
2. 已实测示例：`printf '6\n601\n0\n0\n' | vaspkit`（对称性分析，主菜单 6 → task 601 → 确认 → 退出）
3. 序列不确定时：先不带管道跑一次（会停在交互），根据屏幕提示构造管道输入
4. 交互任务执行前**先告知用户** VASPKIT 会问哪些问题、计划如何回答
5. 执行后检查产物（`ls *.dat *.png`）；若 3 秒内无预期输出，停止并检查 POSCAR 格式与前提文件

## 前置检查（执行前）

```powershell
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && ls EIGENVAL DOSCAR PROCAR CHGCAR LOCPOT PCDAT XDATCAR 2>&1"
```

- **缺 PROCAR** → 投影 DOS/能带做不了（重算时 INCAR 需 `LORBIT=11`）
- **CHGCAR 为 0 字节** → 电荷分析做不了（需 LSF 重算出 CHGCAR）
- **缺 LOCPOT** → 功函数做不了（重算时 INCAR 需 `LVHAR=.TRUE.`）
- **绘图依赖**集群 `~/.vaspkit` 配置：`AUTO_PLOT`、`PYTHON_BIN` 可执行、字体可用；不满足时 vaspkit **只出 `.dat` 不出 `.png`**（静默跳过，不报错）

## 结果拉取

```powershell
python <cc-dir>/scripts/cluster.py pull <calc-dir>/BAND.dat ./out/
python <cc-dir>/scripts/cluster.py pull <calc-dir>/BAND.png ./out/
```

Windows 无 rsync 时 cluster-connect 自动降级 scp，无需额外操作。

## 已知坑（实测）

| 坑 | 现象 | 对策 |
|---|---|---|
| 1.3.x 命令行 flag 静默失败 | 退出码 0、无文件生成 | 一律 printf 管道 |
| vaspkit 1.5.1 的 MSD task（721/722）对 VASP 6 写的 XDATCAR 全 NaN | MSD.dat 全 NaN 且不报错 | 用本地 Python PBC-unwrap 替代（README 附示例） |
| task 111 缺 PROCAR 报错退出 | 总 DOS 理论上不需要 PROCAR 仍报错 | 改用 task 211 间接获取费米能级 |
| 全原子 RDF 峰归属模糊 | 0.94 Å（O-H）、2.69 Å（氢键/K-O 混合） | 元素分辨 RDF（726 或自算） |
| task 102 交互模式覆盖 INCAR | INCAR 被改写 | 执行前自动备份 `INCAR.bak` |

## 权限与安全

- vaspkit 命令：**执行前展示命令**，可直接执行
- `sbatch` 提交 / 文件删除覆盖：**必须用户确认**（复用 cluster-connect 的确认机制）
- 所有远程路径受 cluster-connect `root_dir` 沙箱约束；vaspkit 产物写入计算目录属正常操作

## Acknowledgements

- 知识层（task 映射表、交互规范、检查清单）移植自 [2931798734-commits/vasp-kit-skill](https://github.com/2931798734-commits/vasp-kit-skill)（MIT）
- 执行层复用 [cluster-connect](https://github.com/)（MIT，本仓库姊妹项目）
- VASPKIT：V. Wang et al., *Comput. Phys. Commun.* 267, 108033 (2021)
