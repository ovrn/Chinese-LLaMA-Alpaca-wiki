If you want to quickly experience the model performance without installing other libraries or Python packages, you can use the [scripts/inference_hf.py](scripts/inference_hf.py) script to launch a non-quantized model. The script supports single-card inference for both CPU and GPU. For example, to launch the Chinese-Alpaca-7B model, run the script as follows:

```bash
CUDA_VISIBLE_DEVICES={device_id} python scripts/inference_hf.py \
    --base_model path_to_original_llama_hf_dir \
    --lora_model path_to_chinese_llama_or_alpaca_lora \
    --with_prompt \
    --interactive
```

If you have already executed the `merge_llama_with_chinese_lora_to_hf.py` script to merge the LoRa weights, you don't need to specify `--lora_model`, and the startup method is simpler:

```bash
CUDA_VISIBLE_DEVICES={device_id} python scripts/inference_hf.py \
    --base_model path_to_merged_llama_or_alpaca_hf_dir \
    --with_prompt \
    --interactive
```

Parameter description:

- `{device_id}`: CUDA device number. If empty, inference will be performed on the CPU.
- `--base_model {base_model}`: Directory containing the LLaMA model weights and configuration files in HF format.
- `--lora_model {lora_model}`: Directory of the Chinese LLaMA/Alpaca LoRa files after decompression, or the [🤗Model Hub model name](#Model-Hub). If this parameter is not provided, only the model specified by `--base_model` will be loaded.
- `--tokenizer_path {tokenizer_path}`: Directory containing the corresponding tokenizer. If this parameter is not provided, its default value is the same as `--lora_model`; if the `--lora_model` parameter is not provided either, its default value is the same as `--base_model`.
- `--with_prompt`: Whether to merge the input with the prompt template. **If you are loading an Alpaca model, be sure to enable this option!**
- `--interactive`: Launch interactively for multiple **single-round question-answer** sessions (this is not the contextual dialogue in llama.cpp).
- `--data_file {file_name}`: In non-interactive mode, read the content of `file_name` line by line for prediction.
- `--predictions_file {file_name}`: In non-interactive mode, write the predicted results in JSON format to `file_name`.

Note:

- Due to differences in decoding implementation details between different frameworks, this script cannot guarantee to reproduce the decoding effect of llama.cpp.
- This script is for convenient and quick experience only, and has not been optimized for multi-machine, multi-card, low memory, low display memory, and other conditions.
- When running 7B model inference on a CPU, make sure you have 32GB of memory; when running 7B model inference on a GPU, make sure you have 20GB of display memory.