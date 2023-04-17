接下来以[text-generation-webui工具](https://github.com/oobabooga/text-generation-webui)为例，介绍无需合并模型即可进行**本地化部署**的详细步骤。


#### Step 1: 克隆text-generation-webui并安装必要的依赖
```bash
git clone https://github.com/oobabooga/text-generation-webui
cd text-generation-webui
pip install -r requirements.txt
```

#### Step 2: 将下载后的lora权重放到loras文件夹下，webui默认从`./loras`下加载lora权重
```bash
ls loras/chinese-alpaca-lora-7b
adapter_config.json  adapter_model.bin  special_tokens_map.json  tokenizer_config.json  tokenizer.model
```

#### Step 3: 将HuggingFace格式的llama-7B模型文件放到models文件夹下,webui默认从`./models`下加载hf权重
```bash
ls models/llama-7b-hf
pytorch_model-00001-of-00002.bin pytorch_model-00002-of-00002.bin config.json pytorch_model.bin.index.json generation_config.json
```

#### Step 3: 复制lora权重的tokenizer到models/llama-7b-hf下,webui默认从`./models`下加载tokenizer.model,因此需使用扩展中文词表后的tokenizer.model
```bash
cp loras/chinese-alpaca-lora-7b/tokenizer.model models/llama-7b-hf/
cp loras/chinese-alpaca-lora-7b/special_tokens_map.json models/llama-7b-hf/
cp loras/chinese-alpaca-lora-7b/tokenizer_config.json models/llama-7b-hf/
```

#### Step 4: 修改/modules/LoRA.py文件，大约在第28行
```bash
shared.model.resize_token_embeddings(len(shared.tokenizer))
shared.model = PeftModel.from_pretrained(shared.model, Path(f"{shared.args.lora_dir}/{lora_name}"), **params)
```

#### Step 5: 接下来就可以愉快的运行了，参考https://github.com/oobabooga/text-generation-webui/wiki/Using-LoRAs
```bash
python server.py --model llama-7b-hf --lora chinese-alpaca-lora-7b --cpu
```