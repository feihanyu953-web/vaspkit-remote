# vaspkit-remote

**通过 cluster-connect 远程驱动集群 VASPKIT 的 Claude Code skill。** 零代码——纯「VASPKIT 知识层 + 远程调用模板」，解决「VASPKIT 装在集群、Claude Code 跑在本地」的鸿沟。前置依赖 [cluster-connect](https://github.com/)（同系列开源 skill）。

## 特性

- **200+ VASPKIT task 完整映射**（`references/vaspkit_tasks.md`，含前提文件标注与 1.3.x/1.5.x 版本差异）
- **双模式适配**：1.3.x 与 1.5.x 均支持命令行 flag 模式（`vaspkit -task XXX`）；交互式任务（需回答参数）用 printf 管道
- **前置检查清单**：缺 PROCAR / CHGCAR / LOCPOT 时提前告知，避免白跑
- **已知坑固化**：1.5.1 MSD 全 NaN、1.5.1 菜单 84 不可用、AUTO_PLOT 不出图、task 102 覆盖 INCAR 等，均已写明对策
- **零新增代码**：执行层、沙箱、确认机制全部复用 cluster-connect

## 前置依赖

| 依赖 | 用途 | 安装 |
|---|---|---|
| **cluster-connect skill** | SSH 执行、文件拉取、作业管理 | 见其仓库 README |
| **VASPKIT ≥ 1.3.5**（集群上） | 后处理本体 | `module load` 或 PATH |

## 安装

```powershell
# 全局安装（推荐，所有项目可用）
Copy-Item -Recurse vaspkit-remote $HOME/.claude/skills/

# 或项目级安装
Copy-Item -Recurse vaspkit-remote <your-project>/.claude/skills/
```

安装后重启 Claude Code 即可自动发现。

## 快速开始

**1. 确认 vaspkit 版本**（决定调用模式）：

```powershell
python <cc-dir>/scripts/cluster.py exec "which vaspkit; vaspkit --version 2>&1 | head -3"
```

- 读 banner 中的版本字符串判断版本（`1.3.5 (03 Jul. 2022)` vs `1.5.1 (27 Jan. 2024)`）——"Running VASPKIT Under Command-Line Mode" 在 1.3.x/1.5.x 的 flag 模式下都会打印，不能用作版本判据
- 1.3.x 与 1.5.x 都支持 flag 模式（`vaspkit -task XXX`）

**2. 画总能带（示例，flag 模式）**：

```powershell
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && vaspkit -task 211"
python <cc-dir>/scripts/cluster.py pull <calc-dir>/BAND.dat ./out/
python <cc-dir>/scripts/cluster.py pull <calc-dir>/BAND.png ./out/
```

**3. 1.3.x 用管道**：

```powershell
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && printf '6\n601\n0\n0\n' | vaspkit"
```

> `<cc-dir>` = cluster-connect 安装目录；`<calc-dir>` = 计算目录在集群 `root_dir` 内的路径；多集群加 `--profile <name>`。

## 支持的后处理能力

| 类别 | 代表 task |
|---|---|
| 电子结构 | 能带（PBE/杂化/投影/反折叠）、DOS（总/投影/轨道）、费米面、3D 能带 |
| 电荷/势 | 电荷密度、差分电荷、平面平均电荷、功函数、宏观平均势 |
| MD 轨迹 | MSD、扩散系数、RDF、配位数、速度自关联、振动 DOS、键长键角 |
| 催化/热力学 | 吸附质热修正、气体热修正、d 带中心、NEB 分析、虚频修正 |
| 结构 | 对称性、原胞、超胞、slab、缺陷 |
| 半导体/2D | 带隙、有效质量、Fermi 速度、真空能级校准 |
| 光学/输运 | 光学光谱、Boltzmann 输运 |
| 力学/磁性 | 弹性常数、EOS、磁各向异性能 |

完整 task 表见 `references/vaspkit_tasks.md`。

## 已知限制

- 依赖 cluster-connect 的 `root_dir` 沙箱：计算目录必须在沙箱内才能 pull 结果
- vaspkit 1.5.1 的 MSD task 对 VASP 6 的 XDATCAR 输出全 NaN（vaspkit bug，非本 skill 问题）——建议本地 Python 计算（示例见下）
- printf 管道序列因 task 而异，不确定时先无管道跑一次看提示
- 绘图依赖集群 `~/.vaspkit` 的 AUTO_PLOT / PYTHON_BIN / 字体配置；未配置时只出数据文件

### MSD 替代方案（vaspkit 1.5.1 全 NaN 时）

用本地 Python 对 XDATCAR 做 PBC-unwrap 计算（numpy 即可）：

```python
import numpy as np
lines = open("XDATCAR").read().splitlines()
lat = np.array([[float(v) for v in lines[i].split()] for i in range(2, 5)]) * float(lines[1])
frac = []
i = 7
while i < len(lines):
    if "configuration" in lines[i] or "Direct" in lines[i]:
        i += 1
        fr = []
        for j in range(sum(int(x) for x in lines[6].split())):
            fr.append([float(v) for v in lines[i + j].split()[:3]])
        frac.append(fr); i += len(fr)
frac = np.array(frac)
d = np.diff(frac, axis=0); d -= np.round(d)          # PBC unwrap
uw = np.concatenate([frac[:1], frac[:1] + np.cumsum(d, axis=0)], axis=0)
cart = uw @ lat
msd = np.mean(np.sum((cart - cart[0])**2, axis=2), axis=1)  # A^2
t = np.arange(len(msd)) * 0.5 / 1000.0               # ps (POTIM=0.5 fs 时)
```

## License

MIT。知识层移植自 [2931798734-commits/vasp-kit-skill](https://github.com/2931798734-commits/vasp-kit-skill)（MIT，已保留致谢）。

## Acknowledgements

- [2931798734-commits/vasp-kit-skill](https://github.com/2931798734-commits/vasp-kit-skill) — task 映射表与交互规范的源头（MIT）
- [cluster-connect](https://github.com/) — 本仓库姊妹项目，提供 SSH 执行层
- VASPKIT：V. Wang et al., *Comput. Phys. Commun.* 267, 108033 (2021)
