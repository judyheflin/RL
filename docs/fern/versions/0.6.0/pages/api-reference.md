---
title: "API Reference"
---

This is the Python API reference for NeMo RL (`nemo_rl`) 0.6.0. It is generated from the docstrings and type annotations in the [`nemo_rl`](https://github.com/NVIDIA-NeMo/RL/tree/v0.6.0/nemo_rl) package at the `v0.6.0` tag, and it covers that package only. It does not cover any HTTP service. The command-line entry points are the scripts under [`examples/`](https://github.com/NVIDIA-NeMo/RL/tree/v0.6.0/examples), which are documented in the guides rather than here.

## How to use NeMo RL

For most workflows you do not call the Python API directly. You launch training and evaluation with the example scripts and a YAML config, and override settings from the command line:

```sh
uv run python examples/run_grpo.py --config examples/configs/grpo_math_1B.yaml policy.model_name=Qwen/Qwen2.5-1.5B
```

See [Quick Start](/about/quick-start) and [Installation and Prerequisites](/about/installation) to get set up, and the algorithm guides such as [GRPO](/guides/grpo) and [SFT](/guides/sft) for the configuration options of each recipe.

Use this reference when you extend NeMo RL or read its code, for example to:

- register a custom environment, see [Environments for GRPO Training](/guides/environments)
- call a generation backend directly, see [Generation Interface](/design-docs/generation)
- write or inspect a loss function, see [Loss functions in NeMo RL](/design-docs/loss-functions)
- check a config schema or function signature in the packages below

## Package layout

| Package | Contents |
|---------|----------|
| `nemo_rl.algorithms` | Training loops and loss functions (GRPO, DPO, SFT, and others) |
| `nemo_rl.data` | Datasets, processors and collation |
| `nemo_rl.environments` | Environment interfaces and built-in environments |
| `nemo_rl.models` | Policy and generation backends |
| `nemo_rl.distributed` | Ray cluster, worker group and virtual cluster utilities |
| `nemo_rl.utils` | Config, logging, checkpointing and other shared utilities |

Modules and functions whose names start with an underscore are internal and may change without notice.
