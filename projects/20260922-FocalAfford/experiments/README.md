# FocalAfford 快速可行性实验执行计划

> **给执行 agent：**按本文顺序逐项执行。允许修改实验代码和配置，但不得跳过数据划分保护、对照组或失败结果。执行者不得向本仓库 commit/push；最终只回传本文第 14 节规定的交付包，由主工作区审查后提交。

项目状态入口：[`../README.md`](../README.md)

方案来源：[`../idea/FocalAfford_final-idea-report.pdf`](../idea/FocalAfford_final-idea-report.pdf)

**目标：**用最小但有判别力的实验，回答“几何锚定局部观察、整物体任务语境、优势加权局部修正蒸馏是否分别提供增量”，并生成一份可审查的可行性结论文档。

**总体策略：**先在 LASO 的训练集和验证集完成教师级 A0-A4 对照；只有 Gate A 出现正信号，才进行最小学生级 B0-B4 对照。整个阶段不查看或用于决策 LASO test，不运行 PIAD、鲁棒性集、GLANCE、CMAT 或 AnyUp。

**技术栈：**Linux、Python 3.10、PyTorch 2.1.0、CUDA 11.8、GEAL 官方代码、LASO 官方数据、pytest、pandas/NumPy/SciPy。

**证据状态：**本文件是执行方案，不是实验结果。本文中的参数是首轮试验协议，不表示已经验证有效。

---

## 1. 最终必须回答的问题

最终报告只能根据实际运行结果回答以下四个问题：

1. `Q1 局部细节`：高分辨率几何锚定局部重渲染是否优于全局教师区域预测和普通低分辨率裁剪？
2. `Q2 全局语境`：在相同局部图、文字、位置编码和训练预算下，引入整物体视觉上下文是否进一步改善局部教师？
3. `Q3 学生迁移`：局部教师的增量能否传给只使用点云和语言的三维学生？
4. `Q4 修正机制`：优势加权局部蒸馏是否优于无选择直接蒸馏和相同区域的 GT 重加权？

任何没有对应对照、原始指标或日志的问题，都必须写成“未验证”，不得写成“成立”。

## 2. 本阶段边界

### 2.1 包含

- 固定 GEAL 官方上游版本并跑通环境、渲染器、LASO dataloader 和模型前向。
- 审计 LASO train/val 文件、样本、shape ID 和问题表；确认不存在 train/val shape 泄漏。
- 修复官方训练脚本默认用 `test` 做模型选择的问题，所有开发和门控只用 `val`。
- 实现统一的局部区域清单、局部重渲染、A0-A4 教师对照、B0-B4 学生对照。
- 保存逐样本指标，按 `shape_id` 做配对统计和簇 bootstrap。
- 生成 `FocalAfford_feasibility_conclusion.md` 和完整交付包。

### 2.2 不包含

- 不在 LASO test 上调参、选模型、挑 epoch 或决定是否继续。
- 不跑 PIAD、LASO-C、PIAD-C、GLANCE、CMAT、AnyUp 或完整论文表格。
- 不做大量超参数搜索；局部数量、尺度、温度各只允许一次预设和一次故障性调整。
- 不宣称复现 GEAL 论文数字。官方公开训练脚本在训练中读取 test；本阶段改用 val 后只能称为“统一协议下的内部基线”。
- 不因结果不好而更换数据划分、指标实现、随机种子或删除运行。

## 3. 不可违反的执行规则

1. **test 封存：**任何包含 `split=test` 的训练、评估或分析命令，在本阶段都应由程序直接拒绝。对 `anno_test.pkl` 和 `objects_test.pkl` 只允许检查路径、大小和 SHA-256，不得反序列化或读取样本内容。
2. **区域清单固定：**所有 A/B 变体必须使用同一份由点云几何生成的 region manifest。生成区域时不得读取 affordance label；正样本占比只能在 manifest 固定后作为诊断统计。
3. **预算一致：**同一阶段的变体必须使用相同数据、epoch、batch size、优化器、学习率调度、随机种子和 early-stop 规则。
4. **配对比较：**不同变体必须在相同样本和相同区域上输出逐样本结果；只比较聚合均值不算完成。
5. **训练标签边界：**优势权重 `a_R` 只可在训练 split 计算。val 只用于评估，不得把 val 标签送入权重生成或训练。
6. **失败保留：**OOM、NaN、崩溃、超时和负结果均进入运行索引。不得覆盖日志或用同一 run ID 重跑。
7. **不自动扩展：**Gate A 未通过时不得开始完整 Gate B；Gate B 未通过时不得开始 PIAD 或强基线。

## 4. 已核验的外部起点

执行前再次访问并记录可用性，但不要自行替换来源：

- GEAL 官方仓库：<https://github.com/DylanOrange/geal>
- 本计划核验的 GEAL commit：`3048bb8c859b04b3d7569e2bc186f9880c7109ab`
- GEAL 官方说明：Python 3.10、CUDA 11.8、PyTorch 2.1.0；stage 1/2 入口分别为 `scripts/train_stage1.py` 和 `scripts/train_stage2.py`。
- LASO 官方仓库：<https://github.com/yl3800/LASO>
- LASO 官方数据链接：<https://drive.google.com/file/d/1P4CwQeSALtUOgzhg1ILCiKovE-tcXlSu/view?usp=sharing>
- GEAL 官方 LASO seen checkpoint：`dylanorange/geal` 数据仓库中的 `laso_seen.pt`。

如果上述 commit、数据链接或 checkpoint 不可访问，停止在 `BLOCKED_INPUT.md` 中记录事实；不要搜索并替换成来源不明的镜像。

### 4.1 网络与代理

- 执行设备可使用用户提供或系统继承的代理访问 GitHub、Google Drive、Hugging Face、PyTorch 和 Hugging Face model hub。
- 优先沿用执行环境已有的 `HTTP_PROXY`、`HTTPS_PROXY`、`ALL_PROXY` 和 Git 代理配置，不在脚本中写死代理地址。
- `environment.json` 只记录代理变量“是否设置”，不得记录变量值、用户名、token、cookie 或证书内容。
- 代理失败时保留目标 URL、HTTP 状态和命令错误；不要把未发表材料、数据或日志上传到第三方网盘做中转。

## 5. 工作目录与交付目录

在 Linux 执行机使用以下目录。不要把数据、checkpoint 或完整运行目录放入 `cowork_paper` Git 仓库。

```bash
export WORK_ROOT="$HOME/focalafford-feasibility"
export CODE_ROOT="$WORK_ROOT/geal"
export DATA_ROOT="$WORK_ROOT/data/LASO"
export ARTIFACT_ROOT="$WORK_ROOT/artifacts"
export DELIVERY_ROOT="$WORK_ROOT/delivery"

mkdir -p "$WORK_ROOT" "$DATA_ROOT" "$ARTIFACT_ROOT" "$DELIVERY_ROOT"
```

代码仓库最终应具有以下新增结构：

```text
geal/
├── config/focalafford/
│   ├── a0_global.yaml
│   ├── a1_local.yaml
│   ├── a2_context_only.yaml
│   ├── a3_context_local.yaml
│   ├── a4_plain_crop.yaml
│   ├── b0_geal.yaml
│   ├── b1_direct_local.yaml
│   ├── b2_teacher_difference.yaml
│   ├── b3_advantage_correction.yaml
│   └── b4_gt_reweight.yaml
├── focalafford/
│   ├── __init__.py
│   ├── regions.py
│   ├── local_renderer.py
│   ├── local_teacher.py
│   ├── correction_loss.py
│   ├── split_guard.py
│   └── result_schema.py
├── scripts/focalafford/
│   ├── audit_laso.py
│   ├── build_region_manifest.py
│   ├── smoke_test.py
│   ├── train_teacher.py
│   ├── train_student.py
│   ├── evaluate.py
│   └── summarize.py
└── tests/focalafford/
    ├── test_regions.py
    ├── test_local_renderer.py
    ├── test_ablation_routing.py
    ├── test_correction_loss.py
    ├── test_split_guard.py
    └── test_result_schema.py
```

每次运行写入独立目录：

```text
artifacts/<run_id>/
├── command.txt
├── config.resolved.yaml
├── environment.json
├── git.json
├── stdout.log
├── status.json
├── metrics_per_sample.csv
├── metrics_summary.json
└── checkpoints/best.pt        # 仅训练成功时存在
```

`run_id` 固定为 `<stage>_<variant>_seed<seed>_<YYYYMMDD-HHMMSS>`，例如 `A_a3_seed3407_20260922-131500`。

## 6. Task 0：获取代码并冻结环境

### 6.1 克隆并固定上游

```bash
cd "$WORK_ROOT"
git clone https://github.com/DylanOrange/geal.git geal
cd "$CODE_ROOT"
git checkout 3048bb8c859b04b3d7569e2bc186f9880c7109ab
git switch -c experiment/focalafford-feasibility
git rev-parse HEAD | tee "$ARTIFACT_ROOT/upstream_commit.txt"
```

期望：输出严格等于 `3048bb8c859b04b3d7569e2bc186f9880c7109ab`。

### 6.2 创建环境

```bash
conda create -y -n focalafford python=3.10
conda activate focalafford

python -m pip install --upgrade pip
python -m pip install torch==2.1.0 torchvision==0.16.0 torchaudio==2.1.0 \
  --index-url https://download.pytorch.org/whl/cu118
python -m pip install git+https://github.com/dreamgaussian/dreamgaussian.git#subdirectory=simple-knn
python -m pip install git+https://github.com/ashawkey/kiuikit
python -m pip install -r requirements.txt
python -m pip install pytest gdown

python -m pip install ./thirdparty/diff-gaussian-rasterization
```

### 6.3 环境验收

```bash
python - <<'PY'
import json, platform, subprocess, torch
from pathlib import Path

assert torch.cuda.is_available(), "CUDA is unavailable"
assert torch.version.cuda == "11.8", torch.version.cuda
assert torch.__version__.startswith("2.1.0"), torch.__version__

info = {
    "python": platform.python_version(),
    "torch": torch.__version__,
    "torch_cuda": torch.version.cuda,
    "gpu": torch.cuda.get_device_name(0),
    "gpu_count": torch.cuda.device_count(),
    "driver": subprocess.check_output(
        ["nvidia-smi", "--query-gpu=driver_version", "--format=csv,noheader"],
        text=True,
    ).strip().splitlines(),
}
Path("../artifacts/environment_preflight.json").write_text(
    json.dumps(info, indent=2), encoding="utf-8"
)
print(json.dumps(info, indent=2))
PY

python - <<'PY'
import torch
from diff_gaussian_rasterization import GaussianRasterizationSettings
print("CUDA extension import: OK")
PY
```

验收标准：两段命令退出码为 0，CUDA 可用，扩展可导入。若 CUDA minor version 文本不等于 11.8，但 PyTorch/CUDA extension 能成功编译和前向，记录差异后可继续；不得改换 PyTorch 大版本而不记录。

## 7. Task 1：获取 LASO 与官方 checkpoint

### 7.1 下载 LASO

```bash
mkdir -p "$WORK_ROOT/downloads" "$DATA_ROOT"
cd "$WORK_ROOT/downloads"
gdown --fuzzy \
  'https://drive.google.com/file/d/1P4CwQeSALtUOgzhg1ILCiKovE-tcXlSu/view?usp=sharing' \
  -O LASO_download
file LASO_download
sha256sum LASO_download | tee "$ARTIFACT_ROOT/laso_download.sha256"
```

根据 `file` 的真实输出解包：

```bash
case "$(file -b LASO_download)" in
  *Zip*) unzip -q LASO_download -d "$DATA_ROOT" ;;
  *gzip*) tar -xzf LASO_download -C "$DATA_ROOT" ;;
  *tar*) tar -xf LASO_download -C "$DATA_ROOT" ;;
  *) echo "Unsupported LASO archive type" >&2; exit 2 ;;
esac
```

解包后必须把实际数据根目录规范为 `$DATA_ROOT`，且该目录直接包含：

```text
Affordance-Question.csv
anno_train.pkl
anno_val.pkl
anno_test.pkl
objects_train.pkl
objects_val.pkl
objects_test.pkl
```

若文件在额外子目录中，只移动这七个文件到 `$DATA_ROOT`；不要重命名文件或合并 split。

### 7.2 下载官方 LASO seen checkpoint

```bash
cd "$CODE_ROOT"
mkdir -p ckpt
huggingface-cli download dylanorange/geal laso_seen.pt \
  --repo-type dataset --local-dir ckpt
sha256sum ckpt/laso_seen.pt | tee "$ARTIFACT_ROOT/laso_seen_checkpoint.sha256"
```

checkpoint 仅用于前向和代码路径核查。由于官方训练脚本会在训练过程中读取 test，本阶段不得把该 checkpoint 的 test 表现作为我们的方法证据。

### 7.3 数据审计脚本合同

实现 `scripts/focalafford/audit_laso.py`，命令：

```bash
python scripts/focalafford/audit_laso.py \
  --data-root "$DATA_ROOT" \
  --output "$ARTIFACT_ROOT/laso_audit.json" \
  --do-not-read-test-content
```

脚本必须：

- 对 train/val 的 annotation 和 object 文件计算 SHA-256。
- 输出 train/val annotation 数、object 数、唯一 shape 数、类别数、功能数和 mask 长度分布。
- 验证每条 annotation 的 `shape_id` 在同 split 的 objects 中存在。
- 验证 train 与 val 的 shape ID 交集为空；非空时退出码为 3，并停止实验。
- 验证 mask 数值有限、范围位于 `[0,1]`、长度等于对应点数。
- 只检查 test 文件是否存在及其文件 SHA-256，不反序列化 test 内容。
- 把所有检查写入 JSON；任一硬错误时退出非零。

## 8. Task 2：先建立防泄漏和结果结构

### 8.1 `split_guard.py`

所有训练、评估、manifest 和汇总入口在读取 dataset 之前调用：

```python
assert_development_split(split: str, phase: str)
```

行为：当 `phase == "feasibility"` 且 `split.lower() == "test"` 时抛出 `RuntimeError`；错误信息必须包含 `test split is sealed`。

### 8.2 `result_schema.py`

逐样本 CSV 至少包含以下列：

```text
run_id,stage,variant,seed,split,shape_id,class_name,affordance,
region_id,region_visible_points,region_target_mass,iou,auc,sim,mae
```

学生级全点云结果的 `region_id` 写为空值；不得删列。`metrics_summary.json` 必须包含：

```json
{
  "run_id": "...",
  "status": "completed|failed|timeout",
  "primary_metric": "mae|iou",
  "sample_count": 0,
  "shape_count": 0,
  "metrics": {"iou": null, "auc": null, "sim": null, "mae": null},
  "best_epoch": null,
  "checkpoint": null
}
```

数值未知时使用 JSON `null`，不得填 0。

### 8.3 必写测试

```bash
pytest -q tests/focalafford/test_split_guard.py tests/focalafford/test_result_schema.py
```

测试必须覆盖：

- feasibility/test 被拒绝，feasibility/val 被允许。
- 缺列、重复 `(run_id, shape_id, region_id)`、NaN/Inf 指标会导致 schema 校验失败。
- failed run 可以没有 checkpoint，但必须有 `status.json` 和 `stdout.log`。

只有测试全部通过才进入下一任务。

## 9. Task 3：实现固定的几何区域协议

### 9.1 Region manifest

`build_region_manifest.py` 对 train 和 val 分别生成 JSONL：

```bash
python scripts/focalafford/build_region_manifest.py \
  --data-root "$DATA_ROOT" --split train --setting seen \
  --regions-per-shape 4 --neighbors 256 --seed 3407 \
  --output "$ARTIFACT_ROOT/regions_train.jsonl"

python scripts/focalafford/build_region_manifest.py \
  --data-root "$DATA_ROOT" --split val --setting seen \
  --regions-per-shape 4 --neighbors 256 --seed 3407 \
  --output "$ARTIFACT_ROOT/regions_val.jsonl"
```

区域生成协议固定如下：

1. 使用 GEAL dataloader 输出的归一化点云。
2. 以确定性 FPS 选 4 个 anchor；初始点由 `sha256(shape_id + seed)` 映射到点索引，不能由 label 决定。
3. 每个 anchor 取欧氏距离最近的 256 个点；按距离和点索引稳定排序处理并列。
4. 对 GEAL 的 12 个固定相机，用 renderer 返回的 `rendered_idx` 统计该邻域的可见点数，选择可见点最多的视角；并列时取官方视角顺序中较早者。
5. 每条记录保存 split、shape_id、region_id、anchor index、256 个 point index、view index、可见点数和生成参数。
6. manifest 中不得保存 GT mask、target mass、类别预测或模型分数。

### 9.2 局部重渲染

为每个固定区域产生两类输入：

- `local_highres`：在选定相机下把完整点云重新渲染到 `224x224`；根据该区域可见点在 `rendered_idx` 中的位置取紧包围框，四周扩展 20%，裁剪后双线性缩放到 `112x112`。
- `plain_crop`：从官方 `112x112` 全局深度图使用同一投影范围裁剪，再缩放回 `112x112`。

二者必须使用相同深度转 RGB 和 ImageNet 归一化。局部目标图由同一相机、同一 crop 参数渲染 GT mask 得到。若区域少于 32 个可见点，记录 `valid=false`，所有变体都排除该区域，不得只为某个变体排除。

不同分辨率的教师不得直接比较像素均值。教师评估和阶段 B 的误差计算统一回到 point index：用对应渲染的 `rendered_idx` 把像素预测回投到 region 的可见点，同一点命中多个像素时取均值；没有命中的点记为不可见。任意两种教师做配对比较时，只在二者与 GT 都有效的 point index 交集上计算 MAE/SIM/BCE，并把交集点数写入 `region_visible_points`。交集少于 32 点时，该 region 对所有变体统一无效。

### 9.3 位置编码

每个区域只使用下列 8 维原始几何量，经两层 MLP 投影到 512 维：

```text
anchor_xyz(3), normalized_radius(1), camera_azimuth_sin_cos(2), camera_elevation_sin_cos(2)
```

位置量来自 manifest 和相机，不得包含类别、功能或 GT。

### 9.4 测试

```bash
pytest -q tests/focalafford/test_regions.py tests/focalafford/test_local_renderer.py
```

必须覆盖：

- 同一 shape/seed 两次生成的 manifest 字节一致。
- 打乱或替换 label 后 manifest 不变。
- 每个 region 恰有 256 个合法点索引且 anchor 在区域内。
- 高分辨率和普通裁剪使用同一 view、bbox 和目标区域。
- crop 不越界；空投影和低可见点区域被统一标无效。
- 一个合成点云的回投 point index 与裁剪后的 mask 能对应。

## 10. Task 4：实现 A0-A4 教师对照

### 10.1 共享部分

- DINOv2 ViT-B/14 保持冻结。
- GEAL 发布的 `laso_seen.pt` 由官方 `scripts/evaluation.py` 加载到 `Branch3D`，不是已核实的 stage 1 教师权重。A0 必须在本阶段按与 A1-A4 相同的 train 子集、seed 和 10 epoch 预算训练 `Branch2D`；不得把 3D checkpoint 猜测映射到 `Branch2D`。
- 所有变体使用相同文字、相同 position MLP、相同预测头容量和相同 region manifest。
- 全局特征取 GEAL 最后一层融合后的 patch tokens；局部特征取相同 DINO 层级和相同投影层。
- 训练损失沿用 GEAL stage 1 对区域 GT 的 BCE；模型选择只看 val region MAE。

### 10.2 变体定义

| ID | 输入与计算 | 唯一目的 |
|---|---|---|
| A0 | GEAL 全局教师预测后，按固定 bbox 裁出对应区域 | 全局教师区域起点 |
| A1 | `local_highres + text + position` | 局部视觉是否有价值 |
| A2 | `global visual context + text + position`，local token 替换为同形状可学习 query | 只靠全局信息能否预测局部 |
| A3 | local token 作 query，global task-conditioned tokens 作 key/value，单层 8-head cross-attention，残差接回 local token | 局部细节与全局语境是否互补 |
| A4 | 与 A1 完全相同，但输入 `plain_crop` | 高分辨率几何重渲染是否必要 |

A3 的语境层固定为：`embed_dim=512`、`num_heads=8`、`dropout=0`、一层 cross-attention、残差加 LayerNorm。不允许增加第二层或额外大头部。

### 10.3 消融路由测试

`test_ablation_routing.py` 必须用 forward hook 或显式返回的 debug flags 验证：

- A1/A4 不读取 global context tensor。
- A2 不读取 local image tensor；改变 local image 不改变输出。
- A3 同时读取 local 与 global tensor；分别扰动任一输入会改变输出。
- A0 没有局部模型参数。
- 五个变体使用相同 region IDs。

### 10.4 Smoke 命令

```bash
python scripts/focalafford/smoke_test.py \
  --data-root "$DATA_ROOT" \
  --train-manifest "$ARTIFACT_ROOT/regions_train.jsonl" \
  --val-manifest "$ARTIFACT_ROOT/regions_val.jsonl" \
  --variants a0,a1,a2,a3,a4 \
  --samples 8 --steps 2 --device cuda
```

Smoke 成功标准：

- 所有变体完成前向；A1-A4 完成两步反向。
- loss 和输出无 NaN/Inf，输出范围在 `[0,1]`。
- A3 峰值显存不超过可用显存；如果 OOM，只允许等比例降低 batch size，并对所有 A1-A4 使用相同新 batch size。
- 每个变体产生符合 schema 的临时结果；smoke 指标不得进入可行性结论。

## 11. Task 5：阶段 A 正式快速实验

### 11.1 固定预算

- 数据：LASO seen train/val。
- 训练子集：按 `(object class, affordance)` 分层，对 train 中每组使用由 seed 3407 固定抽取的 20%；不足 5 条的组全部保留。
- 验证：完整 val，不抽样。
- seeds：`3407, 3408`。
- epochs：10。
- batch size：8；若 smoke OOM，则统一改为 4 或 2。
- optimizer/scheduler：沿用 GEAL stage 1 配置，`lr=1e-4`、text encoder 参数 `5e-6`、weight decay `1e-3`、StepLR step 10/gamma 0.5。
- checkpoint：每个 run 仅保存 val MAE 最优 epoch；同一 run 不 early stop。
- 主要指标：region MAE，越低越好。
- 次要指标：region SIM，越高越好。
- 诊断分组：全部有效区域、`region_target_mass>0` 区域、按 object/affordance 分组。target mass 只在评估时计算。

### 11.2 运行命令

```bash
cd "$CODE_ROOT"
declare -A A_CONFIG=(
  [a0]="config/focalafford/a0_global.yaml"
  [a1]="config/focalafford/a1_local.yaml"
  [a2]="config/focalafford/a2_context_only.yaml"
  [a3]="config/focalafford/a3_context_local.yaml"
  [a4]="config/focalafford/a4_plain_crop.yaml"
)
for seed in 3407 3408; do
  for variant in a0 a1 a2 a3 a4; do
    python scripts/focalafford/train_teacher.py \
      --config "${A_CONFIG[$variant]}" \
      --data-root "$DATA_ROOT" \
      --train-manifest "$ARTIFACT_ROOT/regions_train.jsonl" \
      --val-manifest "$ARTIFACT_ROOT/regions_val.jsonl" \
      --train-fraction 0.20 \
      --seed "$seed" \
      --epochs 10 \
      --output-root "$ARTIFACT_ROOT" \
      2>&1 | tee "$ARTIFACT_ROOT/A_${variant}_seed${seed}_console.log"
  done
done
```

运行前用 `printf '%s\n' "${A_CONFIG[@]}"` 核对五个精确配置路径。A0 与 A1-A4 一样按两个 seed 训练；五个变体必须使用相同抽样清单和训练预算。

### 11.3 阶段 A 汇总

```bash
python scripts/focalafford/summarize.py \
  --stage A \
  --artifact-root "$ARTIFACT_ROOT" \
  --cluster-key shape_id \
  --bootstrap-resamples 10000 \
  --bootstrap-seed 20260922 \
  --output "$ARTIFACT_ROOT/gate_a.json"
```

统计方法：

- 对相同 `(shape_id, region_id)` 计算配对差值。
- bootstrap 以 `shape_id` 为簇重采样，不能把同一 shape 的多个 region 当独立样本。
- 报告均值差及 95% percentile CI；不只报告 p 值。
- 两个训练 seed 分别报告，并额外报告 seed 等权平均的逐样本差。

### 11.4 Gate A 判定

`A-local PASS` 同时要求：

- A1 相对 A0 的 MAE 差值 95% CI 上界 `< 0`；
- A1 相对 A4 的 MAE 差值 95% CI 上界 `< 0`；
- 上述方向在两个 seed 中一致；
- SIM 没有出现 95% CI 完全小于 0 的退化。

`A-context PASS` 同时要求：

- A3 相对 A1、A2 的 MAE 差值 95% CI 上界均 `< 0`；
- 两个 seed 方向一致；
- SIM 没有显著退化。

决策：

| 结果 | 后续 |
|---|---|
| A-local FAIL | 停止 B 阶段，结论为核心局部观察在当前协议下不受支持 |
| A-local PASS，A-context FAIL | 使用 A1 作为局部教师进入精简 B 阶段；不再声称全局语境有效 |
| A-local PASS，A-context PASS | 固定 A3 最优 checkpoint，进入完整最小 B 阶段 |
| 任一比较方向不一致或 CI 跨 0 | 标记 INCONCLUSIVE；只允许把 train fraction 提到 40% 后原样重跑一次，不改其他参数 |

不得因为 A3 输给 A1 而调整 attention 层数、头数或局部尺度后继续称为同一预设实验。

## 12. Task 6：实现 B0-B4 学生对照

### 12.1 共享初始化与协议

- 使用 GEAL `Branch3D` 和官方 stage 2 损失作为 B0。
- 所有 B 变体使用相同 seed 的相同三维模型初始权重；每个 seed 在训练前保存一次 `initial_state.pt`，各变体分别加载。
- 全局教师和 Gate A 固定的局部教师均冻结并设为 eval。
- B 阶段仍只用 train/val；模型选择看 val IoU，不读取 test。
- 局部区域沿用阶段 A manifest，不重新采样。

### 12.2 区域误差与优势权重

对训练区域 `R`：

```text
e_g(R) = mean BCE(global_teacher_prediction_R, y_R)
e_l(R) = mean BCE(local_teacher_prediction_R, y_R)
a(R)   = relu(e_g(R) - e_l(R))
w(R)   = stop_gradient(a(R) / (mean_batch(a) + 1e-6))
```

约束：

- `y_R` 仅来自 train。
- 全局、局部预测与 GT 必须在相同可见点索引上比较。
- 没有正优势的 batch，`L_corr=0`，不得除零或强行产生梯度。
- `w(R)` 和两类教师全部停止梯度。
- `L_corr` 为教师局部 context feature 与对应三维 point feature 经各自 64 维投影后的 MSE；先在 point 维取均值，再按 `w(R)` 加权。
- 首轮固定 `lambda_c=0.1`，不得搜索；GEAL 原有一致性权重保持官方配置。

### 12.3 变体定义

| ID | 损失 | 目的 |
|---|---|---|
| B0 | `L_aff + lambda_g * L_GEAL` | 统一协议 GEAL 起点 |
| B1 | B0 + 所有有效局部区域等权 `L_local` | 直接局部蒸馏 |
| B2 | B0 + 按 `mean(abs(t_l-t_g))` 加权，不看谁更准 | 排除“只要教师不同就加权” |
| B3 | B0 + `lambda_c * L_corr`，权重为训练标签优势 | 最终修正蒸馏 |
| B4 | B0 + 使用与 B3 相同 `w(R)` 加权区域 `L_aff`，不蒸馏局部特征 | 排除 GT 重加权解释 |

如果 Gate A 只支持 A1，则 B1/B2/B3 使用 A1 特征；报告必须把“context-guided”标记为未支持。

### 12.4 损失测试

```bash
pytest -q tests/focalafford/test_correction_loss.py tests/focalafford/test_ablation_routing.py
```

必须覆盖：

- `e_l < e_g` 时 `a>0`；相等或更差时 `a=0`。
- 交换教师误差会让优势方向相应交换。
- 全零优势返回精确的有限 0 loss。
- 对 `w`、局部教师和全局教师反向后梯度为 None；学生投影层有有限梯度。
- B4 使用与 B3 字节一致的权重张量，但局部 feature 不进入 loss。
- B0 在关闭新增损失时与未修改 GEAL stage 2 单 batch 输出和 loss 在 `rtol=1e-5, atol=1e-6` 内一致。

## 13. Task 7：阶段 B 最小正式实验

### 13.1 固定预算

- train subset、val、seeds 与阶段 A 相同。
- epochs：12。
- batch size：沿用 smoke 后统一值。
- 主要指标：完整点云 IoU，越高越好。
- 次要指标：SIM、AUC 越高越好，MAE 越低越好。
- 模型选择：val IoU 最优 epoch。
- 首先运行 B0、B1、B3、B4；它们全部完成后再运行 B2。不得用 B2 的结果回头改 B3。

### 13.2 运行命令

```bash
cd "$CODE_ROOT"
for seed in 3407 3408; do
  python scripts/focalafford/train_student.py --config config/focalafford/b0_geal.yaml \
    --data-root "$DATA_ROOT" --train-fraction 0.20 --seed "$seed" --epochs 12 \
    --output-root "$ARTIFACT_ROOT"
  python scripts/focalafford/train_student.py --config config/focalafford/b1_direct_local.yaml \
    --data-root "$DATA_ROOT" --train-fraction 0.20 --seed "$seed" --epochs 12 \
    --teacher-from-gate "$ARTIFACT_ROOT/gate_a.json" --output-root "$ARTIFACT_ROOT"
  python scripts/focalafford/train_student.py --config config/focalafford/b3_advantage_correction.yaml \
    --data-root "$DATA_ROOT" --train-fraction 0.20 --seed "$seed" --epochs 12 \
    --teacher-from-gate "$ARTIFACT_ROOT/gate_a.json" --output-root "$ARTIFACT_ROOT"
  python scripts/focalafford/train_student.py --config config/focalafford/b4_gt_reweight.yaml \
    --data-root "$DATA_ROOT" --train-fraction 0.20 --seed "$seed" --epochs 12 \
    --teacher-from-gate "$ARTIFACT_ROOT/gate_a.json" --output-root "$ARTIFACT_ROOT"
done

for seed in 3407 3408; do
  python scripts/focalafford/train_student.py --config config/focalafford/b2_teacher_difference.yaml \
    --data-root "$DATA_ROOT" --train-fraction 0.20 --seed "$seed" --epochs 12 \
    --teacher-from-gate "$ARTIFACT_ROOT/gate_a.json" --output-root "$ARTIFACT_ROOT"
done
```

每条命令由脚本自身写 `command.txt`、resolved config、环境、Git diff hash 和日志。若进程失败，不自动重跑；先写 `status.json`，修复后使用新 run ID，并在新状态文件中写 `supersedes_run_id`。

### 13.3 Gate B 汇总与判定

```bash
python scripts/focalafford/summarize.py \
  --stage B \
  --artifact-root "$ARTIFACT_ROOT" \
  --cluster-key shape_id \
  --bootstrap-resamples 10000 \
  --bootstrap-seed 20260922 \
  --output "$ARTIFACT_ROOT/gate_b.json"
```

`B-transfer PASS`：固定局部教师的 B1 相对 B0，IoU 配对差 95% CI 下界 `>0`，两个 seed 方向一致，MAE 无显著退化。

`B-correction PASS`：B3 相对 B1 和 B4 的 IoU 配对差 95% CI 下界均 `>0`，两个 seed 方向一致，且 SIM/MAE 不出现显著反向退化。B2 仅用于解释，不参与是否通过的必要条件。

若 CI 跨 0，只允许将 train fraction 从 20% 提到 40%，按相同 seeds、epochs 和配置重跑一次。若方向反转、B3 输给 B4，或 B1 不优于 B0，直接记录未通过，不调 `lambda_c`。

## 14. 运行监控与故障处理

### 14.1 每个训练 run 的监控

- 每 30 秒确认进程存活。
- 每 5 分钟记录 GPU utilization、显存、温度和输出目录大小。
- 30 分钟内日志无新 batch/epoch 才标记 `STALL_SUSPECTED`，但不自动终止。
- 出现 NaN/Inf 立即终止该 run，保存最后 200 行日志和触发 batch 的 sample IDs。
- 单个 smoke 硬超时 30 分钟；单个 A/B 正式 run 硬超时 12 小时。达到硬超时可终止，并标记 timeout。
- 只允许因 OOM 等比例降低所有同阶段变体的 batch size；不得只给某一方法更多资源。

### 14.2 故障决策表

| 故障 | 允许动作 | 禁止动作 |
|---|---|---|
| 数据文件缺失/损坏 | 停止并回传缺失路径、hash、官方来源状态 | 换非官方数据 |
| train/val shape 重叠 | 停止，报告交集和上游文件 hash | 自行删样本后继续 |
| CUDA extension 编译失败 | 保存完整 build log，核对 CUDA/GCC/PyTorch | 静默改 PyTorch 主版本 |
| OOM | 所有同阶段变体统一减 batch | 只降低某个变体或改分辨率 |
| loss NaN | 保存 batch IDs、输入范围、栈信息；修 bug 后新 run | 跳过该 batch |
| 单个 run 崩溃 | 标记 failed，修复后新 run ID | 覆盖原日志、自动重试 |
| 指标低于预期 | 如实完成对照和汇总 | 改 split/seed/metric |

## 15. 最终结论文档

由 `scripts/focalafford/summarize.py` 生成骨架，再由执行 agent 根据真实文件补全：

`$DELIVERY_ROOT/FocalAfford_feasibility_conclusion.md`

必须使用以下结构，不得删掉失败或未验证项：

```markdown
# FocalAfford 快速可行性结论

## 结论摘要
- 总判定：CONTINUE_FULL / CONTINUE_LOCAL_ONLY / REVISE_DISTILLATION / INCONCLUSIVE / STOP
- 一句话依据：
- 证据范围：LASO seen train/val；未查看 test

## 实验身份
- GEAL upstream commit：
- 实验代码 diff SHA-256：
- LASO train/val 文件 SHA-256：
- checkpoint SHA-256：
- CUDA/PyTorch/GPU：
- 完成与失败 run 数：

## 数据与协议核验
- train/val 样本及 shape 数：
- train/val shape 交集：
- test 封存检查：
- region manifest SHA-256：

## Gate A：局部教师
| Variant | Seed | Regions | MAE | SIM | 状态 |
|---|---:|---:|---:|---:|---|

### 配对比较
| Comparison | Metric | Mean delta | 95% CI | 两 seed 同向 | 判定 |
|---|---|---:|---|---|---|

- A-local：PASS / FAIL / INCONCLUSIVE
- A-context：PASS / FAIL / INCONCLUSIVE
- 观察到的失败模式：

## Gate B：三维学生
| Variant | Seed | Shapes | IoU | AUC | SIM | MAE | 状态 |
|---|---:|---:|---:|---:|---:|---:|---|

### 配对比较
| Comparison | Metric | Mean delta | 95% CI | 两 seed 同向 | 判定 |
|---|---|---:|---|---|---|

- B-transfer：PASS / FAIL / NOT_RUN
- B-correction：PASS / FAIL / NOT_RUN
- B2 对机制解释的影响：

## 可支持与不可支持的主张
### 当前证据可支持
- （只写 Gate 明确支持的内容）

### 当前证据不可支持
- 跨数据集泛化、test 性能、优于近期强基线、完整复现和统计显著性之外的因果解释均未在本阶段验证。

## 负结果与异常
| Run ID | 问题 | 是否影响结论 | 证据路径 |
|---|---|---|---|

## 下一步
- 仅列由 Gate 判定直接触发的下一步，不提出无依据的新模块。
```

总判定映射固定为：

| 条件 | 总判定 |
|---|---|
| A-local FAIL | `STOP` |
| A-local PASS，A-context FAIL，B-transfer PASS | `CONTINUE_LOCAL_ONLY` |
| A-local/A-context PASS，B-transfer PASS，B-correction FAIL | `REVISE_DISTILLATION` |
| A-local/A-context/B-transfer/B-correction 全 PASS | `CONTINUE_FULL` |
| 任一关键 Gate 因 CI 跨 0、运行失败或数据问题无法判断 | `INCONCLUSIVE` |

## 16. 必须回传的交付包

执行 agent 不 commit、不 push。完成后生成：

```text
delivery/
├── FocalAfford_feasibility_conclusion.md
├── code.patch
├── untracked_code.tar.gz
├── configs.tar.gz
├── run_manifest.csv
├── metrics_per_sample.tar.gz
├── metrics_summary.tar.gz
├── logs.tar.gz
├── figures.tar.gz                 # 没有图时可省略
├── checkpoints_manifest.csv       # 只列路径、大小、SHA-256，不传大权重
├── environment.json
├── data_manifest.json
├── gate_a.json
├── gate_b.json                    # 未运行 B 时写 NOT_RUN JSON
└── SHA256SUMS
```

生成命令：

```bash
cd "$CODE_ROOT"
git diff --binary 3048bb8c859b04b3d7569e2bc186f9880c7109ab > "$DELIVERY_ROOT/code.patch"
git ls-files --others --exclude-standard -z | \
  tar --null -czf "$DELIVERY_ROOT/untracked_code.tar.gz" --files-from=-
tar -czf "$DELIVERY_ROOT/configs.tar.gz" config/focalafford
tar -czf "$DELIVERY_ROOT/metrics_per_sample.tar.gz" \
  -C "$ARTIFACT_ROOT" $(find "$ARTIFACT_ROOT" -name metrics_per_sample.csv -printf '%P ')
tar -czf "$DELIVERY_ROOT/metrics_summary.tar.gz" \
  -C "$ARTIFACT_ROOT" $(find "$ARTIFACT_ROOT" -name metrics_summary.json -printf '%P ')
tar -czf "$DELIVERY_ROOT/logs.tar.gz" \
  -C "$ARTIFACT_ROOT" $(find "$ARTIFACT_ROOT" -name '*.log' -printf '%P ')

cd "$DELIVERY_ROOT"
find . -maxdepth 1 -type f ! -name SHA256SUMS -print0 | sort -z | \
  xargs -0 sha256sum > SHA256SUMS
sha256sum -c SHA256SUMS
```

`run_manifest.csv` 每个 run 一行，至少包含：run_id、stage、variant、seed、status、command、start/end time、best epoch、primary metric、artifact path。`checkpoints_manifest.csv` 只记录 checkpoint 索引，不上传大文件；如需带回权重，由用户另行指定。

## 17. 完成条件

只有同时满足以下条件，执行任务才算完成：

- 环境、数据和 split 审计通过，test 未被读取用于开发。
- 新增单元测试全部通过，并保留 pytest 输出。
- A0-A4 两个 seeds 全部完成，或失败均有不可恢复证据。
- 只有 Gate A 允许时才运行 B；已运行的 B 对照完整且预算一致。
- 每个完成 run 有 resolved config、命令、日志、逐样本指标和摘要。
- 最终报告中的每个数字都能定位到交付包文件。
- `SHA256SUMS` 校验通过。
- 报告没有“已复现 GEAL”“优于 baseline”“机制有效”等超出当前证据的表述。

完成后只向用户汇报三件事：新发现、判断变化、下一步；不要用运行过程填充结论。
