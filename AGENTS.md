# ZenDNN-tensorflow-plugin

Agent context for the ZenDNN plugin for TensorFlow (zentf) repository.

## Project overview

- Languages: Python (interface), C++ (kernel/plugin implementation), Bazel/Starlark (build)
- Build system: Bazel 7.7.0
- Layout: `tensorflow_plugin/` (plugin sources), `scripts/` (setup helpers for Python/Java/C++), `third_party/` (TensorFlow dependency configuration), `configure.py` (interactive build configuration).
- Versioning: zentf versions follow TensorFlow (e.g., `2.21.0.1` targets TensorFlow `2.21.0`).

## Prerequisites

- Bazel 7.7.0 (`.bazelversion` pins this)
- Git ≥ 1.8
- Python ≥ 3.10 and ≤ 3.13
- TensorFlow 2.21.0 installed in the active Python environment (2.16.0 through 2.21.0 are build-compatible; `./configure` auto-detects)
- A C++ toolchain and conda/venv recommended

Setup example with conda:

```bash
conda create -n tf-v2.21.0-zentf-v2.21.0.1-env python=3.12 -y
conda activate tf-v2.21.0-zentf-v2.21.0.1-env
pip install tensorflow==2.21.0
```

On RHEL/Fedora/AlmaLinux/CentOS, export `ZENDNNL_MANYLINUX_BUILD=1` before building.

## Setup commands

The fastest path is the setup script:

```bash
source scripts/zentf_setup.sh
```

This configures and builds the plugin and sets execution environment variables.

For manual configuration:

```bash
./configure
```

The interactive script will:
- detect the Python library path
- select the matching TensorFlow configuration (`tf_2.21`, etc.)
- copy `workspace.bzl`, `WORKSPACE`, `build_config_util.bzl`, and `BUILD.tpl`
- write the matching `.bazelversion`
- ask about MPI support and optimization flags

## Build commands

```bash
bazel clean --expunge
bazel build -c opt //tensorflow_plugin/tools/pip_package:build_pip_package       --verbose_failures --copt=-Wall --copt=-Werror --spawn_strategy=standalone
```

## Package / install commands

```bash
# Generate a Python wheel
bazel-bin/tensorflow_plugin/tools/pip_package/build_pip_package .

# Install the wheel
python -m pip install ./amd_zentf-*.whl
```

## Test commands

- After installation, validate by importing and running TensorFlow inference on an AMD CPU.
- Java and C++ build/test instructions are in `scripts/java/` and `scripts/c++/` respectively.

## Key conventions

- Bazel `.bazelversion` is 7.7.0; do not change it unless the upstream TensorFlow plugin build explicitly moves to a newer Bazel.
- Configure step copies version-specific build files into place based on the detected TensorFlow version.
- Build flags default to `-march=native -Wno-sign-compare` for `--config=opt`.
- `ZENDNNL_MANYLINUX_BUILD=1` is required for many Linux distribution builds.

## Important gotchas

- The repo defaults to `main`; check out the matching version branch (e.g., `v2.21.0.0`) for reproducible builds.
- MPI support is off by default; opt in during `./configure` if needed.
- Compilation uses `--copt=-Werror`; warnings are treated as errors.
- Generated/copied build files (`WORKSPACE`, `workspace.bzl`, `BUILD.tpl`, `build_config_util.bzl`) are produced by `configure.py`; do not hand-edit them without understanding the configuration flow.
- The C++ interface is documented in `scripts/c++/BUILD_FROM_SOURCE.md`.

## Useful shortcuts

```bash
# One-shot configure + build + install
source scripts/zentf_setup.sh

# Manual clean build
./configure
bazel clean --expunge
bazel build -c opt //tensorflow_plugin/tools/pip_package:build_pip_package
bazel-bin/tensorflow_plugin/tools/pip_package/build_pip_package .
```
