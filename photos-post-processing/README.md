# photos-post-processing

一组用于批量处理标注图片的小脚本：

- 根据图片对应的 `*.json` 标注生成黑白 mask 预览图
- 使用标注 mask 进行抠图（导出透明背景 PNG）
- 将单张图片中的黑色像素置为透明

## 环境与安装

- Python >= 3.9

安装依赖：

```bash
pip install -r requirements.txt
```

## 输入文件约定

默认约定同名配对：

- 图片：`xxx.jpg` / `xxx.png` / ...
- 标注：`xxx.json`（与图片同名，后缀不同）

支持的标注结构（自动识别其一）：

- LabelMe 风格：`{"shapes": [{"label": "...", "points": [[x,y], ...]}]}`
- 通用对象列表：`{"objects": [{"label": "...", "points": [[x,y], ...]}]}` 或 `{"objects": [{"polygon": [[x,y], ...]}]}`
- 直接数组 mask：`{"mask": [[0,1,1,...], ...]}`（大小需与图片一致；非 0 视为前景）

## 脚本

### 1) 生成黑白 mask 预览图

文件：[generate_bw_masks.py](file:///g:/university/innovation/scan/x-label/photos-post-processing/generate_bw_masks.py)

把标注转换为灰度 PNG（白色=前景，黑色=背景）。

```bash
python generate_bw_masks.py --input-dir path/to/data
```

常用参数：

- `--output-dir`：输出目录（默认：`<input-dir>/bw_masks`）
- `--recursive`：递归搜索图片
- `--labels person,car`：只保留指定 label 的多边形
- `--invert-mask`：反转输出（白底黑物）
- `--strict`：遇到坏 json 直接报错退出（默认跳过）

输出命名：

- 与输入目录的相对路径一致，文件名追加 `_mask.png`

### 2) 按标注抠图（导出透明背景 PNG）

文件：[remove_background.py](file:///g:/university/innovation/scan/x-label/photos-post-processing/remove_background.py)

根据 json 生成的 mask，将图片转换为 RGBA，并用 mask 作为 alpha 通道，导出透明背景 PNG。

```bash
python remove_background.py --input-dir path/to/data
```

常用参数：

- `--output-dir`：输出目录（默认：`<input-dir>/no_bg`）
- `--recursive`：递归搜索图片
- `--labels person,car`：只保留指定 label 的多边形
- `--invert-mask`：若 json 标的是背景而不是前景可反转
- `--strict`：遇到坏 json 直接报错退出（默认跳过）

输出命名：

- 与输入目录的相对路径一致，文件名追加 `_nobg.png`

### 3) 单张图片去黑（黑色像素透明化）

文件：[remove_black_pixels.py](file:///g:/university/innovation/scan/x-label/photos-post-processing/remove_black_pixels.py)

把满足 `R,G,B <= threshold` 的像素视为“黑色”，将其 alpha 置为 0。

```bash
python remove_black_pixels.py --input path/to/image.jpg
```

常用参数：

- `--output`：输出 PNG（默认：`<input_stem>_no_black.png`）
- `--threshold`：阈值（默认 0，仅纯黑；例如 10 会更激进）

## 常见问题

- 为什么有的图片被 `[SKIP]`？
  - 通常是缺少同名 `*.json`、json 中没有可识别的 mask 结构、或多边形点数量不足。
- mask 数组尺寸不匹配？
  - `{"mask": ...}` 的二维数组必须是 `(height, width)`，并与图片尺寸一致。
