# RealityScan 贴图工作流自动化脚本

这是一个基于 Python 的自动化工具脚本，旨在通过命令行调用 `RealityScan.exe`，实现三维重建、贴图生成、模型导出及纹理重投影等工作流的完全自动化。

## 🛠 环境要求

- **Python 版本**: Python 3.10 及以上
- **操作系统**: Windows
- **依赖软件**: 必须安装包含 `RealityScan.exe` 

## ✨ 核心功能

1. **环境自检 (`env_check`)**: 自动检查环境变量中是否存在 `RealityScan.exe`，若不存在则支持在运行时动态输入并临时添加。
2. **全自动贴图生成 (`generate_workflow`)**: 从输入照片文件夹开始，自动完成照片对齐、重建区域设置、法线计算、模型清理、UV 展开、贴图计算及项目保存。
3. **模型导出 (`export_compare_mesh_workflow`)**: 自动加载已有项目，并导出指定的模型文件（如 `.obj`）用于外部网格比较或编辑。
4. **纹理重投影 (`reproject_texture_workflow`)**: 将项目中的源模型贴图重新投影到外部导入的新网格上，并导出带有新贴图的模型。

## 🚀 快速开始

将 `function.py` 放入你的工作目录中，然后在其他 Python 脚本中导入并调用所需的工作流函数。

### 1. 基础环境配置

在运行任何工作流之前，建议先运行环境检查，确保程序能找到 `RealityScan.exe`：

```
from function import env_check

# 检查环境变量，如果找不到会自动提示用户输入路径
env_check()
```

### 2. 生成贴图工作流

从照片文件夹直接生成三维模型和贴图，并保存为项目文件：

```python
from function import generate_workflow

generate_workflow(
    input_photo_dir=r"C:\path\to\your\photos",  # 你的输入照片路径
    project_dir=r"C:\path\to\save\project",     # 项目保存路径（不带后缀）
    headless_bool=True,                         # 是否在后台静默运行
    quit_bool=True                              # 运行结束后是否关闭软件
)
```

### 3. 导出模型

加载已有的项目文件，并导出指定的模型（默认导出名称为 "Model 3" 的模型）：

```python
from function import export_compare_mesh_workflow

export_compare_mesh_workflow(
    project_dir=r"C:\path\to\save\project",          # 项目路径（不带后缀）
    export_compare_mesh_dir=r"C:\path\to\export",    # 模型导出目录
    compare_mesh="my_exported_model",                # 导出的文件名
    model_name="Model 3"                             # RealityScan中要导出的模型名
)
```

### 4. 纹理重投影工作流

将一个外部修改好拓扑的网格导入，并将原高模的纹理重新投影到该网格上，然后导出：

```python
from function import reproject_texture_workflow

reproject_texture_workflow(
    project_dir=r"C:\path\to\save\project",
    to_be_reprojected_mesh_dir=r"C:\path\to\import", # 待重投影网格所在目录
    reprojected_mesh_dir=r"C:\path\to\export",       # 最终输出目录
    to_be_reprojected_mesh="my_clean_mesh",          # 导入的文件名
    reprojected_mesh="my_final_textured_mesh",       # 导出的文件名
    reprojection_model="Model 3",                    # 提供贴图的源模型名
    to_be_reprojected_model="Model 4"                # 导入后在软件内的模型名
)
```

## ⚠️ 注意事项

1. **路径格式**: 在 Windows 下，建议使用原始字符串（如 `r"C:\folder\path"`）来传递路径，以避免转义字符报错。
2. **后台运行 (Headless)**: 默认情况下所有工作流都开启了 `headless_bool=True`。如果你希望在运行过程中看到软件的图形界面以监控进度，可以将其设置为 `False`。
3. **文件后缀**: 脚本内部已自动处理了项目的 `.rsproj` 和模型的 `.obj` 后缀，在传递参数时无需重复添加后缀。