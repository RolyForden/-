# FocalAfford Evidence

只记录会影响当前实验协议或论文主张的证据。idea 中的机制解释仍是待验证假设，不计作实验事实。

| 结论或假设 | 核验范围 | 原始来源 | 与当前判断的关系 |
|---|---|---|---|
| GEAL 公开渲染路径使用 12 个固定视角 | 官方代码，固定到 commit `3048bb8c859b04b3d7569e2bc186f9880c7109ab` | `renderer/gaussian_render.py`，<https://github.com/DylanOrange/geal> | 支持把“局部重渲染”定义为对 GEAL 固定全局视角的可检验扩展 |
| GEAL 官方训练环境为 Python 3.10、CUDA 11.8、PyTorch 2.1.0 | 官方 README | <https://github.com/DylanOrange/geal> | 固定第一轮执行环境，减少非方法变量 |
| LASO 官方数据入口公开，GEAL 预期包含 train/val/test 的 annotation 与 object 文件 | LASO 与 GEAL 官方 README | <https://github.com/yl3800/LASO>；<https://github.com/DylanOrange/geal> | 支持使用 train/val 做开发并封存 test |
| FocalAfford 的三个核心增量有效 | 尚未验证 | 无实验结果 | 当前不能声称成立 |

## 尚缺的证据

- A0-A4 在固定 LASO train/val 协议下的逐区域对照。
- 局部教师收益是否能迁移到三维学生。
- 优势加权局部蒸馏是否超过直接蒸馏与相同区域 GT 重加权。
