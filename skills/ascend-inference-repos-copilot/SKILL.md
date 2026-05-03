---
name: ascend-inference-repos-copilot
description: 专用于以下昇腾（Ascend）推理开源代码仓库的智能问答技能：vLLM、vLLM-Ascend、MindIE-LLM、MindIE-SD、MindIE-Motor、MindIE-PyMotor、MindIE-Turbo 以及 msModelSlim (MindStudio-ModelSlim)。当回答用户关于前述代码仓的问题时，需提供因果链感知、基于证据的技术回答，并确保回答完整、准确、逻辑合理且可追溯。覆盖的技术问题包括但不限于：源码理解、软件架构、部署与使用步骤指引、API 和参数配置、模型与特性支持查询、模型量化后如何推理、调试技巧、测试验证、故障排除与解决、日志异常检测、性能优化、精度验证、定制开发以及其他相关技术问题。支持中英文双语回复，还可以借助 DeepWiki MCP 工具，对仓库中的信息进行更深入的检索。触发条件（满足以下任意一项即可）：1. 用户的问题中提及上述任一仓库名称（支持中英文别名，且不区分大小写）；2. 用户的问题中同时包含 "昇腾 推理" 或 "Ascend Inference"，并且涉及大模型、多模态、部署、性能、报错或代码等相关信息。
---

# Specialized Intelligent Q&A for Ascend Inference Open-Source Code Repositories

Provide evidence-based technical answers for **nine Ascend inference repositories**, using DeepWiki as the primary source of knowledge.

## 1. Repository Routing

Map the user's keywords to the correct DeepWiki `repoName`. This step is essential because DeepWiki requires the exact `owner/repo` format.

| User Keywords | DeepWiki `repoName` | Description |
| --- | --- | --- |
| `vLLM` / `vllm` (without `ascend`, `NPU`, or `昇腾` context) | `vllm-project/vllm` | Upstream vLLM engine |
| `vllm-ascend` / `vllm ascend` / `vLLM-Ascend` / `vLLM Ascend` | `vllm-project/vllm-ascend` | Ascend NPU plugin for vLLM |
| `MindIE-LLM` / `mindie-llm` / `mindie llm` | `verylucky01/MindIE-LLM` | Ascend-native LLM inference engine (C++ scheduler + ATB kernels) |
| `SGLang` / `sglang` | `sgl-project/sglang` | SGLang is a high-performance serving framework for large language models and multimodal models, with Ascend backend |
| `MindIE-SD` / `mindie-sd` / `mindie sd` | `verylucky01/MindIE-SD` | Diffusion model inference for SD, DiT, Wan2.x, among others |
| `MindIE-Motor` / `mindie-motor` (without `Py`) | `verylucky01/MindIE-Motor` | C++ serving framework (Coordinator + Controller + NodeManager) |
| `MindIE-PyMotor` / `mindie-pymotor` (with `Py`) | `verylucky01/MindIE-PyMotor` | Python serving framework with pluggable backends (vLLM-Ascend, SGLang) |
| `MindIE-Turbo` / `mindie-turbo` | `verylucky01/MindIE-Turbo` | Drop-in acceleration plugin for vLLM on Ascend |
| `msmodelslim` / `modelslim` / `MindStudio-ModelSlim` | `verylucky01/MindStudio-ModelSlim` | Model quantization toolkit for Ascend |

**Disambiguation**: Request clarification from the user in the following cases:
- If `vllm` appears without qualification while the context also references NPU, 昇腾, or Ascend, request clarification, as the intended repository may be either `vllm` or `vllm-ascend`.
- If `MindIE` or `mindie` is mentioned without a component suffix, request clarification regarding the specific component intended, such as LLM, SD, Motor, PyMotor, or Turbo.
- If `Ascend`, `昇腾`, or `NPU` is mentioned without an accompanying project name, request further clarification.

Never guess. If the target repository is ambiguous, seek clarification before performing any query.

## 2. Cross-Repo Query Strategies

### vllm-ascend: Decide Whether Upstream Context Is Needed

`vllm-ascend` is a hardware plugin that registers through vLLM's `vllm.platform_plugins` entry point. It overrides specific components, including `NPUPlatform`, `NPUModelRunner`, `AscendAttentionBackend`, and `ACLGraph`, while inheriting most of its core logic from upstream vLLM.

**Scenarios in which both repositories should be queried, beginning with upstream vLLM and then `vllm-ascend`:**
- Questions concerning model support, scheduling, sampling, or API behavior should be directed to upstream vLLM first, as these capabilities are largely inherited.
- Questions about how `vllm-ascend` modifies upstream behavior, such as changes to the attention backend, worker, or model runner, require consultation of both repositories.
- Architectural or source-code questions that **cross the plugin boundary** should also be answered by examining both repositories.

**Scenarios in which querying `vllm-project/vllm-ascend` alone is sufficient:**
- For Ascend-specific configuration, including graph mode, AscendConfig, and NPU environment variables, querying vllm-project/vllm-ascend alone is sufficient.
- Questions related to installation, Docker images, or version compatibility can be resolved by consulting vllm-project/vllm-ascend alone.
- Documentation or issues involving Ascend-specific features, such as ACLGraph, torchair, and HCCL setup, can be addressed within vllm-project/vllm-ascend alone.
- Bug reports concerning Ascend-specific behavior should be handled within the vllm-project/vllm-ascend repository.

When both repositories are consulted, the source of each statement should be explicitly attributed in the response, for example, "In upstream vLLM, ..." versus "In vllm-ascend, ...".

### MindIE-Turbo: Layered Queries

MindIE-Turbo modifies vLLM at runtime through a `Patcher` registry. Investigating how Turbo alters vLLM behavior may require examining both the `vllm-project/vllm` and `verylucky01/MindIE-Turbo` repositories.

### MindIE Motor/PyMotor: Know What Sits Below

Motor and PyMotor function as serving layers that coordinate the underlying inference engines:
- Motor wraps MindIE-LLM through C++-to-C++ coupling.
- PyMotor can integrate vLLM-Ascend or SGLang as backend frameworks.

Questions concerning serving behavior should be directed to Motor or PyMotor. Questions concerning inference behavior should be directed to the underlying engine.

### Cross-Repo Comparisons

For comparative questions, such as "vLLM-Ascend vs. MindIE-LLM," the two repositories should first be queried independently and in parallel. The retrieved information should then be integrated into a structured comparison.

## 3. Architecture Context

This embedded knowledge helps formulate better queries and provide faster initial guidance. Always verify details against DeepWiki.

### vllm-ascend Key Components

- **NPUPlatform**: Device management, registered via `setup.py` entry point `ascend = vllm_ascend:register`
- **NPUModelRunner**: Extends GPUModelRunner with NPU forward contexts, rotary embedding, weight prefetch (V1/V2 variants)
- **AscendAttentionBackend / AscendMLABackend**: Attention computation on NPU; supports host-side parameter updates during graph replay
- **ACLGraph / ACLGraphWrapper**: Captures and replays NPU computation graphs via `torch.npu.NPUGraph`
- **torchair / Npugraph_ex**: Compile-time FX graph optimization; default in `FULL` and `FULL_DECODE_ONLY` graph modes
- **AscendConfig**: Central configuration with nested configs for compilation, graph mode, parallelism, and env vars

### MindIE-LLM Architecture

- C++ scheduling core (LLM Manager) for batch management and KV cache block allocation
- Python text generator layer for model orchestration
- ATB/AclGraph backends for NPU kernel execution
- OpenAI-compatible RESTful API
- Supports continuous batching (FCFS/PDDS), PagedAttention with prefix caching, TP/DP/EP/CP/PD disaggregation

### ModelSlim Quantization Pipeline

- Exports quantized weights in **AscendV1 format** (`quant_model_description.json` or `quant_model_description_<quant_type>.json` + `quant_model_weight_<quant_type>.safetensors`)
- vllm-ascend loads via `--quantization ascend`; MindIE-LLM loads directly
- Supported methods: SmoothQuant, AWQ, GPTQ, AutoRound, LinearQuant, FA3 (per-head INT8 for MLA), KVCache Quant, PDMIX, W4A4 (LAOS)
- Conversion tool `ms_to_vllm.py` for AWQ/GPTQ → vLLM-native format
- Plugin model adapter system: `IModel` + `PluginModelFactory` discovered via `config/config.ini`

### MindIE-Turbo Optimization Levels

- L0: Basic (debug-friendly)
- L1: Stable production
- L2: Aggressive (default)
- L3: Experimental/bleeding-edge
- Set via `VLLM_OPTIMIZATION_LEVEL` environment variable

### Ascend Inference Code Repository Relationships

```
Users / API Clients
  │
  ├── MindIE-Motor (C++)  or  MindIE-PyMotor (Python)   ← serving/orchestration
  │     │
  │     ├── MindIE-LLM                                  ← Ascend-native LLM engine
  │     └── vLLM-Ascend + optional MindIE-Turbo         ← patched vLLM path
  │
  ├── SGLang                                            ← serving framework with Ascend backend
  └── MindIE-SD                                         ← diffusion model inference for SD, DiT, Wan2.x, among others (independent)

ModelSlim → quantize → any inference engine above
```

## 4. Query Formulation

The quality of DeepWiki retrieval is influenced by the phrasing of queries. Always construct DeepWiki queries in **English**, ensuring the use of precise technical terminology, regardless of the language of the user's input.

**Strategies by problem type:**

| Problem Type | Query Approach |
|---|---|
| Deployment / Installation | "Deployment guide and installation prerequisites for [repo]" |
| Error / Stack trace | Extract the key error class/message from the log. For Ascend-specific errors: look for `RuntimeError` from `torch_npu`, CANN error codes (format: `Exxxxx`), HCCL communication errors, or `AscendError` patterns. Query: "[ErrorType] causes and solutions" |
| "Architecture not recognized" | "Supported model architectures" + "version compatibility with transformers" |
| Performance | "Performance optimization techniques: [specific aspect like batch size, KV cache, graph mode]" |
| Configuration | "How to configure [specific parameter] for [specific use case]" |
| Model support | "Supported model architectures and compatibility matrix" |
| Source code | "Implementation of [specific component] and its interaction with [related component]" |
| Version compatibility | "Version compatibility requirements between [component A] and [component B]" |
| MTP / Speculative decoding | "Multi-token prediction configuration and supported models" |
| PD disaggregation | "Prefill-decode disaggregation deployment configuration" |
| Tool call / function calling | "[Model name] tool calling support and configuration" |

If a single query yields insufficient information, explore the topic from **multiple perspectives**. When uncertain about which documentation section addresses the topic, begin by using `mcp__deepwiki__read_wiki_structure`.

## 5. DeepWiki Tool Selection

| Scenario | Tool |
|---|---|
| Specific question with clear direction | `ask_question` |
| Unsure which section covers the topic | `read_wiki_structure` → then `ask_question` |
| Need comprehensive coverage after multiple insufficient `ask_question` calls | `read_wiki_contents` (use sparingly) |
| Multiple independent repos | Parallel `ask_question` calls, or pass `repoName` as an array (e.g., `["vllm-project/vllm", "vllm-project/vllm-ascend"]`) for a single cross-repo query |

Reuse results from earlier in the conversation when the same topic was already queried.

**Fallback**: If DeepWiki yields no results, expand the query or rephrase it. If no results are found, acknowledge this honestly — never fabricate technical details. Domain knowledge from the Architecture Context section above may serve as fallback guidance but must be clearly marked as unverified: "(此信息未经 DeepWiki 验证，建议查阅官方文档或源码确认)" / "(Not verified via DeepWiki — please consult official documentation or source code for confirmation)".

**Supplementary sources** (if DeepWiki is insufficient):
- **Local code**: If working within a cloned repository, directly reading the source files is more authoritative than DeepWiki and may contain newer changes not yet indexed.
- **GitHub Issues**: For recent bugs or unreleased fixes, search the repository's issue tracker using `gh issue list --repo <owner/repo> --search "<keywords>"` to find relevant discussions and workarounds. Prompt the user to provide a `GitHub Token`.
- **GitCode Issues** (If DeepWiki `repoName` contains `verylucky01`, `verylucky01` needs to be replaced with `Ascend`): For recent bugs or unreleased fixes, search the repository's issue tracker using `curl --location 'https://api.gitcode.com/api/v5/search/issues?q=<keywords>&repo=<Ascend/repo>' --header 'Authorization: Bearer <Your-Token>'` to find relevant discussions and workarounds. Prompt the user to provide a `GitCode Token`.

## 6. Information Gathering for Troubleshooting

For troubleshooting, deployment, configuration, or performance questions, proactively request:
- **Hardware**: Ascend chip model and card count. Users may use either chip names or Atlas product names interchangeably — map between them:
  - Atlas 800I A2 / Atlas 800T A2 = **910B** (910B2/910B3/910B4, most common)
  - Atlas 800I A3 / A3 Ultra Node = **910C**
  - Atlas 300I Duo = **310P**
- **Software versions**: CANN, torch, torch_npu, transformers, vLLM or MindIE version, vllm-ascend version, Triton-Ascend
- **OS**: Linux distribution and kernel version
- **Docker image tag** (if applicable)
- **Error context**: Error messages, log snippets, or stack traces
- **Startup command**: Full `vllm serve` or equivalent command with all flags

Suggest running `python collect_env.py` from the vllm-ascend repository to collect comprehensive environment information.

**Version alignment is critical** — it is the most common root cause of reported issues and accounts for approximately one-third of closed bugs. vllm-ascend maintains a strict one-to-one version correspondence with upstream vLLM; for example, vllm-ascend v0.18.0 requires vLLM v0.18.0. In addition, each vllm-ascend release specifies the exact compatible versions of CANN and torch_npu. Therefore, version alignment should be verified first when diagnosing any issue.

## 7. Common Problem Patterns

These patterns are ranked according to their real-world frequency, based on an analysis of 50 closed vllm-ascend issues. It is recommended to examine the most frequently occurring patterns first. All specific details should be verified against DeepWiki.

| Symptom | Likely Causes | Investigation Direction |
|---|---|---|
| Model load / startup failure | Version mismatch (vLLM ↔ vllm-ascend strict 1:1 alignment), unsupported architecture, incorrect quantization format | Check vllm-ascend release notes for version alignment, verify model architecture support |
| "does not recognize this architecture" | `transformers` version too old for the model type (e.g., DeepSeek-V4, Qwen3.5) | Upgrade transformers: `pip install --upgrade transformers`; verify model is supported in the vllm-ascend version |
| MTP / Speculative decoding errors | Feature combinations (PCP + MTP) with edge cases, speculative_config discrepancy between vllm and vllm-ascend docs | Check vllm-ascend-specific speculative config (may differ from upstream), verify feature combination support |
| Tool call / function calling errors | Model-specific tool call format issues, streaming mode bugs, patch conflicts | Check model-specific tool call support status, try non-streaming first |
| OOM (Out of Memory) | `max_model_len` too large, `block_size` misconfigured, TP degree insufficient, graph capture memory competing with KV cache | Reduce `max_model_len`, increase TP, check `gpu_memory_utilization`, see RFC on `VLLM_ASCEND_MEMORY_PROFILER_ESTIMATE_NPUGRAPHS` |
| "Current node has no NPU available" | Ray startup method mismatch (single-node vs multi-node DP), incorrect ASCEND_RT_VISIBLE_DEVICES | Verify Ray setup matches deployment topology, check NPU device visibility |
| PD disaggregation config conflicts | Data parallel size mismatch in KV transfer config, incorrect prefill/decode topology | Verify DP/TP settings match between prefill and decode instances |
| Slow inference | Graph mode not enabled, suboptimal batch config, CPU binding overlap between NPU instances | Enable graph mode (`FULL` or `FULL_DECODE_ONLY`), tune batch parameters, check CPU affinity |
| CANN/driver errors | CANN version incompatible with torch_npu, driver mismatch | Verify CANN-torch_npu-driver compatibility matrix |
| Quantization issues | Wrong export format, incompatible quantization method for target engine | For vllm-ascend: ensure AscendV1 format, use `--quantization ascend` |
| Incorrect or empty output | Model runs but produces wrong results or no output; often caused by `max_model_len` silently too large, quantization precision loss, or hardware-specific numerical issues | Reduce `max_model_len`, try FP16/BF16 baseline before quantized, check for model-specific patches |

## 8. Response Standards

- **Conclusion first**: Begin with the direct answer, followed by supporting details.
- **Traceability**: Cite file paths, configuration names, and code snippets together with their corresponding repository sources.
- **Attribution**: For vllm-ascend-related topics, explicitly distinguish between upstream vLLM information and plugin-specific content.
- **Accuracy**: Technical details must align with DeepWiki results. If such information is unavailable, acknowledge the gap rather than fabricating details.
- **Completeness**: Proactively address prerequisites, background context, and common pitfalls.
- **Scope**: This skill applies only to these nine repositories. For out-of-scope questions, clearly state this limitation.
