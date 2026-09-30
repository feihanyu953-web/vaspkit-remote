# vaspkit-remote

**远程驱动集群 VASPKIT 做结果后处理的 Claude Code skill。** 零代码——纯「VASPKIT 知识层 + 远程调用模板」，解决「VASPKIT 装在集群、Claude Code 跑在本地」的鸿沟。执行层复用 [cluster-connect](https://github.com/feihanyu953-web/cluster-connect)（同系列 skill，提供 SSH `exec` / `pull`）。

## 支持的后处理能力（200+ task）

| 类别 | 代表 task | 前提文件 |
|---|---|---|
| 电子结构 | 总能带 211（PBE）、投影能带 213/214/216、能带反折叠 282、总 DOS 111、投影 DOS 113/114/115、费米面、3D 能带 | EIGENVAL / DOSCAR（投影需 PROCAR） |
| 电荷 / 势 | 电荷密度 311、差分电荷 314、功函数 / 平面平均势 426 | CHGCAR / LOCPOT |
| MD 轨迹 | MSD / 扩散系数 721/722/723、RDF / 对关联 725/726、速度自关联、振动 DOS、键长键角 | XDATCAR / PCDAT |
| 催化 / 热力学 | 吸附质热修正 501、气体热修正 502、d 带中心 503、NEB 分析、虚频修正 | OUTCAR |
| 结构 / 对称性 | 空间群 601、原胞、超胞、slab、缺陷 | POSCAR |
| 半导体 / 2D | 带隙 911（VBM/CBM）、有效质量、Fermi 速度、真空能级校准 | EIGENVAL / DOSCAR |
| 光学 / 输运 | 光学光谱、Boltzmann 输运 | — |
| 力学 / 磁性 | 弹性常数、EOS、磁各向异性能 | — |

完整 task 表（含参数与版本差异）见 `references/vaspkit_tasks.md`。

## 特性

- **200+ VASPKIT task 完整映射**，含前提文件标注与 1.3.x/1.5.x 版本差异
- **双模式适配**：读 banner 版本串判别 1.3.x / 1.5.x（注意："Running VASPKIT Under Command-Line Mode" 在两个版本的 flag 模式下都会打印，**不能**用作版本判据）；flag 模式 `vaspkit -task XXX` 与 printf 管道交互（`printf '6\n601\n0\n0\n' | vaspkit`）均支持
- **前置检查清单**：缺 PROCAR / CHGCAR 0 字节 / 缺 LOCPOT 时提前告知，避免白跑
- **已知坑固化**（写明对策）：1.5.1 MSD task 对 VASP 6 的 XDATCAR 全 NaN、菜单 84 不可用、`~/.vaspkit` AUTO_PLOT/PYTHON_BIN/字体未配置时只出 .dat 不出 .png 且静默、printf 序列不完整触发 forrtl severe (24)（已生成文件不受影响）、task 102 交互模式会改写 INCAR（执行前自动备份 INCAR.bak）
- **零新增代码**：执行层、沙箱、确认机制全部复用 cluster-connect

## 前置依赖

| 依赖 | 用途 | 安装 |
|---|---|---|
| **cluster-connect skill** | SSH 执行、文件拉取、沙箱 | 见其仓库 README |
| **VASPKIT ≥ 1.3.5**（集群上） | 后处理本体 | `module load` 或 PATH |

## 安装

```powershell
# 用户级（推荐，所有项目可用）
Copy-Item -Recurse vaspkit-remote $HOME/.claude/skills/
# 或项目级
Copy-Item -Recurse vaspkit-remote <your-project>/.claude/skills/
```

安装后重启 Claude Code 即可自动发现。

## 快速开始

```powershell
# 1. 确认 vaspkit 版本（决定调用模式）
python <cc-dir>/scripts/cluster.py exec "which vaspkit; vaspkit --version 2>&1 | head -3"

# 2. 画总能带（flag 模式）
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && vaspkit -task 211"
python <cc-dir>/scripts/cluster.py pull <calc-dir>/BAND.dat ./out/
python <cc-dir>/scripts/cluster.py pull <calc-dir>/BAND.png ./out/

# 3. 需要参数的任务用 printf 管道
python <cc-dir>/scripts/cluster.py exec "cd <calc-dir> && printf '6\n601\n0\n0\n' | vaspkit"
```

> `<cc-dir>` = cluster-connect 安装目录；`<calc-dir>` = 计算目录在集群 `root_dir` 沙箱内的路径；多集群加 `--profile <name>`。

权限约定：vaspkit 命令展示后可直接执行；`sbatch` 提交、文件删除/覆盖必须先经用户确认。

## 已知限制

- 计算目录必须在 cluster-connect 的 `root_dir` 沙箱内，否则 pull 不到结果
- printf 管道序列因 task 而异，不确定时先无管道跑一次看提示
- 绘图依赖集群 `~/.vaspkit` 的 AUTO_PLOT / PYTHON_BIN / 字体配置；未配置时只出数据文件

### MSD 替代方案（vaspkit 1.5.1 全 NaN 时）

vaspkit 1.5.1 的 MSD task（721/722）对 VASP 6 写出的 XDATCAR 输出全 NaN（vaspkit bug，非本 skill 问题）。用本地 Python 对 XDATCAR 做 PBC-unwrap 计算（numpy 即可）：

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

更完整的轨迹分析（RDF/氢键/扩散系数等）建议配合姊妹项目 [lammps-analyze](https://github.com/feihanyu953-web/lammps-analyze) 思路本地处理。

## 生态位

| 环节 | 工具 |
|---|---|
| 建模（slab / 异质结 / 电解质盒） | [emcbuild](https://github.com/feihanyu953-web/emcbuild) |
| 输入文件生成与校验 | [input_build](https://github.com/feihanyu953-web/input_build) |
| 集群提交 / 传输（执行层） | [cluster-connect](https://github.com/feihanyu953-web/cluster-connect) |
| **集群侧 VASP 后处理** | **本项目** |
| MD 轨迹本地分析 | [lammps-analyze](https://github.com/feihanyu953-web/lammps-analyze) |

## License

MIT。知识层移植自 [2931798734-commits/vasp-kit-skill](https://github.com/2931798734-commits/vasp-kit-skill)（MIT，已保留致谢）。

## Acknowledgements

- [2931798734-commits/vasp-kit-skill](https://github.com/2931798734-commits/vasp-kit-skill) — task 映射表与交互规范的源头（MIT）
- [cluster-connect](https://github.com/feihanyu953-web/cluster-connect) — 本仓库姊妹项目，提供 SSH 执行层
- VASPKIT：V. Wang et al., *Comput. Phys. Commun.* 267, 108033 (2021)
