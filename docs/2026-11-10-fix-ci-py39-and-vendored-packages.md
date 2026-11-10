# 2026-11-10 Python CI (python-package.yml / octave-tests.yml) 修复

## 现象（两类失败）

1. `setup-python` 无法安装 3.9：

   ```
   ##[error]The version '3.9' with architecture 'x64' was not found for Ubuntu 26.04.
   ```
2. `build-2023c` 所有 Python 版本在 flake8 严格检查（`--select=E9,F63,F7,F82`）退出 1：
   - 70 个错误**全部位于 vendored 的 `packages/` 目录**（typing_extensions.py / tensorflow / scipy / pasta 等），
     如 `packages/typing_extensions.py F822 undefined name 'Union' in __all__`、
     `packages/pasta/base/annotate.py F821 undefined name 'target'`。

## 根因

- 2026-10 起 GitHub `ubuntu-latest` 形象迁移到 **Ubuntu 26.04**，其打包的 Python 版本不再包含 3.9，`actions/setup-python` 也没有 26.04 的 3.9 构建；
- 2023C 项目自带 vendored 第三方代码（packages/），其内部 F821/F822 在 Python 3.9+ 语义下不可修复（属于上游历史代码）。

## 修复

`.github/workflows/python-package.yml`、`.github/workflows/octave-tests.yml`：

- `runs-on: ubuntu-latest` → `runs-on: ubuntu-24.04`（3.9–3.12 全部可用）；
- 严格 flake8 检查统一加 `--extend-exclude=packages`，忽略 vendored 目录（非业务代码）。
