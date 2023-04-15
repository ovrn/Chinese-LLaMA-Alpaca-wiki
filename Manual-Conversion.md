### 准备工作

1. 确保机器有足够的内存加载完整模型（例如7B模型需要13-15G）以进行合并模型操作。
2. 务必确认基模型和下载的LoRA模型完整性，检查是否与[SHA256.md](./SHA256.md)所示的值一致，否则无法进行合并操作。原版LLaMA包含：`tokenizer.model`、`tokenizer_checklist.chk`、`consolidated.*.pth`、`params.json`
3. 主要依赖库如下（如果出问题就请安装以下指定版本）：
   - `transformers`（4.28.0测试通过）
   - `sentencepiece`（0.1.97测试通过）
   - `peft`（0.2.0测试通过）
   - python版本建议在3.9以上

```bash
pip install transformers
pip install sentencepiece
pip install peft
```

*注意：本项目不对使用第三方（非Facebook官方）权重的合规性和正确性负责，例如HuggingFace模型库中的`decapoda-research/llama-7b-hf`（use at your own risk）。*


### Step 1: 将原版LLaMA模型转换为HF格式

请使用[🤗transformers](https://huggingface.co/docs/transformers/installation#install-from-source)提供的脚本[convert_llama_weights_to_hf.py](https://github.com/huggingface/transformers/blob/main/src/transformers/models/llama/convert_llama_weights_to_hf.py)，将原版LLaMA模型转换为HuggingFace格式。将原版LLaMA的`tokenizer.model`放在`--input_dir`指定的目录，其余文件放在`${input_dir}/${model_size}`下。执行以下命令后，`--output_dir`中将存放转换好的HF版权重。

```bash
python src/transformers/models/llama/convert_llama_weights_to_hf.py \
    --input_dir path_to_original_llama_root_dir \
    --model_size 7B \
    --output_dir path_to_original_llama_hf_dir
```

### Step 2: 合并LoRA权重，生成全量模型权重

这一步骤会对原版LLaMA模型（HF格式）扩充中文词表，合并LoRA权重并生成全量模型权重。此处可有两种选择：

- 输出PyTorch版本权重（`.pth`文件），使用`merge_llama_with_chinese_lora.py`脚本
  - 以便[使用llama.cpp工具进行量化和部署](#llamacpp量化部署)

- 输出HuggingFace版本权重（`.bin`文件），使用`merge_llama_with_chinese_lora_to_hf.py`脚本（感谢@sgsdxzy 提供）
  - 以便[使用Transformers进行推理](#使用transformers推理)
  - 以便[使用text-generation-webui搭建界面](#使用text-generation-webui搭建界面)


以上两个脚本所需参数一致，仅输出文件格式不同。下面以生成PyTorch版本权重为例，介绍相应的参数设置。

```bash
python scripts/merge_llama_with_chinese_lora.py \
    --base_model path_to_original_llama_hf_dir \
    --lora_model path_to_chinese_llama_or_alpaca_lora \
    --output_dir path_to_output_dir 
```

参数说明：

- `--base_model`：存放HF格式的LLaMA模型权重和配置文件的目录（Step 1生成）
- `--lora_model`：中文LLaMA/Alpaca LoRA解压后文件所在目录，也可使用[🤗Model Hub模型调用名称](#Model-Hub)
- `--output_dir`：指定保存全量模型权重的目录，默认为`./`
- （可选）`--offload_dir`：对于低内存用户需要指定一个offload缓存路径
